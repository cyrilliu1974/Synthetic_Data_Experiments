# Spec-Drift Diagnosis - MechanicalVerifier Low Pass Rate (Phase-C Frozen 294)

> Original: diagnostic report dated 2026-08-25 ("analysis only, no code changes in this step").
> English rendering prepared for this public release; conclusions were later confirmed by the P0/P0.5
> fixes described in `IMPLEMENTATION_LOG_P0_P0.5.md`. Source-code anchors have been adapted to this
> repository's `code/` layout.

---

## 1. One-sentence conclusion

> The 236/294 failures are not a binary choice between "the verifier is too strict" or "all data is
> bad"; they are two stacked layers: **(A)** specification drift between `MechanicalVerifier` and the
> `/main` generator side (including `validator_agent`) over whether `APPLY_LOCAL_UPDATE` is a legal
> S2 action - legal on one side, illegal on the other (`S_TOKEN_BINDING_VIOLATION`, 184 records);
> plus **(B)** the expected failures of `Counterfactual_Near_Miss` and `baseline` cohorts.

- The overall **58/294 pass (19.7%)** does **not by itself violate** the Gate-1 design intent of the
  study design document (sec 4.2: the point is meaningful discrimination, not 100%).
- What genuinely conflicts with the research design: under `main`, `medical`/`financial`
  Successful_Path records (which should be gold positives) went **0/24**, while `legal` went
  **24/24** - fully explained by presence vs absence of `TECH-03 / APPLY_LOCAL_UPDATE`.

---

## 2. Position of the MechanicalVerifier in the study design

| Design doc location | Point |
|---|---|
| sec 1.3 / 4.1 | Phase-C's core question: "did the generator really produce the trajectory structure it claims?" |
| sec 3.5 / 3.5.2 | The verifier is a **rule-based auditor independent of the generator**, deciding `Valid(tau) |= Specification`, not `Match(tau, tau_teacher)` |
| sec 3.5.1 | Three core invariants: **Carry-Forward, Canary Consumption, Terminal Consistency**; Carry-Forward is the most central mechanical invariant |
| sec 4.2 L1005-1007 | **Gate 1 does not pursue 100% pass**; it requires "meaningful discrimination" |
| sec 3.5 L858 | **rule-based != automatically correct**; verifiers can have mis-written rules, spec gaps, insufficient coverage |

> The verifier therefore **should be strict** - strict in being faithful to the specification,
> mechanically reproducible, and able to separate legal from illegal transitions - not strict or
> lax for the sake of moving the pass rate.

---

## 3. Full-data recheck (analysis outputs)

- **Total**: 294 (`data/generated.jsonl` = 294, verification results = 294)
- Total pass: 58; total fail: 236
- failure_category: `FIELD_ERROR` 236, `HARD_PROHIBITION` 0 (no content-level SC_* rules triggered)
- violated_rule: `S_TOKEN_BINDING_VIOLATION` 184 (77.9%), `SV_CARRY_FORWARD_VIOLATION` 52 (22.1%)
- first failure_step: step 2 = 160, step 4 = 60, step 3 = 16

### 3.1 Stratified breakdown (domain x trajectory x condition)

| domain | trajectory | condition | pass / total |
|---|---:|---|---:|
| financial | Counterfactual_Near_Miss | main | 0 / 24 |
| financial | Repair_Path | main | 0 / 24 |
| financial | Successful_Path | baseline | 0 / 18 |
| financial | Successful_Path | main | **0 / 24** |
| financial | Successful_Path | metamorphic | 0 / 8 |
| medical | Counterfactual_Near_Miss | main | 0 / 24 |
| medical | Repair_Path | main | 0 / 24 |
| medical | Successful_Path | baseline | 0 / 18 |
| medical | Successful_Path | main | **0 / 24** |
| medical | Successful_Path | metamorphic | 0 / 8 |
| legal | Counterfactual_Near_Miss | main | 0 / 24 |
| legal | Repair_Path | main | **24 / 24** |
| legal | Successful_Path | baseline | 2 / 18 |
| legal | Successful_Path | main | **24 / 24** |
| legal | Successful_Path | metamorphic | 8 / 8 |

**Key observations:**
- All **58 passing** records carry exactly one surface label: `legal:slot0#TECH-04.TECH-08`
  (58 = 24 + 24 + 8 + 2).
- Failing records are concentrated under the two TECH-03 surfaces
  (`medical:slot0#TECH-03.TECH-04.TECH-08.TECH-12`, 98 records;
  `financial:slot0#TECH-03.TECH-04.TECH-09.TECH-12`, 98 records), plus baseline distortions of the
  legal surface.

### 3.2 Two subsets whose low pass is *by design*

| Subset | Phenomenon | Justified per design doc? |
|---|---|---|
| `Counterfactual_Near_Miss` 72 records (3 domains x 24), all fail | Expected counter-signal / near-miss | **Yes** |
| `baseline` 54 records (format-matched Successful_Path + injected carry-forward break) | Only 2/54 pass (3.7%) | **Yes** - this is Gate 1's **verifier sensitivity control**, not a generator comparison |

