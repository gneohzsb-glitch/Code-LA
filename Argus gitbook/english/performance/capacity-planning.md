# Capacity planning

This page provides a **model** for estimating disk space. Because the event rate differs from
host to host, the final number must be measured on the real host — but the model makes the
order of magnitude clear.

---

## Base formula

```
daily space (pre-compression)  = daily_block_rate × block_size
daily space (post-compression) = daily space ÷ 5.4
total space                    = sum over retained days
```

---

## Parameters

### Block size

Each block is one JSON line:

```json
{"t":1790795115,"e":"openat: /etc/passwd flags=0","ph":"<64 hex>","h":"<64 hex>",
 "u":0,"p":1234,"pp":1,"c":"cat","hn":"host","un":"root","ip":"127.0.0.1"}
```

| Event type | Approximate size |
|---|---|
| Short event (no IP) | ~150 bytes |
| Event with IP | ~200 bytes |
| Aggregation summary block | ~250 bytes |

A conservative average: **~200 bytes**.

### Daily block rate

The rate depends on the host's workload pattern. The range measured on the reference VM:

| Mode | Rate | Blocks/day |
|---|---|---|
| Raw (pre-filter) | ~326 / 30s | ~940,000 |
| Post-noise-filter | ~36 / 30s | ~104,000 |

On a real host, the post-filter rate is typically between **50,000 and 300,000 blocks per
day**, depending on the number of processes and the file-write pattern.

---

## Capacity table

Assuming a 200-byte block size and a 5.4× compression ratio:

| Blocks/day | Raw/day | Compressed/day | 30 days | 90 days | 365 days |
|---|---|---|---|---|---|
| 50,000 | ~10 MB | ~1.9 MB | 57 MB | 170 MB | 690 MB |
| 100,000 | ~20 MB | ~3.7 MB | 111 MB | 330 MB | 1.3 GB |
| 200,000 | ~40 MB | ~7.4 MB | 222 MB | 660 MB | 2.7 GB |
| 300,000 | ~60 MB | ~11 MB | 330 MB | 990 MB | 4 GB |

> These figures do not include a safety margin. Always allow **~20% more** free space than
> the estimate.

---

## Effect of retention modes

| Mode | Effect on space |
|---|---|
| `off` (default) | Unbounded growth; compression only (~5.4×) |
| `archive` | Host space stays constant; data moves to external media |
| `auto` | Host space is freed by deleting the oldest days |

### Example: choosing a policy

Suppose your host produces 100,000 blocks/day and you have 500 GB of disk:

- Compressed/day ≈ 3.7 MB
- With `off`, the disk fills after ~135,000 days (effectively unlimited) — **but** the disk
  is shared with the OS and other data.
- If you want unlimited history, `off` with the 85% warning is sufficient.
- If space is constrained, set `archive` to an external mount.

---

## Capacity versus peak rate

The event rate is not constant. During package installs, compiles, or scans, the rate can
multiply. The model above takes the average. For peaks:

- **Buffering:** the daemon writes every block immediately, so a peak does not lose data; it
  only raises the disk-fill rate temporarily.
- **`performance` mode:** a larger aggregation window (3600s) reduces volume further during
  high-traffic periods.

---

## Practical recommendations

| Need | Suggested setting |
|---|---|
| Unlimited history, large disk | `off` (default) |
| Unlimited history, limited space | `archive` to external media |
| Constant space, deletion-aware | `auto` with a conservative threshold |
| Volume reduction during peaks | `performance` mode + an allowlist derived from real data |

---

## What to measure on your own host

1. **Blocks/day:** run `argus-cli count` on several consecutive days.
2. **Block size:** `sudo du -sh <ARGUS_STORE>/` and divide by the
   block count.
3. **Peak rate:** run `argus-cli stats` repeatedly during busy hours.

Then fill in the formula at the top of this page with your own numbers.
