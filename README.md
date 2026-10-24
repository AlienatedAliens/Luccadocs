# Project Lucca: evidence and results

> This repository contains documents only. Paths like `src/…`, `tests/…` and `logs/…` refer to the main code repository, which is not public yet (planned for [date]). Access is available on request: [your email].

## What it is

A local desktop agent built around two safety mechanisms:

1. **Provenance-gated memory.** Memory writes need an HMAC-SHA256 signature, so model narration and untrusted tool output can't become stored "facts". Rules the agent proposes for itself stay in quarantine until I approve them.
2. **Receipt-checking of action claims.** A supervisor compares what the agent says it did ("I deleted X") against tool-execution receipts from the current turn.

## What works today

- The receipt matcher passes 146 authored tests. On 200 live transcripts from two small models (T=0.6) it went from catching 1 of 23 ungrounded action claims to 23 of 23, with 0 of 100 claim-free controls flagged. These numbers are in-sample (see the erratum).
- Memory recall against the bare model: 10/10 vs 0/10 (the bare model has no memory by design).
- The NLI consistency verifier scores 93/100 on a hand-labeled pair set when given a generous timeout.

## Not established yet

- Whether the matcher fixes hold on new data. That is the pre-registered held-out test below.
- Verifier behavior under production latency: at the 50 ms timeout, 81 of 100 pairs time out, contradiction recall is 0.075, and all 10 downstream consumers fail open.
- Receipt introspection (8/10 vs 7/10) and abstention (9/10 vs 10/10) show no measurable difference from the bare model at n=10.
- All labels are from a single annotator (me).

## Documents

- [Evidence ledger](EVIDENCE_LEDGER.md)
- [Pre-registered held-out protocol](PREREG_MATCHER_SENSITIVITY.md), SHA-256 `b6b1a671491a45839d23552d946e9ee3136a64da97c6ed4098f710fbbded6884`
- [Erratum](ERRATUM_CALIBRATION_2026-10-24.md)

## Planned work

A pre-registered held-out test of the matcher, a fix for the verifier's fail-open path, and a standalone release of the two mechanisms.
