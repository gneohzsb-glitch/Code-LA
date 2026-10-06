# Tests

ARGUS ships an automated test suite that runs with a single command:

```bash
cd ~/argus_v15/tests
sudo ./run_all.sh
```

`run_all.sh` first takes any running installation down through the password-protected
release path (the shield suites need exclusive control of the kernel module), runs every
suite in the correct order, and restores the password, the module, and the daemon at the
end. **No reboot is involved, ever.**

> The removal password can be overridden with `ARGUS_PW=<password>`.

The suite prints one `PASS`/`FAIL` line per assertion and ends with either
`ALL SUITES PASSED` or `FAILURES PRESENT`. The exit status is non-zero if any suite
failed, so it drops straight into CI.

---

## What the suite covers

There are **44 suites** and **631 assertions**. They are grouped below by the part
of the product they exercise.

### Shield and kernel module

| Suite | What it proves |
|---|---|
| `test_shield.sh` | Killing the daemon, `rmmod`, `rmmod -f`, and the password release path |
| `test_shield_reduction.sh` | The shield installs exactly 4 hooks — no more |
| `test_cli_shield.sh` | CLI ↔ shield interaction and status reporting |

### Integrity and the hash chain

| Suite | What it proves |
|---|---|
| `test_verify.sh` | Editing, re-linking, and deletion are all detected |
| `test_aggregation.sh` | Aggregation, crash recovery, and mode change |
| `test_timestamps.sh` | Block timestamp accuracy |
| `test_anchor_hmac.sh` | The daily anchor HMAC, and that key/interval changes apply without a restart |

### Sensor behaviour

| Suite | What it proves |
|---|---|
| `test_no_flood.sh` | Anti-feedback: ARGUS never captures its own output |
| `test_noise_filter.sh` | Noise filtering and the sensitive-path flag |
| `test_list_filters.sh` | `list --external` with no cap, and `sensitive` |
| `test_encryption.sh` | Log encryption: transparent decryption and compatibility |
| `test_gcm.sh` | The AES-GCM codec rejects any edit to ciphertext, tag, or length |
| `test_retention.sh` | Lossless compression, the immutable seal, and the retention modes |

### Access, permissions, and configuration

| Suite | What it proves |
|---|---|
| `test_permissions.sh` | Every file the product creates has the intended mode and owner |
| `test_password_kdf.sh` | The removal password is stored as a slow KDF hash, not plaintext |
| `test_group_access.sh` | Members of the `argus` group can read evidence without `sudo` |
| `test_sudo_hint.sh` | The CLI prints a helpful `sudo`/group hint instead of a bare denial |
| `test_machine_id_stable.sh` | `machine.id` is stable across rebuilds |
| `test_release_no_ssh_kill.sh` | The release path never kills an operator's SSH session |
| `test_config_reread.sh` | Configuration changes are picked up live |
| `test_upgrade_preserves_config.sh` | An upgrade keeps the operator's configuration |
| `test_version.sh` | A single version string is reported everywhere |

### Alerts and evidence

| Suite | What it proves |
|---|---|
| `test_alerts_chain.sh` | Alerts are recorded in the chain and survive a restart |
| `test_evidence_native.sh` | The native evidence format is self-describing and intact |
| `test_no_decoy_leak.sh` | Decoy/cover material never leaks real content |
| `test_exporters.sh` | The SIEM exporters emit well-formed records |
| `test_remove_preserves_keys.sh` | `remove` keeps the keys (destructive; runs last) |

### Licensing and the builder

| Suite | What it proves |
|---|---|
| `test_argus_id.sh` | `argus-id` derives the machine id correctly |
| `test_license_valid.sh` | A valid license verifies |
| `test_license_expired.sh` | An expired license is refused |
| `test_license_tampered.sh` | A modified payload or signature is refused |
| `test_license_wrong_machine.sh` | A license bound to another machine is refused |
| `test_license_replay.sh` | A replayed nonce is refused |
| `test_license_key_rotation.sh` | Key rotation is honoured |
| `test_license_grace_period.sh` | The grace window behaves as designed |
| `test_license_runtime_verify.sh` | The running daemon re-verifies the license |
| `test_license_antitamper.sh` | A debugger attached to the verifier is detected and refused |
| `test_builder.sh` | `argus-builder` produces a sealed, installable package |
| `test_license_integration.sh` | The packaged `install.sh --check-only` accepts a good package |

