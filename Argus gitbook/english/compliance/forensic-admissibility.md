# Forensic admissibility

A record can be presented as **evidence** when its chain of custody and its integrity can be
demonstrated. ARGUS provides three technical instruments for this.

---

## Instrument 1 — Hash chain

Each block is chained to the previous one. Any change (editing, deletion, reordering) is
detectable.

```bash
sudo argus-cli verify
# ALL 368137 BLOCKS VERIFIED — NO BREAKS
```

If the chain is broken, the number of gaps is reported:

```bash
# 18 BROKEN LINKS / 0 BAD HASHES / 126369 BLOCKS
```

This lets you state either "this record has no gaps over interval X" or "it is broken at
block N" — both are admissible propositions.

---

## Instrument 2 — Integrity certificate (manifest)

```bash
sudo argus-cli integrity --from 09/01/2026 --to 09/30/2026
```

The output is a single SHA-256 hash for the entire interval. The key advantage: **an auditor
can verify it independently**, even without ARGUS installed:

```bash
sha256sum <ARGUS_STORE>/2026/September/30.log.gz
# must match the manifest hash
```

This "independent verification" matters for legal admissibility: the correctness of the
record does not hinge on trusting the software vendor.

---

## Instrument 3 — External anchor (proof of time)

The head of the chain is periodically sent to an external destination. If someone later
rewrites the entire chain, the anchored hash no longer matches the chain head.

| Use | Description |
|---|---|
| Proving the record has existed since date X | Timestamped anchor |
| Detecting a full-chain rewrite | Inconsistency with the anchor |

---

## Chain of custody

ARGUS provides the following path for an admissible document:

```
1. The event is recorded in the kernel (cannot be bypassed from userspace)
2. It becomes a chained block (hash over the content)
3. It is written to disk
4. Past days are compressed and sealed immutable
5. verify / integrity proves correctness
6. The manifest is confirmed independently with sha256sum
```

Every step is auditable.

---

## Limitations you should know

| Limitation | Effect |
|---|---|
| Shield scope | The shield defends against tampering by ordinary and non-privileged processes. It is not a boundary against a fully privileged administrator; its purpose is to raise the cost of tampering and block the obvious paths. |
| System clock | Wall-clock timestamps depend on the host clock; the external anchor helps |
| Before ARGUS is installed | Earlier events were never recorded |
| ~1.9% of `openat` events lack a path | They are still recorded, but their path is unknown |

Honesty about these limits is itself part of the forensic value.

---

## Recommendations for presenting evidence

1. Run `integrity` for the interval in question and save the manifest.
2. Run `verify` and record the `NO BREAKS` output.
3. Attach the hashes of the relevant external anchors.
4. Confirm the file paths independently with `sha256sum`.
5. Interpret dates in Iran time (UTC+3:30).

Technical details of the formula and the block types are in [Hash chain](../architecture/blockchain.md).
