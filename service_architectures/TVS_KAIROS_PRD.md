# TVS — Temporal Validity Service
### KAIROS · *"Greek for 'the right moment' — the qualitative dimension of time, as opposed to chronological sequence; the opportune point at which information is still actionable"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `TVS` |
| **Epithet** | `KAIROS` |
| **Full name** | Temporal Validity Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 5–6 (Weeks 29–46) |
| **Build priority** | 8 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — determines whether intelligence is still valid |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**TVS/KAIROS answers: Is this intelligence source still valid — or has it been superseded, expired, or overtaken by events?**

### 1.2 Why This Service Exists

The model does not know the intelligence picture has changed. A source ingested two years ago is presented with the same confidence as a source ingested this morning. A capability assessment that was accurate before a significant programme event is presented as current intelligence after that event — because from the model's perspective, the retrieved document is the retrieved document. Currency failure is the silent failure mode: there is no signal in the output that a previously valid assessment has been superseded.

Without TVS, the Currency axis of THEMIS exists only as a concept. The provenance record can say when a source was ingested, but it cannot say whether that source is still valid at query time. TVS is the service that makes the Currency axis operational — continuously tracking source validity, propagating invalidation through the MOIRAI provenance graph, and surfacing validity signals in ATHENA so analysts know whether the intelligence they are working with is still current.

### 1.3 Why This Service Is Eighth

TVS requires MOIRAI's provenance graph for blast radius propagation (invalidating a source must propagate to all claims derived from it), requires CVS to have verified source existence (invalid sources to validate are sources that CVS has already confirmed exist in the corpus), and requires FGTS/ALETHEIA for the validity signal to feed the ground truth corpus. Phase 5-6 is appropriate because TVS monitoring becomes meaningful only once substantial analytical sessions have accumulated sources in the provenance graph.

### 1.4 Design Principles

- **Currency failure is silent — KAIROS makes it visible.** The primary design goal is surfacing currency information that would otherwise be invisible. An analyst must know that the source supporting their conclusion was valid yesterday but may not be valid today.
- **Validity is continuous, not binary.** Sources do not flip instantaneously from valid to invalid. Validity decays over time according to source-type-specific decay curves. A SIGINT report from yesterday has a different validity profile than a public-record source from five years ago.
- **Invalidation propagates through the graph.** When a source is invalidated, every claim derived from it in the MOIRAI provenance graph is affected. The blast radius query tells operators which sessions and analysts are working with claims that have currency concerns.
- **Validity scores are evidence of concern, not verdicts.** A low validity score prompts the analyst to investigate currency; it does not automatically invalidate their work.

### 1.5 Explicit Out of Scope

- **Determining why a source became invalid.** TVS detects and records validity change; the intelligence basis for an invalidation decision is an analytical judgment.
- **Replacing the analyst's currency assessment.** TVS provides a technical signal; the analyst's domain judgment about what constitutes currency for a specific source type takes precedence.
- **Collection against superseded topics.** TVS identifies currency gaps; tasking collection to fill them is TIS/DIKE's responsibility.

---

## 2. Core Responsibilities

### 2.1 Primary Function

TVS/KAIROS continuously monitors the validity of sources in the intelligence corpus against configurable decay models and external invalidation events, surfaces validity scores and expiry indicators in ATHENA, propagates invalidation through the MOIRAI provenance graph to identify all claims and sessions affected by a source's currency change, and publishes validity events to the provenance chain so that the currency status of every source at the time it was used in an analytical session is permanently recorded.

### 2.2 Secondary Functions

- Decay model management: configurable validity decay functions by source type, collection method, and domain
- External invalidation: receiving invalidation signals from KCS/ARGUS when sources are superseded by new collection or confirmed events
- Validity-weighted retrieval: providing retrieval systems with validity scores so retrieval ranking can weight current sources higher than stale ones
- ERAS integration: supplying `validity_at_capture` scores for each source cited in a reasoning capture
- Batch validity reassessment: periodically recomputing validity scores across the corpus as time passes

### 2.3 What This Service Does Not Decide

