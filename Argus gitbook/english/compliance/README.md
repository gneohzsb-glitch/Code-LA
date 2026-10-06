# Compliance

This section explains how ARGUS supports **forensic admissibility** and **data protection**.
The goal is to provide technical evidence for legal and audit teams — not to claim legal
compliance.

> **Disclaimer:** ARGUS is a technical tool. Whether a given record is accepted in a court of
> law or a compliance review depends on the jurisdiction, your organization's procedures, and
> legal counsel. This page describes only the relevant technical capabilities.

---

## Two axes

| Axis | Question | Page |
|---|---|---|
| Forensic admissibility | Can this record be presented as evidence? | [Forensic admissibility](forensic-admissibility.md) |
| Data protection | Is it compatible with privacy and data regulations? | [Data protection](data-protection.md) |

---

## Why the hash chain matters for compliance

Traditional logging systems can tell you "this event was recorded." ARGUS can tell you
"this event was recorded, **and it has remained untouched from the moment of recording until
now**" — and prove that claim with mathematics (a SHA-256 chain), not with trust in the
operator.

| Feature | Compliance value |
|---|---|
| Hash chain | Proof that the record is unaltered |
| SHA-256 manifest | Independently verifiable evidence (`sha256sum`) |
| External anchor | Proof of time (the record has existed since date X) |
| Encrypted release path | Proof of access control over the system itself |
