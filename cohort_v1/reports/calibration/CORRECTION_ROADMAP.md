# Publishable Correction Roadmap (Phase-C Frozen 294, v2 proposal)

> Original: roadmap document dated 2026-08-25. English rendering prepared for public release.
> **Editor's note (release):** every decision itemized in section 8 was subsequently approved and
> executed - the spec was unified (P0), baseline seed pairing was fixed (Q2 = yes), double-blind
> kappa was run (Q3 = yes), and the full pre-registered v2 rerun was completed (Q4). Evidence:
> `../cohort_v2/reports/PREREGISTRATION.md`, `../cohort_v2/reports/STUDY_COMPLETION.md`, and the
> implementation log in this directory.

---

## 0. Key corrections relative to the v1 (superseded) proposal

| v1 proposal (deprecated) | v2 roadmap (this document) |
|---|---|
| Only "re-verify the 294", no regeneration | The existing 294 must not be the paper's primary cohort; after spec unification, rerun a complete v2 under a new run-id |
| Baseline seed mismatch only annotated as a red flag | Baseline must share seed + constraint graph with its `pair_id` to enter the paper |
| Human kappa not emphasized | Double-blind kappa is a **necessary** component of complete Phase-C evidence (design doc sec 4.6) |
| Gate 2/3 merely restated | Gate 2 agreement computed only on the mechanically-passing subset; Gate 3 JS computed on meaningful failure distributions |
| Claim boundary loosely written | Explicit split: Level 1-2 claimable now / Level 3+ requires a later phase |

> **One sentence:** the existing `58/294` should not be "interpreted" - it should be superseded:
> unify the `APPLY_LOCAL_UPDATE` and carry-forward specification, repair the builder, rerun the
> formal cohort under a new run-id, and use **stratified Gate 1 + double-blind kappa** as the only
> structural-validity evidence chain.

---

## 1. Position of the existing 294 (honest negative result, not primary data)

- The existing 294 records were produced under three inconsistent specifications
  (generator / internal validator / MechanicalVerifier): medical/financial gold was declared complete
  by `/main` but failed Phase-C.
- **Paper position:** appendix / "spec-drift discovery pilot" - showing how the formal pipeline
  exposed specification inconsistency. An honestly reportable negative result; **not** primary
  experimental data.
- The v1 `generated.jsonl` is **not patched**, avoiding local-fix contamination of cohort audits.

---

## 2. P0 - Specification unification decision (unavoidable prerequisite)

**Decision:** `APPLY_LOCAL_UPDATE` is a legal S2 action under Variation_G (consistent with the
Anti-Shortcut / selective-update concept; builder already implements it ledger-preserving).
**Recommended answer: yes.**

One version bump, one PR, three places synchronized:

| File | Action |
|---|---|
| Spec document sec 2.6 S_TOKEN binding table | Add `APPLY_LOCAL_UPDATE` (Variation_G / ledger-preserving only) with boundary conditions |
| `mechanical_verifier.py` `STOKEN_ALLOWED_ACTIONS["S2_SATISFIED"]` | Add `APPLY_LOCAL_UPDATE` |
| Regression tests | Variation_G with APPLY -> pass; missing APPLY -> `SV_VARIATION_G_*` fail (not `S_TOKEN`) |

**Boundary conditions written into the spec** (guarding against "loosening" criticism):
- Legal only around `execution_phase=E1_LOCAL_UPDATE_APPLIED` with `mutable_facts` /
  `state_update_log` present
- No `APPLY_LOCAL_UPDATE` after S1/S5
- `active_constraints / pending / dependencies / resolved` must not be rewritten in that step
  (builder already raises `SV_VARIATION_G_NONLOCAL_MUTATION`; verifier mirrors it)

---

## 3. Generation method: how to produce a publishable v2 cohort

### 3.1 Full rerun under a new run-id
- After spec unification, rerun all 294 under a **new run-id**; never patch old jsonl in place.
- Methods wording: "the v1 pilot exposed spec drift; v2 is a pre-registered cohort."

