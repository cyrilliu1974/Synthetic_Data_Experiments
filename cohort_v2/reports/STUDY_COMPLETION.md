# Phase-C v2 Study Completion (2026-08-25)

> English rendering prepared for public release; file references adapted to this repository's layout.

## Status

**Executed and closed.** Formal cohort `cohort_v2` (294/294, `frozen_spec_v2.json` in `specs/`).
v1 artifacts under `cohort_v1/` preserved.

Primary report: `ANALYSIS_REPORT.md` - Machine summary: `analysis/analysis_summary.json`

---

## Human annotations merged

- **36/36** rows: annotator 1 (`a1_*`) + annotator 2 (`a2_*`) + machine (`v_*`)
- File: `human_annotation/human_annotations.csv`
- kappa output: `analysis/human_agreement.json`
- Guides: `human_annotation/ANNOTATOR_GUIDE.md` (annotators) -
  `human_annotation/ANALYST_APPENDIX.md` (analyst)

| Dimension | human-human kappa | agreement | verifier-human (a1 / a2) | Notes |
|-----------|-------------------|-----------|--------------------------|-------|
| **carryforward** | **1.0** | 36/36 | 1.0 / 1.0 | Perfect; all sample rows structurally clean on carry |
| **first_error** | **0.0** | 24/36 | 0.0 / **1.0** | **12 CF disagreements**: a1=1, a2=0; verifier=0 - a2 aligns with machine |
| **terminal** | **0.0** | 24/36 | **1.0** / 0.0 | **12 CF disagreements**: a1=1, a2=0; verifier=1 - rubric tension on binding vs terminal fields |

**Gate for design doc sec 4.6 (kappa > 0.6 on all dimensions):** **not met** (terminal and
first_error human-human kappa below target).

**Defensible human-verifier claim:** annotator 2 + verifier agree on `first_error` (kappa=1.0);
carryforward unanimous.

---

## Automatic evidence (v2, post P0)

| Gate / probe | Result |
|--------------|--------|
| Gate 1 (paired ablation) | main SP pass **1.0** vs control **0.0**; McNemar b=54,c=0; **seed_mismatch 0/54** |
| Gate 2 | metamorphic agreement **1.0** (12/12 latent groups) |
| Gate 3 | max JS **0.0** (reproducibility across 3 main batches) |
| Mutation probe | sensitivity **1.0** (skip_prerequisite, false_terminal, corrupt_lineage) |
| Surface **narrative_only** LOO | **0.571 ~ chance** (0.5) |
| Surface surface_metadata LOO | 0.816 (metadata separable - not a "no shortcut" claim) |
| Surface action_token LOO | 0.429 ~ chance (task-structure vocabulary; not a generator defect) |
| main gold (SP + Repair) | **144/144** pass |
| main Counterfactual | **0/72** pass (expected - near-miss structural faults injected) |

---

## v1 vs v2 Gate 1 (primary design upgrade)

See `ANALYSIS_REPORT.md`, section "v1 appendix vs v2 primary Gate 1".

| | v1 | v2 |
|---|----|----|
| Baseline seed vs pair | 0/54 shared | **54/54 shared** |
| seed_mismatch red flag | 54/54 | **none** |
| main pass rate | 1.0 | 1.0 |
| control pass rate | 0.037 | **0.0** |
| paired risk difference | 0.963 | **1.0** |
| Interpretation | Post-hoc mutation sensitivity (caveat) | **Paired ablation sensitivity on matched slots** |

---

## Study conclusions (evidence-bound)

### What v2 supports

1. **State carry-forward in gold data:** Main Successful_Path + Repair trajectories (144/144)
   satisfy mechanical carry-forward and terminal checks; the paired ablation (Gate 1 v2) shows that
   disabling carry-forward yields **0/54** verifier passes on matched seeds.
2. **Step-level rule enforcement is auditable:** The verifier detects injected defects (mutation
   probe 1.0); counterfactual near-miss trajectories fail as designed (72/72); metamorphic A/B
   verdicts agree (Gate 2).
3. **Narrative semantic shortcut (partial):** `narrative_only` LOO ~ 0.57 - the domain is **not**
   trivially recoverable from stripped narrative alone. This supports *conditional* use of narrative
   fields for training **if** structural labels remain the admission gate.

### What v2 does **not** support (explicit non-claims)

1. **Downstream model learning:** No training run was executed. Evidence validates **dataset
   construction + mechanical verification**, not that a model trained on this JSONL will maintain
   state in deployment.
2. **Full shortcut elimination:** `surface_metadata` LOO ~ 0.82 - slot/domain metadata remains
   separable. "No semantic shortcut anywhere" is **not** supported.
3. **Generator superiority:** Baseline is a **known ablation control**, not an independent generator
   baseline.
4. **Perfect human-machine rubric alignment:** The terminal dimension on near-miss CF trajectories
   shows annotator disagreement and a verifier-human split (binding fault vs terminal field
   consistency).

### One-sentence answer (for paper abstract / discussion)

> Under pre-registered Phase-C checks, the synthetic pipeline **conditionally** produces structurally
> valid, carry-forward-consistent gold trajectories and a mechanical verifier that discriminates them
> from ablated controls and known mutations; **narrative-only** domain leakage is near chance, but
> **this study does not demonstrate** that training on the data will teach a model long-horizon state
> tracking - that requires a separate training ablation.

---

## Claim boundary (v2 complete)

**Can claim (v2 primary):** Paired Gate 1 ablation; verifier sensitivity; metamorphic invariance;
batch reproducibility; narrative-only probe ~ chance; main gold structural validity (SP/Repair
144/144).

**Retain v1 as appendix:** Verifier sensitivity under the seed-mismatch design; v1 human
`first_error` kappa = 1.0.

**Cannot claim:** kappa > 0.6 on all human dimensions; model learned state persistence; zero
semantic shortcut; independent-generator superiority.
