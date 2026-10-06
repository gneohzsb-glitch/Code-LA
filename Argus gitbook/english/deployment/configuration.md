# Configuration

All configuration files live in `/etc/argus/`. None of them requires a reboot; the daemon reads
configuration at runtime.

---

## File summary

| File | Role | On upgrade |
|---|---|---|
| `export.conf` | export/encryption | updated |
| `anchor.conf` | external anchor | updated |
| `alert.conf` | alert settings | updated |
| `mode` | aggregation filter mode | preserved |
| `allowlist.conf` | aggregation allowlist | preserved |
| `retention.conf` | retention policy | preserved |

---

## `export.conf` — export and encryption

```ini
INTERVAL=60                    # export pass interval (10..3600 seconds)
DESTINATION=FILE:/var/log/argus_export.log
ENCRYPT=0                      # 1 = encrypt new lines
ENCRYPT_KEY_SOURCE=machine     # the only supported source
```

**`DESTINATION`:** `FILE:<path>`, `TCP:<host:port>`, or `UDP:<host:port>`.

**`ENCRYPT`:** with `1`, new lines are encrypted. Earlier days stay plaintext and coexist with
new days. Reading an encrypted log requires `sudo` (the salt is readable only by root). Details
in [Encryption](../security/encryption.md).

---

## `anchor.conf` — external anchor

```ini
INTERVAL=300                   # anchor submission interval (10..86400 seconds)
DESTINATION=FILE:/var/log/argus_anchor.log
FORMAT=JSON
```

The chain head hash is periodically sent to an external destination so that **timestamp
anchoring** becomes possible: if someone later rewrites the entire chain, the anchored hash no
longer matches the chain head.

---

## `alert.conf` — alerts

```ini
MIN_RISK_SCORE=30              # minimum risk score to alert
```

Detection rules are compiled in; this file sets the threshold.

---

## `mode` — aggregation filter mode

A number (`0`, `1`, `2`) or, from the CLI:

```bash
argus-cli mode                   # show
sudo argus-cli mode balanced     # performance | balanced | forensic
```

| Value | Name | Window |
|---|---|---|
| 0 | performance | 3600s |
| 1 | balanced (default) | 300s |
| 2 | forensic | 0 (no aggregation) |

No mode turns logging off.

---

## `allowlist.conf` — aggregation allowlist

Each line says: "this specific executable may read under this path prefix without an individual
block; accumulate it in a window counter."

```
<exe_path>|<path_prefix>|<max_count_per_window>
```

| Field | Meaning |
|---|---|
| `exe_path` | absolute path of the binary, exact match (from `/proc/<pid>/exe`, **not** `comm`) |
| `path_prefix` | matched against the beginning of the opened path |
| `max_count` | a count above this limit is still aggregated, but the summary is flagged as an anomaly; `0` = no cap |

**The shipped file is intentionally empty.** An allowlist must be derived from real data, not
guessed. While it is empty, no event is aggregated.

> **Important:** only **reads** are aggregated. A write under an allowed prefix is always an
> individual block.

An example extraction:

```bash
argus-cli export --format csv --from 09/01/2026 \
  | awk -F, '{print $<comm>","$<path>}' | sort | uniq -c | sort -rn | head
```

---

## `retention.conf` — retention policy

```ini
RETENTION_MODE=off             # off | archive | auto
RETENTION_THRESHOLD_PERCENT=85
RETENTION_DELETE_PERCENT=20
RETENTION_ARCHIVE_PATH=
```

Full details in [Retention](retention.md). This file is re-read by the daemon on every pass, so
changing it takes effect **without a restart**.

---

## Applying changes

| File | How it takes effect |
|---|---|
| `retention.conf` | automatic, next pass (≤10 minutes) |
| `mode` | automatic, ≤5 seconds |
| `allowlist.conf` | automatic |
| `export.conf` / `anchor.conf` | requires `argus-cli service restart` |

None requires a reboot.
