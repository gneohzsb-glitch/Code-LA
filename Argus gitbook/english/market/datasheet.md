# Data Sheet

> **Product:** ARGUS · **Version:** v15.4 · **Vendor:** Gneo HZSB
> **Reference platforms:** Ubuntu 22.04 LTS (kernel 5.15) and Debian 12 (kernel 6.1)

---

## 1. Introduction

ARGUS is a **kernel-level event-recording** system for forensics and security auditing. It
captures every significant system event at the moment it occurs, turns it into a SHA-256-hashed
block chained to the previous block, and writes it to disk. The result is a **tamper-evident
chain**.

The practical goal: after an intrusion, or to demonstrate compliance, answer "what happened,
by which user/process, when, and has anyone tampered with the log?"

---

## 2. Key capabilities

| Capability | Description |
|---|---|
| Kernel-level recording | eBPF sensor, 21 key syscalls (file, authorization, module, signal) |
| Hash chain | `h = SHA-256(prev_hash + event + timestamp + uid + pid)` |
| Tamper resistance | Kernel shield (4 kprobes) + `immutable` seal on archives |
| Smart noise filter | Routine reads are not recorded; writes to the same paths are |
| Event aggregation | Repeated reads are folded into a counter summary block — **never deleted** |
| Sensitive-path flag | Access to `/etc/shadow`, SSH keys, `/root/`, and more is flagged |
| Lossless retention | Past days are compressed, **never deleted**, and sealed |
| Optional encryption | AES-256-CBC with a key locked to the machine (PBKDF2-HMAC-SHA256) |
| SIEM export | JSON/CSV output + live syslog forwarding tagged `ARGUS_SENSITIVE` |
| Integrity certificate | A SHA-256 manifest for a time interval (presentable evidence) |
| External anchor | Sends the chain head to an external destination as proof of time |

---

## 3. Architecture

Three layers, with all logic in userspace; the kernel holds only an eBPF sensor and a small
shield module.

```
┌─────────────────────────────────────────────────────────────┐
│  1. eBPF sensor  (argus_sensor.bpf.c)                        │
│     21 hooks, record only — no hashing, no storage, no policy│
│     Verified by the kernel verifier before it runs           │
└───────────────────────────┬─────────────────────────────────┘
                            │ ring buffer
┌───────────────────────────▼─────────────────────────────────┐
│  2. Daemon (capture engine) — userspace                     │
│     filter/classify → SHA-256 chain → store → alert → export │
│     + retention, external anchor, IP separation              │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│  3. Protection layer (shield module) — 4 kprobes             │
│     kill / tkill / tgkill / delete_module                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Deployment requirements

| Item | Minimum |
|---|---|
| OS | Ubuntu 22.04 LTS or Debian 12 (kernel 5.15+) |
| Kernel | `CONFIG_BPF_SYSCALL=y`, `CONFIG_DEBUG_INFO_BTF=y` |
| Access | root to install and to read encrypted logs |
| Disk space | Depends on event rate; past days are compressed (~5× smaller) |
| Password/service | systemd |

**No GRUB changes. No reboot** — not to install, not to update, not to remove.

---

## 5. Performance (measured on the reference VM)

| Metric | Value |
|---|---|
| Steady-state recording rate | ~36 blocks in 30 seconds (after filtering; ~326 before filtering) |
| Self-capture events | 0 |
| One-day log compression | 46 MB → 8.4 MB (5.4×) |
| `list --external` over a 109 MB day | ~0.6 s |
| `verify` over 368,000 blocks | ~3.4 s |
| Test coverage | 631 automated assertions across 44 test suites, all passing |

---

## 6. Security model

**What is protected:** the daemon cannot be killed with `kill`/`killall`/`killpg`, and the
module cannot be removed with `rmmod` or `rmmod -f`. A signal from PID 1 (systemd) is
deliberately allowed through, so that `systemctl stop/restart` and passwordless updates
remain possible.

**Release path (no reboot):** the token is the SHA-256 hash of the removal password in
`/etc/argus/remove.key`. Without this file, the shield **does not load at all** — so a system
is never left locked in a state that could only be recovered by a reboot.

**Machine lock:** both the shield and encryption are bound to `machine.id`.

**Archive seal:** each compressed day is sealed with the `immutable` flag; the seal is
released without a reboot.

**Daemon resilience:** the systemd unit uses `Restart=always`; a daemon crash does not affect
the kernel.

**Accepted limitation:** the shield makes no claim against a fully privileged administrator.
Its goal is to raise the cost of tampering and block the obvious paths.

---

## 7. Integration

- **SIEM:** live syslog forwarding; sensitive-path events are tagged `ARGUS_SENSITIVE`.
  Destinations: `FILE:` / `TCP:` / `UDP:`.
- **Manual output:** `argus-cli export --format json|csv` (the `flags` column).
- **Evidence certificate:** `argus-cli integrity --from DATE --to DATE`.
- **Chain API:** `argus-cli verify` and `argus-cli block N`.
- **Summary and configuration blocks:** `AGGREGATE`, `CONFIG`, `MODE_CHANGE`, `MAINTENANCE`
  are recorded in the same chain with the same hash formula.

---

## 8. Operation

```bash
# Install
sudo ./install.sh

# Daily use
argus-cli status            # shield, daemon, password, disk usage
argus-cli verify            # verify the chain (links + hashes)
argus-cli list --sensitive  # sensitive-path events only
argus-cli sensitive         # the same filter, as a standalone command
argus-cli tail              # live view
argus-cli mode              # aggregation filter mode (no reboot)
argus-cli retention         # retention policy

# Update without a reboot
sudo argus-cli service restart

# Remove (logs are left intact)
sudo argus-cli remove
```

Log encryption is optional: `ENCRYPT=1` in `/etc/argus/export.conf` and restart the service.

---

## 9. License and delivery

| Component | License | What the customer receives |
|---|---|---|
| Shield module (`argus_shield`) | GPL-2.0 | Binary + full source |
| eBPF sensor (`argus_sensor`) | GPL-2.0 | Binary + full source |
| Daemon, CLI, helper | Proprietary | Binary only |

The full license texts are in `LICENSE/GPL-2.0.txt` and `LICENSE/PROPRIETARY.txt`. The GPL
sources are installed under `/usr/local/share/argus/src/`.

---

## 10. Support

To report an issue or request a feature: Gneo HZSB.
Full documentation: this GitBook.
