# SCRIBE — Semantic Document Version Diff Service
### SCRIBE · *"The recorder who does not merely copy but interprets — who marks not just what changed but what the change means"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `SCRIBE` |
| **Epithet** | `SCRIBE` |
| **Full name** | Semantic Document Version Diff Service |
| **Namespace** | `themis-knowledge` |
| **Layer** | Intelligence Layer — Knowledge Foundation |
| **Build phase** | Year 2 · Q2 |
| **Build priority** | 3 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — detects analytically significant changes between document versions |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**SCRIBE answers: Between version A and version B of this intelligence source or analytical document, what changed — and is that change analytically significant?**

### 1.2 Why This Service Exists

Intelligence documents are updated. A threat assessment from six months ago has been revised. A collection report on a specific target has a new version with updated observations. A finished intelligence product has been corrected. These updates contain information — sometimes the most important information in the corpus.

Plain text diff tools answer the wrong question. They tell you which bytes changed. An intelligence analyst asking whether a source update is significant needs to know whether the capability assessment changed, whether a timeline shifted, whether a new entity was introduced, whether the confidence level was revised. A change from "may acquire" to "has acquired" is a 13-character text diff and an analytically transformational intelligence development.

SCRIBE applies semantic understanding to document versioning. It identifies not just what text changed but what the analytical significance of that change is — whether it represents a capability revision, a timeline change, a new entity introduction, or a cosmetic editorial update — and routes appropriate notifications to analysts whose work depends on the changed content.

### 1.3 Design Principles

- **Significance is analytical, not textual.** A large text change may be analytically insignificant (a reformatted table). A small text change may be analytically critical (a changed modal verb on a capability claim). SCRIBE classifies significance analytically.
- **Affected claim notification is the primary output.** The most operationally valuable output of SCRIBE is not the diff itself — it is the list of analytical claims in active sessions that depend on the changed source. Analysts working with prior-version evidence need to know their source has changed.
- **Version tracking begins at ingestion.** Sources are versioned from the moment they enter the corpus. SCRIBE cannot retroactively version sources that were ingested before it was deployed, but it can version all sources from deployment forward.
- **SCRIBE does not validate the change.** A new version that revises a prior assessment downward is recorded as a CAPABILITY_ASSESSMENT_REVISED change. Whether that revision is correct is an analytical question; SCRIBE surfaces it.

### 1.4 Explicit Out of Scope

- **Document content storage.** SCRIBE stores the semantic diff and significance classification, not the full document content. The corpus stores content; SCRIBE stores the analytical record of what changed.
- **Triggering re-analysis.** SCRIBE notifies analysts that a source they relied on has changed. Whether to re-run an analysis is the analyst's decision.
- **Handling document creation.** SCRIBE diffs versions; a document's first ingestion is not a diff — it is a new source. DPS/CODEX handles document lifecycle.

---

## 2. Core Responsibilities

### 2.1 Primary Function

SCRIBE detects new versions of corpus sources and analytical documents, performs a semantic diff to identify what changed and classify the analytical significance of each change, identifies all analytical claims in the MOIRAI provenance graph that cited the prior version, and notifies the relevant analysts and sessions that a relied-upon source has a significant update.

### 2.2 Secondary Functions

- Significance threshold management: configurable thresholds for what constitutes CRITICAL vs. HIGH vs. MEDIUM vs. LOW significance
- TVS/KAIROS integration: significant changes may constitute invalidation of the prior source version — SCRIBE generates invalidation signals for TVS when appropriate
- Entity change detection: when a document version introduces a new entity or revises an entity relationship, generating entity update signals for OGS/YGGDRASIL
- Batch diff processing: for bulk document updates (e.g., a collection system sends updated reports weekly), queuing and processing diffs asynchronously
- Historical diff access: allowing analysts to query the full version history and diff chain for any source

### 2.3 What This Service Does Not Decide

SCRIBE classifies the significance of document changes. Whether a significant change warrants immediately re-doing an analysis, whether a source with a revised assessment should be considered invalid, and whether the change represents collection improvement or collection degradation are analytical decisions.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
DocumentVersion:
  version_id:              uuid
  source_id:               uuid             # FK → corpus source identifier
  version_number:          int
  content_hash:            str             # SHA-256 of this version's content
  ingested_at:             datetime
  version_notes:           str | null

