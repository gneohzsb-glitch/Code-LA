# License

ARGUS is a **dual-licensed** product: the kernel portion is under GPL-2.0 (with full source),
and the userspace portion is proprietary (binary only).

---

## License split

| Component | License | Is source published? |
|---|---|---|
| Shield module (`kernel_module/argus_shield.c`) | GPL-2.0 | Yes |
| eBPF sensor (`kernel_module/argus_sensor.bpf.c`) | GPL-2.0 | Yes |
| Daemon (`userspace/daemon/*.c`) | Proprietary | No (binary only) |
| CLI (`userspace/cli/cli.c`) | Proprietary | No |
| Helper (`userspace/helper/*.c`) | Proprietary | No |
| Build tools (`build/*.c`) | Proprietary | No |

---

## Why this split?

Any file that uses **GPL-only kernel symbols** or **eBPF helpers** must be GPL. The rest may
remain proprietary.

- **Shield** uses kernel APIs → GPL-2.0
- **eBPF sensor** uses BPF helpers → GPL-2.0
- **Daemon and CLI** run only in userspace → proprietary

Both kernel files carry the `SPDX-License-Identifier: GPL-2.0` marker and
`MODULE_LICENSE("GPL")`. The userspace files carry a "Proprietary and confidential" header.

---

## Customer delivery

| Component | What they receive |
|---|---|
| Shield | Binary (`.ko`) + full source |
| Sensor | Binary (`.bpf.o`) + full source |
| Daemon, CLI, helper | Binary only |

The installer places the two GPL sources and the license texts under
`/usr/local/share/argus/src/`, and **no userspace source** is there. This separation satisfies
the kernel source-disclosure obligation without disclosing the userspace intellectual property.

---

## Full text

- Kernel license: `LICENSE/GPL-2.0.txt`
- Proprietary license: `LICENSE/PROPRIETARY.txt`

---

## Proprietary license summary

> Copyright (C) 2026 Gneo HZSB. All rights reserved.
>
> The userspace components (daemon, IP helper, CLI) are proprietary and confidential. Their
> source is not published; the recipient receives binaries only.
>
> Redistribution of the binary components, in whole or in part, without prior written
> permission from Gneo HZSB is prohibited.
>
> The software is provided "as is," without any warranty.

---

## License-separation test

The `test_source_separation.sh` suite (21 tests) automatically checks this separation: every
kernel source is GPL, every userspace source is proprietary, no userspace source is installed,
and the installer ships both GPL sources. Details in [Tests](testing.md).