### 3.2 Add an admission gate ("first answer whether the data is what you think it is")
After each `/main TERMINAL_COMPLETE`, immediately run single-record mechanical verification;
only passing records append to `generated.jsonl`; failures are marked slot failures and do not enter
the formal cohort.

### 3.3 Handoff discipline
- Teacher decisions may keep short reasons but must not skip reducer state updates.
- Per-domain requirement templates must match the domain; never reuse another domain's template.
- Every job goes through the handoff backend; manual edits of responses are forbidden.

### 3.4 Builder repair (P2, parallel to P0)
`stepwise_builder.py` must guarantee after `APPLY_LOCAL_UPDATE`:
`step[n].state.resolved == step[n-1].next_state.resolved` (core invariant, not verifier-optional).
Add a unit test: Variation_G 7-step trajectory passes carry-forward across the whole chain.

### 3.5 Technique orchestration: keep the frozen-294 design; do not shrink domains
- Do **not** switch medical/financial to `TECH-04.TECH-08` to raise pass rate (that would break
  cross-domain counterbalancing).
- Correct approach: once TECH-03 paths are spec-consistent they should pass symmetrically with legal.

### 3.6 Baseline control: fix seed pairing (precondition for Gate 1 entering the paper)
- Each baseline must share the same `seed` and constraint graph as the main record referenced by its
  `pair_id`.
- Inject the carry-forward ablation only at a pre-specified step.
- Metadata marks `ablation.control=constraint_carry_forward`.
- Otherwise `seed_mismatch 54/54` makes Gate 1 unpublishable.

---

## 4. Verification design usable in the paper

### 4.1 Three-tier verifier roles

| Tier | Component | Paper role |
|---|---|---|
| Generation time | internal validator (`validator_agent`) | development gate; **never** the sole evidence |
| Phase-C | `MechanicalVerifier` | **Gate 1 primary evidence** |
| Human | double-blind + kappa | second opinion independent of the generator |

> Methods must state: external validity is judged by the MechanicalVerifier; the internal validator
> is only a pipeline-consistency check.

### 4.2 Gate 1: stratified tables (total pass rate forbidden as headline)

| Stratum | Expected pass rate | Interpretation |
|---|---|---|
| main / Successful_Path | high (target >= 90%) | gold trajectories structurally legal |
| main / Counterfactual_Near_Miss | low (~0%) | counter-signal effective |
| baseline / Successful_Path | low (control << main) | verifier sensitive to carry-forward mutation |
| main / Repair_Path | medium-high | repair projections legal |
| metamorphic / Successful_Path | high + cross-variant agreement | see Gate 2 |

- Paired statistics reported only for `main SP vs baseline SP` (same `pair_id`, aligned seed) +
  McNemar + RD + CI; labeled `design_role: verifier_sensitivity_control`.

### 4.3 Gate 2: metamorphic agreement computed only on mechanically-passing subsets
- Agreement = 1.0 over "consistently failing" pairs would be unusable.
- Compute violated-rule / failure-step / terminal agreement only on **mechanically passing**
  metamorphic pairs; exact binomial CI.
- Target `Consistency_meta >= ~0.80` is pre-specified, not automatically achieved.

### 4.4 Gate 3: JS computed on meaningful failure distributions
- Before unification, JS=0 only shows "the same bug reproduces".
- After v2, compute per-batch failure-category distributions; report JS + bootstrap CI; interpret as
  reproducibility evidence, not primary.

### 4.5 Mandatory probes (zero generation cost)

| Probe | Status then | v2 target |
|---|---|---|
| Mutation sensitivity | overall 0.67, false_terminal=0 | fix probe/sampling; target ~1.0 |
| Surface leakage LOO | 0.721 (finance/medical 1.0) | approach 0.5; if still high, generation-side counterbalancing needed |

> High surface leakage = cross-domain surface separability, conflicting with the counterbalancing
> goal - report honestly in Discussion.