DocumentDiff:
  diff_id:                 uuid
  source_id:               uuid
  version_from_id:         uuid
  version_to_id:           uuid
  overall_significance:    CRITICAL | HIGH | MEDIUM | LOW | COSMETIC
  changes:                 [SemanticChange]
  tvs_invalidation_warranted:bool
  ogs_entity_updates:      [uuid]           # entity IDs that need updating
  affected_claim_ids:      [uuid]           # MOIRAI claim IDs that relied on prior version
  affected_session_count:  int
  notifications_sent:      bool
  diffed_at:               datetime

SemanticChange:
  change_id:               uuid
  diff_id:                 uuid
  change_type:             CAPABILITY_ASSESSMENT_REVISED |
                           TIMELINE_CHANGED |
                           CONFIDENCE_LEVEL_REVISED |
                           NEW_ENTITY_INTRODUCED |
                           ENTITY_RELATIONSHIP_CHANGED |
                           SOURCE_REMOVED |
                           FACTUAL_CORRECTION |
                           CONTEXT_ADDITION |
                           EDITORIAL_REVISION |
                           COSMETIC
  significance:            CRITICAL | HIGH | MEDIUM | LOW | COSMETIC
  prior_text_hash:         str             # hash of changed text in prior version
  new_text_hash:           str             # hash of changed text in new version
  semantic_description:    str             # plain language: what this change means analytically
  affected_entity_ids:     [uuid]          # OGS entities involved in this change

DiffNotification:
  notification_id:         uuid
  diff_id:                 uuid
  analyst_id_hash:         str
  session_id:              uuid | null
  claim_ids_affected:      [uuid]
  notification_type:       CRITICAL_UPDATE | SIGNIFICANT_UPDATE | ROUTINE_UPDATE
  sent_at:                 datetime
  acknowledged:            bool
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | DocumentVersion, DocumentDiff, SemanticChange, DiffNotification | Indefinite |
| Diff processing queue | Kafka | Asynchronous diff processing for batch updates | 72h |
| Event store | MOIRAI | Signed diff events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| DocumentDiff | Inherits source classification | Compartment-gated |
| SemanticChange | Inherits source classification | Compartment-gated |
| DiffNotification | Inherits session classification | Session-compartmented |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /versions
  Auth:     corpus ingestion service account
  Request:  {
    source_id:             uuid,
    content_hash:          str,
    version_notes:         str | null
  }
  Response: {
    version_id:            uuid,
    version_number:        int,
    diff_queued:           bool        # true if prior version exists; diff will be async
  }
  SLA: p99 < 200ms

GET /sources/{source_id}/diffs
  Auth:     session token | supervisor token
  Response: [DocumentDiff]  (summary — without full change detail)

GET /diffs/{diff_id}
  Auth:     session token | supervisor token
  Response: DocumentDiff with SemanticChange details

GET /sessions/{session_id}/pending-updates
  Auth:     analyst session token | ATHENA service account
  Response: [
    {
      diff_id:             uuid,
      source_id:           uuid,
      overall_significance:str,
      change_count:        int,
      affected_claims:     [uuid]
    }
  ]
  SLA: p99 < 300ms

POST /notifications/{notification_id}/acknowledge
  Auth:     analyst session token
  Response: { acknowledged_at: datetime }

