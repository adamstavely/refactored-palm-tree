# TIS — Tasking Integration Service
### DIKE · *"Greek goddess of justice, moral order, and the way things ought to be — specifically the justice of fair process and rightful procedure; the bridge between what is and what should be done about it"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `TIS` |
| **Epithet** | `DIKE` |
| **Full name** | Tasking Integration Service |
| **Namespace** | `themis-requirements` |
| **Layer** | Intelligence Layer — Intelligence Cycle Completion |
| **Build phase** | Year 2 · Q4 (Addendum F) |
| **Build priority** | 23 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — closes the collection gap loop from detection to tasking to outcome |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**TIS/DIKE answers: How do ATHENA's collection gap signals — detected by ARGUS-LACUNA, surfaced through RQS/HERMES and TVS/KAIROS — become collection requirements that reach the people who can task collection? And how does what gets collected flow back to ATHENA to close the loop?**

### 1.2 Why This Service Exists

The intelligence cycle has a gap. ATHENA surfaces collection gaps: ARGUS-LACUNA identifies what the corpus does not cover, TVS/KAIROS shows sources that have expired, RQS/HERMES flags queries with sparse retrieval coverage. These signals are useful inside ATHENA. But they do not automatically become collection requirements. The gap between "ATHENA knows collection is needed" and "a collection manager tasks collection" is currently manual, informal, and untracked.

TIS/DIKE is the bridge that closes this gap without replacing the existing requirements management infrastructure. THEMIS does not own collection. THEMIS does not task collection. THEMIS generates Collection Gap Requests (CGRs) that integrate into the existing requirements workflow — expressing what ATHENA needs in the vocabulary of the requirements management system, routing them through the appropriate approval process, and tracking them from creation through tasking through collection through outcome.

The return flow is equally important. When collection happens in response to a THEMIS CGR, TIS/DIKE routes the new intelligence back to ATHENA's corpus for ingestion. The loop closes: gap detected → requirement submitted → collection tasked → intelligence ingested → gap filled.

ORACLE's effectiveness feedback is the second output: THEMIS can tell requirements officers which analytical requirements historically produced collection that was most analytically useful, and which produced intelligence that was retrieved but not relied upon. This is the first time the IC has had a data-driven signal of analytical value that can inform requirements prioritisation.

### 1.3 Integration, Not Replacement

TIS/DIKE integrates with existing requirements management systems — it does not replace them. The requirements management system retains full authority over collection tasking. TIS/DIKE submits CGRs into the existing workflow; the requirements management system processes them according to its own priority and approval framework. TIS/DIKE tracks lifecycle status from the requirements management system's APIs.

This is a deliberate design choice. Attempting to replace the requirements management system would make THEMIS's deployment contingent on the larger programme change. TIS/DIKE can be deployed as an add-on that provides value without requiring changes to the existing collection management workflow.

### 1.4 Design Principles

- **CGRs are expressed in requirements-native vocabulary.** The gap identified by ARGUS-LACUNA is expressed in ATHENA's analytical vocabulary. TIS/DIKE translates it into the vocabulary of the requirements management system — the indicators, collection methods, priority criteria, and routing that the system expects.
- **THEMIS does not task collection; it requests it.** The CGR is a request, not a directive. The requirements management system determines whether and how to task collection.
- **ORACLE effectiveness data belongs to the requirements officer, not just to ATHENA.** The signal that a specific collection requirement type historically produced high-analytically-useful intelligence should reach the requirements officer who is prioritising collection. TIS/DIKE surfaces this in the requirements-native interface.
- **The lifecycle record is the accountability artefact.** Every CGR produced by THEMIS, the tasking decision made on it, and the collection outcome (including whether the resulting intelligence was ingested and used) are tracked in a lifecycle record accessible to the IOB.

### 1.5 Explicit Out of Scope

- **Tasking collection directly.** TIS/DIKE submits CGRs to the requirements management system. It does not have authority to task collection itself.
- **Managing existing requirements not originated by THEMIS.** TIS/DIKE tracks the lifecycle of THEMIS-originated CGRs. It does not attempt to manage the full requirements portfolio of the organisation.
- **Corpus ingestion.** TIS/DIKE routes new intelligence to the corpus ingestion pipeline. The ingestion itself is managed by the ingestion infrastructure.

---

## 2. Core Responsibilities

### 2.1 Primary Function

