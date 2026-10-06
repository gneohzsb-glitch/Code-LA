# ARGUS — Documentation

**Kernel-level event recording for forensics and security auditing.**

This documentation is published in two complete languages. Pick one and read it
top to bottom, or jump straight to the page you need — every page stands on its
own.

| Language | Start here | Language | Start here |
|---|---|---|---|
| English | [docs/english/README.md](english/README.md) | فارسی | [docs/persian/README.md](persian/README.md) |

---

## What ARGUS is, in one paragraph

ARGUS watches the kernel for the events that matter — process creation, file
opens on sensitive paths, privilege changes, memory access to other processes,
network activity — and turns each one into a block whose SHA-256 hash is chained
to the block before it. The result is a **tamper-evident log**: change, reorder,
or delete a single byte and `argus-cli verify` points at the break. After an
incident, or for an audit, you can answer one question with confidence:

> **What happened, by which user or process, when — and has anyone touched the log?**

---

## The two languages

* **[English](english/README.md)** — a complete English edition, written to be
  read straight through like a short book, and equally useful as a reference.
* **[فارسی](persian/README.md)** — نسخهٔ کامل فارسی، با همان ساختار و همان
  محتوا.

Both editions describe the same product. When they ever disagree, the product
itself is the source of truth; open an issue and we will reconcile them.

---

## Where to go next

* New to ARGUS? → [Introduction](english/introduction/README.md)
* Deploying it? → [Deployment](english/deployment/installation.md)
* Running it day to day? → [CLI reference](english/cli-reference/README.md)
* Reviewing it for security or compliance? → [Security](english/security/README.md)
  and [Compliance](english/compliance/README.md)

---

> **Licensing:** the kernel components (shield and sensor) ship under GPL-2.0
> with full source; the daemon and CLI are proprietary and ship as binaries.
> See [Licensing](english/appendix/license.md) for the details.

---

> **`internal/` is not part of this book.** It holds support-only material —
> the `ARG-nnn` codebook and the real service names — and is never published
> or shipped. Everything a customer needs is in `english/` and `persian/`.