GET /health
  Response: {
    status, dependencies: { moirai, pces, tvs, ogs, kafka },
    diffs_queued:          int,
    diffs_processed_24h:   int,
    critical_diffs_24h:    int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          SCRIBE_DIFF_COMPLETED
service_id:         "SCRIBE"
session_id:         null
classification:     str
event_payload:
  diff_id:                uuid
  source_id:              uuid
  overall_significance:   str
  change_count:           int
  affected_claim_count:   int
  tvs_invalidation:       bool
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          SCRIBE_CRITICAL_UPDATE_DETECTED
event_payload:
  diff_id:                uuid
  source_id:              uuid
  affected_session_count: int
  dominant_change_type:   str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `SCRIBE_DIFF_COMPLETED` | Document diff completed | MOIRAI, TVS/KAIROS (if invalidation warranted), OGS (if entity updates) |
| `SCRIBE_CRITICAL_UPDATE_DETECTED` | CRITICAL significance diff | MOIRAI, analyst notifications, supervisor alert |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| Corpus ingestion | New source version ingested | Triggers version registration and diff queuing |
| TVS/KAIROS | `TVS_SOURCE_INVALIDATED` | Marks prior document versions as invalidated in version history |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Provenance graph query for affected claims; signed events | Sync query + Async event | Affected claim identification skipped; diff still computed |
| TVS/KAIROS | Temporal Validity | Invalidation signal when document revision warrants it | Async event | TVS not notified; analyst must review manually |
| OGS/YGGDRASIL | Ontology | Entity change signals from document updates | Async event | OGS entity updates deferred |
| Kafka | N/A | Async diff processing queue | Async | Diffs queue in PostgreSQL; processed when Kafka recovers |

### 5.2 Feeds Into

| Service | Epithet | What SCRIBE provides | How |
|---|---|---|---|
| ATHENA | Interface | Pending update notifications for active sessions | API |
| TVS/KAIROS | Temporal Validity | Invalidation signals for significantly revised sources | `SCRIBE_DIFF_COMPLETED` event |
| OGS/YGGDRASIL | Ontology | Entity update signals from document changes | `SCRIBE_DIFF_COMPLETED` event |
| MOS/SAGA | Memory Orchestration | Document revision alerts for matter memory update | API |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Version registration | 50ms | 150ms | 200ms |
| Semantic diff (async) | 2s | 10s | 30s (for large documents) |
| Pending update query (analyst) | 50ms | 150ms | 300ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — SCRIBE unavailability means document version tracking suspended |
| RTO | 30 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Corpus ingestion | Version registration | Service account |
| Analyst session | Pending updates (own session); diff summaries (compartment-scoped) | Session token |
| Supervisor | Team session updates; full diff details | Supervisor token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Semantic diff classification error (CRITICAL classified as LOW) | Medium | P1 — analyst not notified of significant change | Research & Red Team classification benchmarking | Conservative defaults: uncertain significance → HIGH classification |
| Affected claim identification miss (MOIRAI traversal incomplete) | Low | P2 — some analysts not notified | Affected claim count monitoring vs. expected | Alert on diff with high significance but zero affected claims |

### 8.1 Known Design Risks

- **Semantic diff quality depends on the classification model.** Determining whether a textual change represents a CAPABILITY_ASSESSMENT_REVISED versus a CONTEXT_ADDITION requires semantic understanding of intelligence analytical language. The classification model must be trained on intelligence-domain examples — not generic NLP. Resolution path: Research & Red Team to develop labelled training data from historical IC documents before Phase 1.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `scribe.diff.queue_depth` | Gauge | `> 100 pending` | P2 |
| `scribe.diff.critical_rate` | Gauge | `> 10%` of diffs | P2 |
| `scribe.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/scribe/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every document version diff — what changed, its significance, and which analytical claims were affected — is permanently recorded. An oversight body can reconstruct the evolution of any corpus source and identify which analytical sessions were using prior-version evidence.

---

## 11. Implementation Roadmap

### Phase 1 — Version Tracking and Diff Detection (Year 2, Weeks 9–16)

- DocumentVersion and DocumentDiff schemas
- Version registration at corpus ingestion
- Semantic diff pipeline (text diff → semantic change classification)
- MOIRAI provenance graph query for affected claims
- Analyst notification for CRITICAL and HIGH significance diffs
- TVS invalidation signal for significant revisions

**Phase gate criterion:** Document version registration operational. Diff classification produces CRITICAL/HIGH/LOW/COSMETIC correctly on test document pairs. Affected claim notification sent within 60 seconds of diff completion for CRITICAL diffs.

### Phase 2 — Entity Updates, Batch Processing, and Historical Access (Year 2, Weeks 17–24)

- OGS entity update signals from document changes
- Kafka async processing for batch updates
- Historical diff access endpoint
- MOS/SAGA integration for matter memory update
- Analyst acknowledgment tracking

**Phase gate criterion:** OGS entity updates triggered correctly. Batch processing handles 100+ diffs in queue without degradation. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate SCRIBE.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In ATHENA, a notification indicator shows when a source relied upon in the current or a recent session has a significant update. The notification shows: "Source [reference] has been updated. Change: [change_type in plain language]. Significance: [CRITICAL/HIGH]. [N] claims you relied on may be affected." Clicking expands the semantic description of what changed.

### 13.2 What the Analyst Should Do

CRITICAL or HIGH significance updates on relied-upon sources require analyst review before the affected assessment is disseminated. Verify whether the change affects the assessment's central claims. If it does, the assessment should be revised or explicitly caveated before delivery.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q2 gate review.*
