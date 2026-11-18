# tpt-somatic — Project TODO

Owner: TPT Solutions · License: `MIT OR Apache-2.0` (dual) · Rule: no Apache-2.0-only dependencies, no Apache Arrow, no proprietary codecs/blobs.

Decisions: git deps for tpt-* · first hardware ESP32-S3 + ADS1299 · host OS Win/Linux/macOS/RPi · USB serial before BLE · firmware excluded from main workspace · anomaly = rules + ML hook · GitHub Actions CI · release = binaries + Docker + crates.io.

---

## Phase 0 — Foundations & Licensing
- [ ] `git init`, `.gitignore`, default branch, initial commit
- [ ] Create Cargo workspace (host crates only; firmware excluded)
- [ ] Scaffold crates: `tpt-somatic-io`, `-wire`, `-pipeline`, `-bridge`, `-anomaly`, `-app`
- [ ] Scaffold separate `tpt-somatic-firmware` workspace (own target, toolchain, `.cargo/config.toml`)
- [ ] Add `LICENSE-MIT` (copyright TPT Solutions)
- [ ] Add `LICENSE-APACHE` (Apache-2.0, copyright TPT Solutions)
- [ ] Set `license = "MIT OR Apache-2.0"` and `authors`/`repository` in every `Cargo.toml`
- [ ] Add SPDX header (`MIT OR Apache-2.0`) policy to source files
- [ ] README license section + contribution clause (contributions dual-licensed)
- [ ] Reconcile spec: update spec's "MIT License Only" to dual license; convert spec.txt to `DESIGN.md` (remove stray trailing text)
- [ ] `rust-toolchain.toml`, `rustfmt.toml`, `clippy.toml`, MSRV decision
- [ ] `deny.toml`: allow MIT and MIT-OR-Apache-2.0 style licenses; reject Apache-2.0-only, GPL, proprietary; ban `arrow*`
- [ ] Dependency policy doc (how to vet a new dependency)
- [ ] `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`
- [ ] Open Protocol Mandate doc (what is forbidden, procedure for rejecting vendor hardware)

## Phase 1 — Ecosystem Verification
- [ ] Confirm repo URLs and pin tags/revs for tpt-dsp, tpt-strata, tpt-soma
- [ ] Check licenses of tpt-dsp, tpt-strata, tpt-soma are compatible with dual MIT/Apache
- [ ] Verify tpt-dsp-core / tpt-dsp-analysis `no_std` (+alloc) support; if not, decide feature-gating approach
- [ ] Verify tpt-dsp-testkit `assert_no_alloc` / counting-allocator patterns are reusable
- [ ] Read tpt-strata API: batch/schema/serialization, time-series support
- [ ] Read tpt-soma-chronos and tpt-soma-simulacrum expected schema
- [ ] Verify tpt-dsp-audio API for audio feedback
- [ ] Audit full transitive dependency tree with cargo-deny (tokio, serialport, btleplug, crossbeam, bincode, serde, ratatui)
- [ ] Record gaps/blockers in an issues list (missing features in tpt-* crates)

## Phase 2 — CI & Quality Gates (GitHub Actions)
- [ ] fmt check job
- [ ] clippy job (`-D warnings`)
- [ ] cargo-deny job (licenses, bans, advisories, sources)
- [ ] Test matrix: Windows, Linux, macOS
- [ ] Cross-compile checks: aarch64 (RPi/Jetson), armv7
- [ ] Embedded build checks: ESP32-S3, nRF52, RP2040 targets
- [ ] `no_std` build check for wire (and pipeline `no_std+alloc`)
- [ ] Allocation-test job (counting allocator)
- [ ] Doc build + doctests job
- [ ] Fuzz/property-test scheduled job
- [ ] Caching, dependabot/renovate, MSRV job
- [ ] Mirror anything extra from tpt-dsp's CI

## Phase 3 — tpt-somatic-wire (Open Sensor Protocol)
- [ ] Write wire protocol spec (versioned, self-describing)
- [ ] Define frame layout: sync/magic, version, length, sequence, timestamp, channel map, payload, checksum
- [ ] Choose checksum/CRC and justify
- [ ] Framing/resync strategy for UART/USB/BLE streams (COBS or length-delimited)
- [ ] Channel descriptor: modality (EEG/ECG/thermal/IMU), sample format, bit depth, rate, scale/offset
- [ ] Self-describing handshake/descriptor frames
- [ ] Raw uncompressed sample encoding (no codecs)
- [ ] Implement encoder/decoder, `no_std`, allocation-free
- [ ] Error handling: bad CRC, dropped sequence, truncated frame
- [ ] BLE MTU/fragmentation mapping
- [ ] Property tests (round-trip), fuzz tests (decoder never panics)
- [ ] Test vectors / golden files
- [ ] Protocol documentation and versioning/compat policy

## Phase 4 — tpt-somatic-io (Transport)
- [ ] Transport trait abstraction (open standard frames/raw bytes only)
- [ ] USB/Serial reader (`serialport`), non-blocking
- [ ] Device discovery and reconnect handling
- [ ] BLE transport (`btleplug`) with open GATT profile only
- [ ] Mock/synthetic hardware transport for tests
- [ ] Lock-free SPSC queue (`crossbeam`) from reader thread to pipeline
- [ ] Backpressure: drop frames/downsample, never block or panic the read thread
- [ ] Drop/overflow counters and metrics
- [ ] Clock/timestamp alignment across devices
- [ ] Enforce no proprietary decoding logic (review checklist)
- [ ] Unit + integration tests, platform tests (Win/Linux/macOS)

