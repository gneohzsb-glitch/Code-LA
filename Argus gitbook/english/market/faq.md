# FAQ

## General

### How does ARGUS differ from syslog or auditd?

Traditional logs (syslog, auditd, journald) are **editable** and **unchained**: a privileged
attacker can erase the lines that concern them and leave no trace. ARGUS turns each event into
a chained, hashed block, so any change is detectable.

### Does ARGUS prevent intrusions?

No. ARGUS is a **recording and proof** system, not a prevention system. Its job is to give you
a trustworthy record after an incident. (The kernel shield protects only *itself*.)

### Does it require a reboot?

**Never.** Not to install, not to update, not to remove. This is a firm design principle; it
even led us to reject designs based on eBPF LSM and GRUB changes.

### Which operating systems does it run on?

It has been tested on Ubuntu 22.04 LTS (kernel 5.15) and Debian 12 (kernel 6.1). It requires
kernel 5.15+ with `CONFIG_BPF_SYSCALL` and `CONFIG_DEBUG_INFO_BTF` (the default on modern
distributions).

---

## Performance and footprint

### How much space does it take?

It depends on the host's event rate. Past days are compressed automatically (~5× smaller). For
a full model, see [Capacity planning](../performance/capacity-planning.md).

### Are logs ever deleted?

**By default: never.** Past days are only compressed and sealed. Two optional modes
(`archive`, to move data to external media, and `auto`, to delete) exist for hosts with
limited space.

### What is the system overhead?

The eBPF sensor sends only raw events, and the logic lives in userspace. Noise filtering
reduces volume by ~9×. Exact numbers are in [Benchmarks](../performance/benchmarks.md).

---

## Security

### What if the attacker is root?

ARGUS makes no claim against an administrator who is already fully privileged. Its purpose is
to raise the cost of tampering and to make any attempt visible. In practice:

- Obvious attempts (`kill`, `killall`, `rmmod`) are blocked.
- Deleting or editing archives breaks the chain and is detected.
- An attempt to erase the record is itself a recorded event.

### Are logs encrypted?

Optionally. With `ENCRYPT=1`, content is encrypted with AES-256 and the key is locked to the
machine (a copied log is unreadable elsewhere). It is off by default so that reading without
`sudo` remains possible.

### Can ARGUS be removed?

Yes, with a password. `sudo argus-cli remove` removes the binaries and configuration but
**leaves the logs intact**. It never locks up.

---

## Integration

### How does it connect to a SIEM?

Two ways: live syslog forwarding (destinations `TCP:`/`UDP:`/`FILE:` in `export.conf`), or
manual output with `argus-cli export --format json|csv`. Sensitive events are sent with the
`ARGUS_SENSITIVE` tag.

### Does it have an API?

`argus-cli` is a command-line interface. For programmatic access, parse the output of
`export --format json`. There is no REST API.

### Does it work with standard syslog?

Yes, ARGUS forwards events live to a syslog destination.

---

## License

### Do I get the source?

For the kernel components (shield and sensor) **yes** — under GPL-2.0 with full source. For the
daemon and CLI **no** — proprietary, binary only. Details in [License](../appendix/license.md).

### Can I embed it in my own product?

It depends on your contract terms. See `LICENSE/PROPRIETARY.txt`.

---

## Troubleshooting

### The shield does not load.

`/etc/argus/remove.key` must exist. Without it, the shield deliberately does not load (so that
a reboot-only lock-up cannot occur). Run `sudo argus-cli set-password`.

### `verify` says blocks were skipped.

If `ENCRYPT=1` is set, run `verify` with `sudo` (the salt is readable only by root). More in
[Troubleshooting](../deployment/troubleshooting.md).
