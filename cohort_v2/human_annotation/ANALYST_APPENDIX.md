# Human Kappa Annotation Guide (Analyst Appendix with Worked Examples) - v2 Cohort

> English rendering prepared for public release.
> **Cohort:** v2 formal rerun (`cohort_v2` in this repository).
> **Do not distribute to annotators before blind coding is complete.**
> Annotators should read only `ANNOTATOR_GUIDE.md` plus their blind packets.

This appendix contains: stratified manifest, worked examples, expected `v_*` values (after
populate-verifier), and the analysis workflow.

---

## Relationship to v1

| Item | v1 | v2 |
|------|----|----|
| kappa sample record ids | 36 records | same 36 ids |
| Generated text | `cohort_v1/` | **fresh run** `cohort_v2/` |
| May v1 human columns be reused? | - | **No** - independent double-blind recoding required |
| Scale / three dimensions | same | same |
| Verifier rules | post-P0 sync | same |

The v2 design change (paired baseline seeds) does **not** affect the kappa protocol; kappa still
tests whether human structural judgment agrees with the MechanicalVerifier on **this cohort's**
artifacts.

---

## The 36-record stratification (manifest)

See `stratification_manifest.json`: 3 domains x 3 trajectory types x 4 records.

| Trajectory type | Expected first_error distribution (structural validity) |
|-----------------|----------------------------------------------------------|
| Successful_Path | mostly **a1_first_error=1** (no structural errors) |
| Repair_Path | mostly **1** |
| Counterfactual_Near_Miss | mostly **a1_first_error=0** (contains near-miss structural faults) |

Annotators must **not** see the table in advance; kappa measures whether blind judgments match the
structural facts.

---

## Worked example A - gold positive (v2 sample contains `MAIN-0004` and other SP records)

**Type (analyst knows):** Successful_Path
**Blind-reading cues:** resolved accumulates stepwise; terminal + pending=[] only at the last step

**v2 verifier (checkable before populate):** `MAIN-0004` -> `passed=true`

**Expected annotator a1 shape:**

```csv
MAIN-0004,1,,,1,,,1,,
```

| Dimension | Value | Reason |
|------|-----|------|
| terminal | 1 | no premature termination; final step legal |
| carryforward | 1 | each step == previous next.resolved |
| first_error | 1 | no structural errors anywhere |

**After populate-verifier, v_*:** `1,1,1`

---

## Worked example B - near-miss structural fault (v2 sample contains `MAIN-0032` and other CF records)

**Type (analyst knows):** Counterfactual_Near_Miss
**Blind-reading cues:** around some step (commonly near step 4) an action/binding contradicts state;
terminal/carry may still hold individually

**v2 verifier:** `MAIN-0032` -> `passed=false`, `violated_rule=S_TOKEN_BINDING_VIOLATION`,
`failure_step=4`

**Expected annotator a1 shape:**

```csv
MAIN-0032,1,,,0,,,1,,
```

| Dimension | Value | Reason |
|------|-----|------|
| terminal | 1 | termination fields may still be internally consistent |
| carryforward | 1 | inheritance may still hold |
| first_error | **0** | **a structural error was found** |

**After populate-verifier, v_*:** `v_terminal=1`, `v_first_error=0`, `v_carryforward=1`

---

## Semantics of v_* (populate-verifier)

Same scale as human coders:

```python
v_terminal = terminal_consistency.passed
v_carryforward = carry_forward.passed
v_first_error = int(verifier.passed)  # 0 = rejected at any structural check
```

Expected for the v2 cohort (36-record kappa sample): **12 records v_first_error=0**, **24 records
v_first_error=1** (same distribution as v1; not all 36 = 1).

---

## Analyst workflow

### 1. Preparation (after the formal run)

```bash
# initialize / export blind packets if not yet done
python code/experiments/phase_c_validation/code/human_io.py --init --run-dir <run_dir>
python code/experiments/phase_c_validation/code/human_io.py --export-packets --run-dir <run_dir>
```

Hand the **annotator edition** guide + packets + blank CSV to two annotators (**never** the manifest
or this appendix).

### 2. After human coding is complete

```bash
# fill v_* (analyst only; annotators already finished a1/a2)
python code/experiments/phase_c_validation/code/human_io.py --populate-verifier --run-dir <run_dir>

# compute kappa (human-human + human-verifier)
python code/experiments/phase_c_validation/code/human_io.py --run-dir <run_dir>
```

Output: agreement JSON (`human_agreement.json`; stored under `analysis/` in this repository).

### 3. Reporting

Cite the kappa table in the analysis report. Keep each run's CSV separate - never merge across run
directories.

---

## v1 comparison (for appendix use, not kappa reuse)

v1 completed kappa (`cohort_v1/analysis/human_agreement.json`):

- `first_error` human-human kappa = 1.0
- `carryforward` kappa = 1.0
- `terminal` kappa = 0.0 (12 CF records: human terminal=0 vs verifier v_terminal=1 rubric tension)

Report v2 separately after recoding. If terminal disagreement recurs, discuss it as a **rubric
boundary case** in the paper, not as a verifier bug.
