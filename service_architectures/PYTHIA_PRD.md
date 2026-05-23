# PYTHIA — Anticipatory Surfacing Service
### PYTHIA · *"The Oracle at Delphi who spoke before being asked — who saw the shape of what was coming and gave the questioner more than they had thought to ask for"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `PYTHIA` |
| **Epithet** | `PYTHIA` |
| **Full name** | Anticipatory Surfacing Service |
| **Namespace** | `themis-research` |
| **Layer** | Intelligence Layer — Research |
| **Build phase** | Year 2 · Q3 |
| **Build priority** | 8 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin + Currency — surfaces what the analyst should know before they think to ask |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**PYTHIA answers: Based on what this analyst is currently working on — the entities, domains, and analytical questions active in their session — what related intelligence, patterns, or analytical considerations should be surfaced that they have not yet thought to look for?**

### 1.2 Why This Service Exists

The most dangerous intelligence gap is not the one the analyst knows exists — it is the one they do not know to look for. An analyst conducting an assessment of an adversary's technical programme may not realise that: a SIGINT collection gap identified by ARGUS-LACUNA covers precisely the programme stage they are assessing; a SCRIBE diff has updated a key source they relied on in a prior session; MNEMOSYNE has identified a common error pattern for assessments of this type; or MIRROR has found a prior requirement on a structurally similar target that had a DISCONFIRMED outcome for the same claim type.

These signals exist in the platform. Without PYTHIA, each sits in its respective service until the analyst explicitly queries it. PYTHIA monitors active analytical sessions, detects patterns that suggest these signals are relevant, and surfaces them proactively — before the analyst has committed to an approach that would have benefited from knowing them.

Anticipatory surfacing is not a recommendation engine in the commercial sense. It is not "users who worked on this also looked at that." It is a structured pattern-matching service that operates on the analytical context of the session and the accumulated intelligence knowledge of the platform. The signals it surfaces are directly grounded in evidence; the relevance is analytically justified, not inferred from behavioural correlation.

### 1.3 Design Principles

- **Precision over recall.** A signal that is relevant but not surfaced is a missed opportunity. A signal that is surfaced but not relevant creates alert fatigue that trains the analyst to dismiss future signals. PYTHIA optimises for precision — surfacing fewer, higher-confidence signals rather than everything that might be tangentially relevant.
- **Each signal is explicitly grounded.** Every anticipatory signal states where it comes from (MIRROR, MNEMOSYNE, SCRIBE, ARGUS-LACUNA), what specifically it found, and why it is being surfaced now. The analyst can evaluate the relevance judgment; they are not presented with conclusions without reasoning.
- **Signals are advisory, not intrusive.** PYTHIA signals appear in an ambient panel in ATHENA — they do not interrupt the analytical workflow. The analyst decides whether to engage with a signal.
- **Signal quality feeds back into the model.** When an analyst acts on a PYTHIA signal (clicks through, incorporates it into their analysis), this is a positive feedback signal. When they dismiss it, this is a negative signal. PYTHIA tracks acted_upon rates by signal type to calibrate relevance scoring.
- **PYTHIA is Year 2 Q3 because it requires a mature evidence base.** PYTHIA's value is proportional to the depth of the platform's accumulated analytical record. Year 2 Q3 means MIRROR has at least two quarters of profiles, MNEMOSYNE has been extracting institutional knowledge for at least a quarter, and SCRIBE has been tracking document versions for over a year. These are the raw materials PYTHIA works with.

### 1.4 Explicit Out of Scope

- **Generating analytical conclusions.** PYTHIA surfaces signals that may be relevant to the analyst's current work. What to do with those signals is the analyst's analytical decision.
- **Replacing deliberate research.** PYTHIA supplements STOA-driven research; it does not replace the analyst's deliberate research choices.
- **Predicting adversary behaviour.** PYTHIA predicts what the analyst should know, not what the adversary will do.

---

## 2. Core Responsibilities

### 2.1 Primary Function

PYTHIA monitors active ATHENA analytical sessions — tracking the entities, domains, claim types, and collection coverage patterns active in each session — and generates anticipatory signals when it detects pattern matches against MIRROR prior requirements, MNEMOSYNE institutional knowledge, SCRIBE document updates, ARGUS-LACUNA collection gaps, or OGS entity relationships that are relevant to the active session but have not yet been surfaced to the analyst.

### 2.2 Secondary Functions

- Signal prioritisation: ranking signals by relevance score and signal type, presenting the highest-value signals first
- Signal deduplication: suppressing duplicate signals when the analyst has already engaged with the underlying content
- Session pattern detection: identifying when an analyst's session pattern (the order and type of queries being made) suggests they are approaching a common analytical error or gap
- Signal feedback collection: tracking acted_upon and dismissed rates per signal type for model calibration
- Signal expiry: suppressing signals that are no longer relevant as the session advances
- Cross-session anticipation: when an analyst starts a new session on a matter they have worked before, proactively surfacing what changed since their last session (SCRIBE updates, ARGUS-LACUNA gap changes, new MNEMOSYNE nodes)

