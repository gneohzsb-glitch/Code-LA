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

---

## GitHub Pages — the landing page

`index.html` in this repository root is the ARGUS landing page (single file, no
build step). It ships alongside the book; the book itself lives in `english/`
and `persian/`.

To publish it:

1. Push this repository to GitHub with `index.html` at the root.
2. Open the repository: **Settings → Pages**.
3. Under **Build and deployment**, set **Source = Deploy from a branch**.
4. Set **Branch = `main`** (or `master`) and **Folder = `/ (root)`**. Save.
5. Wait about a minute. The site is served at
   `https://<user>.github.io/<repo>/`.

An empty `.nojekyll` file is included at the root so GitHub Pages serves every
file verbatim instead of running it through Jekyll (Jekyll would silently drop
any file or folder whose name starts with an underscore).

The page references, relative to the repository root:

```
assets/
├── argus-hero.webp
├── argus-portrait.webp
├── argus-face.webp
├── favicon.ico
└── fonts/
    ├── Vazirmatn-Thin.ttf … Vazirmatn-Black.ttf   (9 weights, Persian)
    └── SpaceGrotesk-Light.ttf … SpaceGrotesk-Bold.ttf (5 weights, English)
```

Upload the images into `assets/` and the 14 font files into `assets/fonts/`.
Until they are present the page still renders — the image slots simply stay
empty. All three character images ship as **WebP** (25–35% smaller than PNG at
the same visual quality). The only external request the page makes is Google
Fonts for **JetBrains Mono** (the code/CLI face); everything else is local.
