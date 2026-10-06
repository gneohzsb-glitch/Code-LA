# Retention

ARGUS's default policy is one sentence: **compress, never delete, and alert at 85% so you can
decide.** Two optional modes exist for hosts whose disk cannot hold the entire history.

---

## Why isn't deletion the default?

A forensic record that silently discards its own history is worse than useless — it gives you
false confidence. So the default is retention, not deletion.

---

## Default policy (`off`)

| Behavior | Description |
|---|---|
| Compression | each past day (`.log`) is converted to `.log.gz` next to it with zlib |
| Safe ordering | write to `.log.gz.tmp` → atomic `rename` → **only then** `unlink` the original |
| Today's file | **never** touched (the daemon is writing, the alert thread is reading) |
| Partial run | if `.log.gz` exists but `.log` remains, only `.log` is removed |
| Seal | the finished archive is sealed with the `immutable` flag |
| Alert | crossing 85% logs a `Storage Pressure` alert (severity: HIGH) |
| Alert repetition | not repeated until usage falls back below 80% (anti-spam) |

A measured result: a 46 MB log from the previous day shrank to 8.4 MB (**5.4×**), without
discarding a single byte.

---

## Optional modes

The file `/etc/argus/retention.conf`:

```ini
RETENTION_MODE=off             # off | archive | auto
RETENTION_THRESHOLD_PERCENT=85 # at this disk percentage, the mode acts
RETENTION_DELETE_PERCENT=20    # percentage of eligible archives per action
RETENTION_ARCHIVE_PATH=        # destination for archive mode
```

| Mode | Behavior | Reversible? |
|---|---|---|
| `off` | compress + seal. **Nothing is ever removed.** (default) | — |
| `archive` | **moves** the oldest N% of sealed archives to `RETENTION_ARCHIVE_PATH` | yes (data remains) |
| `auto` | **deletes** the oldest N% | **no (data is destroyed)** |

---

## Safety rules (always, in every mode)

Four rules apply in every mode:

1. **Today is never touched.**
2. **The last 7 days are never touched.**
3. **`archive` mode** does nothing until the destination path is a reachable directory (and
   alerts).
4. **The action is recorded in the chain before it happens.** A `MAINTENANCE` block with the full
   list of files and their sizes is written **before** the move/delete — so the record of what
   existed outlives the data itself.

---

## Managing from the CLI

```bash
argus-cli retention                        # show the current policy
sudo argus-cli retention mode off          # off | archive | auto
sudo argus-cli retention threshold 80      # 1..99
sudo argus-cli retention archive-path /mnt/backup
```

Sample output:

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

A configuration change takes effect **without a restart**: the daemon re-reads `retention.conf`
on every pass.

---

## At most one action per crossing

The daemon applies the mode **at most once per crossing of the threshold**. If a disk stays above
the threshold, it does not delete another 20% every 10 minutes; the next action happens only
after usage falls and rises again. This prevents repeated deletion.

---

## The `MAINTENANCE` block

Before any action, a block is recorded in the chain:

```
MAINTENANCE: action=archive threshold=85 used=91 pct=20 keep_days=7 count=3 files=
  /gc/2026/August/12.log.gz:8412345,/gc/2026/August/13.log.gz:7981233,...
```

This means that even if a day's data leaves the disk, the **name and size of the file** remain in
the intact chain.

---

## Practical recommendation

| If… | Recommended mode |
|---|---|
| You have a large disk | `off` (default) |
| Limited space but external media available | `archive` with `RETENTION_ARCHIVE_PATH` |
| Limited space and no external media | `auto` — aware that it is irreversible |

In all modes, before any action, move the archives to external media (release the seal first with
`chattr -i`).