TVS determines whether a source has passed its validity threshold. Whether an analyst may still use a source that is technically stale — because they have domain-specific reasons to believe it remains accurate — is an analytical judgment. TVS flags; the analyst decides.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ValidityRecord:
  record_id:             uuid
  source_id:             uuid              # FK → corpus source
  chunk_id:              str              # retrieval chunk identifier
  source_type:           str              # SIGINT | HUMINT | GEOINT | OSINT | TECHINT | etc.
  ingested_at:           datetime
  decay_model:           str              # FK → DecayModel.model_id
  validity_score:        float            # 0.0–1.0; current validity
  validity_threshold:    float            # below this: flagged for review
  validity_floor:        float            # below this: automatically expired
  last_assessed:         datetime
  status:                VALID | FLAGGED | EXPIRED | INVALIDATED | SUPERSEDED
  invalidated_at:        datetime | null
  invalidation_source:   str | null       # what triggered invalidation
  superseded_by:         uuid | null      # FK → newer ValidityRecord

DecayModel:
  model_id:              uuid
  model_name:            str
  source_type:           str
  collection_method:     str | null
  domain:                str | null
  half_life_days:        float            # time for validity to decay to 0.5
  floor_days:            float            # time for validity to reach validity_floor
  decay_function:        EXPONENTIAL | LINEAR | STEP
  configured_by:         str
  version:               str

InvalidationEvent:
  event_id:              uuid
  source_id:             uuid
  invalidation_type:     SUPERSEDED | CONFIRMED_FALSE | COLLECTION_EXPIRED | MANUAL | ADMIN
  confidence:            HIGH | MEDIUM | LOW
  basis:                 str              # plain language; what triggered invalidation
  triggered_by:          str              # service or analyst who triggered
  timestamp:             datetime
  blast_radius_computed: bool
  blast_radius_summary:  { affected_claims: int, affected_sessions: int }

ValiditySignal:
  signal_id:             uuid
  session_id:            uuid
  turn_id:               uuid
  source_id:             uuid
  validity_at_use:       float            # validity score at the moment the source was used
  flagged:               bool
  flag_reason:           str | null
  surfaced_to_analyst:   bool
  timestamp:             datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | ValidityRecord, DecayModel, InvalidationEvent, ValiditySignal | Indefinite |
| Validity score cache | Redis | Current validity scores for active corpus sources (hot path for retrieval) | 1h TTL + invalidation-event refresh |
| Event store | MOIRAI | Signed validity events | Indefinite |
| Batch reassessment queue | Kafka | Sources due for validity reassessment | 24h retention |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ValidityRecord | Inherits source classification | Compartment-gated per source |
| DecayModel | Controlled Unclassified | Platform metadata; accessible to platform team |
| InvalidationEvent | Inherits source classification | Compartment-gated |
| ValiditySignal | Inherits session classification | Session-compartmented |

### 3.4 Retention and Purge Policy

ValidityRecord retained indefinitely — the historical validity of a source at any point in time is required for retrospective analysis of assessments made when the source was used. InvalidationEvent retained indefinitely. ValiditySignal retained for session lifetime plus seven years. Decay models retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
GET /validity/{source_id}
  Auth:     any service account (PCES-compartment-scoped)
  Response: {
    record:            ValidityRecord,
    current_score:     float,
    status:            str,
    days_since_ingestion: int
  }
  SLA: p99 < 100ms (from Redis cache)

POST /validity/batch
  Auth:     retrieval service account
  Request:  { source_ids: [uuid] }
  Response: [{ source_id, validity_score, status }]
  SLA: p99 < 500ms

POST /invalidate
  Auth:     KCS service account | IOB token | platform operator
  Request:  {
    source_id:         uuid,
    invalidation_type: str,
    confidence:        str,
    basis:             str
  }
  Response: {
    event_id:          uuid,
    blast_radius:      { affected_claims: int, affected_sessions: int, affected_analysts: int }
  }
  SLA: p99 < 2000ms

GET /validity-at/{source_id}?timestamp={datetime}
  Auth:     ERAS service account | IOB token
  Response: { validity_score: float, status: str }   # validity at that historical moment
  SLA: p99 < 500ms

