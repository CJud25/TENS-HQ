# packet-fmp/v1

**ReconRadar (producer) → FMP Calculator (consumer).** Consumer shipped and tested against
golden vectors; the ReconRadar producer is **not yet built** (confirmed by repo search — zero
`packet-fmp` / `packet_fmp` matches in ReconRadar).

A versioned handoff carrying the *observed, public* portion of a pricing scenario — wage-
determination labor lines (or product lines) plus contract context. Internal rates (overhead,
G&A, net proceeds, SUTA, WC) are deliberately **not** in the contract; they stay analyst-entered
and session-only. FMP Calculator's consumer re-verifies every field, rejects any non-`1.x`
version, and builds the identical `Scenario` object manual entry produces.

## Files here

- [`packet-fmp-v1-spec-excerpt.md`](packet-fmp-v1-spec-excerpt.md) — the formal contract (JSON
  schema, validation order, golden-vector design), excerpted verbatim from FMP-Calculator's
  design doc §8.
- [`packet-fmp-v1-services-custodial.json`](packet-fmp-v1-services-custodial.json) — a **handoff
  payload sample** (what crosses the wire: public labor lines + context; internal rates absent
  by design). Its envelope carries `"produced_by": "ReconRadar"` — that names the **future**
  producer this fixture *simulates*; the file is hand-authored in FMP-Calculator's own test
  suite, not emitted by ReconRadar (which has no producer yet).
- [`services_single_occupation.json`](services_single_occupation.json) — a **calculator golden
  vector** (`input` → expected ledger), shared so the future producer can prove correctness by
  reproducing the same ledger. It deliberately **includes** the session-only `indirect_rates` a
  handoff never transmits, because it is a calculator test vector, not a handoff sample. Its
  ledger key is `expected` (alongside a top-level `_arithmetic` string — one line of prose
  pointing at where the expected numbers were hand-computed, not a structured block), while the
  spec §8.3 above names the ledger key `expected_ledger` — a naming drift in the source repo,
  preserved verbatim here so a side-by-side diff doesn't read as an error.

## Live source

- Consumer: [`FMP-Calculator/fmp/handoff.py`](https://github.com/CJud25/FMP-Calculator/blob/fb108d8a0519e8bb84fef5dea83e7555d3bd622a/fmp/handoff.py)
- Handoff fixtures (5): [`FMP-Calculator/tests/fixtures/handoff/`](https://github.com/CJud25/FMP-Calculator/tree/fb108d8a0519e8bb84fef5dea83e7555d3bd622a/tests/fixtures/handoff)
- Golden vectors (5): [`FMP-Calculator/tests/golden/`](https://github.com/CJud25/FMP-Calculator/tree/fb108d8a0519e8bb84fef5dea83e7555d3bd622a/tests/golden)

*Snapshotted at FMP-Calculator `fb108d8` (2026-07-24) — follow the links above for the current
implementation.*
