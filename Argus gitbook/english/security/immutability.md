# The Immutable Seal

Every past day, once compressed, is sealed with the **immutable** flag (`FS_IMMUTABLE_FL`) — the
same effect as `chattr +i`. This prevents a later root from silently overwriting or deleting a
day of the forensic record.

---

## This is not concealment

The seal does not hide the file. The file is, as before:

- listed (`ls`)
- read (`zcat`)
- verified (`argus-cli verify --date …`)

It blocks exactly one thing: **silent overwriting or deletion**.

---

## Releasing the seal (without a reboot)

```bash
sudo chattr -i <ARGUS_STORE>/2026/September/30.log.gz
```

At any time, without a reboot. If you genuinely need to move the file, release the seal first.

---

## The seal is idempotent

The seal is re-applied to every `.log.gz` seen during a scan — not only at the moment of
compression. An archive created before this feature existed therefore gets sealed on the next
pass.

---

## On filesystems without support

If the filesystem does not support the flag (for example, some network filesystems), the file
simply **remains unsealed**. Sealing never fails the compression run — it is best-effort.

---

## What gets sealed?

| Item | Sealed |
|---|---|
| Past-day archives (`.log.gz`) | yes |
| Today's file (`.log`) | no (the daemon is writing to it) |
| `embedded_salt` | yes (at install time) |
| `remove.key` | yes (when the password is set) |

---

## Relationship to retention modes

The `archive` and `auto` retention modes **release the seal first** before moving or deleting an
archive. The seal therefore does not obstruct designed retention work; it only prevents silent
tampering.

Details in [Retention](../deployment/retention.md).
