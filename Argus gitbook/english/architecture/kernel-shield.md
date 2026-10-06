# Kernel Shield

**File:** `kernel_module/argus_shield.c` · **License:** GPL-2.0 · **Size:** ~574 lines

The shield is ARGUS's only hand-written kernel code. Its job is a single sentence:
**if someone tries to kill the daemon or unload the module, stop them** — without the
adversary realizing a shield exists.

---

## 4 kprobes (reduced from 10 in v15.2)

| # | syscall | Target |
|---|---|---|
| 1 | `kill` | Killing by PID, process group, or `-1` |
| 2 | `tkill` | Signal to a thread |
| 3 | `tgkill` | Signal to a specific thread |
| 4 | `delete_module` | `rmmod` and `rmmod -f` |

The six removed hooks (`rt_sigqueueinfo`, `rt_tgsigqueueinfo`, `pidfd_send_signal`,
`ptrace`, `process_vm_readv`, `process_vm_writev`) were secondary paths to the same two
goals: signaling and memory access.

> **A deliberate trade-off:** those hooks were retired as a scoping decision. They offered
> no meaningful protection against a fully privileged operator — who already holds
> administrative control of the host — while enlarging the module's maintenance surface.
> Four hooks instead of ten yield a smaller, more auditable module.

---

## The mechanism: neutralize, not reject

The kprobe sits on `__x64_sys_<name>`. These functions receive `const struct pt_regs *regs`;
zeroing an argument makes the call a no-op:

```
ARG1(sr) = 0        // e.g. turning SIGKILL into 0 = "no signal"
```

**Result:** the syscall does not return an error. `kill` is reported as successful but no
signal is delivered. The adversary gets neither `EPERM` nor any other hint — because they
do not know the shield is there.

---

## Two subtle implementation points

1. **The `pid_t` type:** on the 32-bit ABI it is 32-bit wide and the upper bits of the
   register are undefined. The argument must therefore be read as `(int)`, not `(long)` —
   otherwise `kill(-pgid)` turns into a huge positive number and the logic breaks.

2. **Task lookup:** all lookups are done with `rcu_read_lock()` and
   `pid_task(find_vpid(...))` (because `find_task_by_vpid` is not exported on kernel 5.15).

---

## The release path (no reboot)

```
echo <hash> > /sys/module/argus_shield/parameters/argus_release
```

The token is the content of `/etc/argus/remove.key` — that is, the **SHA-256 hash of the
removal passphrase**. The passphrase itself never reaches the kernel. Writing the correct
token sets `released=true`, clears the daemon PID, and calls `module_put()` so the module
can be removed with `rmmod`.

### Three safeguards on the release path

| Safeguard | Why |
|---|---|
| **Without a token, the module does not load at all** (`-ENOENT`) | Locking without an exit route is forbidden |
| The token is **re-read on every attempt** | Changing the passphrase does not make release impossible |
| The previous token (from load time) is also accepted | Removing the file does not lock the operator out |

---

## Daemon registration

```
echo <pid> > /sys/module/argus_shield/parameters/argus_daemon_pid
```

- Re-registration while a live daemon is protected is rejected with `-EBUSY`, so the shield
  cannot be pointed at a different PID.
- The process start time (`start_time`) is stored so that a **recycled PID** is not mistaken
  for the daemon.

---

## The PID 1 exemption

The shield deliberately does not reject signals from **PID 1 (systemd)**. Without this
exemption, `systemctl stop/restart` would be locked out and every update would require the
removal passphrase. This exemption is what makes "never reboot" practical.

From an ordinary shell, `sudo kill -9 $DPID` is still blocked — only systemd can bring the
service down.

---

## Honesty about the boundaries

The shield makes no claim against a determined root user. A fully privileged operator
retains inherent control over the host, and the documented administrative release route
exists by design. The goal is to **raise the cost and close the obvious avenues**, not to
become invisible. Details are in the [threat model](../security/threat-model.md).

---

## Why not under 200 lines?

The v15.2 shield is about 575 lines (352 lines excluding blanks and comments). Getting under
200 lines is only possible by **removing logic**, not by compressing it:

- Removing the release path → a definitive violation of the "never reboot" requirement
- Dropping `killpg`/`kill(-1)` coverage → a single group-wide signal from a privileged
  process could terminate the daemon
- Removing the comments → kernel code without its rationale is no longer auditable

So "under 200 lines" is accepted as an unmet requirement, with the reason recorded.
