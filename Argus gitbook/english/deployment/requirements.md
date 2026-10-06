# Requirements

## Operating system and kernel

| Item | Minimum | Recommended |
|---|---|---|
| Distribution | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |
| Kernel | 5.15 | 5.15 or 6.1 |
| Architecture | x86_64 | x86_64 |
| `CONFIG_BPF_SYSCALL` | `=y` (distribution default) | — |
| `CONFIG_DEBUG_INFO_BTF` | `=y` (distribution default) | — |

Reference test platforms: Ubuntu 22.04 with kernel `5.15.0-171-generic` and Debian 12 with
kernel 6.1.

---

## Build packages

To build from source:

```bash
sudo apt-get install build-essential clang llvm \
    linux-headers-$(uname -r) \
    libbpf-dev libelf-dev zlib1g-dev libssl-dev pkg-config
```

| Package | For |
|---|---|
| `clang` + `llvm` | compiling the eBPF program |
| `linux-headers-$(uname -r)` | building the shield module |
| `libbpf-dev` + `libelf-dev` | loading and linking eBPF |
| `zlib1g-dev` | retention compression |
| `libssl-dev` | SHA-256 and AES |

---

## Access and services

| Item | Requirement |
|---|---|
| root access | for installation and for reading encrypted logs |
| systemd | for service management (the distribution's standard version is enough) |
| systemd `Restart=always` | for daemon resilience |

---

## Disk space

The space required depends on the **event rate** and cannot be predicted in advance. However:

- Past days are compressed automatically (**~5× smaller**).
- An alert is raised at **85%** disk usage.
- The default policy is **never delete** (compress only).

A conservative estimate: a high-traffic host can produce a few hundred megabytes per day (before
compression). For precise sizing, see
[Capacity Planning](../performance/capacity-planning.md).

---

## Compatibility notes

| Item | Status |
|---|---|
| SELinux | not tested |
| AppArmor | compatible (no specific profile needed) |
| Filesystem without `immutable` support | works; the seal is not applied (best-effort) |
| Container / namespace | out of scope |
| Virtual machine | supported (the reference test platform is a VM) |

---

## No boot changes required

ARGUS requires **no** changes to GRUB, initrd, or boot parameters. All capabilities (eBPF sensor,
kernel shield, encryption) are enabled at runtime by loading the module and starting the service.
This is one of the project's hard design requirements.
