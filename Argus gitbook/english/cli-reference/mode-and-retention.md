# mode and retention

## `argus-cli mode` — aggregation filter mode

```bash
argus-cli mode                   # show the current mode and window size
sudo argus-cli mode balanced     # performance | balanced | forensic
```

```
  ◆ FILTER MODE

  mode: balanced   (window 300s)
```

| Mode | Window | Meaning |
|---|---|---|
| `performance` | 3600s | Aggressive aggregation |
| `balanced` (default) | 300s | Normal aggregation |
| `forensic` | 0 | No aggregation — every event is individual |

**Key points:**

- No mode **turns logging off** — it only changes the aggregation window size.
- Changing the mode itself produces a chain block (`MODE_CHANGE`) and an alert.
- The daemon picks the change up in place within five seconds (**no restart**).
- Aggregation happens only when a rule in `/etc/argus/allowlist.conf` names the path; the
  default is empty.

---

## `argus-cli retention` — retention policy

```bash
argus-cli retention                        # show the current policy
sudo argus-cli retention mode off          # off | archive | auto
sudo argus-cli retention threshold 80      # 1..99
sudo argus-cli retention archive-path /mnt/backup
```

```
  ◆ RETENTION POLICY

  mode:       off
  threshold:  85% full
  act on:     20% of eligible archives
  archive to: (unset)

  off      compress + seal only — nothing is ever removed (default)
  archive  move the oldest days to an external path — nothing destroyed
  auto     delete the oldest days — irreversible
  The current day and the last 7 days are always protected. Writing needs sudo.
```

| Command | Purpose |
|---|---|
| `retention` | Show the policy |
| `retention mode off\|archive\|auto` | Set the mode |
| `retention threshold N` | Disk percentage at which the mode acts (1..99) |
| `retention archive-path P` | Destination for `archive` mode |

| Mode | Behaviour |
|---|---|
| `off` (default) | Compress + seal. Nothing is removed |
| `archive` | Move the oldest N% to an external path |
| `auto` | Delete the oldest N% (irreversible) |

**Safety rules (always):** today is never touched, the last 7 days are preserved, and every
action is recorded in the chain **before it happens**.

Full details are in [Retention](../deployment/retention.md).

---

## How `mode` and `retention` differ

| | `mode` | `retention` |
|---|---|---|
| What it controls | Event aggregation (log volume) | Disk retention (compress/move/delete) |
| Time scale | A seconds-long window | A 10-minute pass |
| Risk of data loss | None | Present in `auto` |
| Config file | `/etc/argus/mode` | `/etc/argus/retention.conf` |
| Applied without restart | Yes (≤ 5 seconds) | Yes (≤ 10 minutes) |
