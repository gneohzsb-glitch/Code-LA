# Log Hiding

ARGUS keeps its history in paths and names that at first glance look like part of the operating
system. This is **obfuscation**, not invisibility. The goal is to raise the cost for a hurried
attacker to find and identify the history — it is not absolute concealment from a determined
analyst.

> **Honesty first:** this layer adds friction; it does not by itself resist an analyst who
> already knows a deployment is present. It is one layer among several, and the hard guarantees
> come from the hash chain and encryption. This layer's job is to make an attacker who does not
> already know what to look for spend more time.

---

## Layer 1 — Disguised path and names

The history is written under a root path whose components are chosen to resemble ordinary
operating-system data (a session/utmp-like tree, a cache-like directory). The exact components
are an **internal detail and are not published** — see [Path reference](../appendix/paths.md);
a document, screenshot, or `grep` that spelled them out would defeat the disguise. What a reader
needs is only the shape:

```
<ARGUS_STORE>/<year>/<month>/<day>.log
```

The file names (`DD.log`) and the date structure (`YYYY/Month/DD`) look natural and unrelated to
security.

---

## Layer 2 — Decoy tree (optional)

The `build/camouflage_engine` tool can create **10,000 decoy directories**. The real directory
is lost among them. Anyone searching for logs with `ls` or `find` is met with a mass of empty
directories.

```bash
sudo argus-cli harden --decoys      # create/rebuild the decoy tree
ARGUS_DECOYS=1 sudo ./install.sh    # or at install time
```

> **v15.4:** earlier installers wrote two metadata files, `log_name` and `decoy_dirs`, into
> `/etc/argus`. Neither was ever read, and `log_name` duplicated information about the real
> storage location that the decoy tree exists to obscure. Both were removed. Decoys are now
> created only on explicit request (off by default), since 10,000 empty directories are not
> security by themselves.
>
> Decoy directories use the same `0700` permissions as the real tree; a `0755` decoy next to a
> `0750` tree would itself point at the real path.

---

## Layer 3 — The immutable seal

Every past day, once compressed, is sealed with the `immutable` flag. This prevents a later root
from silently overwriting or deleting a day of the forensic record. Details in
[The Immutable Seal](immutability.md).

---

## Layer 4 — Machine-locked encryption

If `ENCRYPT=1`, the content of every line is encrypted. The key is derived from `machine.id` +
`embedded_salt`, so a log copied to another machine cannot be read. Details in
[Encryption](encryption.md).

---

## Layer 5 — Binary stripping

Shipped binaries are **stripped**. Without symbols, an attacker using `strings` and `objdump`
cannot quickly extract function names and internal structure. (This is done in `install.sh`; the
build tree is left untouched for developers.)

---

## What is *not* hidden

| Item | Why |
|---|---|
| Kernel module | visible in `lsmod` — intentional; it must be auditable and loadable in the standard way |
| Daemon | an ordinary systemd service |
| Project name in install paths | `/usr/local/bin/argus-cli` and `/usr/local/share/argus/` |

**Why isn't the kernel module disguised?** Because kernel code must be auditable (a GPL
requirement), and disguising a module is itself a rootkit technique that conflicts with the goal
of being *trustworthy*. ARGUS invests confidentiality in its *data*, not in its own *existence*.

---

## An honest summary

| Layer | What it contributes | Nature of the protection |
|---|---|---|
| Disguised path/names | raises the cost of casual discovery | friction, not a guarantee |
| Decoy tree | raises the cost of casual discovery | friction, not a guarantee |
| Immutable seal | prevents silent modification and detects attempts | a reversible-by-design protection |
| Encryption | protects content even if the log is copied | a cryptographic guarantee, while the key is safe |
| Binary stripping | raises the cost of reverse engineering | friction, not a guarantee |

This table describes our claim precisely: concealment creates **cost**; encryption and the chain
create **guarantees**.
