# Introduction

ARGUS is a **kernel-level event-logging system** built for two goals:
**post-incident forensics** and **continuous security auditing**.

Unlike traditional logging systems — which write plain text that anyone with root
access can quietly erase — ARGUS turns every event into a block **chained by hash**.
The chain is locked together so tightly that removing or editing a single block
visibly breaks the entire sequence.

---

## In one sentence

> ARGUS guarantees that the **system's history** cannot be silently rewritten — and
> if someone tries, the log itself exposes the attempt.

---

## What this section covers

| Page | What you will learn |
|---|---|
| [The problem we solve](the-problem.md) | Why ordinary logs are not enough for forensics |
| [Key capabilities](key-capabilities.md) | What ARGUS does — and does not do |
| [Audiences and use cases](audiences.md) | Who uses ARGUS, and in which scenarios |

---

## Design principles

Three principles shape every engineering decision in ARGUS:

1. **Nothing critical runs in the kernel.** The kernel hosts only an eBPF sensor and
   a small shield module; all logic (chain, storage, alerts) lives in user space. If
   the daemon crashes, the kernel stays healthy.

2. **A reboot is never required.** Anything that can be locked must have a *designed*
   way to be unlocked — not a reboot. This principle even led to rejecting designs
   based on eBPF LSM and GRUB modification.

3. **Honesty about limits.** ARGUS makes no impossible claims. Against a determined
   root user, the goal is to **raise the cost**, not to become invisible. These limits
   are stated explicitly in the [threat model](../security/threat-model.md).
