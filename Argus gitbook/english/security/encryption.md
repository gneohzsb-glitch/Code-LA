# Encryption

ARGUS log encryption is **optional** and **off by default**. When enabled, the content of every
line is encrypted with AES-256-CBC and the key is locked to the machine — a log copied to another
machine cannot be read.

---

## Turning it on and off

```bash
# on
sudo sed -i 's/^ENCRYPT=.*/ENCRYPT=1/' /etc/argus/export.conf
sudo argus-cli service restart

# off
sudo sed -i 's/^ENCRYPT=.*/ENCRYPT=0/' /etc/argus/export.conf
sudo argus-cli service restart
```

After it is enabled, new lines begin with the `AGRS1:` marker:

```bash
sudo tail -1 <ARGUS_STORE>/2026/October/01.log | cut -c1-40
# AGRS1:...
```

---

## Key derivation

```
key = PBKDF2-HMAC-SHA256( machine.id + embedded_salt, 500000 iterations )
```

- `machine.id` = the machine fingerprint, derived from `/etc/machine-id` (see
  [Access Control](access-control.md))
- `embedded_salt` = 32 random bytes, readable only by root, sealed `immutable`
- **The key is never stored on disk** — it is derived each time

The result: copying the log *and* `embedded_salt` to another machine is **not enough**, because
`machine.id` is different there.

---

## Line format

```
AGRS1:<base64( IV || ciphertext )>
```

- `AGRS1` = version marker
- A line without this marker is plaintext from an earlier version

So old days (plaintext) and new days (encrypted) coexist in one directory and **no migration is
needed**.

---

## The hash is computed over the plaintext

Order matters:

1. The chain hash is computed over the **plaintext** and **before** encryption.
2. `verify` decrypts first, then checks the hash.

Encryption is therefore **transparent** to the chain, and nothing in the hash formula changes.

---

## Three integration points

| Point | Role |
|---|---|
| `storage_write_line` | encrypts the line before writing |
| `storage_last_block` | decrypts the last line to continue the chain |
| `export_handler` and all readers in `cli.c` | decrypt first, then process |

`retention` and `evidence` work unchanged (the latter hashes the file bytes, so encryption is
transparent to it).

---

## Operational note

`embedded_salt` is readable only by root, so reading an encrypted log requires `sudo`. When run
without root, `verify` explicitly reports the number of unread blocks:

```
◆ 237 ENCRYPTED BLOCK(S) SKIPPED
```

This honest reporting is intentional: better than staying silent about unverified blocks.

---

## Security considerations

| Topic | Status |
|---|---|
| Algorithm | AES-256-CBC |
| Key derivation | PBKDF2-HMAC-SHA256, 500,000 iterations (resistant to brute-force) |
| IV | random per line |
| Machine lock | yes — the key is bound to `machine.id` |
| Key stored on disk | no |
| Authenticated encryption (AEAD) | no — CBC + the hash chain; integrity comes from the hash |

> **Note:** CBC provides confidentiality; authenticity in ARGUS comes from the **hash chain**,
> not from the encryption. This separation is deliberate: each layer does one job, and integrity
> is guaranteed by the chain regardless of the cipher.
