# Data protection

ARGUS records system events — which can include personal information (usernames, file paths,
network connections). This page describes the controls that exist for privacy and data
protection.

> **Disclaimer:** This is a technical description, not legal advice. Compliance with
> regulations (such as GDPR or local law) must be determined with your organization's legal
> counsel.

---

## Data that is recorded

| Field | Can contain personal information? |
|---|---|
| `u` (uid) | Yes — user identity |
| `un` (username) | Yes |
| `p` / `pp` (pid/ppid) | Not directly |
| `c` (process name) | Usually not |
| `e` (event text: file path) | Yes — may contain a username or a personal path |
| `ip` (external address) | Yes — can be personal data |
| `hn` (hostname) | Yes |

---

## Privacy controls

| Control | Mechanism |
|---|---|
| **Encryption at rest** | With `ENCRYPT=1`, content is encrypted with AES-256 |
| **Machine lock** | The key is bound to `machine.id`; a copied log is unreadable elsewhere |
| **Access control** | The salt is readable only by root; reading an encrypted log requires `sudo` |
| **Controlled retention** | The `off` policy never deletes; `auto` performs controlled deletion |
| **Lossless compression** | Data is made smaller, not erased |
| **Immutable seal** | Prevents silent rewriting |

---

## Minimization principles

ARGUS follows several principles that reduce the privacy burden:

| Principle | How |
|---|---|
| **Noise filtering** | Routine reads are not recorded → less data |
| **Aggregation** | Repeated reads are folded into a single counter → fewer blocks |
| **Signal only** | 21 key syscalls, not everything |
| **Conservative default** | Empty allowlist → no aggregation without an operator decision |

---

## Retention period

Retention is the operator's decision and is controlled in `retention.conf`:

| Mode | Period |
|---|---|
| `off` (default) | Unlimited (compressed only) |
| `archive` | Until transfer to external media |
| `auto` | Until the disk reaches a threshold |

To meet a "right to be forgotten" requirement or a retention limit, `auto` mode with an
appropriate threshold — or manual cleanup (after releasing the immutable seal) — can be used.

---

## Right of access and deletion

| Right | How |
|---|---|
| Access to data | `argus-cli export --format json\|csv` |
| Search by user | Filter on the `u`/`un` field in the output |
| Delete data | Manual cleanup of the archive (after releasing the seal), or `auto` mode |

---

## Deployment notes for compliance

1. **Take `ENCRYPT=1` seriously** if a log might leave the host.
2. **Choose a strong removal password.**
3. **Set your retention policy** to match the legal requirements of your jurisdiction.
4. **Limit root access**; keep the salt and `machine.id` safe.
5. **Save integrity manifests** for sensitive intervals.

---

## Data-protection limits

| Limitation | Description |
|---|---|
| Encryption is off by default | It must be enabled deliberately |
| No automatic redaction | ARGUS does not remove or mask fields |
| Immutable seal is reversible | It is a deliberate, reversible control with a documented release path — not a cryptographic guarantee |
| Clock-dependent | The correctness of time ordering depends on the host clock |
