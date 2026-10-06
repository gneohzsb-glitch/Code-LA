# Performance

ARGUS is designed so that its overhead on the host system stays **minimal**, while still
producing a complete and untouched record. This section presents the measured numbers and the
capacity-planning logic behind them.

---

## Performance principles

| Principle | How it is achieved |
|---|---|
| No heavy work in the kernel | The eBPF sensor only emits raw events; the logic lives in userspace |
| Filter at the closest point | Noise is discarded before it reaches the chain |
| Linear writes | Each block is a single append; no file rewrites |
| Aggregation without loss | Repetitive reads are folded into one counter instead of thousands of blocks |
| Background compression | Past days are compressed on the retention thread |

---

## Pages in this section

| Page | Topic |
|---|---|
| [Benchmarks](benchmarks.md) | Measured numbers on the reference VM |
| [Capacity planning](capacity-planning.md) | Estimating space and rate for a real host |

---

## Summary of the numbers

| Metric | Value |
|---|---|
| Recording rate (post-filter) | ~36 blocks per 30 seconds |
| Recording rate (pre-filter) | ~326 blocks per 30 seconds |
| Self-capture events | 0 |
| Compressing one day of logs | 46 MB → 8.4 MB (5.4×) |
| `list --external` on a 109 MB day | ~0.6 seconds |
| `verify` on 368,000 blocks | ~3.4 seconds |

> These numbers were measured on the reference VM (Ubuntu 22.04, kernel 5.15) and depend on
> the hardware and the workload pattern.
