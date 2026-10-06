# Security

ARGUS security rests on three pillars: **tamper-evidence** (the hash chain), **self-protection**
(the kernel shield), and **optional confidentiality** (path concealment and encryption). This
section opens up all three — and, most importantly, explains the **real boundaries** honestly.

---

## The three pillars

| Pillar | Mechanism | Page |
|---|---|---|
| Tamper-evidence | SHA-256 hash chain | [Tamper Resistance](tamper-resistance.md) |
| Self-protection | Kernel shield (4 kprobes) | [Tamper Resistance](tamper-resistance.md) |
| Path confidentiality | Disguised path and names + decoy directories | [Log Hiding](log-hiding.md) |
| Content confidentiality | AES-256, locked to the machine | [Encryption](encryption.md) |
| Archive locking | `immutable` flag | [The Immutable Seal](immutability.md) |

---

## Governing principle: honesty about limits

ARGUS never makes an impossible claim. The threat model is explicit:

- Against an **attacker without root**: protection is strong.
- Against a **determined root**: the goal is to **raise the cost**, not to prevent absolutely.
  A privileged operator with full control of the running kernel can still halt the host or
  interfere below the layer at which ARGUS operates.
- Against an **attacker with physical access**: out of scope.

Full details in [Threat Model](threat-model.md).

---

## "Never locked out" is a security principle

Every lock in ARGUS has a **designed release path**:

- The shield **will not load at all** without a release token.
- Archives are released with `chattr -i`.
- Encryption is optional and is turned off with `ENCRYPT=0`.

The result: **there is no scenario that can only be resolved by a reboot.** This principle even
caused designs based on eBPF LSM and GRUB changes to be rejected.
