# User-Space Daemon

**Directory:** `userspace/daemon/` · **Size:** 18 files, ~2300 lines · **License:** proprietary

The daemon — shown as the *capture engine* in `argus-cli service status` — is the brain of ARGUS: it reads raw events from the
ring buffer, filters them, turns them into chained blocks, writes them to disk, raises
alerts, and forwards to the SIEM.

---

## Files

| File | Role |
|---|---|
| `main.c` | Startup and shutdown ordering |
| `sensor_loader.c` | Loading the eBPF program and attaching the 21 hooks |
| `shield_register.c` | Registering its own PID with the shield (with read-back confirmation) |
| `reader.c` | Reading the ring buffer + anti-feedback filter (hot path) |
| `classifier.c` | Noise/sensitive classifier (split out of reader so it is testable without the eBPF path) |
| `aggregator.c` | Aggregating repetitive reads + modes + crash-safe sidecar |
| `blockchain.c` | Building blocks, hashing, JSON escaping, the `f` field |
| `storage.c` | Writing to the disguised path + reading `.gz` to continue the chain |
| `hasher.c` | SHA-256 |
| `ipcache.c` | Receiving `IP:<pid>:<ip>` over a UNIX socket |
| `threads.c` | Background threads |
| `retention.c` | Compressing past days + retention modes + disk alert |
| `alert_engine.c` | Alert rules |
| `baseline_handler.c` | Baseline learning |
| `remote_anchor.c` | Sending the chain head outbound |
| `export_handler.c` | Log export (with a distinct tag for flagged events) |
| `evidence_handler.c` | Building the SHA-256 manifest for evidence (shared with the CLI) |
| `encryptor.c` | AES-256 with a machine-bound key + log-line codec |

---

## Noise filtering and the sensitive-path flag (`classifier.c`)

Measurement on a real host showed that `openat` accounted for 97% of all blocks, and of
that 97% only ~2.6% was worth keeping. A chain in which the signal is one block in forty is
not a forensic record; it is a haystack.

Classification is applied **only to `openat`** (no other event is filtered):

| Category | Behavior |
|---|---|
| Pure churn (journal spool, locale trees, character devices, `modules.*.bin` caches, public CA certificates) | Discarded — both reads and writes |
| Reads of libraries/`/proc`/`/sys`/`/dev`, directory traversal (`O_DIRECTORY`), relative names arising from it | Discarded — **only if it is a read** |
| A write to any path in the category above | **Kept** (planting a payload in `/lib` or `/dev/shm` must be visible) |
| Sensitive material (`/etc/shadow`, SSH keys, `/root/`, `/etc/argus`, the module tree) | Kept **and flagged** — on every access |
| Writes to system configuration files (`/etc/passwd`, `sudoers`, `pam.d`, `crontab`, `ld.so.cache`) | Kept **and flagged**; reading them is noise |

### Two design decisions that keep the filter honest

1. The "sensitive" check is performed **before** the noise rules, so a relative name such as
   `openat(fd, "id_rsa")` is not mistakenly classified as noise.
2. The flag is stored in the block's `f` field but is **not part of the hash input** — the
   hash formula stays untouched (otherwise every block written to date would become a "bad
   hash"). Because the path is inside the event text (`e`) that is hashed, the flag can be
   recomputed from the hashed data; so excluding it from the hash weakens nothing. `f` is a
   routing hint for the SIEM, not an independent claim.

Every flagged event also raises a `Sensitive Path Access` alert (de-duplicated by type+pid,
so that a loop reading a key file does not flood the alert log).

---

## Event aggregation (`aggregator.c`)

Noise filtering *deletes* very high-frequency events. For the category that must remain but
is large in volume, a third layer was added that **does not delete — it counts**: a
repetitive read-only `openat` is accumulated in a per-window counter and, at the end of the
window, written to the chain as **a single summary block**. No event is lost — it is either
an individual block or a number in a summary.

Four rules keep this citable:

- **Reads only.** No write is ever aggregated. Planting a payload under a prefix that an
  allowlist rule covers stays a first-class event.
- **The key is the real executable, not `comm`.** Identity is read from `/proc/<pid>/exe`
  (dev:ino). `comm` is set by the process itself via `prctl(PR_SET_NAME)`; if the key relied
  on it, an attacker could name themselves `systemd` and vanish. If the exe cannot be
  determined, the event is recorded **individually** (fail-open to recording).
- **The number is inside the hashed text.** The summary is hashed with the existing formula:
  `AGGREGATE: count=N prefix=P exe=E uid=U window=A-B`.
- **Counters are persisted to a sidecar every 5 seconds.** A reboot or crash does not erase
  the window; on the next start the daemon appends any orphaned sidecar to the chain as an
  `AGGREGATE-RECOVERED` block.

### Modes (window size only)

| Mode | Window | Meaning |
|---|---|---|
| `performance` | 3600s | Aggressive aggregation |
| `balanced` (default) | 300s | Normal aggregation |
| `forensic` | 0 | No aggregation — every event is individual |

No mode turns recording off. **The default allowlist is empty**: aggregation happens only
when a rule in `/etc/argus/allowlist.conf` names the exact executable path and path prefix.
Until then, behavior is identical to the previous version.

The SHA-256 of the allowlist file, the mode, and the window are recorded as a
`CONFIG: allowlist-sha256=… mode=… window=…` block whenever they change; tampering with the
allowlist is visible in the chain.

---

## Retention (`retention.c`)

The default policy: **compress, never delete, and warn at 85% so I decide.** Full details are
in [Retention](../deployment/retention.md).

---

## Encryption (`encryptor.c`)

Encryption is optional and enabled with `ENCRYPT=1` in `/etc/argus/export.conf`. The key is
derived with PBKDF2-HMAC-SHA256 (500,000 iterations) from `machine.id` + `embedded_salt`, and
is never stored on disk. Details are in [Encryption](../security/encryption.md).

---

## The IP helper (`ip_resolver.c`)

Reads `/proc/net/tcp` to obtain the inode of sockets with an external connection, then finds
the owner of each inode and tells the daemon "PID X is talking to IP Y". The daemon writes
this into the block's `ip` field.