GET /blast-radius/{source_id}
  Auth:     platform operator | IOB token
  Response: {
    affected_claims:   [uuid],
    affected_sessions: [uuid],
    affected_analysts: [str]    # hashed unless IOB token
  }
  SLA: p99 < 5000ms

GET /session/{session_id}/currency-report
  Auth:     session token | supervisor token
  Response: {
    sources_used:      int,
    flagged_sources:   int,
    expired_sources:   int,
    validity_mean:     float,
    flagged_claims:    [{ claim_id, source_id, validity_score }]
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis, kafka },
    corpus_sources_tracked: int,
    flagged_source_count:   int,
    expired_source_count:   int,
    last_batch_reassessment:datetime,
    last_event_hash:        str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          TVS_SOURCE_FLAGGED
service_id:         "TVS"
session_id:         null         # source-level event, not session-scoped
classification:     str
event_payload:
  source_id:              uuid
  validity_score:         float
  status:                 FLAGGED | EXPIRED
  decay_model:            str
  days_since_ingestion:   int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          TVS_SOURCE_INVALIDATED
event_payload:
  source_id:              uuid
  invalidation_type:      str
  confidence:             str
  blast_radius_claims:    int
  blast_radius_sessions:  int

EventType:          TVS_VALIDITY_SIGNAL
event_payload:
  session_id:             uuid
  turn_id:                uuid
  source_id:              uuid
  validity_at_use:        float
  flagged:                bool
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `TVS_SOURCE_INVALIDATED` | Source invalidated via POST /invalidate | MOIRAI blast-radius query, ATHENA (session currency alerts), FGTS (invalidated citations → corrections) |
| `TVS_SOURCE_FLAGGED` | Validity score crosses threshold | MOIRAI, KCS/ARGUS (for external corroboration check trigger) |
| `TVS_VALIDITY_SIGNAL` | Source used in an ERAS reasoning capture | MOIRAI, ERAS (validity_at_capture field) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| KCS/ARGUS | `KCS_SOURCE_SUPERSEDED` | Triggers invalidation event with SUPERSEDED type |
| OFS/NEMESIS | `OFS_ASSESSMENT_DISCONFIRMED` | Triggers invalidation with CONFIRMED_FALSE type for supporting sources |
| External corpus ingestion | New source ingested | Creates ValidityRecord with initial validity = 1.0 |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Blast radius graph traversal; signed event emission | Sync (blast radius) + Async event | Events buffered; blast radius unavailable — alert fires |
| PCES/AEGIS | Classification Enforcement | Source compartment validation on validity queries | Sync | Validity queries proceed with cached session scope |
| KCS/ARGUS | Knowledge Currency | Source supersession signals | Async event | No external supersession signals; internal decay only |

### 5.2 Feeds Into

| Service | Epithet | What TVS provides | How |
|---|---|---|---|
| ATHENA | Interface | Validity scores and currency flags in source badges | API |
| ERAS/LOGOS | Reasoning Audit | `validity_at_capture` for each source cited | `GET /validity-at/` API |
| CVS/VERITAS | Source Corroboration | Validity scores for corroborating sources | API |
| FGTS/ALETHEIA | Ground Truth | Invalidated citation corrections | `TVS_SOURCE_INVALIDATED` event |
| TIS/DIKE | Tasking Integration | Expired source domains as collection gap signals | `TVS_SOURCE_FLAGGED` event feed |
| Retrieval system | N/A | Validity-weighted retrieval ranking | `GET /validity/batch` |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Source validity queries are compartment-gated. An analyst cannot query the validity of a source outside their session's compartment scope.
- **Compartment inheritance:** ValidityRecord inherits the source's classification. Batch validity queries are filtered to accessible sources only.
- **Failure behavior:** PCES unavailable → validity queries proceed with cached session compartment scope. Invalidation events (write path) proceed without PCES gating — source-level operations are not session-scoped.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Single source validity (cached) | 5ms | 15ms | 100ms |
| Batch validity (retrieval ranking) | 50ms | 150ms | 500ms |
| Invalidation (with blast radius) | 500ms | 1500ms | 2000ms |
| Historical validity lookup | 50ms | 200ms | 500ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Validity lookups/second | 500 (retrieval ranking calls) |
| Batch validity requests/second | 50 |
| Invalidation events/hour | 10 (low frequency; deliberate action) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — TVS unavailability means validity scores fall back to cached values |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |
| RPO | 1 hour (validity score cache rebuild from PostgreSQL) |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| Redis (validity cache) | Validity reads from PostgreSQL (higher latency); batch requests may time out | Retrieval ranking may not apply validity weighting; single lookups slower |
| MOIRAI blast radius | Invalidation proceeds; blast radius computation queued for when MOIRAI recovers | Blast radius results delayed; sessions not immediately notified |
| KCS/ARGUS | No external supersession signals; internal decay continues | Validity scores may not reflect external supersession events |

