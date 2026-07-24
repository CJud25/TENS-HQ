# radar-handoff/v1

**GovCon Recompete Radar (producer) → ReconRadar (consumer). Shipped, both sides live.**

An analyst exports a candidate snapshot from GovCon Recompete Radar as a `radar-handoff/v1`
JSON and uploads it into ReconRadar's Opportunity Packet, where it renders as a caveat-first
"Origin" section — claims from a snapshot, never packet evidence.

## Schema

**Top level**

| Field | Type | Required | Notes |
|---|---|---|---|
| `schema_version` | string | yes | must equal `"radar-handoff/v1"` exactly |
| `snapshot_date` | string | yes | `YYYY-MM-DD` |
| `piid` | string | yes | non-blank; stored verbatim |
| `radar_source_label` | string | no | free-text provenance label |
| `synthetic_sample` | boolean | no | default `false` |
| `claims` | object | no | see below |

**`claims`** — all optional, independently nullable: `referenced_idv_piid`, `recipient_name`,
`recipient_uei`, `awarding_subagency`, `naics_code`, `psc_code`, `type_of_set_aside_code`,
`extent_competed_code` (strings); `number_of_offers_received` (non-negative integer);
`total_obligation`, `base_and_all_options` (numbers); `pop_start_date`, `pop_current_end_date`,
`pop_potential_end_date` (string dates, not strictly validated); `place_of_performance`
(object: `city`, `county` strings; `state` normalized to exactly 2 characters).

## Guardrails (enforced by the consumer's parser)

- 1,000,000-byte upload cap; any string field over 500 characters fails the whole parse.
- Unrecognized fields are ignored but counted and disclosed — never silently dropped.
- A wrong `schema_version` is a fixed parse error, never best-effort.
- The handoff **never feeds the Eligibility Gate, capture-window, or incumbent-leads logic**;
  the `type_of_set_aside_code` specifically is display-only and is never written into the
  analyst set-aside input (no value laundering).
- PIID and place-of-performance *may* prefill the packet's retrieval inputs — verbatim,
  visible, and analyst-editable — never silently promoted into an assessment.

## Files here

- [`ADR-019-radar-handoff-intake.md`](ADR-019-radar-handoff-intake.md) — the design rationale,
  vendored verbatim from ReconRadar (with a dated reconciliation note at the top).
- [`sample_radar_handoff.json`](sample_radar_handoff.json) — a synthetic sample
  (`synthetic_sample: true`, PIID `SYNTH-A2-0001`) showing the exact shape.

## Live source

- Consumer / parser: [`ReconRadar/src/tens_hq/radar_handoff.py`](https://github.com/CJud25/ReconRadar/blob/b10e23d418b955eb7debfac7e953dae5af68d2e8/src/tens_hq/radar_handoff.py)
- Producer / builder: [`GovConRadar/streamlit_app/components/radar_handoff.py`](https://github.com/CJud25/GovConRadar/blob/5e185e147f60c6b138f998d337a3d4c3dc4f7035/streamlit_app/components/radar_handoff.py)
- Consumer tests: [`ReconRadar/tests/test_radar_handoff.py`](https://github.com/CJud25/ReconRadar/blob/b10e23d418b955eb7debfac7e953dae5af68d2e8/tests/test_radar_handoff.py)

*Snapshotted at ReconRadar `b10e23d` / GovConRadar `5e185e1` (2026-07-24) — follow the links
above for the current implementation.*
