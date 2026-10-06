# Shield and password

## `argus-cli set-password`

Set the removal password (first time):

```bash
sudo argus-cli set-password
```

You are asked for the password **10 times**; it is stored in `/etc/argus/remove.key` (as a
SHA-256 hash). The file is sealed with the `immutable` attribute.

> Without this file the shield **does not load at all** — so a system can never become locked
> in a state that only a reboot could clear.

---

## `argus-cli change-password`

Change the removal password:

```bash
sudo argus-cli change-password
```

You are asked for the current password first, then the new password 10 times. After the
change, the shield re-synchronises with the new token — **no restart is needed** (the token
is re-read on every attempt).

---

## `argus-cli release`

Release the shield and stop the daemon **without** removing binaries, configuration, or logs:

```bash
sudo argus-cli release
```

This command:

1. Asks for the removal password and sends the token to the shield
2. Stops the daemon
3. Unloads the module

Use it for upgrades, a clean restart, or troubleshooting. To bring it back:

```bash
sudo argus-cli service restart
```

---

## `argus-cli remove`

Full removal of ARGUS (**logs are left untouched**):

```bash
sudo argus-cli remove
```

It releases the shield → stops the daemon → unloads the module → removes binaries and
configuration. **The log directory remains.**

---

## Manually verifying the shield

To demonstrate that the shield is working, run the following checks. Each one is expected to
**fail to affect the protected daemon**:

```bash
# PID of the protected daemon
DPID=$(sudo cat /sys/module/argus_shield/parameters/argus_daemon_pid)

# 1) Attempt to terminate the daemon
sudo kill -9 $DPID
echo "kill rc=$?"              # rc=0 — reported as "success"
ps -p $DPID -o pid,comm        # yet the process is still alive

# 2) Attempt to unload the module
sudo rmmod argus_shield
# rmmod: ERROR: Module argus_shield is in use

# 3) Forced attempt
sudo rmmod -f argus_shield
# rmmod: ERROR: could not remove module argus_shield: Resource temporarily unavailable

# 4) The correct way to release
sudo argus-cli release
```

**The key point:** the syscall **returns no error**. `kill` reports success, but no signal is
delivered. An operator sees no indication that the shield is present.

> ⚠️ A broadcast signal (`sudo kill -9 -1`) is neutralised in the same way. If the shield were
> **not** active, that command would also take down your own session.

---

## Restarting the service with the shield armed

```bash
sudo argus-cli service restart
```

This works and **requires no password**. The shield deliberately accepts signals from PID 1
(systemd); otherwise the service manager could not bring the daemon down, and every update
would require the password. A signal issued from an ordinary shell remains blocked.

> A full restart takes roughly 40 seconds.

---

## Security command summary

| Command | Password required? | Logs | Binaries | Config |
|---|---|---|---|---|
| `set-password` | — (sets it) | Kept | Kept | Kept |
| `change-password` | Yes (current) | Kept | Kept | Kept |
| `release` | Yes | Kept | **Kept** | **Kept** |
| `remove` | Yes | **Kept** | Removed | Removed |