TIS/DIKE converts ATHENA collection gap signals from ARGUS-LACUNA, RQS/HERMES, TVS/KAIROS, and UCS/TYCHE into structured Collection Gap Requests (CGRs) expressed in requirements-native vocabulary, submits them to the external requirements management system via API, tracks their lifecycle from submission through tasking through collection through ATHENA corpus ingestion, and surfaces ORACLE analytical effectiveness data back to requirements officers in their native workflow.

### 2.2 Secondary Functions

- Gap signal aggregation: consolidating related gap signals across multiple sessions and turns into a single coherent CGR rather than submitting redundant individual requests
- CGR priority scoring: computing an analytical priority score for each CGR based on the frequency and recency of the gap signal, the importance of the dependent assessments, and the UCS/TYCHE epistemic urgency indicator
- Requirements officer feedback interface: a lightweight interface where requirements officers can view pending THEMIS CGRs, their analytical context, and their ORACLE effectiveness ratings
- Corpus ingestion routing: when new collection arrives in response to a THEMIS CGR, routing it to the appropriate corpus ingestion pipeline with the CGR context for source attribution
- Effectiveness reporting: generating structured analytical effectiveness reports from ORACLE data in the vocabulary and format that requirements officers use for collection evaluation

### 2.3 What This Service Does Not Decide

TIS/DIKE generates CGRs and priority scores. Whether a CGR should be tasked, how to prioritise competing collection requirements, and whether new collection adequately addresses the stated gap are decisions owned by the requirements officer and the collection manager. TIS/DIKE provides structured inputs to those decisions; it does not make them.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
CollectionGapRequest:
  cgr_id:                  uuid
  gap_signal_ids:          [uuid]           # FK → ARGUS-LACUNA gap signals that generated this CGR
  domain:                  str
  collection_method:       str              # SIGINT | HUMINT | GEOINT | OSINT | TECHINT | etc.
  gap_description:         str              # plain language: what collection is needed
  analytical_context:      str             # why this matters — linked assessments and their stakes
  priority_score:          float            # 0.0–1.0; computed from frequency, recency, stake
  epistemic_urgency:       str              # from UCS/TYCHE — HIGH | MEDIUM | LOW
  requirements_native_form:{}              # structured CGR in the requirements system's format
  submitted_at:            datetime | null
  external_requirement_id: str | null       # ID assigned by requirements management system
  status:                  DRAFT | SUBMITTED | ACCEPTED | TASKED | COLLECTED | INGESTED | CLOSED | DECLINED

CGRLifecycleRecord:
  record_id:               uuid
  cgr_id:                  uuid
  status:                  str              # status at this lifecycle point
  timestamp:               datetime
  actor:                   str              # who/what caused this status transition
  notes:                   str | null
  collection_ref:          str | null       # collection report reference when collected
  ingestion_ref:           str | null       # corpus ingestion reference when ingested

GapSignalAggregate:
  aggregate_id:            uuid
  domain:                  str
  collection_method:       str
  first_detected:          datetime
  last_detected:           datetime
  signal_count:            int              # how many distinct sessions surfaced this gap
  dependent_sessions:      [uuid]           # sessions whose assessments depend on this gap being filled
  cgr_id:                  uuid | null      # the CGR generated from this aggregate

EffectivenessRecord:
  record_id:               uuid
  requirement_type:        str              # type of analytical requirement
  collection_method:       str
  domain:                  str
  analytical_use_rate:     float            # fraction of responses that were retrieved and relied upon
  confirmation_rate:       float | null     # from OFS/NEMESIS if outcomes available
  sample_size:             int
  time_period:             str
  oracle_basis:            bool
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CollectionGapRequest, CGRLifecycleRecord, GapSignalAggregate, EffectivenessRecord | Indefinite |
| Gap signal cache | Redis | Active gap signal aggregates (deduplication) | 24h TTL |
| Event store | MOIRAI | Signed CGR and lifecycle events | Indefinite |
| Requirements system sync | PostgreSQL | Mirrored external requirement status for offline tracking | Per requirements system lifecycle |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| CollectionGapRequest | Inherits session classification of generating sessions | Compartment-gated; requirements officer access per clearance |
| CGRLifecycleRecord | Inherits CGR classification | Compartment-gated |
| EffectivenessRecord | Controlled Unclassified (aggregate, de-identified) | Requirements officer access |

### 3.4 Retention and Purge Policy

