# Audiences and Use Cases

## Who uses ARGUS?

| Role | Why ARGUS |
|---|---|
| **Incident response (IR) team** | Reconstruct an accurate intrusion timeline; a witness that has not been touched |
| **Compliance and audit team** | Prove the system history has not been rewritten; integrity certificates |
| **Sysadmin / DevOps** | Monitor sensitive changes (permission files, modules) with low overhead |
| **Legal / forensics team** | Evidence that can be independently verified with `sha256sum` |
| **Market research (your team)** | Understand the technical differentiators for competitive comparison and product messaging |

---

## Sample scenarios

### 1. After an intrusion

An attacker has entered, installed a backdoor, and tried to erase the logs.
With ARGUS:

```bash
argus-cli verify              # Is the chain intact?
argus-cli anomalies           # Today's alerts
argus-cli list --sensitive    # Access to sensitive files
argus-cli integrity --from 09/01/2026 --to 09/30/2026   # Certificate for the range
```

If the attacker deleted a line, `verify` shows a `BROKEN LINKS` count; if they changed
content, `BAD HASHES`.

### 2. Proving compliance

In an audit meeting, the question comes up: "How do we know this history was not
tampered with?"

```bash
sudo argus-cli integrity --from 09/01/2026 --to 09/30/2026
# Produces a SHA-256 manifest the auditor can verify independently
```

### 3. Monitoring sensitive changes

Without drowning in log volume:

```bash
argus-cli sensitive           # Only sensitive-path events
argus-cli list --external     # Tally of blocks with an external IP
```

### 4. Streaming to a SIEM

```bash
# Sensitive-path events go to the SIEM with the ARGUS_SENSITIVE tag
argus-cli export --format json
argus-cli export --format csv   # flags column for routing
```

---

## Prerequisite knowledge

- To **run** it: basic familiarity with Linux and the command line is enough.
- To **deploy** it: root access and familiarity with systemd.
- For **deep technical understanding**: the [Architecture](../architecture/README.md)
  and [Security](../security/README.md) sections are recommended.

---

## Suggested reading paths

| If you are… | Start here |
|---|---|
| A product / marketing manager | [Key capabilities](key-capabilities.md) → [Datasheet](../market/datasheet.md) |
| A deployment engineer | [Requirements](../deployment/requirements.md) → [Installation](../deployment/installation.md) |
| A security analyst | [Architecture](../architecture/README.md) → [Threat model](../security/threat-model.md) |
| A day-to-day operator | [CLI reference](../cli-reference/README.md) |
