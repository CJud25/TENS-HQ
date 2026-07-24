*Excerpted verbatim from `CJud25/FMP-Calculator`, `docs/specs/2026-07-23-fmp-calculator-design.md`, §8 "packet-fmp/v1 contract", at commit `fb108d8` (2026-07-24) — [full spec](https://github.com/CJud25/FMP-Calculator/blob/fb108d8a0519e8bb84fef5dea83e7555d3bd622a/docs/specs/2026-07-23-fmp-calculator-design.md).*

---

## 8. packet-fmp/v1 contract

The typed federation handoff. Consumer side + schema + sample files + golden vectors ship in v1;
the **producer** lands later in ReconRadar as a separate slice there (design input 4). Follows the
house radar-handoff pattern: a semver envelope re-verified on arrival.

### 8.1 JSON schema (field names, types, required/optional)

```jsonc
{
  // --- envelope (required) ---
  "kind": "packet-fmp",                 // string, REQUIRED, must equal "packet-fmp"
  "version": "1.0.0",                   // string semver, REQUIRED; consumer accepts major == 1
  "produced_by": "ReconRadar",          // string, REQUIRED (producer app name)
  "produced_at": "2026-07-23",          // string ISO-8601 date, REQUIRED
  "source_packet_id": "…",              // string, OPTIONAL (upstream packet/export id)

  // --- lane (required) ---
  "lane": "services",                   // enum "services" | "products", REQUIRED

  // --- contract context (all OPTIONAL; observed) ---
  "contract_context": {
    "awarding_agency": "…",             // string
    "awarding_sub_agency": "…",         // string (the component — matches on this, not top agency)
    "naics": "561720",                  // string
    "psc": "S201",                      // string
    "place_of_performance": { "county": "…", "state": "FL" },
    "period_of_performance": { "start": "2026-10-01", "end": "2027-09-30" },
    "wd_number": "2015-4281",           // string
    "wd_version": "24",                 // string
    "reference_piid": "…"               // string, OPTIONAL
  },

  // --- labor lines (REQUIRED for lane=services; ≥1) ---
  "labor_lines": [
    {
      "occupation_code": "01111",       // string, REQUIRED
      "occupation_title": "General Clerk I", // string, REQUIRED
      "wd_base_rate": 18.42,            // number ≥ 0, REQUIRED, provenance observed
      "hours": 2080,                    // number > 0, REQUIRED — annual hours PER FTE (never a
                                        //   pre-multiplied line total); effective line hours =
                                        //   hours × fte (pinned in §4 step 1)
      "fte": 1.0,                       // number > 0, OPTIONAL (default 1.0)
      "provenance": "observed"          // enum observed|user-entered|derived, OPTIONAL (default observed)
    }
  ],

  // --- product lines (REQUIRED for lane=products; ≥1) ---
  "product_lines": [
    {
      "item": "…", "unit": "each",      // strings, REQUIRED
      "quantity": 1000,                 // number > 0, REQUIRED
      "materials_cost": 3.10,           // number ≥ 0, REQUIRED
      "direct_labor_hours": 0.25,       // number ≥ 0, REQUIRED
      "direct_labor_rate": 18.42,       // number ≥ 0, REQUIRED
      "packaging_cost": 0.40,           // number ≥ 0, OPTIONAL (default 0)
      "provenance": "observed"
    }
  ],

  // --- optional seeds ---
  "hw_selection": "standard",           // enum standard|eo13706|hawaii_standard|hawaii_eo13706, OPTIONAL
  "notes": "…"                          // string, OPTIONAL
}
```

**Internal rates (overhead, G&A, net proceeds, SUTA, WC) are deliberately NOT in the contract** —
they are user-entered, session-only (non-goal 4). A handoff seeds the *observed* public portion; the
analyst adds internal rates in-session.

### 8.2 Validation & re-verification on arrival

`handoff.load(raw) -> Scenario` performs, in order (fail-fast, typed):
1. **Parse guard** — `args`/payload may arrive as a JSON string; open with
   `data = json.loads(raw) if isinstance(raw, str) else raw` (house lesson: the Workflow/args
   string gotcha).
2. **Envelope check** — `kind == "packet-fmp"` else `HandoffError("not a packet-fmp handoff")`;
   parse `version`, require `major == 1` else `HandoffError("unsupported packet-fmp version {v};
   this build reads 1.x")` (reject 2.x explicitly, do not best-effort a future schema).
3. **Required-field guard** — presence + type for envelope + lane; then lane-specific: `labor_lines`
   ≥ 1 for services, `product_lines` ≥ 1 for products.
4. **Range guard** — `wd_base_rate ≥ 0`, `hours > 0`, `quantity > 0`, rates ≥ 0, `fte > 0`.
5. **Unknown fields** — ignored but counted and logged (forward-compatibility; a `1.1` producer may
   add fields).
6. **Build** — construct the canonical `Scenario` (identical object to manual entry) and return it.

**Error behavior on malformed/missing:** raise typed `HandoffError` with the failing field path; the
app catches it, shows an honest empty state ("This handoff couldn't be read: <reason>. You can enter
the scenario manually.") and falls back to manual entry — **never a crash, never a partial silent
load** (§13).

### 8.3 Sample handoff files + golden-vector pack

Ship under `tests/fixtures/handoff/`:
- `packet-fmp-v1-services-custodial.json` — the seeded custodial demo (12 FTE) as a handoff.
- `packet-fmp-v1-products-unit.json` — a single-item products handoff.
- `packet-fmp-v1-malformed-missing-labor.json`, `…-bad-version-2x.json`,
  `…-negative-rate.json` — the error-path fixtures.

**Golden-vector pack** (`tests/golden/`): each entry is `input_vector → expected full ledger
output` as committed JSON (`{ "input": {…}, "expected_ledger": {…} }`). **These vectors are shared
verbatim so the future ReconRadar producer tests its output against the identical vectors** — the
producer is correct iff its emitted packet, run through this consumer, reproduces the expected
ledger. Because manual entry and handoff produce the same `Scenario`, one golden set covers both
intake paths.

---

