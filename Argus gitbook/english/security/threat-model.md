# Threat Model

Who ARGUS is designed for, what it resists, and where it honestly yields. This is the most
important page in the security section, because **overclaiming is itself a vulnerability**.

---

## In scope

| Attacker | Capability | What ARGUS does |
|---|---|---|
| **Attacker without root** | runs code as an ordinary user | full logging; cannot bypass it |
| **Transient-root attacker** | temporary, hurried root, wants to erase their tracks | `kill`/`killall`/`killpg` blocked; `rmmod` blocked; attempts to delete archives are detected |
| **Unaware attacker** | does not know ARGUS is installed | the disguised path and names plus the decoy tree slow them down |
| **Later data tampering** | a root tomorrow wants to change yesterday's log | archives are `immutable`-sealed; any change breaks the chain |
| **Disclosure of a copied log** | the log leaves the machine | with `ENCRYPT=1`, the key is locked to the machine |

---

## Out of scope

| Attacker | Why it is out of scope |
|---|---|
| **Determined, knowledgeable root** | a fully-privileged operator can legitimately release the shield using the removal password and retains full control of the host's lifecycle. ARGUS assumes the kernel itself is trusted |
| **Attacker with physical access** | can detach the disk; outside the model |
| **Attacker at boot** | if the boot path (GRUB/initrd) is under the attacker's control, the host is already compromised before ARGUS runs |
| **PID 1 impersonation** | taking over PID 1 means the machine is already in the attacker's hands |

> **Note:** against a determined root, ARGUS's goal is to **raise the cost**, not absolute
> prevention. Any stronger protection (such as eBPF LSM) would require changing GRUB and
> **rebooting** — which contradicts the project's hard requirement of never requiring a reboot.
> It is therefore a conscious decision that, against absolute root, the shield yields.

---

## Why the PID 1 exception does not weaken anything

A signal from PID 1 (systemd) passes through so that `systemctl stop/restart` works. This is not
a weakness: taking over PID 1 means the attacker already has full control of the machine, and at
that point the shield is the least of the concerns.

---

## Threat matrix

| Threat | Coverage | Mechanism |
|---|---|---|
| Erasing logs after intrusion | strong | hash chain + immutable seal |
| Killing the daemon with `kill` | strong | shield (4 kprobes) |
| Removing the module with `rmmod` | strong | shield |
| Turning off logging | strong | the sensor lives in the kernel, not userspace |
| Tampering with the aggregation allowlist | strong | allowlist hash in the chain (`CONFIG` block) |
| Process-identity spoofing in aggregation | strong | key = `/proc/<pid>/exe`, not `comm` |
| Disclosure of a copied log | strong (with `ENCRYPT=1`) | machine lock |
| Determined root (host lifecycle control) | accepted residual risk | out of scope |
| Physical access | out of scope | out of scope |

---

## Trust assumptions

- The kernel itself is assumed to be intact.
- `machine.id` and `embedded_salt` are assumed to be protected.
- The removal password is assumed to be strong and secret.
- The system clock (for wall-clock timestamps) is assumed to be reasonable.
- **Members of the `argus` group are authorized to read the entire forensic record.** Membership
  in this group is an operator decision; any member can read the decryption key, so treat it as a
  sensitive access level (like the `adm` group).

---

## Known open items

| Item | Status |
|---|---|
| ~1.9% of `openat` events without a path | deliberately retained (dropping them could hide a sensitive event) |
| Default `ENCRYPT=0` | operator decision; `1` is more secure. Reading is now possible via `argus` group membership (without sudo) |
| `ptrace` is not blocked | accepted; a fully-privileged root can legitimately release the shield |
| HTTP exporter over TLS | the peer certificate is verified by default; `TLS_VERIFY=0` is for lab use only |
| Exporter `TOKEN` in `export.conf` | `0640 root:argus` — keep it secret like a key |
