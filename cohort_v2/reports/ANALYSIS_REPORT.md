# Phase C Validation - Analysis Report (v2 primary)

> English rendering of the original analysis report, prepared for this public release.
> Interpretive strings that were originally non-English have been translated; all numeric fields are
> unchanged. File references adapted to this repository's layout.

**Cohort:** v2 formal run - spec `frozen_spec_v2.json` - completed 2026-08-25
**v1 appendix:** `../../cohort_v1/reports/ANALYSIS_REPORT.md` -
`../../cohort_v1/reports/STUDY_COMPLETION.md`

Total generated: 294 records (216 main + 54 paired baseline + 24 metamorphic).

## v1 appendix vs v2 primary Gate 1

| Metric | v1 | v2 |
|--------|---------------------------|------------------------------|
| Design role | Post-hoc carry-forward mutation sensitivity | **Paired ablation** (matched seed + slot) |
| n_pairs | 54 | 54 |
| Baseline seed = pair seed | 0/54 | **54/54** |
| seed_mismatch red flag | **54/54** (expected in v1) | **0/54** |
| main pass rate (SP pairs) | 1.0 [0.934, 1.0] | 1.0 [0.934, 1.0] |
| control pass rate | 0.037 [0.01, 0.125] | **0.0** [0.0, 0.066] |
| paired risk difference | 0.963 [0.907, 1.0] | **1.0** [1.0, 1.0] |
| McNemar (b, c) | (52, 0) | **(54, 0)** |
| red flags | seed_mismatch | **none** |

**Interpretation:** v2 is the **primary** Gate 1 table for paired-ablation claims. v1 remains valid
appendix evidence for verifier sensitivity under the original seed-mismatch design. Neither table
supports independent-generator superiority.

## Gate 1 - Verifier sensitivity to a known carry-forward mutation

This table is **not** evidence that the full generator outperforms an independent baseline generator.
The control is a format-matched Successful_Path with a post-hoc carry-forward mutation.

- n_pairs = 54
- design_role = verifier_sensitivity_control
- claim_forbidden = Do not report this as full-method superiority over an independent generator baseline.
- main pass rate = 1.0 (95% CI [0.934, 1.0])
- control pass rate = 0.0 (95% CI [0.0, 0.066])
- paired risk difference = 1.0 (95% CI [1.0, 1.0])
- McNemar: b=54, c=0, p=0.0
- seed compared/mismatch = 54/0
- red flags: none

## Gate 2 - Metamorphic (12 latent x 2 variants)

- agreement rate = 1.0 (exact binomial 95% CI [0.757, 1.0])
- exact binomial CI; no large-sample approximation is pretended (study design document, sec 4.6)

## Gate 3 - Failure-distribution JS (reproducibility evidence, not the primary result)

- max JS = 0.0 (bootstrap 95% CI [0.0, 0.0])
- role: reproducibility evidence (NOT primary result); report CI, do not assert < 0.05 alone

## Mutation sensitivity / specificity (zero generation cost)

- n_valid_sampled = 30
- sensitivity by kind = {'skip_prerequisite': 1.0, 'false_terminal': 1.0, 'corrupt_lineage': 1.0}
- overall sensitivity = 1.0 (expected ~ 1.0)
- specificity (reword) = 1.0; specificity (surface) = 1.0 (expected ~ 1.0)

## Surface-leakage probe (leave-one-domain-out Naive Bayes)

### surface_metadata
- description = surface_text + endpoint flag (original Phase-C probe)
- per held-out domain accuracy = {'financial': 0.816, 'legal': 0.816, 'medical': 0.816}
- overall LOO accuracy = 0.816 (chance = 0.5)
- claim_use = metadata separability; not semantic shortcut alone

### action_token
- description = decision.action tokens only - domain action vocabulary separability
- per held-out domain accuracy = {'financial': 0.429, 'legal': 0.429, 'medical': 0.429}
- overall LOO accuracy = 0.429 (chance = 0.5)
- claim_use = task-structure signal; high LOO expected for TECH-03 vs TECH-04 paths - NOT a generator defect

### narrative_only
- description = content_output + rationale with action prefixes stripped
- per held-out domain accuracy = {'financial': 0.571, 'legal': 0.571, 'medical': 0.571}
- overall LOO accuracy = 0.571 (chance = 0.5)
- claim_use = narrative shortcut test - closest to the sec 3.6 semantic-shortcut concern

> Probe action_token near 1.0 indicates separable domain action vocabularies, a normal task-structure
> effect; only probe narrative_only near 0.5 supports a no-narrative-shortcut claim.

## Human double-blind agreement (36 records, double-blind)

- terminal: human-human kappa=0.0 (CI [0.0, 0.0]); agreement=0.667; verifier-human kappa(a1)=1.0 (CI [1.0, 1.0]); verifier-human kappa(a2)=0.0 (CI [0.0, 0.0])
- first_error: human-human kappa=0.0 (CI [0.0, 0.0]); agreement=0.667; verifier-human kappa(a1)=0.0 (CI [0.0, 0.0]); verifier-human kappa(a2)=1.0 (CI [1.0, 1.0])
- carryforward: human-human kappa=1.0 (CI [1.0, 1.0]) - perfect agreement (kappa undefined - zero marginal variance); agreement=1.0; verifier-human kappa(a1)=1.0 (CI [1.0, 1.0]); verifier-human kappa(a2)=1.0 (CI [1.0, 1.0])
- terminal disagreements (n=12): a1=1 vs a2=0; verifier=1
- first_error disagreements (n=12): a1=1 vs a2=0; verifier=0
  - on the 12 Counterfactual_Near_Miss trajectories: rubric tension over whether binding/near-miss
    faults imply terminal=0 (verifier terminal_consistency may still pass)

## Defensible claim (per study design document sec 5.4 / 5.5)

> Gate 1 currently supports only: an independent mechanical verifier can distinguish a complete
> Successful_Path from a **known** carry-forward mutation.
> The pass-rate gap must not be interpreted as "the full generation method beats an independent
> baseline generator".
> Gate 2 / Gate 3 separately test surface invariance and main-method batch reproducibility.

**Must not claim:** the data already lets a model overcome the semantic shortcut; or that the
state-persistence design has improved generator reliability (a separate generation-layer ablation
would be required).