CollectionGapRequest and CGRLifecycleRecord retained indefinitely — the history of THEMIS-originated collection requirements is a permanent accountability and effectiveness record. EffectivenessRecord retained indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /gap-requests/generate
  Auth:     ARGUS-LACUNA service account | RQS service account | TVS service account
  Request:  {
    gap_signal_ids:        [uuid],
    domain:                str,
    collection_method:     str,
    gap_description:       str,
    analytical_context:    str,
    epistemic_urgency:     str
  }
  Response: {
    cgr_id:                uuid,
    action:                CREATED | MERGED,    # MERGED if aggregated into existing CGR
    priority_score:        float
  }
  SLA: p99 < 500ms

POST /gap-requests/{cgr_id}/submit
  Auth:     supervisor token | requirements officer token
  Request:  { requirements_native_form: {} | null }
  Response: {
    cgr_id:                uuid,
    external_requirement_id:str,
    submitted_at:          datetime
  }

GET /gap-requests
  Auth:     supervisor token | requirements officer token | IOB token
  Params:   status: str, domain: str, priority_min: float
  Response: [CollectionGapRequest]

GET /gap-requests/{cgr_id}
  Auth:     supervisor token | requirements officer token | IOB token
  Response: { cgr: CollectionGapRequest, lifecycle: [CGRLifecycleRecord] }

POST /gap-requests/{cgr_id}/lifecycle
  Auth:     requirements management system service account | requirements officer token
  Request:  {
    status:                str,
    collection_ref:        str | null,
    ingestion_ref:         str | null,
    notes:                 str | null
  }
  Response: { record_id: uuid }

POST /gap-requests/{cgr_id}/ingest
  Auth:     corpus ingestion service account
  Request:  {
    collection_ref:        str,
    source_ids:            [uuid]            # newly ingested corpus source IDs
  }
  Response: { cgr_id: uuid, status: INGESTED }

GET /effectiveness/report
  Auth:     requirements officer token | supervisor token | IOB token
  Params:   domain: str, collection_method: str
  Response: {
    records:               [EffectivenessRecord],
    recommendation:        str              # plain-language summary for requirements officer
  }

GET /audit/cgr-summary?from={date}&to={date}
  Auth:     IOB token
  Response: {
    period:                { from, to },
    cgrs_generated:        int,
    submitted_rate:        float,
    accepted_rate:         float,
    tasked_rate:           float,
    ingested_rate:         float,
    mean_priority_score:   float
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis, requirements_system_api },
    pending_cgrs:          int,
    submitted_cgrs:        int,
    requirements_system_reachable: bool,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          TIS_CGR_GENERATED
service_id:         "TIS"
session_id:         null              # CGR is cross-session; gap aggregate drives it
classification:     str
event_payload:
  cgr_id:                 uuid
  domain:                 str
  collection_method:      str
  priority_score:         float
  gap_signal_count:       int
  epistemic_urgency:      str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          TIS_CGR_SUBMITTED
event_payload:
  cgr_id:                 uuid
  external_requirement_id:str
  submitted_by:           str

EventType:          TIS_CGR_INGESTED
event_payload:
  cgr_id:                 uuid
  source_count:           int         # number of sources ingested against this CGR

EventType:          TIS_EFFECTIVENESS_UPDATED
event_payload:
  domain:                 str
  collection_method:      str
  analytical_use_rate:    float
  confirmation_rate:      float | null
  sample_size:            int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `TIS_CGR_GENERATED` | Gap signal produces new CGR | MOIRAI, requirements officer notification |
| `TIS_CGR_SUBMITTED` | CGR submitted to requirements system | MOIRAI |
| `TIS_CGR_INGESTED` | Collection ingested and corpus updated | MOIRAI, KCS/ARGUS (coverage map update), ARGUS-LACUNA (gap may be closed) |
| `TIS_EFFECTIVENESS_UPDATED` | EffectivenessRecord updated from ORACLE data | MOIRAI, requirements officer interface |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| CGS/ARGUS-LACUNA | `CGS_GAP_IDENTIFIED` | Triggers CGR generation or gap signal aggregation |
| RQS/HERMES | `RQS_SPARSE_COVERAGE_FLAGGED` | Contributes to gap signal aggregate for affected domain |
| TVS/KAIROS | `TVS_SOURCE_FLAGGED` | Contributes to gap signal aggregate for expired coverage domain |
| UCS/TYCHE | `UCS_PROFILE_RESOLVED` (collection needed) | Contributes to gap signal aggregate with epistemic urgency signal |
| OFS/NEMESIS | `OFS_OUTCOME_CLASSIFIED` | Updates EffectivenessRecord for affected requirement type and collection method |
| ORACLE | Effectiveness data (via API) | Updates EffectivenessRecord |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| CGS/ARGUS-LACUNA | Collection Gap Service | Primary gap signal source | Async event | Gap signals queue; CGRs generated when ARGUS-LACUNA recovers |
| MOIRAI | Provenance | CGR and lifecycle events | Async event | Events buffered; CGR still generated in PostgreSQL |
| PCES/AEGIS | Classification Enforcement | CGR access control | Sync | Proceeds with cached session scope |
| External requirements management system | N/A | CGR submission and lifecycle tracking | Sync API | CGRs remain in SUBMITTED state; lifecycle tracking stalled; alert |
| ORACLE (Year 2+) | Outcome Intelligence | Effectiveness data for requirements reporting | Sync query | Effectiveness records not updated |

