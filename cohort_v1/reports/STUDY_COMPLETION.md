# Phase-C v1 Study Completion (2026-08-25)

> **Note:** English rendering of the original study-completion document, prepared for this public
> release. File references have been adapted to this repository's layout (`automatic_checks/`
> artifacts appear under `analysis/` here).

## Human annotations merged

- **36/36** rows: annotator 1 (`a1_*`) + annotator 2 (`a2_*`) + machine (`v_*`)
- File: `human_annotation/human_annotations.csv`
- kappa output: `analysis/human_agreement.json`

## Human agreement summary

| Dimension | human-human kappa | agreement | verifier-human (a1 / a2) | Notes |
|-----------|-------------------|-----------|--------------------------|-------|
| **first_error** | **1.0** | 36/36 | 1.0 / 1.0 | Both annotators align with verifier structural pass/fail |
| **carryforward** | **1.0** | 36/36 | perfect | All rows = 1 (no carry breaks in sample) |
| **terminal** | **0.0** | 24/36 | a1=0.0 vs v; a2~1.0 vs v | **12 disagreements**: a1=0, a2=1 on near-miss trajectories |

### Terminal disagreement (substantive)

On 12 structurally-invalid trajectories (near-miss), **annotator 1** marked `terminal=0` while
**annotator 2** marked `terminal=1`. Verifier `v_terminal=1` (terminal *field* consistency passes;
binding fault is separate).

**Interpretation:** Rubric ambiguity - whether a binding violation implies `terminal=0` even when
`completion`/`pending` fields are internally consistent. Recommend a Discussion footnote; an optional
rubric v2 would clarify terminal vs binding.

**Gate for design document sec 4.6:** `first_error` kappa > 0.6 **met**; overall multi-dimension
kappa target **not met** due to the terminal rubric split.

---

## Automatic evidence (v1, post P0)

| Gate / probe | Result |
|--------------|--------|
| Gate 1 | main SP 1.0 vs control 0.037; seed_mismatch 54/54 (v1 design) |
| Gate 2 | metamorphic agreement 1.0 |
| Gate 3 | JS = 0 (reproducibility) |
| Mutation probe | sensitivity 1.0 |
| Surface narrative_only LOO | **0.578 ~ chance** |
| main gold (SP+Repair) | 144/144 pass after P0 spec sync |

---

## v2 (executed - see primary completion doc)

The formal v2 cohort was completed 2026-08-25. Primary Gate 1 + study conclusions:
`../cohort_v2/reports/STUDY_COMPLETION.md` and `../cohort_v2/reports/ANALYSIS_REPORT.md`.

---

## Claim boundary (v1 complete)

**Can claim:** Structural validity pipeline; verifier sensitivity; metamorphic invariance;
human-verifier agreement on **overall structural error** (first_error); narrative surface probe ~
chance.

**Cannot claim:** kappa > 0.6 on all dimensions; paired baseline ablation (that is v2); generator
superiority.
