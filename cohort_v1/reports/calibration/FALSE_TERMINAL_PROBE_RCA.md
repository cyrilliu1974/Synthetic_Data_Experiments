# P0.5 Root-Cause Analysis - `false_terminal` Mutation Sensitivity = 0.0

**Date:** 2026-08-25
**Scope:** v1 frozen-294 cohort (`cohort_v1` in this repository)
English rendering prepared for public release; content unchanged, code paths adapted to `code/`.

## Symptom

`analysis/mutation_probe.json` reported `sensitivity_by_kind.false_terminal = 0.0` with
`n_skipped = 0` - injections ran but the verifier never rejected.

## Root cause (confirmed)

**Primary: probe injection bug (a), not verifier invariant gap (b).**

Formal `/main` trajectories use a **step-level completion dict** only on the **final** step
(`completion.terminal = true`). Mid steps have `completion = null`.

`inject_defect(..., "false_terminal")` (pre-fix) picked `s = randint(0, len(steps)-2)` and either:

1. Set `next_state.completion = true` on mid steps - ignored by the verifier's dict-contract early
   branch (only steps with `completion.terminal=true` are checked), or
2. Re-toggled `terminal=true` on the **legitimate final step** where `next_state.pending` is already
   `[]` - verifier correctly passes (`not pending and inv_passed -> continue`).

Manual confirmation on `MAIN-0001`:

| Step | `completion` | `next_state.pending` |
|------|--------------|----------------------|
| 1-4  | `null`       | non-empty            |
| 5    | `{terminal: true}` | `[]`           |

Injecting `{terminal: true}` on step 2 with pending non-empty -> **rejected**
(`SV_TERMINAL_CONSISTENCY`). Verifier behavior is correct when the defect hits the right contract.

## Fix

`offline_generator.inject_defect` (`false_terminal`):

- Candidate steps = non-final steps with **non-empty** `next_state.pending_constraints`
- Set the formal `/main` terminal triple on `step.completion` (create dict if absent)

## Post-fix validation

Re-run on the v1 cohort passes (after the P0 `APPLY_LOCAL_UPDATE` sync):

```
false_terminal: rejected=144/144, still_passed=0
```

Verifier `_check_terminal_consistency` unchanged - no dual-contract widening needed.

## Claim boundary

Pre-fix `false_terminal=0` must **not** be cited as "terminal invariant broken". It was a
**mutation-probe targeting bug**. Re-run the mutation probe after any fix before reporting
sensitivity.
