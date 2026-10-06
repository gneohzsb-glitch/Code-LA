# Key Capabilities

## Recording and integrity

| Capability | Description |
|---|---|
| **Kernel-level recording** | eBPF sensor covering 21 key syscalls (file, permission, module, signal) |
| **Hash chain** | `h = SHA-256(prev_hash + event + timestamp + uid + pid)` — each block is locked to the previous one |
| **Tamper detection** | `verify` recomputes both the *link* and the *hash* of every block; content edits, reordering, and mid-chain deletions are all detected |
| **Integrity certificate** | A SHA-256 manifest for a time range; presentable, independently verifiable evidence |
| **External anchoring** | Sending the chain head to an external destination for timestamp anchoring |

## Self-protection

| Capability | Description |
|---|---|
| **Kernel shield** | A small module with 4 kprobes; blocks `kill`/`killall`/`killpg` against the daemon and `rmmod`/`rmmod -f` |
| **Release path** | A token-protected release; without it the shield **does not load at all** (so a reboot lock-out can never occur) |
| **Archive sealing** | Past days are sealed with the `immutable` flag; released with `chattr -i`, no reboot required |
| **Daemon resilience** | A systemd unit with `Restart=always`; if the daemon crashes it returns automatically and re-arms the shield |

## Performance and maintenance

| Capability | Description |
|---|---|
| **Noise filtering** | Routine reads (`/proc`, libraries) are not recorded; writes to the same paths are |
| **Event aggregation** | Repetitive, known-noisy reads are collapsed into a single counted summary block — they are **not deleted** |
| **Lossless retention** | Past days are compressed (~5× smaller), **never deleted**, and sealed |
| **Retention modes** | `off` (default), `archive` (move to external media), `auto` (delete) — the latter two are opt-in |
| **85% alert** | The operator is warned before the disk fills and decides for themselves |

## Security and integration

| Capability | Description |
|---|---|
| **Optional encryption** | AES-256-CBC with a machine-bound key (PBKDF2-HMAC-SHA256, 500,000 iterations) |
| **Sensitive-path flag** | Access to `/etc/shadow`, SSH keys, `/root/`, and similar is recorded with a distinct flag and tagged to the SIEM |
| **SIEM export** | JSON/CSV output plus live syslog forwarding with the `ARGUS_SENSITIVE` tag |
| **Path obfuscation** | Logs are kept under a disguised path and name, so a casual look at the filesystem does not reveal them |

---

## What ARGUS is **not**

Honesty about the boundaries is part of the design:

- **It is not antivirus.** ARGUS records events; it does not block intrusions.
- **It makes no impossible claim against a determined root user.** A fully privileged
  operator retains inherent control over the host; the goal is to close the obvious
  avenues and raise the cost.
- **Encryption is off by default.** Enabling it is the operator's decision.
- **It is not a SIEM replacement.** ARGUS produces a trustworthy *source* of events;
  a SIEM collects and correlates them.