---

## 7. Security Model

### 7.1 Authentication

Retrieval services query validity via service account. Analyst-facing endpoints use session tokens. Invalidation requires KCS service account, IOB token, or operator token. Analyst-initiated invalidation is not permitted — this is a deliberate restriction.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Retrieval service | `GET /validity/*`, `POST /validity/batch` (compartment-scoped) | Service account |
| Analyst session | `GET /session/{id}/currency-report` for own sessions | Session token |
| KCS/ARGUS | `POST /invalidate` with SUPERSEDED type | KCS service account |
| Platform operator | Full invalidation authority | Operator token |
| IOB | `POST /invalidate`; blast radius; historical validity | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/tvs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/tvs/db-credentials` | 30 days |
| Redis credentials | `themis/tvs/redis-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

**Adversarial invalidation**: an attacker who gains access to the invalidation endpoint could invalidate large numbers of legitimate sources, degrading the corpus. Mitigation: invalidation is restricted to KCS service account, operator, and IOB — not analyst-accessible. Bulk invalidation (> 10 sources in one operation) requires IOB token and dual authorization.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Decay model miscalibration (sources expire too fast or too slow) | Medium | P2 — misleading currency signals | Analyst complaint rate on FLAGGED sources; Research & Red Team audit | Decay models configurable; domain experts review by source type |
| Blast radius computation timeout (large graph traversal) | Low | P1 — affected sessions not notified promptly | Blast radius latency monitoring | Async blast radius with immediate acknowledgment; streaming notification |
| Validity cache staleness after rapid invalidation events | Low | P2 — stale validity scores served briefly | Cache invalidation event consumption lag | Event-driven cache invalidation on TVS_SOURCE_INVALIDATED |

### 8.1 Known Design Risks

- **Decay model half-lives are not empirically derived.** The initial decay model half-lives are domain-expert estimates, not empirical measurements. A SIGINT report's actual utility half-life varies enormously by collection type, programme stage, and geopolitical dynamics. The models need to be calibrated against OFS/NEMESIS outcome data over time — an assessment built on a source at 0.3 validity that was later confirmed represents a source that remained valid longer than the decay model predicted. Resolution path: OFS/NEMESIS outcomes feed TVS decay model calibration in Year 2.
- **Validity decay does not capture step-change invalidation.** The decay models assume gradual currency loss. In intelligence analysis, sources often remain fully valid until a specific event makes them instantly stale. The STEP decay function addresses this partially, but requires knowing the step date in advance. External invalidation events (POST /invalidate) are the primary mechanism for step-change invalidation; the decay model is a background signal, not the primary validity determination.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `tvs.validity.lookup.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `tvs.corpus.flagged_ratio` | Gauge | `> 20%` of tracked sources | P2 |
| `tvs.corpus.expired_ratio` | Gauge | `> 10%` of tracked sources | P2 |
| `tvs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `tvs.invalidation.blast_radius.latency_p99` | Histogram | `> 5000ms for 1h` | P1 |
| `tvs.cache.miss_rate` | Gauge | `> 10% on validity lookups` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces, redis, kafka },
  corpus_sources_tracked: int,
  flagged_ratio:          float,
  expired_ratio:          float,
  pending_reassessments:  int,
  last_batch_at:          datetime,
  last_event_hash:        str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":         "ISO-8601",
  "service":           "TVS/KAIROS",
  "event":             "SOURCE_FLAGGED | SOURCE_INVALIDATED | VALIDITY_BATCH | BLAST_RADIUS_COMPUTED",
  "source_id":         "uuid | null",
  "validity_score":    0.0,
  "status":            "string",
  "blast_radius":      { "claims": 0, "sessions": 0 },
  "duration_ms":       0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/tvs/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant

