# Test report

This report records the results of the full, capillary test campaign over ARGUS v15.4:
what was tested, in what environment, which bugs were found and fixed, and what the final
result was.

---

## Executive summary

| Item | Value |
|---|---|
| Version under test | ARGUS v15.4 (with the v15.5 information-hygiene work) |
| Date of run | 2026-10-06 |
| Environment | Ubuntu 22.04 VM, kernel 5.15.0-171 |
| Suites | **44** |
| Assertions | **631** |
| Result | **44 of 44 suites green, 0 failures** |
| Reboots | **none** (at any step) |

> No bug was left open at the end of the campaign. Every failure seen along the way was
> root-caused, fixed, and the affected suite re-run until green.

---

## Environment

- **OS:** Ubuntu 22.04 (server), inside VirtualBox.
- **Kernel:** `5.15.0-171-generic` — the kernel the shield module and the eBPF sensor load
  on.
- **Privileges:** every suite that needs it runs with `sudo`.
- **Build:** from the source tree with `make`, then installed with `install.sh`.
- **License:** a one-year package built with `argus-builder` for this machine's id was
  installed and active; `argus-cli license` reported `VALID` and the daemon journal showed
  `LICENSE state=VALID`.

---

## Method

Every suite runs with a single command:

```bash
sudo ./tests/run_all.sh
```

`run_all.sh`:

1. Takes any running installation down through the password-protected release path (the
   shield suites need exclusive control of the module).
2. Runs the suites in the correct order — some need a live daemon, some a stopped one.
3. Restores the removal password, the shield module, and the daemon.
4. Ends with either `ALL SUITES PASSED` or `FAILURES PRESENT`, and returns the matching
   exit code.

**No reboot is required at any step** — every lock-down has a designed release path.

---

## Results by suite

| # | Suite | Passing assertions |
|---|---|---|
| 1 | Shield (`test_shield.sh`) | 18 |
| 2 | Shield reduction (`test_shield_reduction.sh`) | 33 |
| 3 | CLI ↔ shield (`test_cli_shield.sh`) | 16 |
| 4 | Chain verify (`test_verify.sh`) | 11 |
| 5 | Aggregation (`test_aggregation.sh`) | 26 |
| 6 | Anti-feedback (`test_no_flood.sh`) | 5 |
| 7 | Block timestamp (`test_timestamps.sh`) | 4 |
| 8 | Retention (`test_retention.sh`) | 35 |
| 9 | Noise filter (`test_noise_filter.sh`) | 12 |
| 10 | List filters (`test_list_filters.sh`) | 8 |
| 11 | Encryption (`test_encryption.sh`) | 14 |
| 12 | Backward compatibility (`test_backward_compatibility.sh`) | 13 |
| 13 | Source separation (`test_source_separation.sh`) | 21 |
| 14 | Permissions (`test_permissions.sh`) | 13 |
| 15 | Password KDF (`test_password_kdf.sh`) | 9 |
| 16 | Machine id (`test_machine_id_stable.sh`) | 9 |
| 17 | Release without SSH kill (`test_release_no_ssh_kill.sh`) | 15 |
| 18 | Alerts chain (`test_alerts_chain.sh`) | 14 |
| 19 | Single version (`test_version.sh`) | 10 |
| 20 | Config re-read (`test_config_reread.sh`) | 13 |
| 21 | Upgrade keeps config (`test_upgrade_preserves_config.sh`) | 10 |
| 22 | GCM codec (`test_gcm.sh`) | 10 |
| 23 | Anchor HMAC (`test_anchor_hmac.sh`) | 15 |
| 24 | Sudo hint (`test_sudo_hint.sh`) | 15 |
| 25 | Group access (`test_group_access.sh`) | 9 |
| 26 | No decoy leak (`test_no_decoy_leak.sh`) | 5 |
| 27 | Native evidence (`test_evidence_native.sh`) | 10 |
| 28 | SIEM exporters (`test_exporters.sh`) | 14 |
| 29 | argus-id (`test_argus_id.sh`) | 33 |
| 30 | License valid (`test_license_valid.sh`) | 13 |
| 31 | License expired (`test_license_expired.sh`) | 9 |
| 32 | License tampered (`test_license_tampered.sh`) | 8 |
| 33 | License wrong-machine (`test_license_wrong_machine.sh`) | 7 |
| 34 | License replay (`test_license_replay.sh`) | 7 |
| 35 | License key rotation (`test_license_key_rotation.sh`) | 10 |
| 36 | License grace period (`test_license_grace_period.sh`) | 13 |
| 37 | License runtime verify (`test_license_runtime_verify.sh`) | 7 |
| 38 | License anti-tamper (`test_license_antitamper.sh`) | 6 |
| 39 | Builder (`test_builder.sh`) | 44 |
| 40 | License integration (`test_license_integration.sh`) | 14 |
| 41 | Capillary (`test_capillary.sh`) | 16 |
| 42 | Remove preserves keys (`test_remove_preserves_keys.sh`) | 28 |
| 43 | No name leak (`test_no_name_leak.sh`) | 21 |
| 44 | Diagnostic codes (`test_codes.sh`) | 8 |
| | **Total** | **631** |

---

## Bugs found and fixed

The campaign did not only confirm "green"; it surfaced several real bugs and a couple of
test flakinesses, all of which were fixed.

### 1) Unbounded export-thread loop — CPU spin and a stuck shutdown

