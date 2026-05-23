# THEMIS Platform — Implementation Roadmap
## Sequencing, Timing, and Control Gates · v2.0
*Updated to reflect full 42-service platform including FGS/PLUTUS and HADES*

---

## Roadmap Overview

The THEMIS platform deploys across three implementation horizons and nine build phases spanning four years. Each phase has explicit entry criteria, deliverables, and control gates requiring sign-off before the next phase begins. No phase begins without gate confirmation.

```
H1 — GOVERNANCE FOUNDATION       Months 1–17    23 platform services
H2 — INTELLIGENCE CYCLE          Year 2         13 intelligence layer services
H3 — WARNING LAYER & ORACLE      Year 3–4        6 warning services + outcome intelligence
PROPOSED — COMPETENCE AXIS       Year 2–3+       2 proposed services (ARB approval required)
```

**Total approved services:** 42 (23 platform + 15 intelligence layer + HADES + FGS)
**Total proposed services:** 2 (CPS/APORIA, ODS/LETHE — pending ARB approval)

---

## Pre-Phase Prerequisites

The following must be confirmed as operational or contracted before Phase 1 begins. These are not THEMIS engineering tasks — they are organisational prerequisites.

| Prerequisite | Owner | Required by | Risk if absent |
|---|---|---|---|
| Machine-readable clearance system API | Security authority | Phase 1 start | PCES cannot validate analyst clearances — no sessions possible |
| Conflict of interest registry API | Security authority | Phase 1 start | PCES cannot complete CoI checks — sessions blocked |
| Requirements management system API | Requirements authority | Year 2 Q4 | TIS/DIKE cannot submit CGRs — manual workaround only |
| Physical security infrastructure for HADES | Facilities | Phase 5 start | HADES air-gapped indices cannot be deployed |
| IOB charter and governance framework | Leadership/IOB | Phase 1 start | GC items cannot be resolved — several Phase 1–2 services blocked |
| GC-3: Query-type authorization taxonomy | IOB | Phase 1 start | PCES classifier training cannot begin |
| GC-4: Within-requirement role-tier access | IOB | Phase 1 start | PCES tier deployment blocked |
| GC-2: Retrieval gap indicator disclosure | IOB | Year 2 Q4 | CGS/ARGUS-LACUNA analyst surface cannot be built |
| FGS tokenomics model (initial) | Leadership/IOB | Phase 1 start | Token allocation periods cannot open |
| Consumption data governance policy | IOB | Phase 1 start | FGS analyst data collection blocked |
| Research & Red Team capability benchmark commitment | R&RT | Phase 1 start | CPS/APORIA proposal cannot proceed |
| IAS/SCUDO IC-specific threat catalog | R&RT | Phase 5 start | IAS/SCUDO Phase 1 blocked without IC patterns |

---

## Phase 1 — Safety Gates, Financial Governance, and Core Interaction Layer
**Weeks 1–8 · Entry: Prerequisites confirmed · Gate: Gate-1**

### Services Deployed

| Service | Epithet | Deliverables |
|---|---|---|
| PCES | AEGIS | Session initialization, per-request validation, CoI detection, classification log |
| PGS | NOMOS | Input and output screening, interaction classification, ICD 203 compliance |
| FGS | PLUTUS | Transaction recording, account management, SCBS balance integration, tokenomics model |
| MGS | TERMINUS | MCP registry (initial entries), gateway enforcement, classification boundary |
| PRS | PROMETHEUS | Prompt versioning, session record, mutation hint mapping |

### Phase Gate-1 Criteria

All five criteria must be verified before Phase 2 begins:

- [ ] Every ATHENA session produces a PCES compartment decision event in MOIRAI
- [ ] PGS screening fires correctly on ICD 203 test violation cases; interaction classification accuracy ≥ 85% on labelled test set
- [ ] FGS records a TokenTransaction for every inference call; period allocation opened with MOIRAI event
- [ ] MGS blocks unregistered MCP server calls and classification boundary violations in test
- [ ] PRS version hashes logged to MOIRAI for every session turn that uses a prompt
- [ ] **Governance gate:** IOB has reviewed Phase 1 deployment; GC-3 and GC-4 confirmed satisfied

---

## Phase 2 — Agent-Native Foundation and Interaction Layer Completion
**Weeks 5–28 · Entry: Gate-1 · Gate: Gate-2**

### Services Deployed

