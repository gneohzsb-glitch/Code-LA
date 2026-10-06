# Access and the `argus` group

> **Who this page is for:** the administrator who installs ARGUS, and the SOC
> lead who wants the analysts to work without typing `sudo` all day.

ARGUS records what happens on a host. That data is evidence, so by default it is
**root-only**: the daemon runs as root, the kernel shield answers only to root,
and the stored events are readable by root alone. This page explains *why* that
is the default, and how to give your analysts exactly the access they need —
no more — by adding them to a single group.

---

## Why everything is root-only by default

Three things about ARGUS make root the natural default owner:

1. **The daemon and the shield.** The shield is a kernel module that refuses to
   be unloaded or have its protected processes killed. The daemon registers with
   it. Both are privileged by nature.
2. **The data is evidence.** Event records, the hash chain, and the license live
   under directories that are mode `0750`, owned `root:argus`. A user who is not
   in the `argus` group cannot read them at all.
3. **Least privilege.** Most ARGUS commands either change state (start/stop the
   shield, edit configuration, release the password) or expose raw evidence.
   Defaulting to root keeps the quiet, safe default.

The cost of that default is obvious: every read needs `sudo`, which is slow for
an analyst and noisy in a ticket. The `argus` group removes that cost for the
read-only work, without weakening the write side.

---

## The `argus` group

The installer creates a system group named **`argus`** and hands it read access
to the ARGUS data, while leaving the sensitive pieces root-only. The result is a
clean split:

| | A member of the `argus` group | A user outside the group |
|---|---|---|
| Read events, search, count | **yes** | no |
| Verify the chain (`argus-cli verify`) | **yes** | no |
| Export / forward to a SIEM | **yes** | no |
| Read an encrypted log (decrypt) | **yes** | no |
| Change configuration (mode, retention, forwarding) | no — needs root | no |
| Release the shield password | no — needs root | no |
| Unload the shield | no — needs root | no |

In short: **reading is delegated, changing is not.** An analyst can investigate
freely; only a small set of privileged actions still requires root.

> **Note.** The exact on-disk location of the event store is intentionally not
> published in this documentation. It is an internal detail, and keeping it
> undocumented is part of how ARGUS protects the evidence. Use the `argus-cli`
> commands below — they are the supported interface and always find the store
> for you.

---

## Adding an analyst to the group

One command, run once per user, as root:

```bash
sudo usermod -aG argus <username>
```

The user must **log out and back in** (or start a new session) for the new group
membership to take effect — group membership is decided when a session opens.

Verify it worked:

```bash
id <username>          # the output should include "argus"
```

From that session on, the read-only commands work **without `sudo`**:

```bash
argus-cli status
argus-cli list --last 50
argus-cli verify
argus-cli count
argus-cli export --format json --from 10/01/2026
```

If a command still asks for root, the session has not picked up the group yet —
log out and back in, or run `newgrp argus` to refresh the current shell.

---

## Removing access

Take a user out of the group the same way you put them in:

```bash
sudo gpasswd -d <username> argus
```

Their next session simply will not have the group, and the read commands will
refuse again. Nothing else changes; the evidence is untouched.

---

## How this helps the SOC

A well-run SOC wants three things from an endpoint agent: fast access to the
data, a clear line around what can be changed, and an audit trail. The `argus`
group delivers all three:

* **No `sudo` tax on investigations.** Triage, search, and chain verification
  are the bulk of an analyst's day. Doing them without elevation keeps the
  workflow fast and keeps the `sudo` log meaningful — when it *is* used, it is
  because something privileged really happened.
* **A clean separation of duties.** Analysts read; administrators change. A
  compromised analyst account cannot reconfigure the agent, mute a destination,
  or release the shield.
* **Evidence stays confidential by default.** Because reads are group-gated
  rather than world-readable, the event data is never exposed to unrelated
  accounts on the host.

A common pattern is to make the group part of your standard SOC image, so a new
analyst is productive the moment their account is provisioned:

```bash
# on the endpoint, once per analyst
sudo usermod -aG argus "$SOC_USER"
```

---

## A note on the CLI and `sudo`

A few commands are **always** privileged and will tell you so if you run them
without root — for example `argus-cli mode`, `argus-cli retention`,
`argus-cli forward`, `argus-cli release`, and `argus-cli remove`. That is by
design: they change the security posture of the host, not just what you can see.

If a read command reports that the log is root-only, you are almost certainly
running it as a user who is **not** in the `argus` group. Add the user (above)
or run the single command with `sudo` — the message is deliberately explicit so
the two cases never look the same.

---

## Summary

* ARGUS is root-only by default because it produces evidence and controls a
  kernel shield.
* The `argus` group grants **read-only** access to that evidence, with no
  `sudo`, for the analysts who need it.
* Add with `usermod -aG argus <user>`; remove with `gpasswd -d <user> argus`.
* Privileged actions stay privileged — reading is delegated, changing is not.
* The store's exact location is not documented on purpose; use `argus-cli`.