## Phase 5 — tpt-somatic-pipeline (Real-time DSP)
- [ ] Pipeline architecture and stage trait (preallocated buffers)
- [ ] Frame ingestion from io queue
- [ ] Filtering via tpt-dsp-core (notch, bandpass, detrend)
- [ ] Windowed FFT/PSD, RMS, spectral descriptors via tpt-dsp-analysis
- [ ] EEG features: band powers, gamma synchrony/coherence
- [ ] ECG features: R-peak detection, HRV, HRV coherence
- [ ] Thermal and motion features
- [ ] Multi-channel/multi-modal time alignment
- [ ] Feature vector output type
- [ ] Allocation-free hot path, proven with counting allocator tests
- [ ] `no_std + alloc` build (or feature gate per Phase 1 outcome)
- [ ] Property-based tests for feature extraction
- [ ] Benchmarks and latency budget breakdown (target < 50 ms end-to-end)

## Phase 6 — tpt-somatic-bridge (tpt-strata → tpt-soma)
- [ ] Define metric schema (e.g. "EEG Gamma Synchrony", "HRV Coherence") aligned to tpt-soma-chronos
- [ ] Map to tpt-soma-simulacrum calibration inputs
- [ ] Per-second / per-epoch aggregation
- [ ] Build tpt-strata columnar batches (no Arrow)
- [ ] Serialization and session metadata (device, sample rate, protocol version)
- [ ] Output sinks: file and tpt-soma handoff
- [ ] Slow disk / slow tpt-soma handling: degrade gracefully, no blocking of read thread
- [ ] Integration test: mock hardware → pipeline → valid tpt-strata batches
- [ ] Schema versioning and migration notes

## Phase 7 — tpt-somatic-firmware (separate workspace, no_std)
- [ ] Select embedded HAL/runtime (verify licenses; MIT/Apache-compatible only)
- [ ] ESP32-S3 board bring-up
- [ ] ADS1299 SPI driver (raw ADC values only)
- [ ] Stream via tpt-somatic-wire over USB/UART
- [ ] BLE streaming over open GATT service
- [ ] Sample timing / DRDY interrupt handling, timestamping
- [ ] Reference node hardware doc (BOM, wiring, schematic notes)
- [ ] Hardware-in-loop smoke test with host io crate
- [ ] ECG front-end support (e.g. ADS1292R/MAX30003)
- [ ] Thermal sensor support
- [ ] IMU/motion sensor support
- [ ] nRF52 port
- [ ] RP2040 port
- [ ] Flash/build/debug instructions
- [ ] Procedure for replacing vendor firmware on off-the-shelf boards

## Phase 8 — tpt-somatic-app (Biofeedback UI)
- [ ] CLI skeleton (device select, session start/stop)
- [ ] ratatui TUI dashboard: live signals, bands, HRV, device status
- [ ] Low-latency audio feedback via tpt-dsp-audio
- [ ] Haptic feedback output path
- [ ] Feedback mapping config (metric → audio/haptic parameters)
- [ ] Session recording and playback CLI
- [ ] Config file and logging
- [ ] Cross-platform audio/terminal testing (Win/Linux/macOS/RPi)

## Phase 9 — tpt-somatic-anomaly
- [ ] Define scope vs tpt-soma-pathos (document boundary)
- [ ] Rule engine: declarative multi-modal correlation rules
- [ ] Example rule: thermal bloom + HRV spike + motion stillness
- [ ] Statistical improbability scoring
- [ ] Pluggable ML inference trait (runtime chosen later; license-vetted)
- [ ] Event output format and bridge integration
- [ ] Unit tests with synthetic scenarios

## Phase 10 — Integration, Performance & Safety Validation
- [ ] End-to-end test with mock hardware
- [ ] End-to-end test with real ESP32-S3 + ADS1299 node
- [ ] Measure sensor-to-biofeedback latency (< 50 ms)
- [ ] Backpressure/chaos tests (slow disk, dropped link, flooded queue)
- [ ] Long-duration soak test
- [ ] Allocation and panic audit of hot path
- [ ] Data-integrity check (raw samples bit-exact through pipeline inputs)
- [ ] Security review (untrusted device input, parser robustness)

## Phase 11 — Packaging & Deployment
- [ ] `cargo build --release` / `cargo run --bin tpt-somatic-app` verified
- [ ] Standalone binaries (Windows, Linux, macOS, ARM) with embedded assets
- [ ] Release automation in GitHub Actions
- [ ] Dockerfile and image for server use (USB/serial passthrough notes)
- [ ] Raspberry Pi and Jetson Nano optimized builds
- [ ] Embedded firmware release artifacts

## Phase 12 — Documentation & Release
- [ ] Architecture docs and ecosystem diagram
- [ ] Wire protocol spec published
- [ ] User guide, hardware guide, firmware flashing guide
- [ ] API docs (docs.rs-ready)
- [ ] CHANGELOG and versioning policy
- [ ] Final `cargo deny` + license audit
- [ ] Publish crates to crates.io (switch git deps to versions as needed)
- [ ] Tag v0.1.0 and publish release