| Service | Epithet | Deliverables |
|---|---|---|
| SCBS | SENTINEL-CAP | Session capability envelopes, pre-call estimation, anomalous spend detection, FGS integration |
| CBS | BROKER | Capability surface registry, handle issuance, proxy calls, scope violation detection |
| SKS | DAEDALUS | Skill registry, graduated sharing, SKS-to-STOA integration point |
| MOIRAI | MOIRAI (Phase 1) | Event ledger, hash chain, per-service signing; PCES/PGS events already chained |

*Note: MOIRAI Phase 1 (event ledger) deploys at the start of Phase 2, retroactively chaining Phase 1 events. This is the designed sequence: PCES and PGS buffer events in Phase 1; MOIRAI ingests them at Phase 2 start.*

### Phase Gate-2 Criteria

- [ ] MOIRAI hash chain continuous from Phase 1 first event through Phase 2 last event; chain integrity verification passes
- [ ] Agent sessions bounded by SCBS envelopes; spend exceedance suspends session within 1 second in test
- [ ] CBS scope violation detected and SCBS escalation triggered correctly in test
- [ ] SKS catalog serves approved skills to STOA integration point
- [ ] FGS SCBS balance query integration operational; session budgets respect team allocation
- [ ] **Governance gate:** ARB reviews Agent-Native capability surface governance framework; CBS capability surface registry has at least 5 ARB-approved entries

---

## Phase 3 — Core Infrastructure and Document Provenance
**Weeks 9–28 · Entry: Gate-2 · Gate: Gate-3**

### Services Deployed

| Service | Epithet | Deliverables |
|---|---|---|
| MOIRAI | MOIRAI (Phase 2) | Provenance graph (Neo4j), blast radius traversal, TSA anchoring, audit certificates |
| TCS | MIMIR | Bayesian posteriors, calibration state machine, gaming probability (Phase 1) |
| CVS | VERITAS | Source existence verification, fabrication detection, GRND badge |
| MDS | KRONOS | Model version registry, drift assessment, stale flagging, MCP version tracking |
| DPS | CODEX | Document lifecycle tracking, AI usage disclosure, session-to-document linkage |
| RSS | ROLLBACK | Pre-action snapshots, rollback execution, tiered storage |

### Phase Gate-3 Criteria

- [ ] MOIRAI provenance graph traversal returns correct 2-hop neighbourhood for test entities; TSA anchor covers 100% of events > 26 hours old
- [ ] TCS confidence signals served to ATHENA for all active sessions; PRIOR ONLY state correctly surfaced
- [ ] CVS fabrication detected and MOIRAI event produced for test fabricated citations; GRND badge displayed in ATHENA
- [ ] MDS model version change triggers TCS stale flagging within 60 seconds in test
- [ ] DPS document-to-session linkage produces AI usage disclosure for test document
- [ ] RSS pre-action snapshot created before every test write action; rollback restores correct state
- [ ] **Governance gate:** IOB reviews MOIRAI chain audit certificate format; confirms acceptable for oversight use

---

## Phase 4 — Quality Layer Completion and Adversarial Defence
**Weeks 29–46 · Entry: Gate-3 · Gate: Gate-4**

### Services Deployed

| Service | Epithet | Deliverables |
|---|---|---|
| FGTS | ALETHEIA | Correction intake, five-factor weighting, corpus versioning, TCS posterior updates |
| TVS | KAIROS | Source validity tracking, decay models, invalidation events, MOIRAI blast radius |
| RQS | HERMES | Retrieval quality assessment, coverage scoring, sparse coverage flagging |
| IAS | SCUDO | Input and retrieval screening (OWASP + IC patterns), MCP response screening via MGS |
| MAS | EIDOLON | Image and document authenticity (Phase 1); video and audio (Phase 2) |
| HADES | HADES (P1) | IAS/SCUDO, CVS, MAS ingestion pipelines; Research & Red Team query interface |

### Phase Gate-4 Criteria

- [ ] FGTS weighted corrections reaching TCS and updating posteriors; supervisor confirmation workflow operational
- [ ] TVS validity scores displayed on all ATHENA source citations; invalidation event triggers blast radius computation
- [ ] RQS sparse coverage flagging generating TIS/DIKE gap signals in test
- [ ] IAS blocks all OWASP LLM Top 10 test patterns; IC-specific pattern catalog operational
- [ ] MAS risk badges displayed on all image and document sources in ATHENA
- [ ] HADES receiving IAS, CVS, and MAS events; Research & Red Team query interface accessible from designated terminals only; air-gapped network isolation validated
- [ ] **Governance gate:** Research & Red Team presents first adversarial intelligence summary from HADES to IOB; IAS threat catalog updated with at least 5 new patterns derived from HADES findings

