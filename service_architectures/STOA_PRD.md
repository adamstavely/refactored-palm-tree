# STOA — Research Orchestration Service
### STOA · *"Greek for the covered walkway where scholars convened to reason together — the structured space where inquiry becomes synthesis"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `STOA` |
| **Epithet** | `STOA` |
| **Full name** | Research Orchestration Service |
| **Namespace** | `themis-research` |
| **Layer** | Intelligence Layer — Research |
| **Build phase** | Year 2 · Q2 |
| **Build priority** | 5 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — produces a documented methodology trail for AI-assisted multi-step research |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**STOA answers: Given a complex analytical requirement, how is it decomposed into tractable sub-questions, what skills and sources are applied to each, and how is the resulting synthesis documented in an auditable methodology trail?**

### 1.2 Why This Service Exists

Complex intelligence requirements are not single-turn questions. An assessment of an adversary's technical programme requires: identifying what is known about the programme's history, assessing current development stage, evaluating likely timelines, characterising collection coverage, and synthesising these strands into a coherent analytical product. Each of these is a distinct research activity. Doing them in a single ATHENA session produces a session with no internal structure — the AI moves from one strand to the next without the analyst being able to see how the research was organised or review intermediate conclusions before they are synthesised.

STOA provides that structure. It decomposes a complex requirement into sub-questions, routes each to the appropriate analytical skill from SKS/DAEDALUS, retrieves relevant intelligence via the corpus, assembles partial answers, and synthesises them — all while building a documented methodology trail that is itself an analytically auditable record. The methodology trail is what makes STOA-assisted research different from multi-turn ATHENA sessions: it is transparent about how the research was organised, what was asked in what order, and what intermediate conclusions informed the synthesis.

### 1.3 Design Principles

- **The methodology trail is the primary accountability artefact.** The synthesis is the output. The methodology trail is the accountability record. Both must be produced; neither substitutes for the other.
- **Decomposition is analyst-guided, not AI-autonomous.** STOA proposes a decomposition of the requirement into sub-questions. The analyst reviews and approves the decomposition before research begins. An AI-autonomous decomposition that the analyst did not review is not accountable research.
- **Skills from SKS/DAEDALUS, not ad hoc prompting.** Every sub-question is addressed using a versioned, approved skill from the registry. STOA does not ad hoc prompt — this is precisely what distinguishes STOA research from unstructured chat. The skill version used is logged to MOIRAI.
- **Partial answers are reviewable before synthesis.** The analyst can review each sub-question's partial answer before STOA proceeds to synthesis. This is the human-in-the-loop checkpoint that makes STOA research reviewable rather than black-box.
- **Compounding value through MIRROR and ORACLE.** When MIRROR identifies similar prior requirements, STOA can initialise sub-questions with the analytical approaches that worked on those requirements. When ORACLE is operational, STOA can weight sub-question approaches by predicted accuracy. The service improves as the research layer matures.

### 1.4 Explicit Out of Scope

- **Executing collection.** STOA identifies collection gaps; TIS/DIKE routes the gap to requirements officers.
- **Making analytical conclusions.** STOA synthesises what the research produced. The analytical conclusion is the analyst's responsibility; STOA provides the structured research that informs it.
- **Managing finished analytical products.** DPS/CODEX manages document lifecycle. STOA produces the research record that informs a document.

---

## 2. Core Responsibilities

### 2.1 Primary Function

STOA orchestrates multi-step analytical research — receiving a complex requirement from an ATHENA session, proposing a decomposition into sub-questions for analyst approval, executing each sub-question using versioned SKS/DAEDALUS skills with corpus retrieval, producing reviewable partial answers, synthesising them into a structured research output, and generating a methodology trail that records every step, every skill invoked, every source retrieved, and every intermediate conclusion.

### 2.2 Secondary Functions

