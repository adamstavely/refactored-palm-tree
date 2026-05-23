# RQS — Retrieval Quality Service
### HERMES · *"Greek messenger of the gods — he who ensures that what is sent arrives accurately and completely at its destination"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `RQS` |
| **Epithet** | `HERMES` |
| **Full name** | Retrieval Quality Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 5–6 (Weeks 29–46) |
| **Build priority** | 9 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — assesses whether the right intelligence reached the context window |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**RQS/HERMES answers: Did the retrieval system return the right intelligence for this query — and is anything important missing?**

### 1.2 Why This Service Exists

The GRND/PARAM/SYNTH distinction depends on retrieval working correctly. A claim is labelled GRND because it was retrieved — but if the retrieval system returned the wrong chunks, the claim is effectively PARAM dressed as GRND. The source badge says GRND; the actual provenance is parametric, because what was retrieved did not actually inform the claim.

Retrieval quality failures are invisible to analysts. The AI presents retrieved content as if it is the relevant intelligence on the topic. If the retrieval system returned a document that is tangentially related rather than directly relevant, the analyst cannot easily detect this without examining every retrieved chunk. If a highly relevant document exists in the corpus but was not retrieved, the analyst may not know it exists.

RQS/HERMES makes retrieval quality visible: relevance scoring tells the analyst how well the retrieved content matched the query, coverage assessment tells the analyst if important intelligence may have been missed, and the retrieval audit trail records what was retrieved so that the full retrieval context is part of the provenance record.

### 1.3 Why This Service Is Ninth

RQS requires MOIRAI for the retrieval audit trail, PCES for session-scoped corpus access, and TVS for validity-weighted retrieval ranking. It is Phase 5-6 because retrieval quality monitoring requires accumulated data on what retrieval looks like in practice — meaningful relevance and coverage baselines can only be established after a sufficient volume of analytical sessions.

### 1.4 Design Principles

- **Retrieval audit trail is part of the provenance record.** The set of chunks retrieved for a query is as analytically significant as what the AI said about those chunks. If retrieval was poor, the provenance record shows this.
- **Coverage assessment is directional, not exhaustive.** RQS can signal that a topic area has sparse retrieval coverage — not that specific documents were missed. Exhaustive enumeration of missed relevant documents requires knowledge of what the corpus contains, which is itself a retrieval problem.
- **Reformulation recommendations are advisory.** When retrieval quality is low, RQS may suggest query reformulations. These are signals to the analyst, not automated re-queries. Automated re-querying without analyst awareness would change the session context without consent.
- **Quality scores feed calibration, not gating.** Low retrieval quality informs TCS/MIMIR that calibration in this interaction may be less reliable — it does not block the session.

### 1.5 Explicit Out of Scope

- **Retrieval execution.** RQS assesses retrieval results; it does not execute retrieval queries.
- **Corpus construction or indexing.** RQS monitors retrieval quality against an existing corpus; it does not manage the corpus.
- **Determining what documents should be in the corpus.** RQS identifies coverage gaps; filling them is a collection management function handled through TIS/DIKE.

---

## 2. Core Responsibilities

### 2.1 Primary Function

RQS/HERMES receives the retrieval results for each query in an ATHENA session, assesses their relevance to the query, estimates the coverage of the retrieved set relative to what the corpus likely contains on this topic, generates a retrieval quality signal for TCS/MIMIR calibration routing, and publishes the full retrieval context — what was retrieved, its relevance scores, and its coverage assessment — to the MOIRAI provenance record.

### 2.2 Secondary Functions

- Query reformulation recommendations: when retrieval relevance is low, generating alternative query phrasings for analyst consideration
- Session retrieval summary: end-of-session report showing retrieval quality across all turns
- Retrieval pattern monitoring: tracking retrieval relevance and coverage distributions across sessions for Research & Red Team analysis
- STOA integration: providing retrieval quality signals to STOA research orchestration for methodology trail documentation
- Sparse coverage domain flagging: identifying domains with consistently low retrieval coverage for collection gap reporting via TIS/DIKE

### 2.3 What This Service Does Not Decide

RQS assesses retrieval quality. Whether a session with low retrieval quality should be halted, whether the analyst should requery, and whether coverage gaps represent collection requirements are analyst and operational decisions. RQS provides data; humans decide.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
RetrievalAssessment:
  assessment_id:         uuid
  session_id:            uuid
  turn_id:               uuid
  query_text:            str              # the query issued to the retrieval system
  query_type:            str              # SEMANTIC | KEYWORD | HYBRID
  chunks_retrieved:      int
  relevance_scores:      [ChunkRelevance]
  mean_relevance:        float
  coverage_score:        float            # 0.0–1.0; estimated corpus coverage
  coverage_level:        HIGH | MEDIUM | LOW | SPARSE
  quality_signal:        RetrievalQualitySignal
  reformulation_suggestions: [str] | null
  timestamp:             datetime

ChunkRelevance:
  chunk_id:              str
  source_id:             uuid
  relevance_score:       float            # 0.0–1.0
  relevance_tier:        HIGH | MEDIUM | LOW | MARGINAL
  validity_score:        float            # from TVS at retrieval time
  included_in_context:   bool            # whether this chunk was used in context assembly

