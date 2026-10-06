# Troubleshooting

## Service status

```bash
argus-cli service status
argus-cli logs --last 30
```

Healthy journal messages:

```
argus: sensor attached 21/21 hooks (0 skipped)
argus: shield protecting pid <N>
argus: daemon active (pid=<N>)
```

---

## Shield kernel messages

```bash
sudo dmesg | grep -i "argus shield"
```

---

## Log path

```bash
sudo ls -l <ARGUS_STORE>/$(date -u -d '+12600 seconds' '+%Y/%B')/
```

Past days have been converted to `.log.gz`. To read manually:

```bash
sudo zcat <ARGUS_STORE>/2026/September/30.log.gz | tail -5
```

> The date in the directory name is computed in **Iran time (UTC+3:30)**, not UTC.

---

## Common problems

### The shield does not load

```bash
ls /etc/argus/remove.key          # must exist
sudo dmesg | tail -5              # reports the reason for rejection
```

The shield intentionally does not load **without** `remove.key`. To create it:

```bash
sudo argus-cli set-password
sudo modprobe argus_shield
```

### `insmod: File exists`

The module is already loaded. Release it first:

```bash
sudo argus-cli release
```

### Install: `Operation not permitted`

The old binary carries the immutable seal:

```bash
sudo chattr -i /usr/local/bin/argus-cli
sudo ./install.sh
```

### The daemon does not come up or is stuck

```bash
argus-cli service status
sudo dmesg | grep -i 'argus shield' | tail -5
```

If `argus-cli service status` shows an extra daemon process (an orphan):

```bash
sudo argus-cli service restart
```

### `verify` rejects encrypted blocks

If `ENCRYPT=1`, run `verify` with `sudo` (the salt is readable only by root):

```bash
sudo argus-cli verify
```

Run without root, and the number of unread blocks is reported:

```
◆ 237 ENCRYPTED BLOCK(S) SKIPPED
```

### Disk full / Storage alert

```bash
argus-cli status | grep STORAGE
sudo grep -A2 'Storage' /var/log/argus_alerts.log | tail -6
sudo du -sh <ARGUS_STORE>/
```

The default policy is: **nothing is deleted, only compressed.** If space is still short:

```bash
argus-cli retention                      # current policy
sudo argus-cli retention mode archive    # move to external media
sudo argus-cli retention archive-path /mnt/backup
```

(Release sealed archives with `chattr -i` before moving them manually.)

### The shield is armed but I cannot restart the service

It should work. `systemctl restart` comes from PID 1 and is not rejected. If it is locked:

```bash
sudo argus-cli service restart
argus-cli logs --last 20
```

### `kill -9` on the daemon returns no error but the daemon does not die

This is **correct behavior**. The shield neutralizes the signal without returning an error —
intentionally, so an attacker sees no sign of it.

---

## Live chain statistics

```bash
sudo tail -f <ARGUS_STORE>/2026/October/01.log | cut -c1-160
```

---

## If everything fails

The way back is always open:

```bash
sudo argus-cli release             # release the shield + stop the daemon
sudo argus-cli service restart
```

There is no scenario that can only be resolved by a reboot.
