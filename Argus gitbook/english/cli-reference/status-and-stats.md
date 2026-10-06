# status and stats

## `argus-cli status`

A system-wide status overview:

```bash
argus-cli status
```

```
   ╔══════════════════════════════════════════════════════════╗
   ║              ARGUS // SURVEILLANCE SYSTEM              ║
   ╚══════════════════════════════════════════════════════════╝

  SHIELD:    ◆ ACTIVE
  RELEASE:   ◆ locked
  DAEMON:    ◆ 712
  LICENSE:   ◆ VALID (2026-12-31, 88 days)
  PASSWORD:  ◆ SET
  STORAGE:   ◆ 93% used  [!] HIGH — free space or archive old days
```

| Line | Meaning |
|---|---|
| `SHIELD` | Whether the shield module is loaded |
| `RELEASE` | `locked` (token armed) or `released` |
| `DAEMON` | PID of the protected daemon, or `NOT PROTECTED` |
| `LICENSE` | License status and days remaining |
| `PASSWORD` | Whether a removal password is set |
| `STORAGE` | Disk usage percentage; warns above 85% |

> To see the `PASSWORD` line, run the command with `sudo` (the password file is readable by
> root only).

---

## `argus-cli count`

A complete block count across four ranges:

```bash
argus-cli count
```

```
  ◆ BLOCK COUNT

  Today:   32768
  Week:    196412
  Month:   154585
  Total:   368483
```

| Range | Definition |
|---|---|
| `Today` | Today (local time) |
| `Week` | The last 7 days |
| `Month` | The current month |
| `Total` | All history |

`count` includes both compressed days (`.log.gz`) and encrypted days.

---

## `argus-cli stats`

A fast count of today's blocks:

```bash
argus-cli stats
```

Use `count` when you need the full breakdown across ranges.

---

## `argus-cli service` and `argus-cli logs`

```bash
argus-cli service status
```

```
  ◆ ARGUS SERVICES

  capture engine  active
  resolver        active
```

| Line | Meaning |
|---|---|
| `capture engine` | The daemon that records, chains and stores |
| `resolver` | The helper that maps a PID to its remote IP |

`sudo argus-cli service restart` restarts both. A restart detaches the eBPF sensor and
leaves a capture gap, so prefer a configuration change (applied within 5 s) unless you have
installed a new binary.

```bash
argus-cli logs --last 100
```

`logs` prints the last N service-log lines (default 40). The log is read **through ARGUS**,
so no service name has to be typed or shown.

> **Services are named by role.** ARGUS runs its services under deliberately ordinary
> systemd names; the CLI never prints them, and neither does this book. Use
> `service status` / `logs` to observe them.

---

## Diagnostic codes

When a command fails, the last line carries a short, opaque code:

```
  ◆ ACCESS DENIED [ARG-205]
```

The code identifies the failure without describing how ARGUS works internally. Quote it to
support; the meaning is looked up on our side. Normal (`◆` green) output carries no code.

---

## How `status`, `stats`, and `count` differ

| Command | What it reports | Speed |
|---|---|---|
| `status` | System state (shield, disk, password) | Immediate |
| `stats` | Today's block count | Fast |
| `count` | Four count ranges (today/week/month/total) | Somewhat slower (walks the whole tree) |
