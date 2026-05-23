# MOS — Memory Orchestration Service
### SAGA · *"Norse for the accumulated narrative memory of a community — not just what happened, but what was learned, what was settled, and what remains uncertain"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MOS` |
| **Epithet** | `SAGA` |
| **Full name** | Memory Orchestration Service |
| **Namespace** | `themis-knowledge` |
| **Layer** | Intelligence Layer — Knowledge Foundation |
| **Build phase** | Year 2 · Q1–Q2 |
| **Build priority** | 2 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust — manages the memory context that shapes how the AI reasons across sessions |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MOS/SAGA answers: What does ATHENA already know — from prior sessions on this matter, from the analyst's established positions, from institutional analytical history — that should inform this session's context?**

### 1.2 Why This Service Exists

Every ATHENA session starts from the same baseline: the full corpus, no prior context about this analyst's work on this matter, no accumulated knowledge about what has been established or remains uncertain. Session-by-session, the AI reconstructs the same context from scratch. An analyst who has run fifteen sessions on a specific target must re-establish what they know in every new session, and the AI cannot build on prior analytical work.

Human analytical expertise is not stateless. A senior analyst who has spent six months on a specific programme target has an accumulated mental model — what is established, what is uncertain, what has been confirmed, what has been revised. ATHENA without memory treats every session as if this analyst has never worked on this matter before.

MOS/SAGA changes this. It assembles a structured memory context at session initialization — drawing from prior sessions on the same matter, the analyst's established positions and corrections, institutional analytical patterns from MNEMOSYNE, and OGS entity state — and makes this context available to the AI in a compressed, prioritised form that fits within the context window without displacing retrieval.

Memory orchestration is not the same as storing everything. The context window is finite. MOS/SAGA's design problem is selecting and compressing what is most relevant, not storing everything indefinitely. The design principle is: give the AI the structured summary that a well-briefed colleague would have, not the raw archive.

### 1.3 Design Principles

- **Memory is assembled at session start, not streamed during it.** The memory context is computed and compressed before the session begins. Mid-session memory updates are lightweight additions, not full re-computation.
- **Memory types are distinct and assembled separately.** Matter memory, analyst memory, and institutional memory have different provenance, different update rates, and different relevance criteria. They are assembled and weighted separately, then merged into the session context.
- **Compression is analytically informed.** A summary that preserves the most analytically significant facts (established capabilities, open questions, correction history) is more valuable than a summary that preserves the most recent facts. MOS/SAGA uses analytical significance, not recency, as its primary compression criterion.
- **Memory does not override retrieval.** Memory context is injected as structured background. Retrieval is still the primary evidence source. Memory is what the analyst and the AI know going in; retrieval is what they discover during the session.
- **Gaming detection applies to memory-driven sessions.** A session with strong memory context changes the gaming probability calculation — an analyst who verifies claims that memory already established as incorrect shows a different pattern than one who verifies genuinely new claims.

### 1.4 Explicit Out of Scope

- **Long-term storage of session content.** MOIRAI stores all session events. MOS/SAGA stores the compressed memory representation — not the full session transcript.
- **Knowledge extraction from sessions.** MNEMOSYNE extracts institutional knowledge. MOS/SAGA consumes what MNEMOSYNE produces.
- **Entity state management.** OGS manages the entity graph. MOS/SAGA queries OGS for entity context relevant to this session.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MOS/SAGA assembles a structured, compressed memory context at the start of each ATHENA session — drawing from prior session records via MOIRAI, the analyst's correction and calibration history via FGTS/ALETHEIA and TCS/MIMIR, relevant entity state from OGS/YGGDRASIL, and institutional knowledge from MNEMOSYNE — and injects this into the session context in a form the AI can use without displacing retrieval content from the context window.

### 2.2 Secondary Functions

- Matter memory management: maintaining a structured record of what has been established, what is uncertain, and what has been revised across sessions on the same matter or requirement
- Analyst profile assembly: compressing the analyst's calibration history and domain expertise signals into a session-ready profile
- Context window budget management: ensuring the memory injection stays within a configurable token budget, prioritising the highest-significance memory elements when budget is tight
- Session memory update: receiving new analytical conclusions from each session and updating the matter memory record for future sessions
- Cross-session correction tracking: identifying when a claim established as fact in a prior session is subsequently corrected, and surfacing this in the current session's memory context

### 2.3 What This Service Does Not Decide

MOS/SAGA assembles memory context from available records. Whether the assembled context is analytically accurate, whether a prior conclusion should be revised, and whether memory-driven context changes how an analyst should approach a new session are analytical decisions. MOS/SAGA provides structured prior context; the analyst decides how to use it.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
MatterMemory:
  matter_id:               uuid             # FK → external requirement or matter identifier
  domain:                  str
  subject_entity_ids:      [uuid]           # FK → OGS entities this matter concerns
  established_facts:       [MemoryFact]
  open_questions:          [MemoryQuestion]
  revised_positions:       [MemoryRevision]
  last_session_id:         uuid
  last_updated:            datetime
  session_count:           int