### 2.3 What This Service Does Not Decide

PYTHIA identifies potentially relevant signals. Whether a signal is actually relevant to the specific analytical question the analyst is addressing, whether to act on it, and how to incorporate it into the analysis are analyst decisions. PYTHIA proposes; the analyst judges.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
AnticipatorySignal:
  signal_id:               uuid
  session_id:              uuid
  signal_type:             SIMILAR_REQUIREMENT |
                           INSTITUTIONAL_PATTERN |
                           DOCUMENT_UPDATE |
                           COLLECTION_GAP |
                           ENTITY_CONNECTION |
                           PRIOR_SESSION_UPDATE |
                           OUTCOME_WARNING
  source_service:          MIRROR | MNEMOSYNE | SCRIBE | ARGUS_LACUNA | OGS | ORACLE
  source_ref_id:           uuid              # ID in the source service
  content:                 str               # the signal in plain analytical language
  justification:           str               # why this is being surfaced now
  relevance_score:         float             # 0.0–1.0
  priority:                HIGH | MEDIUM | LOW
  created_at:              datetime
  expires_at:              datetime          # signals expire as session context changes
  status:                  PENDING | SURFACED | ACTED_UPON | DISMISSED | EXPIRED

SessionProfile:
  profile_id:              uuid
  session_id:              uuid
  active_entities:         [uuid]            # OGS entity IDs active in this session
  active_domains:          [str]
  active_claim_types:      [str]
  query_pattern:           str               # compressed description of session query pattern
  last_updated:            datetime

SignalFeedback:
  feedback_id:             uuid
  signal_id:               uuid
  action:                  ACTED_UPON | DISMISSED
  time_to_action_seconds:  int
  feedback_at:             datetime

SignalModel:
  model_id:                uuid
  signal_type:             str
  precision_rate:          float             # acted_upon / (acted_upon + dismissed)
  surfaced_count:          int
  last_calibrated:         datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | AnticipatorySignal, SessionProfile, SignalFeedback, SignalModel | Session + 2 years |
| Active signal cache | Redis | Pending signals for active sessions (fast ATHENA polling) | Session TTL |
| Event store | MOIRAI | Signed signal and feedback events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| AnticipatorySignal | Inherits session classification | Session-compartmented |
| SessionProfile | Inherits session classification | Session-compartmented |
| SignalFeedback | Controlled Unclassified | De-identified; Research & Red Team access |

---

## 4. API Contract

### 4.1 Endpoints

