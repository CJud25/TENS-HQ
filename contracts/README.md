# Integration contracts

TENS HQ's "federation, not fusion" claim rests on typed JSON contracts crossing repo
boundaries — each versioned, fail-closed on schema mismatch, and re-verified by the
receiving app rather than trusted blindly. This folder vendors the real spec/sample/golden
evidence for each contract, snapshotted at a pinned commit, alongside links to the live
source.

| Contract | Producer → Consumer | Status |
|---|---|---|
| [`radar-handoff-v1/`](radar-handoff-v1/) | GovCon Recompete Radar → ReconRadar | Shipped, both sides |
| [`packet-fmp-v1/`](packet-fmp-v1/) | ReconRadar → FMP Calculator | Consumer shipped; producer queued |

These are dated snapshots for portfolio legibility, not the live source of truth — follow the
links inside each folder for the current implementation.