### 5.2 Feeds Into

| Service | Epithet | What TIS provides | How |
|---|---|---|---|
| Requirements management system | N/A | Structured CGRs in requirements-native format | External API |
| Corpus ingestion pipeline | N/A | New collection routing with CGR context | `POST /gap-requests/{id}/ingest` |
| KCS/ARGUS | Knowledge Currency | Coverage map update trigger on successful ingestion | `TIS_CGR_INGESTED` event |
| CGS/ARGUS-LACUNA | Collection Gap Service | Gap closure signal when ingestion completes | API notification |
| IOB Reporting | Oversight | CGR summary; effectiveness reports | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** CGR access is compartment-gated. A requirements officer without access to the generating session's compartment cannot view the full analytical context of the CGR — they receive a redacted version with the gap description and collection method but not the underlying assessment context.
- **Cross-clearance handling:** The requirements management system may have a different classification architecture. TIS/DIKE translates CGRs into the requirements system's classification vocabulary before submission, using GC-8 policy for forecast product governance (which applies to TRS-derived CGRs) and the standard gap request policy for others.
- **Failure behavior:** PCES unavailable → CGR generation proceeds; access endpoints unavailable.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| CGR generation | 200ms | 500ms | 1000ms |
| Gap signal aggregation | 50ms | 150ms | 300ms |
| CGR submission to requirements system | 500ms | 2000ms | 5000ms |
| Lifecycle status update | 100ms | 300ms | 500ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Gap signals processed/hour | 200 |
| CGRs generated/day | 20 (low volume; gap signals are aggregated) |
| CGRs submitted/day | 10 (submission requires human review) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — TIS unavailability means gap signals queue; CGRs not generated until recovery |
| MOIRAI event durability | 99.999% |
| RTO | 30 minutes |
| RPO | 1 hour |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Operational impact |
|---|---|---|
| Requirements management system API | CGRs remain in SUBMITTED state; lifecycle tracking stalled; alert P1 | Requirements officers cannot receive THEMIS CGRs via API; manual workaround required |
| ORACLE | Effectiveness records not updated; existing records still served | Requirements effectiveness reporting uses stale data; noted explicitly |
| ARGUS-LACUNA | Gap signals not received; existing CGRs continue lifecycle tracking | No new CGRs generated until ARGUS-LACUNA recovers |

---

## 7. Security Model

### 7.1 Authentication

Gap signal sources use service accounts. Supervisors and requirements officers use role-specific tokens. Requirements management system uses a service account for lifecycle status updates. IOB uses IOB token.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ARGUS-LACUNA / RQS / TVS / UCS | Gap signal submission | Service account |
| Supervisor | CGR review and submission approval; lifecycle view (compartment-scoped) | Supervisor token |
| Requirements officer | CGR view (redacted to clearance level); lifecycle update; effectiveness report | Requirements officer token |
| Requirements management system | Lifecycle status updates | Service account |
| Corpus ingestion | `POST /gap-requests/{id}/ingest` | Service account |
| IOB | Full access including audit summary | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/tis/signing-key` | 90 days |
| PostgreSQL credentials | `themis/tis/db-credentials` | 30 days |
| Requirements system API credentials | `themis/tis/requirements-api-key` | 90 days |
| Redis credentials | `themis/tis/redis-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Requirements management system API unavailable | Medium | P1 — CGRs cannot be submitted | Requirements system health check | CGRs queued in SUBMITTED state; manual submission fallback process documented |
| CGR never tasked (submitted but not actioned) | High (structural) | P2 — collection gap remains unfilled | CGR age monitoring | Alert on CGRs in SUBMITTED state > 30 days; requirements officer escalation |
| Gap signal duplication (same gap submitted many times) | Medium | P2 — requirements system receives redundant requests | GapSignalAggregate deduplication | Aggregation by domain + collection_method + time window before CGR generation |
| Corpus ingestion after collection does not close gap | Medium | P2 — gap technically closed but analytically still open | Post-ingestion gap re-evaluation | TVS/KAIROS and ARGUS-LACUNA re-evaluate after ingestion; gap marked as closed only when coverage density improves |