> After excluding these two classes, the "unjustifiably low pass" concentrates in the
> `main / Successful_Path` and `main / Repair_Path` **medical / financial** branches.

---

## 4. Root-cause decomposition: why medical/financial failed completely

### Cause A - Spec drift: three-way inconsistency over `APPLY_LOCAL_UPDATE` (primary cause; 184 S_TOKEN binding failures)

**Trajectory-shape contrast:**

- **Legal passing example** `MAIN-0001` (`TECH-04.TECH-08`, 5 steps):
  `CHECK_INITIAL_STATE -> RESOLVE_* -> RESOLVE_* -> RESOLVE_* -> TERMINAL_DERIVED` -> **pass**
- **Medical failing example** `MAIN-0080` (`TECH-03.*`, 7 steps):
  `CHECK_INITIAL_STATE -> APPLY_LOCAL_UPDATE(status) -> RESOLVE_* x4 -> TERMINAL_DERIVED` ->
  **step 2 flagged S_TOKEN_BINDING_VIOLATION**

The three components' positions on legal S2 actions:

| Component | Does S2 allow `APPLY_LOCAL_UPDATE`? | Basis |
|---|---|---|
| `code/failure_distribution/mechanical_verifier.py` `STOKEN_ALLOWED_ACTIONS` | **No** (only `DERIVE / APPLY_L2_SCAFFOLD / CHECK_INITIAL_STATE / TERMINAL_DERIVED`; `RESOLVE_*` grandfathered) | sec 2.6 binding table |
| `validator_agent.py` | **Yes** (and Variation_G missing it must raise `SV_VARIATION_G_SELECTIVE_UPDATE_MISSING`) | Variation_G must contain a ledger-preserving `APPLY_LOCAL_UPDATE` |
| `stepwise_builder.py` | **Yes** (explicitly generates the `APPLY_LOCAL_UPDATE|status` transition) | Variation_G selective-update scenario |

Under the design doc's Anti-Shortcut / Variation_G concept, "update a local version without breaking
the dependency ledger" is a legitimate research scenario; the builder indeed produces such
trajectories, but the Phase-C `MechanicalVerifier` specification never added that action to its
binding table, so:

> generator considers it legal -> `/main TERMINAL_COMPLETE` -> Phase-C verifier flags
> S_TOKEN_BINDING_VIOLATION @ step 2

This is **insufficient spec coverage / unaligned validator pair** (anticipated by design doc sec
3.5) - not "TECH-03 trajectories should never exist".

**Quantification:** `APPLY_LOCAL_UPDATE` appears only in the two `TECH-03.*` surfaces (medical /
financial, 98 each = 196). Of these, 184 were intercepted at `S_TOKEN_BINDING_VIOLATION @ step 2`;
the remaining 12 were intercepted earlier by other rules with the same root cause.

### Cause B - `SV_CARRY_FORWARD_VIOLATION` (true defects, 52 records)

Design doc sec 3.5.1 defines **Carry-Forward** as the most central invariant;
`_check_carry_forward` checks `step[n].state.resolved == step[n-1].next_state.resolved`.

- **16/18 legal-baseline failures** are **deliberate ablation** (`repair_enforcement` skipping 4B) -
  part of the Gate-1 control design, **not a bug**.
- The remaining 36 (including some main step-3/4 breaks) are genuine **constraint-binding /
  state-persistence failures** when `RESOLVE_*` chains fail to inherit the previous `resolved` -
  exactly the semantic-shortcut mechanism the design targets (persistence layer).

> **Conclusion:** `S_TOKEN` failures are **spec-drift false kills**; `SV_CARRY_FORWARD` failures are
> **true defect detections demanded by the design**. The two must not be conflated.

---

## 5. Is the existing validator standard reasonable?

### Reasonable parts (consistent with the design doc)

1. **Strict Carry-Forward** - should be strict; it operationalizes the paper's central proposition.
2. **S-Token binding + `RESOLVE_*` grandfathering** - reasonable as the `/main` stepwise encoding of
   `DERIVE`.
3. **No 100%-pass requirement** - counterfactual/baseline are supposed to fail; matches Gate 1.
4. **Independent verifier separated from the generator** - correct direction.

### Unreasonable parts (relative to the full specification)

1. **`APPLY_LOCAL_UPDATE` absent from the Phase-C verifier** while builder + internal validator
   implement Variation_G - **spec drift**.
2. **The spec document's sec 2.6 binding table itself omits full semantics for
   `APPLY_LOCAL_UPDATE`** - spec text, internal validator, and mechanical verifier were never
   synchronized.
3. **A single overall pass rate (58/294 = 19.7%) has no explanatory power** - stratified reporting
   is mandatory (see sec 6 below and the roadmap).

---