RetrievalQualitySignal:
  signal_id:             uuid
  assessment_id:         uuid
  quality_level:         HIGH | MEDIUM | LOW | POOR
  sparse_coverage:       bool
  low_relevance:         bool
  calibration_weight_adjustment: float   # multiplier for TCS calibration weight in this session

RetrievalAuditEntry:
  entry_id:              uuid
  session_id:            uuid
  turn_id:               uuid
  retrieval_system:      str             # which retrieval system was used
  query_embedding_ref:   str             # reference to query embedding (not stored in full)
  chunks_returned:       [str]           # chunk IDs
  chunks_used:           [str]           # chunk IDs included in context window
  timestamp:             datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | RetrievalAssessment, ChunkRelevance, RetrievalAuditEntry | Session + 7 years |
| Event store | MOIRAI | Signed retrieval events | Indefinite |
| Pattern analytics | Elasticsearch | Retrieval quality patterns across sessions for Research & Red Team | 2 years |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| RetrievalAssessment | Inherits session classification | Session-compartmented |
| RetrievalAuditEntry | Inherits session classification | Session-compartmented; IOB access for full audit |
| Quality patterns | Controlled Unclassified (aggregated, de-identified) | Research & Red Team access |

### 3.4 Retention and Purge Policy

RetrievalAssessment and RetrievalAuditEntry retained for session lifetime plus seven years. Quality pattern data in Elasticsearch retained for two years (rolling). MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /assess/retrieval
  Auth:     retrieval gateway service account
  Request:  {
    session_id:          uuid,
    turn_id:             uuid,
    query_text:          str,
    query_type:          str,
    chunks_retrieved:    [{ chunk_id, source_id, retrieval_score }],
    chunks_used:         [str]
  }
  Response: {
    assessment_id:       uuid,
    quality_signal:      RetrievalQualitySignal,
    relevance_summary:   { mean: float, high_tier: int, marginal: int },
    coverage_level:      str,
    reformulation_suggestions: [str] | null
  }
  SLA: p99 < 500ms

GET /assessment/{assessment_id}
  Auth:     session token | supervisor token | IOB token
  Response: RetrievalAssessment with ChunkRelevance details

GET /session/{session_id}/quality-summary
  Auth:     session token | supervisor token
  Response: {
    session_id:          uuid,
    turn_count:          int,
    mean_relevance:      float,
    mean_coverage:       float,
    sparse_turns:        int,
    low_quality_turns:   int
  }

GET /audit/{session_id}/retrieval-record
  Auth:     IOB token
  Response: {
    session_id:          uuid,
    audit_entries:       [RetrievalAuditEntry],
    assessments:         [RetrievalAssessment]
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces },
    assessments_today:  int,
    mean_quality_24h:   float,
    last_event_hash:    str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          RQS_RETRIEVAL_ASSESSED
service_id:         "RQS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  assessment_id:          uuid
  chunks_retrieved:       int
  chunks_used:            int
  mean_relevance:         float
  coverage_level:         str
  quality_level:          str
  sparse_coverage:        bool
  calibration_adjustment: float
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          RQS_SPARSE_COVERAGE_FLAGGED
event_payload:
  domain:                 str
  query_pattern:          str          # anonymised query pattern
  coverage_score:         float
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `RQS_RETRIEVAL_ASSESSED` | Every retrieval assessment | MOIRAI, TCS/MIMIR (calibration weight adjustment), ATHENA |
| `RQS_SPARSE_COVERAGE_FLAGGED` | Coverage score < 0.2 in a domain | MOIRAI, TIS/DIKE (collection gap signal), ARGUS-LACUNA |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| TVS/KAIROS | `TVS_SOURCE_INVALIDATED` | Invalidated source chunks flagged as invalid in subsequent relevance scoring |
| PCES/AEGIS | `PCES_SESSION_GRANTED` | Session compartment scope used to filter corpus for coverage assessment |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed retrieval audit events | Async event | Events buffered; assessment still proceeds |
| PCES/AEGIS | Classification Enforcement | Session scope for coverage assessment corpus filtering | Sync | Coverage assessment proceeds with cached session scope |
| TVS/KAIROS | Temporal Validity | Validity scores for retrieved chunks | Sync | Relevance scoring proceeds without validity weighting |

### 5.2 Feeds Into

