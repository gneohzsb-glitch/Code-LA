# The Problem We Solve

## An ordinary log is a poor witness

After a breach, the security team turns to the system's history. But conventional logs
carry several fundamental weaknesses that make them unreliable for forensics:

| Weakness | Consequence |
|---|---|
| **Editable** | An attacker with root access simply removes the lines about themselves |
| **Deletable** | The whole log file is removed; no trace remains |
| **No chain** | Deleting a middle line leaves no detectable gap |
| **No root of trust** | There is no way to prove the file has been untouched since it was written |
| **Can be silenced** | Logging is switched off and the system continues quietly |

The result: in a court of law or a compliance review, the log cannot be relied on as
**evidence**.

---

## What makes a log "evidence"?

For a record to be citable, three conditions must hold:

1. **Completeness** — every meaningful event is recorded, not just the ones that are
   logged by default.
2. **Provable integrity** — any change (edit, deletion, reordering) is detectable.
3. **Resistance to being switched off** — an attacker cannot stop the recording or take
   the system out of service.

---

## The ARGUS solution

ARGUS meets these three conditions with three mechanisms:

| Condition | ARGUS mechanism |
|---|---|
| Completeness | An eBPF sensor in the kernel covering 21 key syscalls; it cannot be bypassed from user space |
| Provable integrity | A SHA-256 hash chain; each block embeds the hash of the previous block |
| Resistance | A shield module blocks killing the daemon and unloading the module; archives are sealed `immutable` |

---

## Why is this hard?

Three real constraints stand in the way of any solution:

- **Performance:** kernel-level recording is expensive; every event must be processed
  cheaply.
- **Volume:** event rates are high; an ordinary disk fills up within weeks.
- **Not locking yourself out:** a system that can only be unlocked by rebooting is a
  disaster in production.

ARGUS addresses all three deliberately: noise filtering and aggregation for performance
and volume, and the "never reboot" principle for lock-out safety. Details are in
[Architecture](../architecture/README.md).