---

## Phase 5 — Knowledge Layer and Reasoning Audit
**Weeks 47–66 · Entry: Gate-4 · Gate: Gate-5 (H1 Complete)**

### Services Deployed

| Service | Epithet | Deliverables |
|---|---|---|
| KCS | ARGUS | Watchlist monitoring, supersession signals to TVS, coverage map, model knowledge currency |
| ERAS | LOGOS | Reasoning capture, claim extraction, completeness scoring, adversarial reasoning records to HADES |
| HADES | HADES (P2) | ERAS adversarial reasoning record integration, CASSANDRA integration (anticipatory) |

### Phase Gate-5 — H1 Complete

This is the H1 completion gate. All 23 approved platform services are operational. The gate requires:

- [ ] KCS watchlist evaluation producing TVS supersession signals in test; coverage map populated for all indexed corpus domains
- [ ] ERAS reasoning captures produced for all ATHENA sessions; unsupported claim flags surfaced in ATHENA verification queue; completeness scores feeding PGS output screening threshold
- [ ] HADES adversarial reasoning records from ERAS indexed and queryable; manipulation mechanism annotation workflow operational
- [ ] **Full platform chain audit:** MOIRAI chain integrity verified from Phase 1 first event through Phase 5 last event; TSA coverage ≥ 99%; chain audit certificate accepted by IOB in formal review
- [ ] **Governance gate (H1 closing review):** IOB formal assessment of H1 platform. All 8 GC items status reviewed. ARB sign-off on H2 readiness. Research & Red Team presents HADES adversarial intelligence assessment to IOB.

---

## H2 Transition — Year 2 Pre-work

Before Year 2 intelligence layer services begin, the following must be in place:

| Prerequisite | Required by | Notes |
|---|---|---|
| GC-5: Outcome classification definition | OFS/NEMESIS Year 2 Q2 | Without GC-5, OFS cannot classify outcomes |
| GC-6: Consumer package content policy | PCS/IRIS Year 2 Q3 | Without GC-6, PCS cannot define package content |
| GC-8: Forecast product governance | TRS/CHRONOS Year 3 | Required before CHRONOS forecast products |
| OGS entity graph initial population | Year 2 Q1 | Analytic standards authority operational commitment required |
| MNEMOSYNE data floor assessment | Year 2 Q1 | Confirm FGTS/OFS data volume sufficient for extraction |

---

## Phase 6 — Intelligence Cycle Completion (Addendum F)
**Year 2 Q1–Q4 · Entry: Gate-5 + H2 pre-work · Gate: Gate-6**

### Services Deployed

| Service | Epithet | Quarter | Key deliverables |
|---|---|---|---|
| UCS | TYCHE | Q1 | Uncertainty characterisation, three analyst questions, taxonomy v1.0 |
| OFS | NEMESIS | Q2 | Outcome event ingestion, FGTS high-weight corrections, GC-5 required |
| PCS | IRIS | Q3 | Consumer package generation, falsification indicators, GC-6 required |
| TIS | DIKE | Q4 | Gap-to-requirement pipeline, requirements system API, CGR lifecycle |

### Phase Gate-6 Criteria

- [ ] UCS uncertainty profiles produced for all analytical claims; three ATHENA questions display and record analyst resolutions
- [ ] OFS outcome events trigger FGTS corrections within 60 seconds of supervisor confirmation
- [ ] PCS consumer packages generated within 3 seconds of DPS dissemination event; GC-6 package format implemented
- [ ] TIS CGRs submitted to requirements management system API successfully; lifecycle tracking operational
- [ ] **Governance gate:** IOB reviews first batch of OFS outcome data; GC-5 adequacy confirmed; PCS consumer package format approved for policymaker delivery

---

## Phase 7 — Knowledge Foundation (themis-knowledge)
**Year 2 Q1–Q3 · Entry: Gate-5 · Gate: Gate-7**

### Services Deployed (parallel to Phase 6)

| Service | Quarter | Data floor | Key deliverables |
|---|---|---|---|
| OGS/YGGDRASIL | Q1 | Corpus established | Entity graph, resolution index, ontology v1.0 |
| MOS/SAGA | Q1–Q2 | OGS operational | Session memory assembly, matter memory, analyst profiles |
| SCRIBE | Q2 | Corpus versioned | Semantic diff, significant change notification, ATHENA badges |
| MNEMOSYNE | Q3 | 18 months FGTS/ERAS | Knowledge graph, extraction pipeline, MOS/SAGA integration |

