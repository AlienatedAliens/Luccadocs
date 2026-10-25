# Project Lucca: evidence and results

> This repository contains public evidence, pre-registration protocols, and research ledgers for Project Lucca. The complete companion agent codebase, test suite (1,243+ tests), and development history are public at [github.com/AlienatedAliens/ProjectLucca](https://github.com/AlienatedAliens/ProjectLucca). File paths (`src/…`, `tests/…`, `logs/…`) reference that repository directly.

## Research Publications & Empirical Manuscripts

1. **Foundational Preprint (Published & Citable):**
   - *Provenance-Gated Memory and Deontic Output Supervision for Epistemically Grounded Autonomous Agents*
   - **DOI:** [`10.5281/zenodo.23649049`](https://doi.org/10.5281/zenodo.23649049) &bull; **ORCID:** [`0009-0004-0133-0463`](https://orcid.org/0009-0004-0133-0463)
   - **HTML / PDF Mirror:** [alienatedaliens.github.io/ProjectLucca](https://alienatedaliens.github.io/ProjectLucca/)

2. **Empirical Study 1 — Memory Poisoning Ablation (Pre-registered `prereg-poisoning-v1`):**
   - *Cryptographic Provenance Gates Mitigate Multi-Session Memory Poisoning in Autonomous Agents*
   - **Protocol:** [`PREREG_MEMORY_POISONING_ABLATION.md`](PREREG_MEMORY_POISONING_ABLATION.md)
   - **Manuscript PDF:** [Download Paper 1 PDF](https://alienatedaliens.github.io/ProjectLucca/papers/01_memory_poisoning_ablation.pdf)
   - **Key Finding:** Provenance gating eliminates recursive self-poisoning (0.0% vs 100.0% in Naive RAG, exact McNemar $p = 6.98 \times 10^{-10}$), preserves conversational continuity (100.0% vs 0.0% in deletion), retains 96.7% benign calculation continuity, and confirms the empirical open boundary on user misinformation (100.0% absorption). Dual-annotator agreement: $\kappa = 0.7683$.

3. **Empirical Study 2 — Held-Out Deontic Output Supervision & SOAC (Pre-registered `prereg-supervision-v1`):**
   - *Why Speech-Regex Action Supervisors Miss Live Output: An Empirical Evasion Taxonomy and Held-Out Structured Action Contract Evaluation*
   - **Protocol:** [`PREREG_DEONTIC_SUPERVISION_HELD_OUT.md`](PREREG_DEONTIC_SUPERVISION_HELD_OUT.md)
   - **Manuscript PDF:** [Download Paper 2 PDF](https://alienatedaliens.github.io/ProjectLucca/papers/02_deontic_supervision_evasion.pdf)
   - **Key Finding:** Evaluated across $N=120$ frozen held-out prompts. Strict speech regex detected only 1/16 (6.25%) live ungrounded action claims, confirming 4 grammatical evasion modes. Structured-Output Action Contracts (SOAC) achieve 6.82 $\mu$s mean latency (11x faster than regex) with 0/40 false alarms on controls, while revealing 13.3% prose leakage that necessitates GBNF decoding constraints.

---

## What it is

A local desktop companion agent built around two core verification mechanisms and a local NLI verifier:

1. **Provenance-gated memory (`ProvenanceGate`).** Memory writes require an authentic domain-separated HMAC-SHA256 signature (`ADR-006`), ensuring model narration (`LLM_NARRATION`) and untrusted tool outputs cannot become persistent stored beliefs. Self-induced procedural rules remain strictly in quarantine (`D-REAS-06`) until human review.
2. **Receipt-checking of action claims (`TruthInReceiptMatcher`).** A deterministic clause-level syntactic extractor and receipt unifier (sub-50 µs on CPU, zero-LLM) intercepts streaming output and compares what the agent says it did ("I deleted X") against authentic execution receipts (`ToolReceipt`) from the active turn (aborting mid-stream if a stateful tool ran, or flagging for post-turn audit on read-only turns).
3. **Semantic consistency verifier (`VerifierWorker`).** An in-process OpenVINO MiniLM-L6 cross-encoder running locally on CPU for memory consistency and contradiction checking (12.2 ms mean latency, 0 timeouts).

---

## Documents & Evidence

- [Evidence ledger](EVIDENCE_LEDGER.md) — Benchmark results, Wilson 95% score CIs, dual-annotator metrics, and documented failure modes.
- [Pre-registered Poisoning Protocol](PREREG_MEMORY_POISONING_ABLATION.md) — Tagged `prereg-poisoning-v1`.
- [Pre-registered Supervision Protocol](PREREG_DEONTIC_SUPERVISION_HELD_OUT.md) — Tagged `prereg-supervision-v1`.
- [Pre-registered legacy held-out protocol](PREREG_MATCHER_SENSITIVITY.md) — Protocol SHA-256: `b6b1a671491a45839d23552d946e9ee3136a64da97c6ed4098f710fbbded6884`.
- [Calibration erratum (2026-10-24)](ERRATUM_CALIBRATION_2026-10-24.md) — Self-disclosed audit of baseline replication differences, clause counts, and hash verifications.
- [Fail-open vulnerability analysis](FAILOPEN_ANALYSIS.md) — Systematic audit of the 10 NLI downstream consumers and the 3-tier remediation plan.
- [Library extraction plan](EXTRACTION_PLAN.md) — Packaging specification for the standalone `agent-provenance-guard` Python library.
