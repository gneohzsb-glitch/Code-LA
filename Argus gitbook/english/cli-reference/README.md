# CLI Reference

`argus-cli` is the operator interface to ARGUS. Most commands run **without `sudo`**; the
ones that read or write protected files require it.

---

## Command index

| Command | Purpose |
|---|---|
| [`status`](status-and-stats.md) | Shield, daemon, password, and disk status |
| [`count`](status-and-stats.md) | Block counts: today / week / month / total |
| [`stats`](status-and-stats.md) | Block count for today |
| [`list`](list-and-filters.md) | Last 100 blocks; supports filters |
| [`sensitive`](list-and-filters.md) | Only sensitive-path events |
| [`block`](list-and-filters.md) | Display a block, a range, or a list |
| [`tail`](list-and-filters.md) | Live view |
| [`verify`](verify-and-integrity.md) | Verify the chain (links + hashes) |
| [`integrity`](verify-and-integrity.md) | Integrity certificate for a range |
| [`anomalies`](verify-and-integrity.md) | Today's alerts |
| [`export`](export-and-siem.md) | JSON/CSV export |
| [`mode`](mode-and-retention.md) | Aggregation filter mode |
| [`retention`](mode-and-retention.md) | Retention policy |
| [`set-password`](shield-and-password.md) | Set the removal password |
| [`change-password`](shield-and-password.md) | Change the removal password |
| [`release`](shield-and-password.md) | Release the shield (no removal) |
| [`remove`](shield-and-password.md) | Full removal (logs are kept) |
| `help <command>` | Detailed help for a single command |

---

## Inline help

```bash
argus-cli help            # full command list
argus-cli help list       # help for the list command
argus-cli help retention  # help for the retention command
```

---

## Global options

| Option | Purpose |
|---|---|
| `--no-color` | Disable ANSI color |
| `--date MM/DD/YYYY` | Restrict to a single day |
| `--from MM/DD/YYYY` | Start from a date |
| `--last N` | Last N blocks |

**Environment variable:** `NO_COLOR=1` also disables color (a common standard). Color is
disabled automatically when output is piped (non-TTY) and when `TERM=dumb`, so scripts
always see plain text.

---

## Shell completion

A bash completion script is installed at `/etc/bash_completion.d/argus-cli`. To enable it
in a running shell:

```bash
source /etc/bash_completion.d/argus-cli
```

`argus-cli <Tab>` then suggests commands, and `argus-cli list --<Tab>` suggests options.
