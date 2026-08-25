# P0 + P0.5 Implementation Summary (2026-08-25)

English rendering prepared for public release; code paths adapted to this repository's `code/`
layout.

## Changes made

### P0.5 - `false_terminal` RCA + probe fix

**Root cause:** `inject_defect(false_terminal)` targeted wrong steps (see
`FALSE_TERMINAL_PROBE_RCA.md`). Verifier invariant intact.

**Code:** `code/experiments/phase_c_validation/code/offline_generator.py` - pick mid-steps with
non-empty `next_state.pending`; write the formal `step.completion` terminal triple.

**Result:** `mutation_probe.json` -> `false_terminal: 1.0` (was 0.0).

### P0 - `APPLY_LOCAL_UPDATE` spec sync

**Root cause:** Spec drift - `validator_agent` / `stepwise_builder` allow `APPLY_LOCAL_UPDATE` under
Variation_G; `MechanicalVerifier` did not.

**Code:** `code/failure_distribution/mechanical_verifier.py`
- Add `APPLY_LOCAL_UPDATE` to `STOKEN_ALLOWED_ACTIONS["S2_SATISFIED"]`
- Block `APPLY_LOCAL_UPDATE` after S1/S5 (same as `DERIVE`)

**Tests:** `code/failure_distribution/tests/test_mechanical_verifier.py` - 3 new cases.

### P2 - builder carry-forward

**Finding:** After P0 re-verification on the **existing v1 `generated.jsonl`** (no regeneration):

| Stratum | Pass rate |
|---------|-----------|
| main Successful_Path (all domains) | **72/72** |
| main Repair_Path | **72/72** |
| main Counterfactual_Near_Miss | **0/72** (expected) |
| metamorphic | **24/24** |
| baseline | **2/54** (52 intentional `SV_CARRY`) |

**Conclusion:** No `SV_CARRY_FORWARD_VIOLATION` on any `main` record. P2 builder changes **not
required** for the v1 artifacts - prior failures were spec drift (`S_TOKEN`), not carry-forward bugs.

## v1 cohort headline (re-verified, same 294 records)

- **170/294 (57.8%)** overall pass - expected given 72 counterfactual + 52 baseline controls designed
  to fail.
- **Gate 1:** main SP pass **1.0** (54/54 paired arms); control **0.037**; McNemar b=52, c=0;
  seed_mismatch red flag unchanged (v1 design).
- **Gate 2:** agreement **1.0** (12/12 latent pairs).

Artifacts refreshed: verification results, analysis summary, analysis report, mutation probe.

## Not done at that time (next steps per agreed plan; since executed as cohort_v2)

1. **v2 pre-registration** - paired baseline same seed (design change) -> executed; see
   `../cohort_v2/reports/PREREGISTRATION.md`.
2. Full formal rerun with a new run-id -> executed (`cohort_v2`).
3. Human kappa -> completed for both cohorts (36 samples each).
4. Surface probe reinterpretation - action-token vs narrative-only dual report -> adopted.
5. Optional: fix `skip_prerequisite` inject targeting on longer TECH-03 horizons (same bug class as
   false_terminal).