### 4.6 Human annotation (mandatory for complete study evidence)
1. Stratified sampling from the v2 cohort (including medical/financial TECH-03, not just legal
   TECH-04)
2. Two blind annotators coding terminal / first-error / carry-forward
3. Cohen's kappa + CI, target kappa > 0.6
4. Run the analysis tooling
> Without kappa, automatic-only evidence is incomplete by design-doc standards.

---

## 5. Claimable vs non-claimable (epistemic boundary)

### Claimable (Level 1-2, after spec self-consistency + v2)
- A trajectory-validation pipeline with generator / MechanicalVerifier separation
- MechanicalVerifier distinguishes complete Successful_Path from a known carry-forward mutation
  (Gate 1 sensitivity)
- Data conditionally provides structural counter-signal against the semantic shortcut (with
  counterbalancing + hold-out)
- Metamorphic / batch reproducibility as supporting evidence (with CI, not primary)

### Not claimable (requires a later phase)
- Models have overcome the semantic shortcut
- State-persistence design has improved generator reliability (needs ablation)
- Gate 1 pass-rate gap = full-method superiority
- Any unstratified total pass rate = cohort quality

---

## 6. Execution roadmap (as planned at the time)

```
Week 1 - Spec & Code
  |- P0: unify APPLY_LOCAL_UPDATE spec (spec doc + mechanical_verifier + tests)
  |- P2: fix stepwise_builder carry-forward (Variation_G path)
  '- fix baseline seed pairing logic
Week 2 - Pilot
  |- new run-id, --limit 12 (4 main SP per domain)
  |- immediate per-record mechanical verification
  '- confirm main SP pass > 0 in all three domains
Weeks 3-4 - Full cohort
  |- complete 294 under phase-c-frozen-294-v2 (new run-id; v1 untouched)
  |- verify + mutation + surface probes
  '- stratified Gate 1/2/3 analysis
Week 5 - Human & Paper
  |- double-blind kappa (stratified sample incl. medical/financial TECH-03)
  |- finalize analysis report + summary JSON
  '- write Methods + Results from stratified tables
```

### Minimal publishable package (if resources constrained)
1. Spec unification + regression tests
2. Stratified sub-cohort: 72 records (24 main SP per domain) rerun
3. Paired Gate 1 + 8 metamorphic pairs + kappa on 36 samples
4. Honest statement: full 294 deferred to appendix / future release

---

## 7. Execution checklist (order of operations)

| # | Action | Files | Backup/versioning |
|---|---|---|---|
| 1 | Back up verifier / spec doc / prompts / builder / frozen spec current versions | backups dir | version tag on first line |
| 2 | P0: sync `APPLY_LOCAL_UPDATE` across spec doc + verifier binding table | see sec 2 | minor version |
| 3 | P2: fix builder carry-forward + unit test | stepwise_builder | synchronized |
| 4 | Fix baseline seed pairing (shared seed + constraint graph) | frozen_spec v2 draft | never overwrite v1 |
| 5 | Add admission gate (per-record verify after TERMINAL) | formal driver | synchronized |
| 6 | Pilot `--limit 12` verifying three-domain main SP pass > 0 | new run-id | - |
| 7 | Full v2 rerun 294 + probes + stratified analysis | v2 evidence directory | new directory |
| 8 | Double-blind kappa stratified sampling (incl. medical/financial TECH-03) | annotations CSV | - |
| 9 | Update README change log + gate interpretations | README | changelog lives in README only |

---

## 8. Decisions requested (all since resolved)

- **Q1 (P0):** Is `APPLY_LOCAL_UPDATE` a legal S2 action under Variation_G? - *Resolved: yes.*
- **Q2:** Fix baseline seed pairing? - *Resolved: yes.*
- **Q3:** Include double-blind kappa? - *Resolved: yes.*
- **Q4:** Full v2 rerun vs minimal path? - *Resolved: full v2 rerun, executed and closed.*

*(Original decision-request framing retained for historical fidelity; see editor's note above.)*
