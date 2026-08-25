# Human Kappa Annotation Guide (Annotator Edition, Double-Blind) - v1 Cohort

> English rendering prepared for public release.
> This file contains no record-level example answers, trajectory types, or machine verdicts.
> The analyst's companion is `ANALYST_APPENDIX.md` (do not open before blind coding is complete).

---

## Who may see what

| File | Annotators |
|------|------------|
| `human_annotation/packets/*.json` | yes - the only input |
| `human_annotation/human_annotations.csv` | yes - fill only **a1_*** / **a2_*** columns |
| `ANNOTATOR_GUIDE.md` (this file) | yes |
| `stratification_manifest.json` | no |
| `ANALYST_APPENDIX.md` | no (not even after coding, unless you are the analyst) |
| the **v_*** columns in the CSV | no |
| `data/generated.jsonl` / `verification/verifier_results.jsonl` | no |

---

## Three dimensions (uniform 0/1 scale)

**`1` = the dimension holds structurally (no violation)**
**`0` = the dimension has a structural violation**

### `terminal`
Check step by step: `completion` / `terminal` versus `next_state.pending_constraints` and
`invariant_check.passed`:

- **1**: no "pending non-empty yet terminal claimed"; no "should be complete but not terminal"
  (except legitimate S5/HARD_STOP exceptions)
- **0**: any step violates the rules above

### `carryforward`
From **step 2 onward**, check step by step:

```text
step[i].state.resolved_constraints  ==  step[i-1].next_state.resolved_constraints
```

- **1**: consistent throughout
- **0**: any step where resolved is wiped, jumps, or under-inherits

### `first_error`
After reading the **entire** trajectory:

- **1**: **no** structural error of any kind found (terminal / carryforward / action-binding / etc.)
- **0**: **at least one** structural error found (note the step for yourself only; do **not** write
  it into the CSV)

Do not guess whether a record "should" pass or fail - judge only from the steps fields.

---

## Procedure

1. Open `human_annotation/packets/<packet_id>.json` (only `packet_id`, `horizon`, `surface_slot`,
   `steps`)
2. Review under the three rules above
3. In the CSV, fill `a1_terminal`, `a1_first_error`, `a1_carryforward` on the matching row
   (values `0`/`1`)
4. The second annotator independently fills `a2_*` (never seeing your a1 values)
5. When both are done, hand over to the analyst for kappa computation (not an annotator task)

---

## Universal review checklist (apply to every packet)

```text
[ ] Step 1: state.resolved empty [] (or consistent with the initial state)?
[ ] Steps 2..N-1: completion does not falsely terminate? resolved inherited?
[ ] Last step: terminal consistent with pending=[]?
[ ] Whole trajectory: any binding / orphan / other structural contradiction?
-> No problems:        terminal=1, carryforward=1, first_error=1
-> Only terminal bad:  terminal=0, others as observed
-> Only carry bad:     carryforward=0, usually first_error=0
-> Other structural error only: first_error=0, other two as observed
```

---

## CSV format

```csv
record_id,a1_terminal,a2_terminal,v_terminal,a1_first_error,a2_first_error,v_first_error,a1_carryforward,a2_carryforward,v_carryforward
MAIN-xxxx,1,,,1,,,1,,
```

- Fill only **a1_*** (or a2_*); leave **v_*** empty
- Values may be `0`/`1`, `yes`/`no`, or `pass`/`fail`

---

## Single-annotator note

You may complete all `a1_*` first. **Human-human kappa requires a second person filling a2_***; one
person filling both columns cannot count as double-blind kappa.
