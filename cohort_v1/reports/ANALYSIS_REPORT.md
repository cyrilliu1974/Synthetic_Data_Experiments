# Phase C Validation - Analysis Report (v1 appendix)

> **Note:** English rendering of the original analysis report, prepared for this public release.
> Interpretive strings that were originally non-English have been translated; all numeric fields
> are unchanged. File references adapted to this repository's layout.

Total generated: 294 records (216 main + 54 paired baseline + 24 metamorphic).

## Gate 1 - Verifier sensitivity to a known carry-forward mutation

This table is **not** evidence that the full generator outperforms an independent baseline
generator. The control is a format-matched Successful_Path with a post-hoc carry-forward mutation.

- n_pairs = 54
- design_role = verifier_sensitivity_control
- claim_forbidden = Do not report this as full-method superiority over an independent generator baseline.
- main pass rate = 1.0 (95% CI [0.934, 1.0])
- control pass rate = 0.037 (95% CI [0.01, 0.125])
- paired risk difference = 0.963 (95% CI [0.907, 1.0])
- McNemar: b=52, c=0, p=0.0
- seed compared/mismatch = 54/54
- red flags: ['seed_mismatch (54/54 pairs; not the same frozen slot)']

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
- per held-out domain accuracy = {'financial': 0.816, 'legal': 0.837, 'medical': 0.816}
- overall LOO accuracy = 0.823 (chance = 0.5)
- claim_use = metadata separability; not semantic shortcut alone

### action_token
- description = decision.action tokens only - domain action vocabulary separability
- per held-out domain accuracy = {'financial': 0.673, 'legal': 0.837, 'medical': 0.816}
- overall LOO accuracy = 0.776 (chance = 0.5)
- claim_use = task-structure signal; high LOO expected for TECH-03 vs TECH-04 paths - NOT a generator defect

### narrative_only
- description = content_output + rationale with action prefixes stripped
- per held-out domain accuracy = {'financial': 0.571, 'legal': 0.592, 'medical': 0.571}
- overall LOO accuracy = 0.578 (chance = 0.5)
- claim_use = narrative shortcut test - closest to the sec 3.6 semantic-shortcut concern

> Probe action_token near 1.0 indicates separable domain action vocabularies, a normal
> task-structure effect; only probe narrative_only near 0.5 supports a no-narrative-shortcut claim.

## Human double-blind agreement (36 records, double-blind)

- terminal: human-human kappa=0.0 (CI [0.0, 0.0]); agreement=0.667; verifier-human kappa(a1)=0.0 (CI [0.0, 0.0]); verifier-human kappa(a2)=1.0 (CI [1.0, 1.0])
- first_error: human-human kappa=1.0 (CI [1.0, 1.0]); agreement=1.0; verifier-human kappa(a1)=1.0 (CI [1.0, 1.0]); verifier-human kappa(a2)=1.0 (CI [1.0, 1.0])
- carryforward: human-human kappa=1.0 (CI [1.0, 1.0]) - perfect agreement (kappa undefined - zero marginal variance); agreement=1.0; verifier-human kappa(a1)=1.0 (CI [1.0, 1.0]); verifier-human kappa(a2)=1.0 (CI [1.0, 1.0])
- terminal disagreements (n=12): a1=0 vs a2=1 on 12 near-miss trajectories - rubric ambiguity on whether a binding fault implies terminal invalidity

## Defensible claim (per study design document sec 5.4 / 5.5)

> Gate 1 currently supports only: an independent mechanical verifier can distinguish a complete
> Successful_Path from a **known** carry-forward mutation.
> The pass-rate gap must not be interpreted as "the full generation method beats an independent
> baseline generator".
> Gate 2 / Gate 3 separately test surface invariance and main-method batch reproducibility.

**Must not claim:** the data already lets a model overcome the semantic shortcut; or that the
state-persistence design has improved generator reliability (a separate generation-layer ablation
would be required).