- Decomposition recommendation: using MIRROR similar requirement data and MNEMOSYNE institutional knowledge to recommend sub-question structure
- Gap detection during research: when a sub-question produces sparse retrieval, generating a gap signal for ARGUS-LACUNA via RQS/HERMES
- Partial answer review interface: surfacing each sub-question's partial answer to the analyst before synthesis
- Research session resumption: pausing and resuming research sessions across multiple ATHENA sessions on the same matter
- ORACLE integration (Year 3): using predicted accuracy by sub-question type to weight synthesis
- Research template library: pre-defined decomposition templates for common requirement types (technical programme assessment, intent analysis, order of battle, etc.)

### 2.3 What This Service Does Not Decide

STOA proposes research structure and executes approved research. Whether the decomposition is analytically appropriate, whether a partial answer is sufficient to proceed, and whether the synthesis is analytically sound are human decisions made by the analyst at review checkpoints. STOA organises and documents; the analyst judges.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ResearchSession:
  research_id:             uuid
  analyst_session_id:      uuid             # FK → ATHENA/PCES analyst session
  matter_id:               uuid | null      # FK → MOS/SAGA MatterMemory
  requirement_context:     str              # the initial analytical requirement
  decomposition_approved:  bool
  sub_questions:           [ResearchSubQuestion]
  methodology_trail:       [MethodologyStep]
  synthesis:               str | null       # the final synthesised research output
  status:                  DECOMPOSING | REVIEWING | RESEARCHING | SYNTHESISING | COMPLETE | PAUSED | ABANDONED
  created_at:              datetime
  completed_at:            datetime | null

ResearchSubQuestion:
  subquestion_id:          uuid
  research_id:             uuid
  sequence:                int
  question:                str
  skill_id:                uuid | null      # FK → SKS/DAEDALUS
  skill_version_hash:      str | null
  retrieved_chunk_ids:     [str]
  rqs_assessment_id:       uuid | null      # FK → RQS retrieval quality
  ias_screening_id:        uuid | null      # FK → IAS adversarial screening
  partial_answer:          str | null
  partial_confidence:      high | medium | low | uncertain | null
  gap_identified:          bool
  gap_cgr_id:              uuid | null      # FK → TIS/DIKE CGR if gap submitted
  analyst_reviewed:        bool
  analyst_notes:           str | null

MethodologyStep:
  step_id:                 uuid
  research_id:             uuid
  sequence:                int
  step_type:               DECOMPOSE | RETRIEVE | SKILL_INVOKE | PARTIAL_ANSWER |
                           ANALYST_REVIEW | SYNTHESISE | GAP_IDENTIFIED | REVISION
  description:             str
  skill_ref:               uuid | null
  skill_version_hash:      str | null
  source_refs:             [str]
  timestamp:               datetime

ResearchTemplate:
  template_id:             uuid
  name:                    str
  requirement_type:        str              # e.g., TECHNICAL_PROGRAMME | INTENT | ORDER_OF_BATTLE
  sub_question_templates:  [str]
  skill_suggestions:       [{ subquestion_pattern: str, skill_id: uuid }]
  maintained_by:           str
  version:                 str
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | ResearchSession, ResearchSubQuestion, MethodologyStep, ResearchTemplate | Indefinite |
| Active session cache | Redis | In-progress research sessions (fast resume) | 24h TTL |
| Event store | MOIRAI | Signed research and methodology events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ResearchSession | Inherits analyst session classification | Session-compartmented |
| MethodologyStep | Inherits research session classification | Session-compartmented |
| ResearchTemplate | Controlled Unclassified | Platform-wide |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /research/start
  Auth:     ATHENA service account | analyst session token
  Request:  {
    analyst_session_id:    uuid,
    matter_id:             uuid | null,
    requirement_context:   str,
    template_id:           uuid | null
  }
  Response: {
    research_id:           uuid,
    proposed_decomposition:[{ sequence: int, question: str, skill_suggestion: str | null }],
    mirror_prior_count:    int,        # similar prior requirements found
    mnemosyne_patterns:    int         # relevant institutional knowledge nodes
  }
  SLA: p99 < 3000ms

POST /research/{research_id}/approve-decomposition
  Auth:     analyst session token
  Request:  {
    approved_questions:    [{ subquestion_id: uuid, approved: bool, revised_question: str | null }]
  }
  Response: { research_id: uuid, status: RESEARCHING }