```
GET /signals/{session_id}/pending
  Auth:     ATHENA service account
  Response: [AnticipatorySignal ordered by priority, relevance_score]
  SLA: p99 < 200ms (from Redis cache)

POST /signals/{session_id}/update-context
  Auth:     ATHENA service account
  Request:  {
    active_entities:       [uuid],
    active_domains:        [str],
    active_claim_types:    [str],
    recent_query_summary:  str
  }
  Response: { signals_generated: int, high_priority_count: int }
  SLA: p99 < 1000ms

POST /signals/{signal_id}/feedback
  Auth:     ATHENA service account
  Request:  { action: ACTED_UPON | DISMISSED }
  Response: { signal_id: uuid, feedback_recorded: bool }

GET /signals/model-quality
  Auth:     Research & Red Team | IOB token
  Response: [SignalModel]

GET /health
  Response: {
    status, dependencies: { moirai, mirror, mnemosyne, scribe, argus_lacuna, ogs, redis },
    active_sessions_monitored:int,
    signals_generated_24h: int,
    acted_upon_rate_24h:   float,
    dismissed_rate_24h:    float,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          PYTHIA_SIGNAL_SURFACED
service_id:         "PYTHIA"
session_id:         uuid
classification:     str
event_payload:
  signal_id:              uuid
  signal_type:            str
  source_service:         str
  priority:               str
  relevance_score:        float
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          PYTHIA_SIGNAL_ACTED_UPON
event_payload:
  signal_id:              uuid
  signal_type:            str
  time_to_action_seconds: int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `PYTHIA_SIGNAL_SURFACED` | Signal created and sent to analyst | MOIRAI |
| `PYTHIA_SIGNAL_ACTED_UPON` | Analyst acts on a signal | MOIRAI, feedback model update |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| SCRIBE | `SCRIBE_CRITICAL_UPDATE_DETECTED` | Checks if affected source_id is in any active session's relied-upon sources; generates DOCUMENT_UPDATE signal |
| ARGUS-LACUNA | `CGS_GAP_IDENTIFIED` | Checks if gap domain matches active sessions; generates COLLECTION_GAP signal |
| MNEMOSYNE | `MNEMOSYNE_KNOWLEDGE_UPDATED` | Checks if updated domain/claim_type matches active sessions |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MIRROR | Requirement Similarity | Similar requirement signals | Sync query on session context update | No SIMILAR_REQUIREMENT signals generated |
| MNEMOSYNE | Institutional Knowledge | Institutional pattern signals | Sync query + Async event | No INSTITUTIONAL_PATTERN signals generated |
| SCRIBE | Document Diff | Document update signals | Async event | No DOCUMENT_UPDATE signals generated |
| CGS/ARGUS-LACUNA | Collection Gap | Gap signals | Async event | No COLLECTION_GAP signals generated |
| OGS/YGGDRASIL | Ontology | Entity relationship signals | Sync query | No ENTITY_CONNECTION signals generated |
| MOIRAI | Provenance | Signed signal events | Async event | Events buffered |

### 5.2 Feeds Into

| Service | Epithet | What PYTHIA provides | How |
|---|---|---|---|
| ATHENA | Interface | Anticipatory signal panel | API |
| Research & Red Team | Signal quality | Acted_upon and dismissed rates by signal type | Signal model API |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Pending signals retrieval (cached) | 10ms | 30ms | 200ms |
| Context update and signal generation | 200ms | 500ms | 1000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — PYTHIA unavailability means proactive signals not generated; sessions continue |
| RTO | 15 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ATHENA | Signal retrieval; context update; feedback | Service account |
| Research & Red Team | Signal model quality | Research team token |
| IOB | Full signal and feedback access | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Alert fatigue (too many low-relevance signals) | Medium | P2 — analyst ignores all signals including high-value ones | Dismissed rate monitoring | Precision threshold: < 30% acted_upon rate triggers relevance model recalibration |
| Signal staleness (session context changes but signals not updated) | Low | P2 — irrelevant signals continue appearing | Signal expiry monitoring | Signals expire on session context update if context drift is significant |

### 8.1 Known Design Risks

- **The relevance model requires acted_upon feedback to calibrate.** Early PYTHIA will have a poorly calibrated relevance model because there is no feedback history. The first months of operation will require deliberate Research & Red Team oversight to prevent either excessive alert fatigue or excessive signal suppression. Resolution path: initial deployment with conservative precision threshold (only HIGH confidence signals surfaced); threshold gradually relaxed as feedback data accumulates.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `pythia.signals.acted_upon_rate` | Gauge | `< 20%` (alert fatigue likely) | P2 |
| `pythia.signals.pending.latency_p99` | Histogram | `> 200ms for 5m` | P1 |
| `pythia.moirai.emit.failure_rate` | Counter | `> 0.1%` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/pythia/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every signal surfaced and every analyst action on that signal is permanently recorded. This provides a complete record of what anticipatory intelligence was available to analysts during their sessions and whether they acted on it.

---

## 11. Implementation Roadmap

### Phase 1 — Session Monitoring and Core Signal Generation (Year 2, Weeks 25–32)

- SessionProfile schema and context update endpoint
- Signal generation from SCRIBE document updates and ARGUS-LACUNA gaps
- Signal cache and pending signals endpoint for ATHENA
- Signal feedback collection
- MIRROR and MNEMOSYNE signal generation (limited until data floor met)

**Phase gate criterion:** SCRIBE and ARGUS-LACUNA signals surface correctly in ATHENA signal panel. Session context update triggers signal refresh. Acted_upon and dismissed feedback recorded. Signal expiry on context change functional.

### Phase 2 — Relevance Calibration, ORACLE Integration, and Cross-Session Surfacing (Year 2, Weeks 33–40)

- Relevance model calibration from feedback data
- ORACLE outcome warning signals (Year 3 capability; integration point prepared)
- Cross-session prior session update signals (what changed since last session on this matter)
- OGS entity relationship signals
- Signal model quality reporting for Research & Red Team

**Phase gate criterion:** Acted_upon rate > 30% after two months of operation. Cross-session update signals surface correctly on matter session resumption. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate PYTHIA.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

An ambient PYTHIA panel in ATHENA shows pending signals sorted by priority. Each signal shows: type (similar requirement, document update, collection gap, etc.), source service, and a one-sentence description. Expanding the signal shows the full content and justification. Two buttons: "Explore this" (marks ACTED_UPON and opens the relevant service) and "Not relevant" (marks DISMISSED).

### 13.2 What the Analyst Should Do

Review HIGH priority signals before significant analytical commitments. The most valuable signals are often OUTCOME_WARNING (MIRROR found a similar requirement with a DISCONFIRMED outcome for this claim type) and DOCUMENT_UPDATE (a source you relied on has been significantly revised). If you regularly dismiss PYTHIA signals, consider whether the signal types being generated are miscalibrated — feedback the Research & Red Team about patterns of irrelevant signals.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q3 gate review.*
