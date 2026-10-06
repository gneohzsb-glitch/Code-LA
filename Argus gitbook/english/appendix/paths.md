# Path reference

> **Note on scope.** The event store — where the chain log and its archives
> actually live — is deliberately **not** listed on this page. Keeping its
> location out of the documentation is part of how ARGUS protects the evidence.
> Use the `argus-cli` commands to read it; they always resolve the current
> location for you, and they honour the `argus` group's access rules.

## Binaries and resources

| Item | Path |
|---|---|
| Operator CLI | `/usr/local/bin/argus-cli` |
| GPL sources (shield + sensor) | `/usr/local/share/argus/src/` |
| Bash completion | `/etc/bash_completion.d/argus-cli` |

> The paths of the daemon, the resolver and the sensor are **internal** and
> are not published. ARGUS runs them under deliberately ordinary-looking
> names; a doc or a screenshot that names them would defeat that. Use
> `argus-cli service status` and `argus-cli logs` to observe them instead.

---

## Configuration

| Item | Path |
|---|---|
| Export/encryption | `/etc/argus/export.conf` |
| External anchor | `/etc/argus/anchor.conf` |
| Alerts | `/etc/argus/alert.conf` |
| Filter mode | `/etc/argus/mode` |
| Aggregation allowlist | `/etc/argus/allowlist.conf` |
| Retention policy | `/etc/argus/retention.conf` |

---

## Security

| Item | Path | Permissions |
|---|---|---|
| Removal password (hash) | `/etc/argus/remove.key` | `0600` |
| Password KDF (PBKDF2) | `/etc/argus/pass.kdf` | `0600` |
| Machine fingerprint | `/etc/argus/machine.id` | `0640 root:argus` + immutable |
| Encryption salt | `/etc/argus/embedded_salt` | `0640 root:argus` + immutable |
| Anchor signing key | `/etc/argus/anchor.key` | `0640 root:argus` |

> The kernel module's own path and its control parameters are internal and
> are not published. `argus-cli status` reports the shield state without
> exposing them.

---

## Data and logs

The event store is protected: it is mode `0750`, owned `root:argus`, and readable
only by root and members of the `argus` group. Its exact root is an internal
detail and is not published here.

| Item | Where |
|---|---|
| Chain log (today) | the active day file inside the protected store |
| Archive (past days) | the compressed day files inside the protected store |
| Chain counter | the counter file inside the protected store |
| Integrity certificate | `/var/log/argus_evidence/cert_*.txt` |
| Runtime socket | `/run/argus/ip.sock` |
| Alert log | `/var/log/argus_alerts.log` |
| Export log | `/var/log/argus_export.log` |
| Anchor log | `/var/log/argus_anchor.log` |

---

## Services

The two ARGUS services run as ordinary-looking systemd units; their unit
names are internal and are not published. Drive them through the CLI:

| Task | Command |
|---|---|
| State | `argus-cli service status` |
| Restart | `sudo argus-cli service restart` |
| Log | `argus-cli logs --last 100` |

---

## Notes

- **Dates** in the store's directory names are computed in **Iran time
  (UTC+3:30)**, not UTC.
- The **immutable seal** is applied to the compressed day archives, the removal
  key, and the encryption salt; it is released without a reboot.
- To find the current location of the store on a host where you are root, ask
  the product rather than hard-coding a path: the CLI is the supported
  interface.