## 6. Re-reading Gates 1/2/3 (avoiding wrong claims)

- **Gate 1**: the reported `main 0.333 vs control 0.037, RD 0.296, McNemar p=0.0` measures
  **verifier sensitivity to a known carry-forward mutation** and must **not** be read as "full TGAA
  beats an independent baseline" (explicitly forbidden; `seed_mismatch 54/54`).
- **Gate 2**: agreement `1.0 [0.757,1.0]` looks perfect, but only the 8 legal metamorphic pairs
  passed mechanically; medical/financial pairs "consistently fail" due to the same drift -
  **high agreement with low pass**; interpret separately from Gate 1.
- **Gate 3**: max JS 0.0 arises because all batches' failure distributions are dominated by the same
  drift - this is **reproducibility of the drift**, not verified generation stability.
- **Mutation probe**: pre-fix `false_terminal sensitivity 0.0` plus surface-leakage LOO 0.721
  (financial/medical 1.0, legal 0.163) indicated residual surface-pattern leakage versus the
  cross-domain counterbalancing goal (later traced to a probe-targeting bug; see RCA doc).

---

## 7. Cross-check against the external long-text diagnosis

| External judgment | Data verdict | Aligned? |
|---|---|---|
| Low pass rate mainly reflects imprecise validator standard -> actually spec drift + some true defects | 184 spec-drift + 52 true defects + 72 expected near-miss = stacked layers | Yes |
| medical/financial Successful_Path wipeout is implausible | 0/24 in both domains, conflicting with gold expectation | Yes |
| legal Successful_Path 24/24 pass is plausible | observed 24/24 | Yes |
| Counterfactual/baseline low pass is plausible | expected-failure structure of 72 + 52 | Yes |
| Variation_G / APPLY_LOCAL_UPDATE three-way inconsistency | located with file/line anchors | Yes |
| Carry-Forward is the core invariant; its failures are true defects | design doc sec 3.5.1 + 52 observed | Yes |

---

## 8. Fix priorities (recommendations at the time; since executed)

### P0 - Unify the specification (precondition of a self-consistent cohort)

- **Decision required**: is `APPLY_LOCAL_UPDATE` a legal S2 action under Variation_G?
  - If **yes** (recommended): write it into the spec binding table and
    `STOKEN_ALLOWED_ACTIONS["S2_SATISFIED"]`, synchronizing TC_* boundary conditions (no APPLY after
    S1/S5).
  - If **no**: rewrite the builder's Variation_G path to stop emitting the action.
- **Risk if ignored**: any total pass rate remains unusable for paper claims.
- *(Status: resolved as "yes"; implemented - see implementation log.)*

### P1 - Add binding-table regression tests

- Single code change point: `STOKEN_ALLOWED_ACTIONS` in `code/failure_distribution/mechanical_verifier.py`.
- Regression: legal Variation_G passes; Variation_G missing `APPLY_LOCAL_UPDATE` fails with
  `SV_VARIATION_G_SELECTIVE_UPDATE_MISSING` (or equivalent), not `S_TOKEN_BINDING`.
- **Risk**: relaxing the table without tests lets future drift happen silently again.

### P2 - Report Gate 1 stratified (paper-writing fix)

- Report pass rates by `condition x trajectory_type x domain x surface`; never headline 58/294.
- Separate tables: main Successful_Path (gold) vs counterfactual vs baseline.

### P3 - Track true defects

- Repair builder-side ledger inheritance behind the 52 `SV_CARRY_FORWARD_VIOLATION` cases
  (especially `RESOLVE_*` chains after `APPLY_LOCAL_UPDATE`).
- Post-fix re-verification showed no remaining carry-forward failures on `main` records; see the
  implementation log.

---

## 9. Non-claimable boundary (design doc sec 5.4, epistemic boundary)

- **Must not claim**: models already overcome the semantic shortcut; state-persistence design has
  improved generator reliability (requires a generation-layer ablation).
- **May claim**: Phase-C verifies the verifier distinguishes complete Successful_Path from a known
  carry-forward mutation; the data **conditionally** provides structural counter-signal against the
  semantic shortcut (subject to counterbalancing and hold-out validation).

---

## 10. Appendix - data fingerprint (paths adapted to this repository)

- `cohort_v1/verification/verifier_results.jsonl`: 294 rows, `passed=True 58 / False 236` (as first
  run, pre-fix)
- `cohort_v1/data/generated.jsonl`: 294 rows
- `cohort_v1/analysis/analysis_summary.json`: Gate1 `main 0.333 / control 0.037` (54 pairs), Gate2
  `1.0`, Gate3 `0.0`, mutation `0.667`, surface LOO `0.721` (first-run snapshot values)
- Key source anchors (this repository): `code/failure_distribution/mechanical_verifier.py`
  (S_TOKEN binding table and check with `RESOLVE_*` grandfathering);
  generator-side validator and stepwise builder implementing Variation_G with `APPLY_LOCAL_UPDATE`.