**Symptom:** after a while the daemon reached 100% CPU and `systemctl stop` never returned
(main was stuck in `pthread_join`).

**Root cause:** the inner read loop in the export thread had no bound; on a large log it
kept refilling and never reached a stop point.

**Fix:** a per-pass cap `EXPORT_MAX_PER_PASS 2000`, a re-check of `running` in the inner
loop, `snprintf` for the log path, and a conditional sleep (`export_sleep`) that clamps a
value below 1 up to 1.

### 2) Live-log race in `test_list_filters`

**Symptom:** "CLI count does not match raw log count" — sometimes 50109 vs 50105.

**Root cause:** the log is live and the daemon appends blocks while the test runs, so two
reads never see the same byte count; an exact-equality check is flaky by construction — not
a miscount.

**Fix:** take the raw count on both sides of the CLI read and require the CLI total to fall
inside the `[R1, R2]` window.

### 3) False negative in the GCM test

**Symptom:** "the codec accepted a tampered tag".

**Root cause:** flipping the last base64 character only changed padding-discarded bits, so
the tag was not actually altered.

**Fix:** step back six characters so a meaningful bit changes.

### 4) Fail-open when the license file is missing

**Symptom:** with `LICENSE_ENFORCE=1`, a missing license file fell through to "community"
mode (open).

**Fix:** behaviour changed to **fail-closed**; a missing file restricts the install and
chains a LICENSE block.

### 5) Second license enforcement point

**Root cause:** the license was enforced in only one place.

**Fix:** `blockchain_add_event()` drops raw events when restricted — via a `weak` symbol so
test harnesses that link only `blockchain.c` still build.

### 6) Shared fatal mask

**Root cause:** the fatal bits were computed separately in the library, CLI, and daemon.

**Fix:** `ARGUS_LIC_FATAL_MASK` (bits 0..9 plus bit 14) is defined once in a shared header
and used by all three consumers.

### 7) Debugger detection (layer 7)

**Added:** `being_traced()` reads `TracerPid` from `/proc/self/status`; a tracer attached ⇒
`E_TAMPER` ⇒ fatal. This is defense-in-depth, not a guarantee (release binaries are
stripped).

### 8) Builder-pipeline breakages

- **Path flattening:** `make-payload.sh` flattened the `tools/argus-license` directory and
  `make` could not build the target — fixed with `mkdir -p` of the parent chain.
- **CLI without a key:** a prebuilt `argus-cli` (built without the public key) shipped in
  the package and `make` skipped rebuilding it ⇒ `UNKNOWN KEY`. Fixed by removing build
  outputs from the payload.
- **Missing verifier/decryptor:** the package had neither. Now `argus-license-verify` (with
  the public key baked in) and `argus-payload` are built into the package, and `install.sh`
  unseals the binaries before installing.
- **Hidden refusal reason:** `--quiet` in `install.sh` suppressed the reason the test greps
  for; removed.

### 9) Release hardening

`strip --strip-all` on the userspace binaries and `strip --debug` on the module, in
`build.sh`.

### 10) `argus-cli help` had two `config` topics (v15.5)

**Symptom:** `argus-cli help config` printed the *license* help and returned, and
`argus-cli help license` fell through to "no detailed help".

**Root cause:** two `if (!strcmp(t, "config"))` branches; the first matched and returned,
so the second was dead code and the `license` topic did not exist.

**Fix:** the first branch is now `license`; `service` and `logs` got their own topics.

### v15.5 — information hygiene

Two new suites close the "don't describe the internals" requirement:

- **`test_no_name_leak.sh` (21):** every operator-facing command is run and its output is
  grepped for internal unit/binary/path tokens; the source and `install.sh` are checked too.
  The `service` view now names roles (`capture engine`, `resolver`), and a new
  `argus-cli logs` reads the journal with `-o cat` so no unit name is shown or typed.
- **`test_codes.sh` (8):** failures carry an opaque `ARG-nnn` code, and every code defined
  in `common/argus_codes.h` is documented in the internal codebook. The codebook is not
  linked from the customer GitBook.

The camouflaged names were also scrubbed from every customer doc page (both languages);
the real names are kept out of the customer book entirely.

---

## The capillary test

`test_capillary.sh` is a deliberately hostile pass over the edges:

- The CLI never crashes on empty, over-long, or malformed arguments.
- A hostile configuration (`INTERVAL=0`) does not wedge the daemon.
- Idle CPU stays flat over a measured window (no busy loop).
- `systemctl stop` returns promptly (no shutdown hang).

---

## Conclusion

- **44 of 44 suites** and **631 assertions** green; no failure left.
- Real bugs (daemon CPU spin / stuck shutdown, fail-open license) were fixed.
- Test flakinesses (live-log race, GCM false negative) were replaced with sounder checks so
  that "green" stays meaningful.
- The release path (builder → sealed package → install → `VALID`) was verified end to end.
- **No reboot** was needed at any point.

### Honest limits

- The tests ran on one VM and one kernel; other kernels may exercise the eBPF hooks
  differently.
- The anti-debug layer is defense-in-depth, not an absolute guarantee; the goal is to raise
  the cost of cracking, not to make it impossible.
- Passing tests are not a substitute for an independent security review.

---

## Reproduce

```bash
cd ~/argus_v15
make
sudo ./tests/run_all.sh
```

For the full list of suites and notes on each, see [Tests](testing.md).