### Compatibility and structure

| Suite | What it proves |
|---|---|
| `test_backward_compatibility.sh` | Older on-disk formats still verify |
| `test_source_separation.sh` | GPL and proprietary sources stay separated |
| `test_capillary.sh` | Argument edges, hostile config, idle CPU, and prompt shutdown |

### Information hygiene (v15.5)

| Suite | What it proves |
|---|---|
| `test_no_name_leak.sh` | No operator output names an internal unit, binary or path |
| `test_codes.sh` | Failures carry an `ARG-nnn` code, and every code is documented |

---

## Notes on selected suites

### `test_verify.sh`

It builds a synthetic chain for a past date (so it never touches the live log) and checks
four cases:

| Case | Expected result |
|---|---|
| Intact chain | `ALL N BLOCKS VERIFIED — NO BREAKS` |
| Edited content | `BAD HASHES` |
| Re-linked block | `BROKEN LINKS` |
| Block deleted from the middle | `BROKEN LINKS` |

### `test_retention.sh`

- The `retention.conf` file is installed and defaults to `off`
- `argus-cli retention` reports the policy
- Invalid values (unknown mode, threshold 0 or 150) are rejected
- A written value persists in the configuration and is restored
- Safety rules exist in the daemon source: `off` returns before any deletion, the 7-day
  window is preserved, and the action is recorded in the chain before it happens

This test never sets `auto`/`archive` mode, so no real data is at risk.

### `test_list_filters.sh`

- The census header of `list --external` is present
- The count for each IP matches the reported total
- The total lies inside an independent window taken from the raw log on both sides of the
  CLI read (the log is live, so an exact-equality check would be flaky by construction)
- No fixed cap remains in the `cli.c` source
- `sensitive` shows only flagged blocks
- `list --sensitive` and `sensitive` agree

### `test_capillary.sh`

A deliberately hostile pass over the edges:

- The CLI never crashes on empty, over-long, or malformed arguments
- A hostile configuration (`INTERVAL=0`) does not wedge the daemon
- Idle CPU stays flat over a measured window (no busy loop)
- `systemctl stop` returns promptly (no shutdown hang)

### `test_license_antitamper.sh`

The verifier reads its own `TracerPid`; if a debugger or tracer is attached it raises the
`TAMPERED` state and the license is treated as fatal. The suite confirms a clean run stays
`VALID` and that `strace`/`gdb` runs are refused. This is defense-in-depth, not a
guarantee — see the licensing appendix for the honest limits.

---

## Running suites individually

```bash
sudo ./test_shield.sh
sudo ./test_shield_reduction.sh
./test_cli_shield.sh
./test_verify.sh
./test_aggregation.sh          # daemon stopped
./test_no_flood.sh 20
./test_timestamps.sh
sudo ./test_retention.sh
sudo ./test_noise_filter.sh
sudo ./test_list_filters.sh
sudo ./test_encryption.sh
sudo ./test_gcm.sh
sudo ./test_capillary.sh
```

> Some tests require a running daemon and some a stopped one; `run_all.sh` handles the
> ordering itself. Running a suite by hand out of order may fail its preconditions.

---

## Building the test harness

Some tests build a separate C harness that links the real source:

```bash
gcc -O2 -Wall -I userspace/daemon -I common \
    -o /tmp/argus_ret_test tests/ret_test.c \
    userspace/daemon/retention.c \
    userspace/daemon/alert_engine.c \
    userspace/daemon/storage.c \
    userspace/daemon/blockchain.c \
    userspace/daemon/hasher.c \
    userspace/daemon/ipcache.c \
    userspace/daemon/encryptor.c -lz -lpthread -lcrypto
```

The retention harness runs the real `retention_compress_past_days()` function on a
synthetic day and inspects the result — without restarting the daemon.

See [Test report](test-report.md) for the results of the most recent full campaign.
