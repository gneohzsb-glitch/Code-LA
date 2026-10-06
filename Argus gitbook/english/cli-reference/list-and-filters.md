# list and filters

## `argus-cli list`

Default: the last 100 blocks.

```bash
argus-cli list                   # last 100 blocks
argus-cli list --last 20         # last 20 blocks
argus-cli list --from 09/30/2026 # a specific day
```

| Option | Purpose |
|---|---|
| `--last N` | Last N blocks |
| `--external` | Tally blocks that have an external IP (no cap) |
| `--sensitive` | Only sensitive-path events |
| `--from MM/DD/YYYY` | A specific day |
| `--no-color` | Disable color |

A block that opened a sensitive path is tagged `[SENSITIVE]`.

---

## `argus-cli list --external`

A complete tally of blocks that carry an external IP, **with no cap**, sorted from
heaviest traffic first:

```bash
argus-cli list --external
```

```
  ◆ 12148 BLOCKS WITH EXTERNAL IP — 4 DISTINCT

  10.0.2.2:        6668 blocks
  185.125.188.55:  3164 blocks
  91.189.91.83:    168 blocks
  ...
```

> **History:** this command once capped at 10,000 matches and 256 distinct IPs and reported
> an incomplete total. It now counts every block in a single pass using an open-addressing
> table, removing the extra `wc -l` pass (a 109 MB day completes in ≈ 0.6 seconds).

---

## `argus-cli sensitive`

Only sensitive-path events, as a standalone command:

```bash
argus-cli sensitive
argus-cli sensitive --last 10
argus-cli sensitive --from 09/30/2026
```

This is equivalent to `argus-cli list --sensitive`; both share the same code path and
produce identical output.

A "sensitive path" is access to `/etc/shadow`, SSH keys, `/root/`, `/etc/argus`, the module
tree, and writes to system configuration files.

---

## `argus-cli block`

Display a specific block, a range, or a list:

```bash
argus-cli block 1                # a single block
argus-cli block 5-10             # a range
argus-cli block 1,3,7            # a list
argus-cli block 42 --date 09/30/2026
```

---

## `argus-cli tail`

A live view of new blocks:

```bash
argus-cli tail
argus-cli tail --from 09/30/2026
# Ctrl+C to exit
```

---

## Reading compressed days

Every read command also reads compressed days (`.log.gz`). `verify` works on a compressed
day and reports on its own that the file is compressed.

```bash
argus-cli list --from 09/30/2026    # still works if that day is compressed
```

---

## Practical examples

```bash
# Busiest external IPs
argus-cli list --external | head -20

# All access to sensitive files today
argus-cli sensitive

# Blocks for a specific day
argus-cli list --from 09/30/2026 | head -50

# Inspect a particular block
argus-cli block 42
```
