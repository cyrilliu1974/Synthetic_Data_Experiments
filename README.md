# Synthetic Data Experiments - Public Release

Companion artifact release for the study of **mechanically verified synthetic decision-trajectory
data** (working name TGAA-style pipeline): a generator that produces multi-step decision
trajectories under hash-bound handoff discipline, an **independent mechanical verifier** that judges
structural validity, and a pre-registered validation protocol ("Phase C") run on two frozen cohorts.

- **Cohort v1** (`cohort_v1/`): 294-record pilot cohort. Its initially low pass rate triggered a
  full instrument-calibration investigation (specification drift between generator-side validators
  and the independent verifier), documented here as an honest negative result plus fixes.
- **Cohort v2** (`cohort_v2/`): 294-record **pre-registered primary cohort** after calibration,
  with paired baseline seeds. This is the primary evidence for Gate 1 claims.

Everything is released under frozen specifications so that every number below can be recomputed from
the shipped artifacts.

---

## 1. Repository map

```text
.
|-- README.md                        this file
|-- specs/
|   |-- frozen_spec_v1.json          frozen generation spec, cohort v1 (294 records)
|   '-- frozen_spec_v2.json          frozen generation spec, cohort v2 (paired baseline seeds)
|
|-- cohort_v1/                       pilot cohort "phase-c-frozen-294"
|   |-- run_manifest.json            run identity / configuration record
|   |-- data/generated.jsonl         all 294 generated trajectories
|   |-- verification/verifier_results.jsonl   per-record mechanical verifier verdicts
|   |-- analysis/                    analysis outputs (summary, probes, agreement)
|   |-- reports/                     study completion + analysis report (English)
|   |   '-- calibration/             spec-drift diagnosis, RCA, fix log, roadmap
|   |-- human_annotation/            blind packets, CSV, stratification manifest, guides
|   '-- session_ledgers/             batch manifest + compressed per-slot session states
|
|-- cohort_v2/                       pre-registered primary cohort "phase-c-frozen-294-v2"
|   '-- (same layout as cohort_v1, plus reports/PREREGISTRATION.md)
|
|-- handoff_archive/                 byte-intact hash-bound envelopes
|   |-- jobs/                        request payloads as dispatched
|   '-- responses/                   response payloads as returned
|
'-- provenance/source_snapshot.json  provenance fingerprint of the source tree

Source code is not included in this release and is available upon request for review purposes.
```

---

## 2. Cohort design (both cohorts)

294 records = **216 main + 54 baseline + 24 metamorphic**, over three domains
(financial / legal / medical) and three trajectory types:

| Trajectory type | Role | Expected verifier outcome |
|---|---|---|
| Successful_Path (SP) | gold positives | pass |
| Repair_Path | repair projections of SP | pass |
| Counterfactual_Near_Miss | injected near-miss structural faults (counter-signal) | fail |
| baseline (54) | format-matched SP with carry-forward ablation (Gate 1 control) | fail |
| metamorphic (24) | 12 latent groups x 2 surface variants (Gate 2 pairs) | agree |

The mechanical verifier checks nine rule families, including next-state completeness, state
carry-forward (`step[n].state.resolved == step[n-1].next_state.resolved`), initial-state
consistency, dependency closure, no-orphan constraints, terminal consistency, action-token binding,
canary consumption, and content rules.

---

## 3. Headline results

### 3.1 Instrument calibration story (why there are two cohorts)

The first frozen run scored **58/294 (19.7%)** overall. Full-cohort forensics
(`cohort_v1/reports/calibration/SPEC_DRIFT_DIAGNOSIS.md`) attributed failures to two stacked layers:

- **184 x S_TOKEN_BINDING_VIOLATION**: specification drift - the generator-side validators accepted
  `APPLY_LOCAL_UPDATE` as a legal S2 action (Variation_G selective update) while the independent
  verifier's binding table did not. False kills, not bad data.
- **52 x SV_CARRY_FORWARD_VIOLATION**: genuine defects (plus intentional baseline ablations).

Fixes applied (P0 binding-table sync + regression tests; P0.5 mutation-probe targeting bug fixed -
see `FALSE_TERMINAL_PROBE_RCA.md`), then re-verification of the same 294 records gave:
main SP 72/72, main Repair 72/72, counterfactual 0/72 (expected), metamorphic 24/24, baseline 2/54
(by design). Rather than patching the pilot, the team then **pre-registered and executed cohort v2**
with paired baseline seeds.

