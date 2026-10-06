# Glossary

| Term | Definition |
|---|---|
| **eBPF** | Extended Berkeley Packet Filter; a technology for running safe programs in the kernel, proven safe by the verifier before execution |
| **Sensor** | The ARGUS eBPF program; 21 kprobes that place raw events in a ring buffer |
| **Shield** | The ARGUS kernel module; 4 kprobes that protect the daemon |
| **Daemon** | The ARGUS service that turns raw events into chained blocks (shown as the *capture engine* in `argus-cli service status`) |
| **Block** | A single JSON line that chains one event to the previous block by hash |
| **Hash chain** | The sequence of blocks, each carrying the hash of the previous block |
| **Genesis** | The first block; its `prev_hash` is 64 zeros |
| **kprobe** | A kernel hook point that intercepts the execution of a function |
| **Ring buffer** | A kernel circular buffer in which the sensor places events |
| **CO-RE** | Compile Once – Run Everywhere; reading kernel structures without version dependence |
| **Verifier** | The part of the kernel that proves an eBPF program's safety before execution |
| **immutable** | A filesystem flag (`FS_IMMUTABLE_FL`) that blocks rewriting and deletion |
| **Archive** | The `.log.gz` file for a past day, compressed and sealed |
| **Aggregation** | Folding repeated reads into a single counter summary block, without deletion |
| **allowlist** | The set of rules that determine which (executable, path prefix) may be aggregated |
| **SENSITIVE flag** | A marker for access to a sensitive path; stored in the `f` field (unhashed) |
| **Noise filter** | Discarding routine reads (`/proc`, libraries) |
| **Feedback ring buffer** | A loop that would occur if the daemon recorded its own events; closed with `daemon_pid_map` |
| **PBKDF2** | A password-based key-derivation function; ARGUS uses PBKDF2-HMAC-SHA256 with 500,000 iterations |
| **AES-256-CBC** | A symmetric encryption algorithm for content confidentiality |
| **Machine lock** | Binding the encryption key to `machine.id`, so a copied log is unreadable elsewhere |
| **External anchor** | Sending the chain head to an external destination as proof of time |
| **Manifest** | A file containing the SHA-256 hash of a time interval (integrity certificate) |
| **Chain of custody** | The proven path of how evidence was kept |
| **Obfuscation** | Hiding the path/name of the log; it raises cost rather than guaranteeing |
| **Self-capture** | An event produced by the daemon itself; it must be zero |
| **Filter mode** | `performance` / `balanced` / `forensic`; the size of the aggregation window |
| **MAINTENANCE block** | A block that records a maintenance action before it happens |
