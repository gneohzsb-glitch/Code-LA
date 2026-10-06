# ARGUS — English edition

**Kernel-level event recording for forensics and security auditing.**

> Version 15.4 · by Gneo HZSB

ARGUS captures the events that matter the moment they happen, turns each one into
a block whose SHA-256 hash is chained to the block before it, and writes it to
disk. The result is a **tamper-evident chain**: change, reorder, or remove a
single byte and `argus-cli verify` shows exactly where the break is.

The practical goal is simple. After an intrusion — or to demonstrate compliance —
you can answer, with confidence:

> **What happened, by which user or process, when — and has anyone touched the log?**

---

## Who this documentation is for

This GitBook is written for **market researchers, pre-sales engineers, and
technical managers**, and it doubles as an operator's manual. Every section
stands on its own; if you have time for only one page, start with
[Key capabilities](introduction/key-capabilities.md) and the
[Datasheet](market/datasheet.md).

---

## At a glance

| Property | ARGUS |
|---|---|
| Event capture | At the kernel level, with an eBPF sensor (21 syscall hooks) |
| Tamper-evidence | A SHA-256 hash chain; each block is locked to the previous one |
| Self-protection | A kernel "shield" that resists killing the daemon or unloading the module |
| Retention | Past days are compressed and sealed; **never deleted by default** |
| Encryption | Optional AES-256-GCM, with a key locked to the machine |
| Reboot required | **No** — not to install, not to upgrade, not to remove |
| Integration | JSON/CSV output, live syslog forwarding, an integrity certificate |

---

## Quick start

```bash
make                 # build everything from source
sudo ./install.sh    # install
argus-cli status     # shield, daemon, password, disk use
argus-cli verify     # verify the chain
```

Full guides live in [Installation](deployment/installation.md) and the
[CLI reference](cli-reference/README.md).

---

## Current status

| Metric | Value |
|---|---|
| Test coverage | **631 automated assertions across 44 suites, all green** |
| Shield hooks | 4 (down from 10) |
| eBPF sensor hooks | 21 |
| Self-capture events | 0 |
| Reboot required | No |

---

> **Licensing:** the kernel components (shield and sensor) ship under GPL-2.0
> with full source; the daemon and CLI are proprietary and ship as binaries.
> See [Licensing](appendix/license.md) for the details.
