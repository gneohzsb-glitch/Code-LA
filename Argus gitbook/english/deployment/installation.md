# Installation

## Prerequisites

Satisfy [Requirements](requirements.md). Then:

---

## Step 1 — Build

```bash
cd ~/argus_v15
make clean
make
```

Seven files are produced:

| File | Role |
|---|---|
| `build/argus_id` | machine-fingerprint tool |
| `build/camouflage_engine` | decoy-tree builder |
| `kernel_module/argus_shield.ko` | shield module |
| `kernel_module/argus_sensor.bpf.o` | eBPF sensor |
| `userspace/argus-cli` | operator CLI |
| the other `userspace/` binaries | the daemon and the resolver (their names are internal) |

---

## Step 2 — Install

```bash
sudo ./install.sh
```

The installer does the following:

| Step | What |
|---|---|
| 1 | userspace binaries → `/usr/local/bin/` and **stripped** |
| 2 | eBPF sensor → `/usr/local/share/argus/` |
| 3 | GPL sources → `/usr/local/share/argus/src/` |
| 4 | configuration files → `/etc/argus/` |
| 5 | `machine.id` (machine fingerprint) |
| 6 | `embedded_salt` (immutable-sealed, permission 600) |
| 7 | decoy tree (optional, with `ARGUS_DECOYS=1`) |
| 8 | shield module → `/lib/modules/.../argus/` and load it |
| 9 | systemd units + bash completion |

---

## Step 3 — If no removal password is set

If `remove.key` is absent, you will see:

```
[!] no removal password set, so the shield was NOT loaded.
```

This is **intentional**: the shield is never armed without an exit path (otherwise it could only
be cleared by a reboot). The continuation commands are printed right there:

```bash
sudo argus-cli set-password      # enter the password 10 times
sudo modprobe argus_shield
sudo argus-cli service restart
```

---

## Configuration behavior: first install vs. upgrade

Three configuration files are **operator state** and are **not overwritten** on upgrade:

| File | Behavior |
|---|---|
| `/etc/argus/mode` | created only if absent |
| `/etc/argus/allowlist.conf` | created only if absent |
| `/etc/argus/retention.conf` | created only if absent |

The other three (`export.conf`, `anchor.conf`, `alert.conf`) are updated on every install.

The rationale: an upgrade must not silently discard a filter mode or allowlist that the operator
derived from real data.

---

## Step 4 — Verify the installation

```bash
argus-cli status
```

Expected output:

```
  SHIELD:    ◆ ACTIVE
  RELEASE:   ◆ locked
  DAEMON:    ◆ <pid>
  LICENSE:   ◆ ...
  PASSWORD:  ◆ SET
  STORAGE:   ◆ <n>% used
```

Then the integrity of the chain:

```bash
sudo argus-cli verify
# ALL <N> BLOCKS VERIFIED — NO BREAKS
```

---

## Post-install checks

```bash
# are the services active?
argus-cli service status

# is the module loaded?
lsmod | grep argus_shield

# did the sensor attach all hooks?
argus-cli logs --last 20 | grep hooks
# argus: sensor attached 21/21 hooks (0 skipped)
```

---

## Bash completion

The installer places a bash completion file at `/etc/bash_completion.d/argus-cli`, which is
sourced automatically in new shells. Typing `argus-cli ` and pressing `Tab` suggests subcommands
and valid flags:

```
$ argus-cli <Tab>
block        count        help         list         mode         retention    ...
$ argus-cli list --<Tab>
--external  --from      --last     --sensitive  --no-color
$ argus-cli retention mode <Tab>
archive  auto  off
```

If it is not active in the current session:

```bash
source /etc/bash_completion.d/argus-cli
```

Completion suggests only the flags and arguments the CLI actually accepts, so a stray `Tab`
never proposes an invalid option.

---

## Install troubleshooting

| Problem | Solution |
|---|---|
| `insmod: File exists` | the module is already loaded; run `sudo argus-cli release` first |
| shield does not load | `ls /etc/argus/remove.key`; without it, it intentionally does not load |
| `Operation not permitted` during install | the old binary carries the immutable seal; `sudo chattr -i /usr/local/bin/argus-cli` |
| daemon does not come up | `argus-cli logs --last 30` |

More in [Troubleshooting](troubleshooting.md).