### 3.2 Gates and probes

| Evidence | Cohort v1 (appendix) | Cohort v2 (primary) |
|---|---|---|
| Gate 1 design role | post-hoc mutation sensitivity (seed_mismatch 54/54 red flag) | **paired ablation** (seeds shared 54/54) |
| Gate 1 main SP pass rate | 1.0 [0.934, 1.0] | 1.0 [0.934, 1.0] |
| Gate 1 control pass rate | 0.037 [0.010, 0.125] | **0.0** [0.0, 0.066] |
| Paired risk difference | 0.963 [0.907, 1.0] | **1.0** [1.0, 1.0] |
| McNemar (b, c) | (52, 0) | **(54, 0)** |
| Gate 2 metamorphic agreement | 1.0 [0.757, 1.0] | **1.0** [0.757, 1.0] |
| Gate 3 max JS across batches | 0.0 [0.0, 0.0] | 0.0 [0.0, 0.0] |
| Mutation probe sensitivity | 1.0 (post-fix; false_terminal 144/144 rejected) | **1.0** by kind |
| Mutation specificity (reword / surface) | 1.0 | 1.0 |
| Surface LOO: metadata | 0.823 | 0.816 |
| Surface LOO: action_token | 0.776 | 0.429 (~ chance) |
| Surface LOO: narrative_only | 0.578 (~ chance) | **0.571** (~ chance) |

Pre-specified targets from the study design document (sec 4.6): metamorphic agreement >= ~0.80
(met), JS < 0.05 (met), human kappa > 0.6 on all dimensions (**not met** - see below).

### 3.3 Human double-blind agreement (36 stratified records per cohort)

| Dimension | v1 human-human kappa | v2 human-human kappa | Note |
|---|---|---|---|
| first_error | **1.0** | 0.0 | v2: annotator split on 12 CF records; annotator 2 aligns with verifier |
| carryforward | **1.0** | **1.0** | unanimous in both cohorts |
| terminal | 0.0 | 0.0 | recurring rubric tension: does a binding fault imply terminal=0? |

The terminal disagreement is analyzed as a rubric boundary case (binding fault vs terminal-field
consistency), not a verifier bug; blind packets, coding sheets, and both annotation guides are
released so others can replicate or revise the protocol.

### 3.4 Claim boundary (binding for any reuse)

**Supported:** structural-validity pipeline; verifier discriminates gold trajectories from ablated
controls and known mutations; metamorphic invariance; batch reproducibility; narrative-only leakage
near chance.

**Not supported:** full-method superiority over an independent generator baseline; downstream model
learning outcomes; zero semantic shortcut anywhere (metadata remains separable); kappa > 0.6 on all
human dimensions.

---

## 4. Integrity and disclosure notes

1. **Hash-bound envelopes are byte-intact.** Everything under `handoff_archive/` is cryptographic
   evidence: recorded hashes bind each job to its response. These files were copied without any
   modification. Some job payloads embed template fragments in languages other than English; these
   are intentionally left untouched because editing would invalidate the recorded hashes.
2. **Translated interpretive strings.** A small number of prose fields inside six analysis JSONs
   (probe descriptions, expectation notes) were translated into English for this release; numeric
   and verdict fields are byte-faithful. All narrative documents were rewritten in English; where a
   document was translated rather than originally written in English, this is stated in its header.
3. **Session ledgers.** Per-slot session states (354 MB JSON per cohort in the working archive) are
   shipped as gzip tarballs excluding console logs (`slot_sessions_no_console_logs.tar.gz`,
   ~20 MB per cohort), preserving full auditability of intermediate states.
4. **Human-subject-free.** Annotation packets contain synthetic text only; annotator identities are
   not included (columns a1_/a2_ only).

---

## 5. Citation

If you use these artifacts, please cite the accompanying paper and this repository:

- Repository: <https://github.com/cyrilliu1974/Synthetic_Data_Experiments/>
- Paper: *Mechanically Verified Synthetic Decision-Trajectory Data via Hash-Bound Handoff*
  (citation block to be finalized at publication).

## 6. License

License to be added before public announcement (planned: permissive research license).
