# Tamper Resistance

ARGUS defends itself on two fronts: **data integrity** (the log cannot be changed silently) and
**system survival** (the daemon cannot be killed and the module cannot be removed).

---

## Front one: data integrity

The hash chain guarantees that any change to the history is detectable. `argus-cli verify`
recomputes both the links and the hashes:

| Attack | Result |
|---|---|
| Editing the content of a block | `BAD HASHES` |
| Reordering / re-linking | `BROKEN LINKS` |
| Deleting a block from the middle | `BROKEN LINKS` |
| Deleting the whole file | a date gap in the sequence of days |

Formula and block types in [Hash Chain](../architecture/blockchain.md).

---

## Front two: system survival

The kernel shield (4 kprobes) closes the obvious paths:

| Attacker action | Result |
|---|---|
| `kill -9 <daemon>` | the syscall is reported as "successful", but no signal is delivered |
| `kill -9 -1` (broadcast) | neutralized |
| `killpg` (process group) | neutralized |
| `rmmod argus_shield` | `Module is in use` |
| `rmmod -f argus_shield` | `Resource temporarily unavailable` |

**Key point:** the syscall **does not return an error**. The attacker sees no sign that the
shield exists — no `EPERM`, no error message. This is *silent neutralization*, not *loud
rejection*.

---

## The PID 1 exception

The shield deliberately does not reject signals from PID 1 (systemd). Without this,
`systemctl stop/restart` would lock up and every update would require the removal password.

```bash
sudo argus-cli service restart   # works, no password
sudo kill -9 $DPID                              # still blocked
```

---

## Daemon resilience

The systemd unit is configured with `Restart=always` and `RestartSec=3`. If the daemon crashes
for any reason, systemd brings it back and re-arms the shield. No kernel thread is needed for
recovery — the standard service manager does the job.

---

## The release path: why we are never locked out

Without an exit path, every lock would become "only releasable by a reboot". ARGUS forbids this:

- The shield **will not load without a release token** (`-ENOENT`).
- The token = the SHA-256 hash of the removal password in `/etc/argus/remove.key`.
- The token is **re-read on every attempt**, so changing the password never makes removal
  impossible.
- The original token (from load time) is also accepted, so clearing the file does not lock the
  operator out.

```bash
sudo argus-cli release    # prompts for the removal password and releases the shield
```

---

## Proof tests

These claims are confirmed by automated tests:

| Test suite | What it proves |
|---|---|
| `test_shield.sh` | killing the daemon, `rmmod`, `rmmod -f`, the release path |
| `test_shield_reduction.sh` | the shield has exactly 4 hooks |
| `test_verify.sh` | edits, re-linking, and deletion are detected |
| `test_retention.sh` | lossless compression + the immutable seal |

Details in [Testing](../appendix/testing.md).
