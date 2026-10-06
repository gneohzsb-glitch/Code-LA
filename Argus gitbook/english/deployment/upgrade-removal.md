# Upgrade and Removal

None of these steps requires a reboot. The shield deliberately does not reject signals from PID 1
(systemd), so the service manager can always bring the daemon down.

---

## Routine upgrade (no shield change)

If only userspace code (daemon/CLI) changed:

```bash
cd ~/argus_v15
make
sudo ./install.sh
sudo argus-cli service restart
```

`systemctl restart` **does not ask for a password** — this is the PID 1 exception.

> A full restart takes about 40 seconds (the baseline thread is finishing its work).

---

## Full upgrade (with a shield change)

If the kernel module (shield) code changed, the module must be removed and reloaded. Use the
passworded path:

```bash
sudo argus-cli release                   # release the shield + stop the daemon (data remains)
cd ~/argus_v15 && make && sudo ./install.sh
sudo argus-cli service restart
```

`release` prompts for the removal password, releases the shield, stops the daemon, and removes
the module. After `install.sh`, the shield is re-armed automatically.

---

## Changing the removal password

```bash
sudo argus-cli change-password
```

After the change, the shield synchronizes itself with the new token — no restart is needed (the
token is re-read on every attempt).

---

## Full removal (logs remain)

```bash
sudo argus-cli remove
```

This command:

1. releases the shield
2. stops the daemon
3. removes the module
4. deletes the binaries and settings

**The log directory is left untouched** — removing ARGUS never deletes the forensic record.

---

## The difference between `release` and `remove`

| | `release` | `remove` |
|---|---|---|
| Releases the shield | yes | yes |
| Stops the daemon | yes | yes |
| Removes the module | yes | yes |
| Deletes the binaries | **no** | yes |
| Deletes the settings | **no** | yes |
| Preserves the logs | yes | yes |
| Use | upgrade/restart | final removal |

---

## Notes

- `release` and `remove` both require the removal password.
- If you run `systemctl start` again after `release`, the shield is re-armed automatically.
- To remove archives manually, release the seal first: `sudo chattr -i <path>/DD.log.gz`
- **None of these operations requires a reboot.**
