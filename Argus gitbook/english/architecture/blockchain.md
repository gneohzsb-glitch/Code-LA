# Hash Chain

The core of ARGUS's tamper-evidence is the **hash chain**. Every event becomes a block whose
hash embeds the hash of the previous block. This dependency makes it impossible to remove or
edit any block silently.

---

## The hash rule

```
h = sha256( prev_hash + event + timestamp + uid + pid )
```

- `prev_hash` is the hash of the previous block.
- The **first block** of each chain has `prev_hash` = 64 zeros (genesis).

> **Important:** the `f` field (sensitive flag) is deliberately **not part of the hash
> input**. This keeps the hash formula unchanged from earlier versions, so every historical
> block remains valid.

---

## Block format

Each block is one JSON line:

```json
{"t":1790795115,"e":"openat: /etc/passwd flags=0","ph":"<64 hex>","h":"<64 hex>",
 "u":0,"p":1234,"pp":1,"c":"cat","hn":"host","un":"root","ip":"127.0.0.1"}
```

| Field | Meaning |
|---|---|
| `t` | timestamp (wall clock, seconds) |
| `e` | event text (hashed) |
| `ph` | previous block's hash |
| `h` | this block's hash |
| `u` | uid |
| `p` | pid |
| `pp` | ppid |
| `c` | process name (`comm`) |
| `hn` | hostname |
| `un` | username |
| `ip` | associated external IP (when present) |
| `f` | flag (sensitive), **not hashed** |

---

## Block types

ARGUS produces several block types, all chained with **the same hash formula**:

| Type | Source | Purpose |
|---|---|---|
| Normal event | `blockchain_add_event()` | Any recorded syscall |
| `AGGREGATE` | `blockchain_add_synthetic()` | Counted summary of aggregated reads |
| `AGGREGATE-RECOVERED` | `blockchain_add_synthetic()` | Recovery of an orphaned sidecar after a crash |
| `CONFIG` | `blockchain_add_synthetic()` | Allowlist hash + mode + window on change |
| `MODE_CHANGE` | `blockchain_add_synthetic()` | Filter mode change |
| `MAINTENANCE` | `blockchain_add_synthetic()` | A retention action (`archive`/`auto` mode) **before** it happens |

Synthetic blocks carry a **wall-clock** timestamp (not monotonic), because they describe a
window that has closed and has no monotonic sensor stamp. **No new field was added to the
hash formula**, so all previous blocks remain valid.

---

## Why does this work?

If an attacker wanted to remove or edit a middle block numbered `k`:

1. The hash of block `k` changes (because the content changed).
2. Block `k+1` still holds the old `ph` → a **broken link**.
3. If the attacker also updates the `ph` of block `k+1`, the hash of block `k+1` changes →
   the break moves to `k+2`.
4. To hide it completely, every block to the end of the chain would have to be recomputed —
   in effect, rewriting the entire history.

`argus-cli verify` recomputes both the **link** and the **hash**, so all three attacks are
detected:

| Attack | Detected by |
|---|---|
| Editing a block's content (hash left untouched) | `BAD HASHES` |
| Reordering/re-linking a block | `BROKEN LINKS` |
| Deleting a block from the middle | `BROKEN LINKS` |

---

## Recovery after restart

Every daemon restart that started the chain from scratch would break it. On startup, ARGUS
reads the last `.log` (or `.log.gz`) block of the day and continues `prev_hash` from it. This
was one of the bugs fixed along the way (item 7.13).

---

## The in-memory capacity limit

The daemon keeps a retention limit in memory (FIFO), but every block is written to disk
immediately; memory is therefore never the real ceiling on history. The history lives on disk
and is read by `verify` and `list`.
