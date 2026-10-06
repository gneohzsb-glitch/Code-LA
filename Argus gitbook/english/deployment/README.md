# Deployment

This section covers everything needed to put ARGUS on a real host: from requirements, to
installation, configuration, retention, and removal — all **without any reboot**.

---

## Lifecycle

```
build (make) → install (install.sh) → configure → operate → upgrade → remove
                                          ↓
                                (all without a reboot)
```

---

## Quick summary

```bash
# 1. build
make

# 2. install
sudo ./install.sh

# 3. if no removal password is set:
sudo argus-cli set-password      # enter the password 10 times
sudo modprobe argus_shield
sudo argus-cli service restart

# 4. verify
argus-cli status
argus-cli verify
```

---

## Pages in this section

| Page | Topic |
|---|---|
| [Requirements](requirements.md) | OS, kernel, access, space |
| [Installation](installation.md) | step by step, what gets installed, install troubleshooting |
| [Configuration](configuration.md) | configuration files and parameters |
| [Retention](retention.md) | retention policy and modes |
| [Upgrade and Removal](upgrade-removal.md) | reboot-free upgrade and clean removal |
| [Troubleshooting](troubleshooting.md) | common problems and solutions |

---

> **Principle:** none of these steps — not installation, not upgrade, not removal — requires a
> reboot. Every lock has a designed release path.
