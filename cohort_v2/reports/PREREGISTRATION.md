# Phase-C v2 Pre-registration (2026-08-25)

> English rendering prepared for public release. Placed here because it governs the v2 cohort.
> Status at publication: **executed and closed (2026-08-25)**.

## What changed in v2

Only **baseline seed pairing** and **Gate 1 interpretation**:

- `frozen_spec_v2.json`: each `BASE-####` inherits `seed` from its `pair_id` `MAIN-####`
  (54/54 aligned).
- Same builder ablation (`_apply_phase_c_baseline_ablation`), same `training_admission: false`.
- New evidence directory for the formal driver run with `--spec frozen_spec_v2.json`.

## What did NOT change

- 294 record count and condition mix (216 main + 54 baseline + 24 metamorphic).
- MechanicalVerifier rules (post P0 `APPLY_LOCAL_UPDATE` sync).
- Evidence cohort policy: **all 294 retained**, failures included in stratified tables.
- Counterfactual 0/72 pass expectation.
- Human kappa protocol (36-sample stratified CSV).

## Gate 1 claim upgrade

| | v1 | v2 |
|---|----|----|
| Design role | Verifier sensitivity to **post-hoc** carry-forward mutation | **Paired ablation** sensitivity on matched seed + constraint graph |
| seed_mismatch flag | Expected red flag (54/54) | **Observed 0/54** |
| Forbidden claim | Full generator superiority | Same - still not independent generator superiority |
| Allowed claim | Verifier detects known mutation on format-matched SP | Verifier discriminates main vs ablated control **on paired slots** |

## Observed results (v2 formal run)

| Gate / probe | v2 result |
|--------------|-----------|
| Gate 1 | main 1.0 vs control 0.0; McNemar b=54,c=0; seed_mismatch **0/54** |
| Gate 2 | metamorphic agreement **1.0** |
| Gate 3 | max JS **0.0** |
| Mutation probe | sensitivity **1.0** |
| Surface narrative_only LOO | **0.571** ~ chance |
| main SP + Repair | **144/144** pass |
| Human kappa (36) | carryforward kappa=1.0; terminal & first_error human-human kappa=0.0 (CF rubric tension) |

v1 retained under `cohort_v1/` (see its reports directory).

## Execution checklist

| Step | Status |
|------|--------|
| 1. Generate spec | done |
| 2. Formal run -> v2 evidence directory | done, 294/294 |
| 3. verify + probes + stratified analysis | done |
| 4. Human kappa on v2 sample (fresh blind coding) | done |
| 5. Report: v1 appendix vs v2 primary Gate 1 | done (see `ANALYSIS_REPORT.md`) |

## Explicit non-goals for v2

- No admission gate dropping failed main-gold from the evidence cohort.
- No conflation of training admission with evidence inclusion.
- No claim that surface LOO ~ 1.0 on the action-token probe is a generator defect.
- No claim that downstream model training outcomes were measured (requires a separate ablation).
