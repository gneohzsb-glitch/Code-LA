# verify and integrity

## `argus-cli verify`

A full chain verification: both the **link** and the **hash** of every block are recomputed.

```bash
sudo argus-cli verify
sudo argus-cli verify --date 09/30/2026
```

A healthy result:

```
  ◆ ALL 368137 BLOCKS VERIFIED — NO BREAKS
```

A result with tampering:

```
  ◆ 18 BROKEN LINKS / 0 BAD HASHES / 126369 BLOCKS
```

| Indicator | Meaning |
|---|---|
| `NO BREAKS` | The chain is intact |
| `BROKEN LINKS` | A block was moved/removed (or `prev_hash` does not read) |
| `BAD HASHES` | A block's content changed without its hash being recomputed |

> **Important:** `verify` does not only check links — it **recomputes the hash as well**.
> Content edits are therefore detected even if the chain link is left consistent.

### When encryption is enabled

`verify` on an encrypted log requires `sudo` (the salt is readable by root only). Without
root, the number of unread blocks is reported:

```
◆ 237 ENCRYPTED BLOCK(S) SKIPPED
```

### Reading compressed days

`verify` also works on `.log.gz` and reports that the file is compressed.

---

## `argus-cli integrity`

An integrity certificate for a time range — a **presentable piece of evidence**:

```bash
sudo argus-cli integrity --from 09/01/2026 --to 09/30/2026
```

Output:

```
  ARGUS INTEGRITY CERTIFICATE
  ============================
  Date Range:         09/01/2026 to 09/30/2026
  Total Events:       ...
  Files Verified:     ...
  Chain Hash:         <sha256>
  Verified:           YES - NO TAMPERING
  ============================
  VERIFICATION: sha256sum <log_file> should match hash above
```

The manifest produces a single SHA-256 hash for the whole range that **an auditor can verify
independently with `sha256sum`** — without ARGUS.

---

## `argus-cli anomalies`

Today's alerts:

```bash
argus-cli anomalies
argus-cli anomalies --date 09/30/2026
```

> **v15.4:** this command now reads **from the chain** — the same typed blocks
> `{"e":"ALERT"}` / `{"e":"HINT"}` that `argus-cli alerts` reads, using the same parser.
> The previous version scanned raw event text for the string `"ANOMALY"`, which was never
> emitted anywhere in the codebase, and re-implemented detection logic with a separate regex
> that had drifted from the alert engine. There is now a single source of truth.

Alert rules include: brute-force, module removal, privilege escalation, file deletion, log
tampering, chain breakage, out-of-hours access, rapid fire, anomalous behaviour, storage
pressure, filter-mode change, and sensitive-path access.

---

## How `verify` and `integrity` differ

| | `verify` | `integrity` |
|---|---|---|
| Scope | One day or all history | An arbitrary range |
| Output | Gap counts | Manifest + hash |
| Use case | Routine health check | Evidence for audit/court |
| Requires root | Only when encryption is enabled | Yes |

---

## What is detected?

| Scenario | What `verify` reports |
|---|---|
| Healthy chain | `NO BREAKS` |
| Content edit (link left intact) | `BAD HASHES` |
| Re-linked block | `BROKEN LINKS` |
| Block removed from the middle | `BROKEN LINKS` |

All four cases are covered by `tests/test_verify.sh`.
