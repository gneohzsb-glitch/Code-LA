# export and SIEM

## `argus-cli export`

Export a day's blocks into `$HOME`:

```bash
argus-cli export --format json
argus-cli export --format csv
argus-cli export --format csv --from 09/30/2026
```

| Option | Purpose |
|---|---|
| `--format json` | JSON output |
| `--format csv` | CSV output (with a `flags` column) |
| `--from MM/DD/YYYY` | A specific day |

---

## The `flags` column in CSV

The CSV output has a real `flags` column used for **SIEM routing**:

| Value | Meaning |
|---|---|
| `1` | Sensitive-path event (`[SENSITIVE]`) |
| `2` | Aggregation summary whose count exceeds the ceiling (anomaly) |
| `0` | Ordinary event |

> **History:** the header previously declared eight columns while the body wrote raw JSON
> lines, so the header and rows never aligned. This was fixed in v15.1.

---

## Live forwarding

Beyond manual export, the daemon forwards events live to a destination. The destination is
configured in `/etc/argus/export.conf` and can be changed **without a restart** (a config
watcher re-reads it).

### syslog destinations (RFC5424)

```ini
DESTINATION=FILE:/var/log/argus_export.log
DESTINATION=TCP:siem.example.com:514
DESTINATION=UDP:siem.example.com:514
```

Sensitive-path events are sent with the tag **`ARGUS_SENSITIVE`** and alert priority
(`<12>`), so they can be routed without parsing the JSON.

### JSON exporters (v15.4 — Phase 6)

Three HTTP-based destinations, each sending every block as a **POST with a JSON body**
(libcurl; over TLS when the URL is `https://`):

```ini
# Generic webhook
DESTINATION=HTTP:https://collector.example.com/ingest
TOKEN=<bearer-token>          # optional → Authorization: Bearer

# Splunk HEC
DESTINATION=SPLUNK:https://splunk.example.com:8088/services/collector/event
TOKEN=<hec-token>             # → Authorization: Splunk <token>

# Elasticsearch
DESTINATION=ELASTIC:https://es.example.com:9200/argus/_doc
TOKEN=<api-key>               # optional → Authorization: ApiKey

TLS_VERIFY=1                  # 0 only for a lab self-signed certificate
```

| Destination | Body format |
|---|---|
| `HTTP:` | `{"host":..,"ts":..,"event":<block>}` |
| `SPLUNK:` | `{"time":..,"host":..,"sourcetype":"argus","event":<block>}` |
| `ELASTIC:` | `{"@timestamp":"<RFC3339>","host":{"name":..},"event":<block>}` |

`<block>` is the same JSON object held in the chain (so what reaches the SIEM is exactly what
was hashed). If a POST fails, the daemon does not stop and the next block is attempted
again.

---

## External anchoring (timestamp anchoring)

The head of the chain is periodically sent to an external destination
(`/etc/argus/anchor.conf`). If the chain were ever rewritten wholesale later, the anchored
hash would no longer match the chain head.

```ini
INTERVAL=300
DESTINATION=FILE:/var/log/argus_anchor.log
FORMAT=JSON
```

---

## Integration examples

### Sending to a syslog-based SIEM

```ini
DESTINATION=TCP:siem.internal:601
```

### Extracting sensitive events from CSV

```bash
argus-cli export --format csv --from 09/30/2026 \
  | awk -F, 'NR==1 || $<flags_col>==1'
```

### Deriving an allowlist from real data

```bash
argus-cli export --format csv --from 09/01/2026 \
  | awk -F, '{print $<comm>","$<path>}' | sort | uniq -c | sort -rn | head -20
```

Review each pair, and if it genuinely is a daemon talking to itself, add it to
`allowlist.conf`.

---

## Block types in the output

A SIEM can route by block type using the tag:

| Block type | Source |
|---|---|
| Ordinary event | Any recorded syscall |
| `AGGREGATE` | Aggregation summary |
| `AGGREGATE-RECOVERED` | Recovery of an orphaned sidecar |
| `CONFIG` | allowlist/mode/window change |
| `MODE_CHANGE` | Filter-mode change |
| `MAINTENANCE` | Retention action |

Details are in [The Hash Chain](../architecture/blockchain.md).