### 10.2 What This Service Attests

The MOIRAI record for TVS proves the currency status of every source used in every analytical session at the time it was used. An oversight body can query TVS events to determine whether an assessment was built on sources that were flagged or expired at use time — even if those sources were later invalidated or restored.

### 10.3 What This Service Cannot Prove

TVS proves the validity score at the time of use, computed from the configured decay model. It does not prove the decay model accurately reflects the intelligence's actual validity. It does not prove that a source with validity_score = 0.9 was actually current intelligence — only that the model had not yet decayed it to the flagging threshold.

---

## 11. Implementation Roadmap

### Phase 1 — Validity Tracking and Decay (Weeks 29–36)

- ValidityRecord schema with initial validity = 1.0 on ingestion
- DecayModel configuration for primary source types (SIGINT, HUMINT, GEOINT, OSINT, TECHINT)
- Batch validity reassessment (daily scheduled job)
- `GET /validity/{source_id}` and `POST /validity/batch` endpoints
- Redis validity score cache with event-driven invalidation
- `TVS_SOURCE_FLAGGED` MOIRAI event emission
- Basic ATHENA source badge integration (validity score display)

**Phase gate criterion:** Validity scores computed and cached for all corpus sources. Decay correctly reduces scores over time in test corpus. ATHENA displays validity badge on source citations.

### Phase 2 — Invalidation, Blast Radius, and ERAS Integration (Weeks 37–46)

- `POST /invalidate` endpoint with blast radius computation
- MOIRAI blast radius traversal integration
- `TVS_SOURCE_INVALIDATED` event and affected session notification
- `GET /validity-at/` historical validity endpoint for ERAS
- KCS/ARGUS supersession event consumption
- Currency report endpoint for session and supervisor
- Validity-weighted retrieval ranking integration

**Phase gate criterion:** Invalidation triggers blast radius and notifies affected sessions within 10 seconds (p95). ERAS receives historical validity scores for reasoning captures. KCS supersession events trigger invalidation. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate TVS deployment.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

Each source citation in ATHENA shows a validity indicator alongside the CVS verification badge: a green clock (VALID, score > 0.7), amber clock (FLAGGED, score 0.3–0.7), red clock (EXPIRED, score < 0.3), or grey X (INVALIDATED). Hovering shows the validity score, the days since ingestion, and the decay model applied.

### 13.2 What the Analyst Should Do

FLAGGED or EXPIRED: verify currency through independent means before relying on this source. Check whether newer collection exists on this topic. If the source is critical to the assessment and currency cannot be confirmed, caveat the assessment explicitly. INVALIDATED: do not use this source. If you believe the invalidation is incorrect, contact your supervisor.

### 13.3 What the Signal Does Not Mean

VALID does not mean the intelligence is accurate — it means the decay model has not yet flagged it. EXPIRED does not mean the intelligence was wrong — it means the platform cannot confirm it remains current. Analysts with domain expertise may have grounds to extend reliance on an expired source; this judgment must be documented in the assessment.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Decay model calibration from outcome data.** The initial decay half-lives are expert estimates. OFS/NEMESIS outcome data will eventually show which source types retained validity longer than the model predicted. The Research & Red Team should design a calibration study of decay parameters after Year 1 outcome data is available.

### 14.2 Operational Assumptions

- **Assumption 1: Source ingestion timestamps are reliable.** TVS computes validity from ingestion date. If source timestamps in the corpus are inaccurate (backdated, missing, or set to ingestion date rather than collection date), decay calculations will be incorrect. Resolution path: corpus ingestion pipeline must capture both collection date and ingestion date; TVS uses collection date where available.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 5 gate review.*
