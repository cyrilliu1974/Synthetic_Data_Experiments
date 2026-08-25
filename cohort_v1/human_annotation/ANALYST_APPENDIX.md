# Human Kappa Annotation Guide (Analyst Appendix with Worked Examples) - v1 Cohort

> English rendering prepared for public release.
> **Do not distribute to annotators before blind coding is complete.**
> Annotators should read only `ANNOTATOR_GUIDE.md` plus their blind packets.

This appendix contains: stratified manifest explanation, worked examples, expected `v_*` values
(after populate-verifier).

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

## Worked example A - gold positive (sample contains `MAIN-0004` and other SP records)

**Type (analyst knows):** Successful_Path
**Blind-reading cues:** resolved accumulates stepwise; terminal + pending=[] only at the last step

**Annotator a1:**

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

## Worked example B - near-miss structural fault (sample contains `MAIN-0032` and other CF records)

**Type (analyst knows):** Counterfactual_Near_Miss
**Blind-reading cues:** around some step (commonly near step 4) an action/binding contradicts state;
terminal/carry may still hold individually

**Annotator a1:**

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

Expected for this cohort: **12 records v_first_error=0**, **24 records v_first_error=1**
(not all 36 = 1).

---

## Analyst commands (original workspace layout)

```bash
python code/experiments/phase_c_validation/code/human_io.py --populate-verifier --run-dir <run_dir>
python code/experiments/phase_c_validation/code/human_io.py --run-dir <run_dir>
```

Output: agreement JSON (`human_agreement.json`). In this repository, analysis outputs are stored
under `analysis/`.