GET /research/{research_id}/partial-answers
  Auth:     analyst session token
  Response: [ResearchSubQuestion with partial_answer populated]

POST /research/{research_id}/approve-partial
  Auth:     analyst session token
  Request:  {
    subquestion_id:        uuid,
    approved:              bool,
    notes:                 str | null
  }
  Response: { subquestion_id: uuid, next_step: str }

POST /research/{research_id}/synthesise
  Auth:     analyst session token
  Response: {
    research_id:           uuid,
    synthesis:             str,
    status:                COMPLETE
  }
  SLA: p99 < 5000ms

GET /research/{research_id}/methodology-trail
  Auth:     analyst session token | supervisor token | IOB token
  Response: ResearchSession with full MethodologyStep array

GET /templates
  Auth:     analyst session token
  Response: [ResearchTemplate]

GET /health
  Response: {
    status, dependencies: { moirai, sks, mirror, mnemosyne, pces, redis },
    active_research_sessions:  int,
    completed_24h:             int,
    mean_subquestions:         float,
    last_event_hash:           str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          STOA_RESEARCH_STARTED
service_id:         "STOA"
session_id:         uuid
classification:     str
event_payload:
  research_id:            uuid
  subquestion_count:      int
  template_used:          str | null
  mirror_prior_count:     int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          STOA_METHODOLOGY_STEP
event_payload:
  research_id:            uuid
  step_type:              str
  skill_version_hash:     str | null
  source_count:           int

EventType:          STOA_RESEARCH_COMPLETE
event_payload:
  research_id:            uuid
  subquestion_count:      int
  gaps_identified:        int
  synthesis_tokens:       int
  total_duration_seconds: int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `STOA_RESEARCH_STARTED` | Research session initialised | MOIRAI, MOS/SAGA (matter context update) |
| `STOA_METHODOLOGY_STEP` | Each methodology step executed | MOIRAI (methodology trail) |
| `STOA_RESEARCH_COMPLETE` | Synthesis complete | MOIRAI, ORACLE (research outcome for pattern learning) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| SKS/DAEDALUS | Skill invocation | Sub-question skill result feeds partial answer |
| RQS/HERMES | Retrieval quality | Coverage gaps trigger gap signal |
| MIRROR | Similar requirement data | Informs decomposition recommendation |
| MNEMOSYNE | Knowledge nodes | Informs decomposition and skill selection |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| SKS/DAEDALUS | Skill Registry | Sub-question skill execution | Sync | Sub-question proceeds with direct prompting; skill versioning unavailable |
| MOIRAI | Provenance | Methodology trail events | Async event | Events buffered; research continues |
| MIRROR | Requirement Similarity | Prior requirement context for decomposition | Sync query | Decomposition proceeds without prior context |
| MNEMOSYNE | Institutional Knowledge | Knowledge patterns for decomposition | Sync query | Decomposition proceeds without institutional context |
| RQS/HERMES | Retrieval Quality | Coverage gap detection per sub-question | Sync | Gaps not detected automatically |
| IAS/SCUDO | Adversarial Screening | Sub-question response screening | Sync | Responses not screened; alert |
| PCES/AEGIS | Classification Enforcement | Session scope on research | Sync | Session scope from cached context |

### 5.2 Feeds Into

| Service | Epithet | What STOA provides | How |
|---|---|---|---|
| ATHENA | Interface | Research session workflow; methodology trail | API |
| DPS/CODEX | Document Provenance | Research sessions contributing to documents | Research_id linked at document creation |
| ORACLE | Outcome Intelligence | Research outcomes for pattern learning | `STOA_RESEARCH_COMPLETE` event |
| TIS/DIKE | Tasking Integration | Collection gaps identified during research | Gap signal via RQS |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Research initialisation + decomposition | 1000ms | 2000ms | 3000ms |
| Sub-question execution (per question) | 2000ms | 5000ms | 10000ms |
| Synthesis | 2000ms | 4000ms | 5000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — STOA unavailability suspends multi-step research; single-turn ATHENA continues |
| RTO | 15 minutes |

### 6.3 Graceful Degradation

| Dependency unavailable | Behavior | Impact |
|---|---|---|
| MIRROR | Decomposition proceeds without prior requirement context | Analyst receives no similar-requirement suggestions |
| MNEMOSYNE | Decomposition proceeds without institutional patterns | Analyst receives no institutional knowledge suggestions |
| SKS/DAEDALUS | Sub-questions proceed with direct prompting | Skill versioning unavailable; methodology trail records unversioned approach |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst session | Research start; partial review; synthesis | Session token |
| Supervisor | Methodology trail review (team scope) | Supervisor token |
| IOB | Full methodology trail access | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Decomposition produces sub-questions that don't address the requirement | Medium | P2 — research misses the analytical question | Analyst review checkpoint | Mandatory analyst review and approval of decomposition before research begins |
| Synthesis contradicts individual partial answers | Low | P2 — synthesis is internally inconsistent | Analyst synthesis review | Partial answers surfaced with synthesis; analyst can reject and revise |
| Research session grows unbounded (many sub-questions, long synthesis) | Low | P2 — token budget exceeded | Sub-question count monitoring | Configurable maximum sub-question count; SCBS token budget applies |

### 8.1 Known Design Risks

- **The decomposition quality depends on the requirement articulation.** A vaguely stated requirement produces a vague decomposition. STOA cannot compensate for imprecise analytical requirements. Resolution path: PRS/PROMETHEUS prompt engineering templates for requirement articulation — helping analysts state requirements precisely before initiating STOA research.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `stoa.research.latency_p99` | Histogram | `> 10s per sub-question` | P2 |
| `stoa.research.abandonment_rate` | Gauge | `> 20%` | P2 |
| `stoa.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/stoa/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every research session's methodology trail is permanently recorded — every sub-question asked, every skill invoked, every source retrieved, every partial answer reviewed, every gap identified. An oversight body can reconstruct the complete research methodology for any STOA-assisted analysis.

---

## 11. Implementation Roadmap

### Phase 1 — Core Orchestration and Methodology Trail (Year 2, Weeks 9–16)

- ResearchSession, ResearchSubQuestion, MethodologyStep schemas
- Research start, decomposition, partial answer, and synthesis endpoints
- SKS/DAEDALUS skill invocation for each sub-question
- Methodology trail MOIRAI events
- Analyst review checkpoints at decomposition and partial answer stages
- Basic research template library (3–5 common requirement types)

**Phase gate criterion:** Complete research session (start → decomposition → research → synthesis → methodology trail) produced for test requirement. Methodology trail MOIRAI events chained correctly. Analyst review checkpoints function as gates — research does not proceed without approval.

### Phase 2 — MIRROR, MNEMOSYNE, and Gap Integration (Year 2, Weeks 17–24)

- MIRROR integration for prior requirement context in decomposition
- MNEMOSYNE integration for institutional knowledge in decomposition
- RQS/HERMES gap detection per sub-question
- ORACLE integration (Year 3 capability; integration point only in Phase 2)
- Research session pause/resume
- Expanded template library

**Phase gate criterion:** MIRROR prior requirements appear in decomposition suggestions. MNEMOSYNE knowledge nodes appear in sub-question context. RQS gap signals generate TIS/DIKE gap requests. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate STOA. The research template library content is maintained by the analytic standards authority as an operational policy artefact.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

From an ATHENA session, the analyst invokes STOA by stating a complex requirement. STOA returns a proposed decomposition (the sub-questions it proposes to research) with the MIRROR prior requirement count and MNEMOSYNE pattern count that informed it. The analyst reviews, approves or modifies, and STOA begins research. After each sub-question, the partial answer surfaces for review before the next sub-question begins. After all sub-questions, STOA proposes a synthesis. The analyst reviews the synthesis against the partial answers and approves or requests revision.

### 13.2 What the Analyst Should Do

Review the decomposition carefully — this is the most important step. If the decomposition misses an important analytical dimension or includes irrelevant sub-questions, modify it before approving. Review each partial answer before the next sub-question begins; conclusions from prior sub-questions inform later ones, so errors compound. At synthesis, compare the synthesis against the partial answers to confirm internal consistency.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q2 gate review.*
