# eBPF Sensor

**File:** `kernel_module/argus_sensor.bpf.c` · **License:** GPL-2.0 · **Size:** ~206 lines

The sensor is the heart of event recording. 21 kprobe programs are attached to key
syscalls, and each places a raw event into a **ring buffer**. The sensor makes no
decisions: it does not hash, store, or filter. Its job is only to "listen and forward".

---

## The 21 syscalls covered

| Group | Syscalls | Count |
|---|---|---|
| **File** | `unlink`, `unlinkat`, `rename`, `renameat`, `truncate`, `ftruncate`, `chmod`, `fchmodat`, `rmdir`, `openat` | 10 |
| **Process** | `setuid`, `setgid`, `setresuid`, `setreuid`, `kill`, `tkill`, `tgkill` | 7 |
| **Module and mount** | `init_module`, `finit_module`, `delete_module`, `mount` | 4 |

This set is chosen deliberately: file changes, permission changes, user-identity changes,
signal delivery, module load/unload, and mount — exactly the paths an attacker uses for
persistence and for covering their tracks.

---

## The raw event

The event structure (`common/argus_events.h`) contains:

| Field | Meaning |
|---|---|
| Event ID | Which syscall |
| timestamp | Monotonic (for accuracy and ordering) |
| `pid` / `tgid` | Process and thread group |
| `uid` | Effective user |
| `comm` | Process name |
| path | When applicable |

`ppid` (parent process) is read from `task_struct->real_parent` with **CO-RE**, with no
dependency on the exact kernel version.

---

## The anti-feedback map

A map named `daemon_pid_map` is used in the `emit()` function: **the daemon itself never
generates events.** Without this, every log write by the daemon would create a new event
and start an endless feedback loop (the most severe bug fixed in the project's history).

Measured result: **0 self-capture events**.

---

## Why eBPF and not a hand-written module?

| Criterion | Hand-written module (v14) | eBPF sensor (v15) |
|---|---|---|
| Memory safety | The developer's responsibility | Proven by the kernel verifier |
| Loading | Always | Only if the verifier approves |
| Kernel attack surface | Large | Small and bounded |

The eBPF sensor cannot write outside its permitted maps and buffers; any invalid access is
rejected at load time, not at run time.