### 8.1 Known Design Risks

- **The requirements management system API is a hard external dependency.** TIS/DIKE's primary function — getting CGRs into the collection workflow — requires a machine-readable API from the requirements management system. If no such API exists, TIS/DIKE can only generate CGRs that must be manually submitted, which removes the automation value. Resolution path: API access must be confirmed and contracted before Year 2 Q4 deployment. This is an organisational prerequisite, not an engineering task.
- **The CGR vocabulary translation is domain-specific.** Converting ATHENA's gap description into a requirements-native CGR format requires understanding how the specific requirements management system categorises and prioritises requests. The `requirements_native_form` field is a structured translation of the gap description into the system's vocabulary. This translation logic must be developed in collaboration with requirements system owners before Phase 1. It cannot be built generically.
- **ORACLE effectiveness data requires operational maturity to be meaningful.** The most valuable signal TIS/DIKE can provide to requirements officers — which collection types historically produce analytically useful intelligence — requires OFS/NEMESIS outcome data and ORACLE's analytical model to have sufficient training data. In Year 2, this data will be sparse. The effectiveness reports should be clearly marked with their sample size; small sample sizes produce misleading recommendations. Minimum meaningful sample size: 20 requirements with outcome data per domain-collection_method combination.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `tis.cgr.pending_age_max_days` | Gauge | `> 30 days` in SUBMITTED state | P1 |
| `tis.cgr.acceptance_rate` | Gauge | `< 50%` of submitted CGRs | P2 |
| `tis.requirements_system.reachable` | Gauge | `false` | P1 |
| `tis.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `tis.gap_signal.dedup_rate` | Gauge | For monitoring; no alert threshold | Informational |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces, redis, requirements_system_api },
  pending_cgrs:                int,
  submitted_cgrs:              int,
  max_pending_age_days:        int,
  requirements_system_reachable:bool,
  last_event_hash:             str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":           "ISO-8601",
  "service":             "TIS/DIKE",
  "event":               "CGR_GENERATED | CGR_SUBMITTED | LIFECYCLE_UPDATED | CGR_INGESTED",
  "cgr_id":              "uuid",
  "domain":              "string",
  "collection_method":   "string",
  "status":              "string",
  "priority_score":      0.0,
  "duration_ms":         0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/tis/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant

### 10.2 What This Service Attests

The MOIRAI record for TIS proves every collection gap identified by THEMIS that was converted into a CGR, submitted to the requirements management system, and tracked through to ingestion. An oversight body can reconstruct the complete collection tasking history driven by THEMIS gap signals — what was requested, whether it was tasked, whether collection was produced, and whether that collection was ingested into the corpus.

### 10.3 What This Service Cannot Prove

TIS proves the CGR was submitted and the lifecycle was tracked as recorded. It does not prove that the collection that was ingested actually addressed the stated gap — it proves collection was ingested against a CGR. Whether the ingested collection was analytically useful is determined by ATHENA's retrieval quality monitoring (RQS/HERMES) and ultimately by OFS/NEMESIS outcome data.

---

## 11. Implementation Roadmap

### Phase 1 — CGR Generation and Submission (Year 2, Weeks 33–40)

- CollectionGapRequest, GapSignalAggregate, CGRLifecycleRecord schemas
- Gap signal aggregation logic (deduplication by domain + collection_method + time window)
- CGR generation from ARGUS-LACUNA, RQS, TVS, UCS signals
- Priority scoring computation
- Supervisor review and submission workflow
- Requirements management system API integration (requires external API agreement)
- MOIRAI event emission: `TIS_CGR_GENERATED`, `TIS_CGR_SUBMITTED`
- Basic requirements officer view interface

**Phase gate criterion:** Gap signals from ARGUS-LACUNA produce CGRs within 30 minutes (allowing for aggregation window). Supervisor review and submission workflow produces correctly formatted CGR in requirements system. MOIRAI events produced for all CGRs. Requirements management system API confirmed accessible.

### Phase 2 — Lifecycle Tracking, Ingestion Routing, and Effectiveness Reporting (Year 2, Weeks 41–48)

- Lifecycle status update API (requirements system callback)
- Corpus ingestion routing on collection against CGR
- KCS/ARGUS coverage map update trigger on ingestion
- ARGUS-LACUNA gap closure notification
- ORACLE effectiveness data integration
- EffectivenessRecord computation and requirements officer reporting
- IOB audit endpoint
- Gap re-evaluation post-ingestion

**Phase gate criterion:** Lifecycle tracking reflects requirements system status updates. Ingestion against a CGR triggers coverage map update and gap re-evaluation. Effectiveness report generated with ORACLE data (requires ORACLE to have sufficient training data). ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-8 | Forecast product governance — governs how TRS/CHRONOS-derived CGRs are formatted and submitted when they contain forecast-based gap identification | TRS-derived CGRs in Phase 2 |
| External API agreement | Requirements management system must expose a machine-readable API for CGR submission and lifecycle tracking | Phase 1 deployment — without API access, TIS cannot automate CGR submission |

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In ATHENA, the gap notification panel (populated by ARGUS-LACUNA) shows whether a gap signal has been converted into a CGR and the current CGR status: DRAFT, SUBMITTED, TASKED, or INGESTED. When collection arrives against a gap the analyst flagged, ATHENA surfaces a notification: "New collection ingested against a gap you identified. [Domain]. [Collection method]. [Source count] sources now available." The analyst can immediately query against the newly ingested sources.

### 13.2 What the Analyst Should Do

When UCS/TYCHE characterises a gap as epistemic with HIGH urgency, check whether a CGR already exists for this domain and collection method before requesting a new one. Duplicate CGRs waste requirements officer attention. If a gap has been open more than 30 days with no TASKED status, escalate to your supervisor — the requirement may need re-prioritisation.

When new collection arrives against a previous gap: re-run the affected assessment with the new collection before the next analytical cycle. Do not assume the gap is fully closed — check TVS/KAIROS validity on the new sources and RQS/HERMES coverage before treating the gap as filled.

### 13.3 What the Signal Does Not Mean

A CGR status of ACCEPTED does not mean collection will happen — it means the requirements officer has acknowledged the request. TASKED means collection has been directed. INGESTED means intelligence has entered the corpus. Gap closed does not mean all assessments affected by the gap have been updated — it means the collection coverage gap has been addressed; updating assessments is the analyst's responsibility.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Requirements-native CGR format.** The `requirements_native_form` field must be structured in the vocabulary of the specific requirements management system. This is a joint engineering task with requirements system owners. The TIS/DIKE schema provides a flexible container; the translation logic must be configured per requirements system. Resolution path: requirements system owner engagement before Phase 1 begins — minimum 3 months lead time.

### 14.2 Operational Assumptions

- **Assumption 1: Requirements management system API access exists and is accessible from THEMIS infrastructure.** This is the single highest-risk assumption for TIS/DIKE. Without API access, the service cannot automate CGR submission. Resolution path: this must be confirmed as a go/no-go condition for Phase 1 before Year 2 Q4 begins.
- **Assumption 2: Requirements officers will engage with the THEMIS effectiveness reporting interface.** If requirements officers do not use TIS/DIKE's effectiveness reports in their prioritisation decisions, the feedback loop value is zero. Resolution path: requirements officer engagement and training must be part of the Year 2 deployment plan — TIS/DIKE's operational value depends on human adoption, not just technical integration.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — incorporates Addendum F tasking integration service design |

---

## Appendix A: CGR Lifecycle Reference

| Status | Meaning | Owner | Typical Duration |
|---|---|---|---|
| DRAFT | CGR generated; awaiting supervisor review | TIS/DIKE | Hours to days |
| SUBMITTED | Submitted to requirements management system | Requirements system | Days to weeks |
| ACCEPTED | Requirements officer acknowledged; in prioritisation | Requirements officer | Days to weeks |
| TASKED | Collection directed against this requirement | Collection manager | Weeks |
| COLLECTED | Collection produced; intelligence in transit | Collection system | Days |
| INGESTED | Intelligence ingested into ATHENA corpus | Corpus ingestion | Hours |
| CLOSED | Gap addressed; lifecycle complete | TIS/DIKE | — |
| DECLINED | Requirements officer declined; rationale logged | Requirements officer | — |

*Note: Typical durations are illustrative. Actual durations depend on collection method, target difficulty, and requirements prioritisation.*

---

## Appendix D: Red Team Findings

*Pending — Year 2 Q3 gate review. Requirements management system API access must be confirmed before red team assessment can be scoped.*
