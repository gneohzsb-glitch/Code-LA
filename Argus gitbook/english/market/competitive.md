# Competitive differentiation

This page compares ARGUS against competing categories. The goal is transparency: ARGUS is
designed for one specific need and is strong at it; for other needs, a better tool may exist.

---

## Competing categories

| Category | Examples | What it does |
|---|---|---|
| Traditional system logging | syslog, rsyslog, journald | Event recording, without tamper-evidence |
| Kernel auditing | auditd | Syscall recording, without a hash chain |
| Log management / SIEM | ELK, Splunk, Wazuh | Collection and correlation |
| File integrity | AIDE, Tripwire | Hashing of system files |
| Forensic systems | Incident-response tooling | Post-incident analysis |

---

## Comparison matrix

| Feature | ARGUS | syslog/auditd | SIEM | AIDE/Tripwire |
|---|---|---|---|---|
| Kernel-level recording | ✅ eBPF | ✅ | ❌ | ❌ |
| Hash chain (tamper-evident) | ✅ | ❌ | ❌ | ❌ |
| Self-protection (shield) | ✅ | ❌ | ❌ | ❌ |
| Independently verifiable certificate | ✅ | ❌ | Partial | ✅ (files) |
| Event recording (not just files) | ✅ | ✅ | ✅ | ❌ |
| No reboot | ✅ | ✅ | ✅ | ✅ |
| Automatic compression | ✅ | ✅ | ✅ | — |
| SIEM integration | ✅ | ✅ | ✅ | ✅ |
| Encryption machine lock | ✅ | ❌ | Sometimes | ❌ |

---

## Four key differentiators

### 1. Provable tamper-evidence

Competitors **record** events. ARGUS records events **and proves they are unaltered** — with a
SHA-256 chain. The difference is that in a courtroom or a compliance review, "trust us"
becomes "verify it yourself."

### 2. Self-protection

If the recording system can be turned off, the rest of its capabilities are meaningless. With
its kernel shield (4 kprobes), ARGUS prevents the daemon from being killed and the module from
being removed — silently, without the attacker realizing.

### 3. "Never reboot" as a principle

Many kernel-level security solutions require a GRUB change or eBPF LSM, which in turn requires
a reboot. ARGUS deliberately rejected these so it can be deployed in production.

### 4. Lossless retention

Competitors typically delete logs after a while. By default ARGUS **never deletes** — it only
compresses and seals. Deletion is an explicit operator decision, not default behavior.

---

## Where ARGUS is **not the competitor**

Honesty is part of the sale:

| Need | Better tool |
|---|---|
| Collecting logs from thousands of hosts | SIEM (Splunk, ELK) |
| Correlating multi-source events | SIEM |
| Preventing intrusions (firewall/IDS) | Prevention tools |
| Checking the integrity of static files | AIDE/Tripwire |
| Post-incident memory/disk analysis | Dedicated IR tools |

ARGUS produces a **trustworthy event source**; a SIEM collects and correlates it. The two are
complementary, not competitors.

---

## Positioning

```
                    High tamper-evidence
                            ▲
                            │
             ARGUS ●        │
                            │
   Event level ◄────────────┼─────────────► File level
                            │
                            │        ● AIDE
        ● auditd/syslog     │
                            │
                    Low tamper-evidence
```

ARGUS sits in an area that is less covered: **kernel-level event recording + provable
tamper-evidence + no reboot**. That combination is the core differentiator.