MemoryFact:
  fact_id:                 uuid
  content:                 str              # the established fact in plain analytical language
  established_in_session:  uuid
  established_at:          datetime
  evidence_type:           str              # GRND | PARAM | SYNTH | VERIFIED
  confidence:              HIGH | MEDIUM | LOW
  confirmed_by_outcome:    bool             # true if OFS/NEMESIS has confirmed this

MemoryQuestion:
  question_id:             uuid
  content:                 str              # the unresolved analytical question
  first_identified:        datetime
  last_active:             datetime
  gap_cgr_id:              uuid | null      # FK → TIS/DIKE CGR if collection requested

MemoryRevision:
  revision_id:             uuid
  original_fact_id:        uuid
  revised_content:         str
  revision_reason:         str
  revised_in_session:      uuid
  revised_at:              datetime

SessionMemoryContext:
  context_id:              uuid
  session_id:              uuid
  matter_id:               uuid | null
  assembled_at:            datetime
  token_budget:            int
  tokens_used:             int
  context_summary:         str              # the compressed context injected into session
  components_included:     [str]           # what was included (matter, analyst, institutional, entity)
  components_excluded:     [str]           # what was excluded and why (budget constraints)

AnalystMemoryProfile:
  profile_id:              uuid
  analyst_id_hash:         str
  domain_expertise:        [{ domain: str, signal: HIGH | MEDIUM | LOW }]
  known_calibration_gaps:  [{ domain: str, claim_type: str }]
  recent_correction_pattern:str            # compressed summary of recent corrections
  last_updated:            datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | MatterMemory, SessionMemoryContext, AnalystMemoryProfile | Indefinite |
| Memory context cache | Redis | Pre-assembled contexts for active matters (fast session initialization) | 4h TTL |
| Compressed memory store | PostgreSQL (JSONB) | Compressed matter memory snapshots | Indefinite |
| Event store | MOIRAI | Signed memory assembly and update events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| MatterMemory | Inherits matter classification | Compartment-gated; cross-matter queries require compartment overlap |
| AnalystMemoryProfile | Controlled Unclassified | Analyst (own) and supervisor access |
| SessionMemoryContext | Inherits session classification | Session-compartmented |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /contexts/assemble
  Auth:     ATHENA service account
  Request:  {
    session_id:            uuid,
    analyst_session_id:    uuid,
    matter_id:             uuid | null,
    subject_entity_ids:    [uuid],
    token_budget:          int,            # tokens available for memory injection
    domain:                str
  }
  Response: {
    context_id:            uuid,
    context_summary:       str,            # the injected memory text
    tokens_used:           int,
    components:            [str],
    matter_memory_id:      uuid | null
  }
  SLA: p99 < 1000ms

POST /matters/{matter_id}/update
  Auth:     ATHENA service account (session close trigger)
  Request:  {
    session_id:            uuid,
    new_facts:             [{ content, evidence_type, confidence }],
    new_questions:         [{ content }],
    revisions:             [{ original_fact_id, revised_content, reason }]
  }
  Response: { matter_id: uuid, updated: bool }

GET /matters/{matter_id}
  Auth:     analyst session token | supervisor token
  Response: MatterMemory with full fact/question/revision history

GET /matters/{matter_id}/summary
  Auth:     analyst session token
  Response: { matter_id, established_count, question_count, revision_count, last_session_at }

POST /matters
  Auth:     analyst session token | ATHENA service account
  Request:  { matter_id: str, domain: str, subject_entity_ids: [uuid] }
  Response: { matter_id: uuid }

GET /analyst/{analyst_id_hash}/profile
  Auth:     session token (own) | supervisor token | IOB token
  Response: AnalystMemoryProfile

