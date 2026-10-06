# Architecture

ARGUS has three layers, with a clear boundary between "kernel" and "user space". All
logic lives in user space; the kernel runs only an eBPF sensor and a small shield module.

```
┌─────────────────────────────────────────────────────────────┐
│ User space                                                  │
│                                                             │
│  capture engine           ← Brain: reads the ring buffer,   │
│  (daemon)                    blockchain, storage, alerts,   │
│                              anchoring                      │
│                                                             │
│  resolver                 ← PID → IP mapping                │
│                                                             │
│  argus-cli                ← Operator: status/verify/list/…  │
├─────────────────────────────────────────────────────────────┤
│ Kernel boundary                                             │
├─────────────────────────────────────────────────────────────┤
│ eBPF sensor               ← 21 hooks, ring buffer           │
│ shield module             ← 4 kprobes, tamper-resistant     │
└─────────────────────────────────────────────────────────────┘
```

---

## Why this split?

Every line of code that runs in the kernel is kernel attack surface. ARGUS pulls logic out
of the kernel wherever possible:

- The **eBPF sensor** emits raw events and makes no decisions. The advantage: the kernel
  verifier proves it safe *before it runs*, so a memory error in it is impossible.
- The **shield** is the only hand-written kernel module, and its job is solely to "protect
  the daemon's life".
- The **daemon** holds all the logic, and if it crashes the kernel stays healthy.

---

## Comparison with the previous-generation architecture (v14)

| | v14 | v15 |
|---|---|---|
| Sensor | ~24 hand-written kprobes in a kernel module | A single eBPF program (verified by the kernel verifier) |
| Custom kernel code | One large module | A small shield module + an eBPF program |
| Logic (blockchain, storage, alerts) | Mixed into the kernel | Entirely in user space |

---

## Components in this section

| Page | Topic |
|---|---|
| [eBPF sensor](ebpf-sensor.md) | 21 hooks, raw events, the anti-feedback map |
| [Kernel shield](kernel-shield.md) | 4 kprobes, the signal-neutralizing mechanism, the release path |
| [User-space daemon](userspace-daemon.md) | 18 files, filtering, aggregation, retention, encryption |
| [Hash chain](blockchain.md) | The hash formula, block format, block types |
| [Data flow](data-flow.md) | The path of a syscall from kernel to disk |