### Phase Gate-7 Criteria

- [ ] OGS resolution rate > 70% on test surface forms; subgraph queries serve STOA correctly
- [ ] MOS/SAGA context assembly completes within 1 second for matters with > 5 prior sessions
- [ ] SCRIBE CRITICAL diffs notify affected analysts within 60 seconds; semantic change classification benchmarked ≥ 80% accuracy
- [ ] MNEMOSYNE extraction pipeline producing HIGH-strength nodes for domains with > 20 evidence records; de-identification audit confirms no analyst identity in any node

---

## Phase 8 — Research Intelligence (themis-research)
**Year 2 Q2–Q3 · Entry: Gate-7 · Gate: Gate-8**

### Services Deployed

| Service | Quarter | Data floor | Key deliverables |
|---|---|---|---|
| STOA | Q2 | OGS, SKS operational | Research decomposition, methodology trail, analyst review checkpoints |
| MIRROR | Q2 | 50 requirements | Requirement similarity, collection effectiveness, data floor reporting |
| PYTHIA | Q3 | MIRROR + MNEMOSYNE | Anticipatory signals, session context monitoring, feedback model |

### Phase Gate-8 Criteria

- [ ] STOA complete research session (start → synthesis → trail) produced for test requirement; methodology trail MOIRAI events chained
- [ ] MIRROR similarity results meaningful for domains with > 50 profiles; data_floor_met field correctly reported
- [ ] PYTHIA acted_upon rate > 30% after 60 days of operation; session context updates trigger signal refresh
- [ ] **Governance gate:** IOB reviews STOA methodology trail; confirms analytical accountability standard met

---

## Phase 9 — Warning Layer (themis-warning)
**Year 2 Q4–Year 3 · Entry: Gate-8 + GC-8 resolved · Gate: Gate-9**

### Services Deployed

| Service | Quarter | Key deliverables |
|---|---|---|
| CGS/ARGUS-LACUNA | Year 2 Q4 | Collection gap assessment, epistemic signal to UCS, gap signals to TIS |
| TRS/CHRONOS | Year 3 | Trajectory models, scenario spaces, leading indicators, UCS temporal component |
| ADS/CASSANDRA | Year 3 | Behavioural baselines, anomaly detection, unusual absence |
| WSF/LACHESIS | Year 3 | Weak signal fusion, precursor pattern matching, SENTINEL feed |
| ORACLE | Year 3 | Outcome patterns (200 requirement data floor), accuracy predictions |
| CRF/JANUS | Year 3 | Cross-requirement overlap, contradiction detection, supervisor notification |
| SWS/SENTINEL | Year 3–4 | Strategic warning synthesis, multi-source corroboration, lifecycle tracking |

### Phase Gate-9 Criteria

- [ ] CGS epistemic signal correctly feeds UCS uncertainty profiles; gap signals reach TIS within 30 minutes
- [ ] CHRONOS trajectory models with scenario spaces and leading indicators produced for test entities; GC-8 forecast product format operational
- [ ] CASSANDRA anomaly detection demonstrates meaningful deviation scores on known historical test anomalies
- [ ] LACHESIS precursor pattern library contains ≥ 10 validated patterns; fusion established for test signal sets
- [ ] ORACLE predictions carry correct evidence count and INSUFFICIENT maturity labels for data-scarce domains
- [ ] SENTINEL warning generated from multi-service test signals; source_diversity ≥ 3 before warning issued
- [ ] **Governance gate:** IOB reviews first SENTINEL strategic warning and lifecycle; confirms warning governance protocol adequate

---

## Proposed Services — ARB Decision Track

These two services are not on the approved roadmap. They require ARB approval before they can be built.

| Service | Proposal requirements | Earliest possible deployment |
|---|---|---|
| CPS/APORIA | R&RT evaluation programme design; IOB hard ceiling endorsement; pilot evaluation results | Year 2+ (post Gate-5) |
| ODS/LETHE | Latency benchmarking (p99 < 50ms validated); OOD validity study; distribution characterisation methodology; IOB threshold policy | Year 3+ (post Gate-7) |

**ARB decision point:** Both proposals should be reviewed together as the Competence Axis package, no earlier than Gate-5 completion when the R&RT has sufficient operational data to inform the evaluation methodology.

---

## Control Gate Summary