| Service | Epithet | What RQS provides | How |
|---|---|---|---|
| TCS/MIMIR | Trust Calibration | Retrieval quality signal as calibration weight adjustment | MOIRAI event |
| STOA | Research Orchestration | Retrieval quality for methodology trail documentation | API |
| TIS/DIKE | Tasking Integration | Sparse coverage domain flags as collection gap signals | `RQS_SPARSE_COVERAGE_FLAGGED` event |
| ATHENA | Interface | Quality summary for session header; per-turn coverage indicator | API |
| IOB Reporting | Oversight | Full retrieval audit records | Audit endpoint |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Session token validated on analyst-facing endpoints. Retrieval gateway uses service account.
- **Compartment inheritance:** RetrievalAssessment inherits session classification. Retrieval audit records accessible only to IOB for cross-session analysis.
- **Failure behavior:** PCES unavailable → analyst endpoints unavailable; retrieval gateway assessment proceeds.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Retrieval assessment | 100ms | 300ms | 500ms |
| Session quality summary | 50ms | 200ms | 500ms |
| Full audit record | 200ms | 1000ms | 3000ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Assessments/second | 50 (one per retrieval call, per active analyst) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — RQS unavailability degrades retrieval quality visibility; sessions continue |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |
| RPO | 1 hour |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Events buffered; assessment still produced | No analyst-facing impact; provenance gap logged |
| TVS | Validity-weighted relevance falls back to unweighted relevance | Relevance scores may rank stale sources higher |

---

## 7. Security Model

### 7.1 Authentication

Retrieval gateway uses service account. Analyst session token for session-facing endpoints. IOB token for full audit records.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Retrieval gateway | `POST /assess/retrieval` | Service account |
| Analyst (own session) | Session quality summary | Session token |
| Supervisor | Team session quality summaries | Supervisor token |
| IOB | Full audit records | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/rqs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/rqs/db-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Relevance model miscalibration (poor relevance scoring) | Medium | P2 — misleading quality signals | Research & Red Team benchmarking | Relevance model benchmarked on labelled retrieval test set |
| Coverage overestimation (claims high coverage when corpus is sparse) | Medium | P2 — analyst unaware of intelligence gaps | Coverage calibration against known topics | Coverage model validated on topics with known corpus density |

### 8.1 Known Design Risks

- **Coverage assessment is inherently uncertain.** Determining whether a retrieval result covers the relevant intelligence on a topic requires knowing what the relevant intelligence is — which is what the retrieval is trying to find. Coverage assessment uses indirect signals (query-corpus density, term frequency patterns, cluster analysis of retrieved chunks). These are informative but not reliable. The coverage score should be surfaced as a directional indicator, not a precise measurement.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `rqs.assessment.latency_p99` | Histogram | `> 500ms for 5m` | P1 |
| `rqs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `rqs.mean_relevance_24h` | Gauge | `< 0.4 sustained over 4h` | P2 |
| `rqs.sparse_coverage_rate` | Gauge | `> 30% of sessions` | P2 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces },
  assessments_today:   int,
  mean_relevance_24h:  float,
  sparse_rate_24h:     float,
  last_event_hash:     str
}
```

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/rqs/signing-key`
- **Chain participation:** Yes
- **What it attests:** The full retrieval context for every analytical session turn is permanently recorded — what was retrieved, its relevance, and its coverage assessment. This makes the retrieval context an auditable part of the provenance record alongside the AI's output.

---

## 11. Implementation Roadmap

### Phase 1 — Relevance Scoring and Audit Trail (Weeks 29–36)

- RetrievalAssessment schema and `POST /assess/retrieval` endpoint
- Chunk relevance scoring against query
- RetrievalAuditEntry logging (what was retrieved vs. what was used)
- MOIRAI event emission: `RQS_RETRIEVAL_ASSESSED`
- Basic ATHENA quality indicator

**Phase gate criterion:** Every retrieval in ATHENA produces a RetrievalAssessment and a MOIRAI event. Relevance scores demonstrate meaningful differentiation on test query set.

### Phase 2 — Coverage, Reformulation, and Integration (Weeks 37–46)

- Coverage assessment implementation
- `RQS_SPARSE_COVERAGE_FLAGGED` event and TIS/DIKE integration
- Query reformulation suggestions
- TCS/MIMIR calibration weight adjustment integration
- Session quality summary endpoint
- IOB audit endpoint

**Phase gate criterion:** Coverage assessment demonstrates correlation with known-sparse domains. TCS calibration weight adjustment applied for low-quality retrievals. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate RQS deployment.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In each ATHENA session turn, a retrieval quality indicator shows the coverage level (HIGH / MEDIUM / LOW / SPARSE) and mean relevance. LOW or SPARSE coverage shows an amber indicator with the message: "Retrieval coverage for this query is limited. Important intelligence may not have been retrieved." Query reformulation suggestions are shown as clickable options if the relevance is low.

### 13.2 What the Analyst Should Do

SPARSE coverage: consider whether the query can be reformulated to improve coverage. Review the retrieved chunks to assess whether they address the core analytical question. If coverage appears genuinely limited, this may be a collection gap — note it in your analytical record. LOW relevance: the retrieved content may not be directly relevant to your query. Review chunks carefully before relying on them.

### 13.3 What the Signal Does Not Mean

HIGH coverage does not mean all relevant intelligence was retrieved — only that the corpus appears to have reasonable density on this topic. LOW relevance does not mean the AI's response was wrong — the AI may have produced accurate parametric knowledge despite low retrieval relevance. It means the retrieved content was not the primary basis for the response.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Coverage assessment model validity.** The coverage assessment relies on indirect corpus density signals. The Research & Red Team should validate coverage scores against ground-truth queries where corpus coverage is known. Resolution path: validation study on 50 topics with known corpus density before Phase 2 gate.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 5 gate review.*
