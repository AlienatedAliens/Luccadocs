# Project Lucca: evidence and results

> This repository contains public evidence, pre-registration protocols, and research ledgers for Project Lucca. The complete companion agent codebase, test suite (1,243+ tests), and development history are public at [github.com/AlienatedAliens/ProjectLucca](https://github.com/AlienatedAliens/ProjectLucca). File paths (`src/…`, `tests/…`, `logs/…`) reference that repository directly.

## What it is

A local desktop companion agent built around two core verification mechanisms and a local NLI verifier:

1. **Provenance-gated memory (`ProvenanceGate`).** Memory writes require an authentic domain-separated HMAC-SHA256 signature (`ADR-006`), ensuring model narration (`LLM_NARRATION`) and untrusted tool outputs cannot become persistent stored beliefs. Self-induced procedural rules remain strictly in quarantine (`D-REAS-06`) until human review.
2. **Receipt-checking of action claims (`TruthInReceiptMatcher`).** A deterministic clause-level syntactic extractor and receipt unifier (sub-50 µs on CPU, zero-LLM) intercepts streaming output and compares what the agent says it did ("I deleted X") against authentic execution receipts (`ToolReceipt`) from the active turn (aborting mid-stream if a stateful tool ran, or flagging for post-turn audit on read-only turns).
3. **Semantic consistency verifier (`VerifierWorker`).** An in-process ModernBERT-base NLI classifier (149M parameters) running locally via OpenVINO for memory consistency and contradiction checking.

## What works today

- The receipt matcher passes 146 authored tests (73 unit + 73 grammar tests; 186 in the full authored suite). On 200 live transcripts from two small models ($T=0.6$) it went from catching 1 of 23 ungrounded action claims to 23 of 23, with 0 of 100 claim-free controls flagged. These numbers are strictly in-sample calibration (see [Erratum](ERRATUM_CALIBRATION_2026-10-24.md)).
- Memory recall against the bare model: 10/10 vs 0/10 (+100 pp, exact McNemar $p = 0.00195$; bare model has no memory by construction).
- The NLI consistency verifier achieves 93/100 accuracy (contradiction P/R 1.000 / 1.000) on a hand-labeled benchmark when given an unconstrained 500 ms evaluation window.

## Not established yet

- **Out-of-sample generalization:** Whether the matcher fixes hold on new, unseen prompts and verbs. This is the subject of the pre-registered held-out evaluation below.
- **Production verifier timeout degradation:** Under the 50 ms interactive budget on CPU, mean latency is 57.1 ms, causing 81 of 100 pairs to time out. Contradiction recall drops to 0.075, and all 10 downstream consumers currently fail open.
- **Null results on small contrastive batteries ($n=10$):** Receipt introspection (8/10 vs 7/10, $p = 1.000$) and calibrated abstention (9/10 vs 10/10, $p = 1.000$) show no statistically significant difference from the bare model.
- **TTS safety guard keyword regex failure:** Blocked 0 of 10 conversational harmful probes (3/10 refused by native model weights). It is not relied upon.
- **Single-annotator labeling:** All development and calibration labels are from a single annotator (the author).

## Documents

- [Evidence ledger](EVIDENCE_LEDGER.md) — Benchmark results, Wilson 95% score CIs, and documented failure modes.
- [Pre-registered held-out protocol](PREREG_MATCHER_SENSITIVITY.md) — Pre-registered evaluation plan (SHA-256: `b6b1a671491a45839d23552d946e9ee3136a64da97c6ed4098f710fbbded6884`).
- [Calibration erratum (2026-10-24)](ERRATUM_CALIBRATION_2026-10-24.md) — Self-disclosed audit of baseline replication differences, clause counts, and hash verifications.
- [Fail-open vulnerability analysis](FAILOPEN_ANALYSIS.md) — Systematic audit of the 10 NLI downstream consumers and the 3-tier remediation plan.
- [Library extraction plan](EXTRACTION_PLAN.md) — Packaging specification for the standalone `agent-provenance-guard` Python library.

## Planned work (Grant Deliverables)

1. **A pre-registered held-out test of the matcher:** 250 fresh prompts across three model scales (Gemma 4 12B, E4B, E2B), including unseen verbs absent from development data. Outputs will be labeled blind and shuffled with supervisor flags hidden, with a second independent annotator double-labeling a subset for inter-annotator agreement (Cohen's $\kappa$). Raw labels will be published. Protocol SHA-256: `b6b1a671491a45839d23552d946e9ee3136a64da97c6ed4098f710fbbded6884`.
2. **Remediating the verifier's fail-open path and latency bottleneck:** Refactoring all 10 downstream consumers to fail closed/quarantine on timeout, introducing an asynchronous two-phase commit queue for memory/belief updates (evaluated at 500 ms where contradiction recall is 1.000), and evaluating OpenVINO INT8 quantization aiming to clear the 50 ms budget, reporting any recall impact (see [Fail-open analysis](FAILOPEN_ANALYSIS.md)).
3. **Standalone open-source release (`agent-provenance-guard`):** Extracting the HMAC provenance gate and deontic receipt matcher into a zero-bloat, documented standalone Python library with zero LLM dependencies and sub-millisecond execution so other agent frameworks can drop them in (see [Extraction plan](EXTRACTION_PLAN.md)).