| Gate | Phase | Timing | Sign-off required |
|---|---|---|---|
| Gate-0 | Pre-Phase | Before Week 1 | Leadership confirms prerequisites; IOB charter in place |
| Gate-1 | Phase 1 | Week 8 | Cell Lead + Engineering Lead + IOB review |
| Gate-2 | Phase 2 | Week 28 | Cell Lead + Engineering Lead + ARB (agent capability surfaces) |
| Gate-3 | Phase 3 | Week 28 | Cell Lead + Engineering Lead + IOB (MOIRAI chain audit) |
| Gate-4 | Phase 4 | Week 46 | Cell Lead + Engineering Lead + R&RT + IOB (adversarial assessment) |
| **Gate-5 (H1 Close)** | Phase 5 | Week 66 | **Full ARB + IOB formal assessment; H2 authorisation required** |
| Gate-6 | Phase 6 | Year 2 Q4 | Cell Lead + IOB (GC-5, GC-6 confirmed; outcome data review) |
| Gate-7 | Phase 7 | Year 2 Q3 | Cell Lead + Engineering Lead + Analytic Standards Authority |
| Gate-8 | Phase 8 | Year 2 Q3 | Cell Lead + IOB (methodology trail standard) |
| Gate-9 | Phase 9 | Year 3–4 | Cell Lead + IOB (warning governance protocol) |

---

## Key Policy Dependencies by Phase

| Phase | GC Item | Decision Required |
|---|---|---|
| Phase 1 | GC-3 | Query-type authorisation taxonomy |
| Phase 1 | GC-4 | Within-requirement role-tier access |
| Phase 1 | FGS Tokenomics | IOB-approved allocation model |
| Phase 4 | (none — operational) | — |
| Phase 4–5 | HADES Physical Security | Facilities commitment |
| Year 2 Q2 | GC-5 | Outcome classification definition |
| Year 2 Q3 | GC-6 | Consumer package content policy |
| Year 2 Q4 | GC-2 | Retrieval gap indicator disclosure |
| Year 3 | GC-8 | Forecast product governance |

---

## Data Floor Requirements

Several intelligence layer services require accumulated data before they become analytically meaningful. These floors cannot be shortened by engineering effort.

| Service | Data floor | Realistic availability | Risk if deployed early |
|---|---|---|---|
| TCS/MIMIR (CALIBRATED state) | 30 weighted corrections per analyst-domain cell | Year 2+ for most cells | PRIOR ONLY state extended; misleading confidence signals |
| FGTS (corpus useful for TCS) | 100+ corrections across domains | Year 2 | Sparse corpus; TCS posteriors statistically weak |
| MIRROR | 50 requirements with profiles | Year 2 Q1–Q2 | Low-confidence similarity; data_floor_met = false |
| MNEMOSYNE | 18 months FGTS + ERAS data | Year 2 Q3 | TENTATIVE nodes only; institutional knowledge thin |
| ORACLE | 200 requirements with outcomes | Year 3–5 (realistic) | INSUFFICIENT maturity on all predictions |
| SENTINEL (meaningful warnings) | All warning layer services mature | Year 4 | Low-signal-diversity warnings; alert fatigue risk |

---

## Deployment Assumptions and Risk Registry

| Risk | Probability | Phase affected | Mitigation |
|---|---|---|---|
| Clearance system API not ready | Medium | Phase 1 blocked | Contract requirement; prerequisite gate |
| R&RT bandwidth insufficient for IAS catalog | Medium | Phase 4 degraded | IAS catalog content is Phase 4 prerequisite; pre-Phase 4 commitment required |
| OGS entity graph quality poor at Year 2 Q1 | High | Phase 7 degraded | Analytic standards authority graph quality programme planned |
| OFS/NEMESIS outcome data floor takes > 5 years | High (structural) | ORACLE perpetually weak | ORACLE maturity expectations set correctly; not gate-blocking |
| Requirements management system API access denied | Medium | TIS/DIKE blocked | Alternative: manual CGR submission with reduced automation |
| MOIRAI Neo4j performance degradation at scale | Low | Phase 3 degraded at Year 3+ | Load testing at 10M and 100M nodes before Gate-3 |
| Physical security for HADES not ready | Low | Phase 4 delayed | Facilities prerequisite; Gate-0 confirmation required |

---

*THEMIS Implementation Roadmap v2.0 · 42 approved services · 2 proposed services · 9 control gates*
*Document maintained by: THEMIS Platform Team*
*Governance: changes to phase sequencing or gate criteria require ARB approval*
