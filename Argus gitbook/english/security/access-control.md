# Access Control

ARGUS has several access-control mechanisms, each protecting a different level: from file
access, to the operational password, to the machine lock.

---

## Access levels

| Level | What is required | For what |
|---|---|---|
| **`argus` group member** | group membership (no sudo) | all read commands: `list`, `tail`, `stats`, `count`, `verify`, `alerts`, `anomalies`, `integrity` |
| **Ordinary user (outside the group)** | nothing | sees no log data; the CLI directs them to `sudo`/the group |
| **root (`sudo`)** | root access | install, `remove`, `retention` (writes), `harden`, any mode change |
| **Removal password** | password + token | `release`, `remove`, `change-password` |
| **Encryption** | group membership or root | reading encrypted content (key from `machine.id` + `embedded_salt`) |

---

## The `argus` group (v15.4)

Before v15.4 the log tree used `0755/0644`, which left the whole forensic record readable by
every ordinary user on the machine. v15.4 hardens this into a group-based model:

| Path | Permission | Owner |
|---|---|---|
| `/etc/argus` | `2750` (setgid) | `root:argus` |
| Log directories | `2750` (setgid) | `root:argus` |
| Log files and keys | `0640` | `root:argus` |

The **setgid** bit on the directories means that files and subdirectories created by the daemon
(root) inherit the `argus` group from the start.

Adding an operator:

```bash
sudo usermod -aG argus <user>     # then log out / log in once
```

- **Group member** → reads without `sudo` (the decryption key is available too).
- **Outside the group** → does not even have `stat` on the tree; the CLI prints "run with sudo".
- The `0755/0644` model was deliberately removed.

Permission compliance can be checked with `sudo argus-cli harden --check`.

---

## What is root-only?

| Item | Why |
|---|---|
| `remove.key` / `pass.kdf` | `0600`, root only; the hash/derivation of the removal password |
| `embedded_salt` | `0640 root:argus`, `immutable`-sealed; the root of the encryption key |
| `machine.id` | `0640 root:argus`, `immutable`-sealed; the machine fingerprint |
| `anchor.key` | `0640 root:argus`; the HMAC signing key for the anchor |

---

## The removal path: two stages

Removing ARGUS requires two things, not one:

1. **The removal password** — whose hash is compared against `remove.key`.
2. **Writing the token into the kernel** — `shield_release(hash)` sends the token to the shield.

Neither is sufficient on its own. Someone who has only read the `remove.key` file cannot proceed
without the password, and someone who knows the password but does not have the file is likewise
blocked — though the shield also accepts the load-time token, so the operator is never locked
out.

---

## Machine lock

Both the shield and encryption are bound to `machine.id`:

```
machine.id = sha256( /etc/machine-id )
```

> **v15.4:** derivation uses `/etc/machine-id` only. Earlier versions also folded in
> hostname/MAC/CPU, which meant a hardware change or a rename could permanently orphan encrypted
> logs. If `/etc/machine-id` is missing, the only fallback source is the DMI UUID.

- A log copied to another machine **cannot be decrypted** (the derived key differs).
- Copying `remove.key` to another machine is also useless, because the shield is not armed there.

---

## File-level privilege separation

| File | Permission |
|---|---|
| `/etc/argus/remove.key`, `/etc/argus/pass.kdf` | `0600` |
| `/etc/argus/machine.id`, `embedded_salt`, `anchor.key` | `0640 root:argus` |
| Day log files | `0640 root:argus` |
| `.log.gz` archives | `0640 root:argus` + `immutable` seal |

Sealing and unsealing are done with the ioctl (`FS_IOC_SETFLAGS`), not `chattr(1)` — because
`chattr` can be replaced via `PATH` and would run as root.

---

## Software license separation (GPL)

| Component | License | Source |
|---|---|---|
| Shield + sensor (kernel) | GPL-2.0 | fully published |
| Daemon, CLI, helpers | proprietary | binary only |

This separation satisfies the GPL's legal requirement while preserving the intellectual property
of the userspace portion. Details in [License](../appendix/license.md).

---

## What access control is *not*

- ARGUS is not an **identity-management** system.
- It has no **RBAC**; access is based on root/non-root, not roles.
- It is not a **sudo replacement**; it uses the standard Linux access model.