GET /health
  Response: {
    status, dependencies: { moirai, ogs, mnemosyne, fgts, redis },
    active_matters:        int,
    context_assemblies_24h:int,
    cache_hit_rate:        float,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          MOS_CONTEXT_ASSEMBLED
service_id:         "MOS"
session_id:         uuid
classification:     str
event_payload:
  context_id:             uuid
  matter_id:              uuid | null
  tokens_used:            int
  token_budget:           int
  components_included:    [str]
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          MOS_MATTER_UPDATED
event_payload:
  matter_id:              uuid
  session_id:             uuid
  facts_added:            int
  questions_added:        int
  revisions_made:         int
```

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| MOIRAI | `SESSION_TURN_CREATED` | Session content available for end-of-session matter memory update |
| FGTS/ALETHEIA | `FGTS_CORRECTION_WEIGHTED` | Analyst correction pattern updated in AnalystMemoryProfile |
| OFS/NEMESIS | `OFS_OUTCOME_CLASSIFIED` | Memory facts confirmed by outcome marked `confirmed_by_outcome: true` |
| MNEMOSYNE | `MNEMOSYNE_KNOWLEDGE_UPDATED` | Relevant institutional knowledge nodes available for next context assembly |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| OGS/YGGDRASIL | Ontology | Entity state for session context | Sync query | Entity context not included; matter context still assembled |
| MOIRAI | Provenance | Prior session records for matter assembly | Sync query + Async event | Prior session context not available; basic analyst profile still assembled |
| MNEMOSYNE | Institutional Knowledge | Institutional knowledge nodes for session context | Sync query | Institutional knowledge not included; matter and analyst context still assembled |
| FGTS/ALETHEIA | Ground Truth | Analyst correction history for profile | Async event | Analyst profile uses cached version |
| TCS/MIMIR | Trust Calibration | Analyst calibration gaps for profile | Sync query | Calibration gaps not reflected |

### 5.2 Feeds Into

| Service | Epithet | What MOS provides | How |
|---|---|---|---|
| ATHENA | Interface | Session memory context (injected at session start) | API |
| STOA | Research Orchestration | Matter memory as analytical context for research decomposition | API |
| TCS/MIMIR | Trust Calibration | Memory-informed gaming probability signal (memory-driven sessions have different patterns) | API |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Context assembly (cached matter) | 100ms | 300ms | 1000ms |
| Context assembly (cold matter) | 500ms | 1500ms | 2500ms |
| Matter update (session close) | 200ms | 500ms | 1000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — MOS unavailability means sessions start without prior context |
| RTO | 15 minutes |

### 6.3 Graceful Degradation

| Dependency unavailable | Behavior | Impact |
|---|---|---|
| OGS | Entity context not assembled; matter and analyst context still assembled | Session starts without entity relationship context |
| MNEMOSYNE | Institutional knowledge not assembled; matter and analyst context still assembled | Session starts without historical pattern context |
| MOIRAI | Prior session context not available; analyst profile used only | Session starts without matter history |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ATHENA | Context assembly; matter update | Service account |
| Analyst (own) | Matter summary (own matters); analyst profile (own) | Session token |
| Supervisor | Matter history (team scope) | Supervisor token |
| IOB | Full access | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Memory context too large for token budget | Medium | P2 — memory truncated; important context dropped | Token budget tracking | Analytical significance scoring prioritises which elements survive truncation |
| Stale matter memory (facts established before subsequent revision not updated) | Medium | P2 — analyst receives outdated context | Revision tracking; last_updated monitoring | Revisions always supersede original facts in assembled context |
| Cross-matter information leakage (matter A context bleeds into matter B assembly) | Low | P1 — potential intelligence contamination | Compartment check on matter assembly | Hard compartment boundary enforced on matter memory queries |

### 8.1 Known Design Risks

- **The compression problem has no perfect solution.** Deciding which elements of a six-month analytical history are most relevant to compress into 2,000 tokens is an inherently lossy transformation. The analytical significance scoring model will make mistakes. Resolution path: the assembled context is surfaced to the analyst at session start for review — the analyst can flag if important context was omitted, contributing to the significance model's calibration.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `mos.assembly.latency_p99` | Histogram | `> 2500ms for 5m` | P1 |
| `mos.token_budget.utilisation_mean` | Gauge | `> 90%` (compression pressure high) | P2 |
| `mos.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/mos/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every session memory context assembled is permanently recorded with what was included, what was excluded, and how many tokens were used. An oversight body can determine exactly what prior context was available to the AI in any session.

---

## 11. Implementation Roadmap

### Phase 1 — Matter Memory and Session Context (Year 2, Weeks 1–12)

- MatterMemory, MemoryFact, SessionMemoryContext schemas
- Context assembly from MOIRAI prior sessions + OGS entity state
- Matter memory update at session close
- Token budget management and significance-based compression
- ATHENA session initialization integration

**Phase gate criterion:** Context assembly completes within 1000ms for matters with > 5 prior sessions. Token budget respected. Matter memory updates at session close. Analyst-visible context preview in ATHENA.

### Phase 2 — Analyst Profile, Institutional Integration, and Cross-Session Revision (Year 2, Weeks 13–20)

- AnalystMemoryProfile assembled from FGTS and TCS signals
- MNEMOSYNE institutional knowledge integration
- Cross-session revision tracking (revised facts supersede originals)
- OFS/NEMESIS outcome confirmation flagging
- Matter memory quality review interface for analysts

**Phase gate criterion:** Analyst profile reflects FGTS correction history. MNEMOSYNE knowledge nodes included in assembly. Revised facts correctly supersede originals in assembled context. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate MOS. The memory context content policy — what types of prior session content may be included in memory assembly — is an analytic standards authority operational policy, not a GC policy decision.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

At session start, ATHENA shows a memory context panel: "Based on prior work on this matter: [established facts count] established, [open questions count] unresolved, [revisions count] revised since last session." Expanding shows the compressed matter summary and the list of open analytical questions. The analyst can dismiss the context (it will still be injected but not displayed) or mark specific elements as no longer relevant.

### 13.2 What the Analyst Should Do

Review the assembled memory context at session start. If established facts appear to be stale or incorrect, mark them for revision — this improves the matter memory for all analysts working on the same matter. Contribute to open questions by addressing them explicitly during the session — end-of-session updates will mark them as resolved or escalate them.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q1 gate review.*
