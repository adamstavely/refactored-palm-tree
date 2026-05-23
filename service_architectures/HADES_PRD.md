# HADES — Adversarial Intelligence Repository
### HADES · *"Greek god of the underworld and the realm itself — the place where what has been blocked, detected, or defeated goes; not oblivion but preservation; the carefully tended record of what tried to get through and what succeeded"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `HADES` |
| **Epithet** | `HADES` |
| **Full name** | Adversarial Intelligence Repository |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 5–6 (Weeks 29–46) with Phase 2 expansion at Phase 7–8 |
| **Build priority** | Quality layer — alongside IAS/SCUDO, expanding with ERAS/LOGOS |
| **Owner team** | Research & Red Team (operational); THEMIS Platform Team (infrastructure) |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Cross-cutting — the institutional memory of how the platform has been attacked and where it has failed |

> **Access note:** HADES is not an analyst-facing service. It is accessible exclusively to the Research & Red Team, the THEMIS Platform Team for security maintenance, and the IOB for oversight. Regular analysts, supervisors, and standard service accounts do not have access. This restriction is itself a security measure — the contents of HADES describe precisely how to attack the platform.

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**HADES answers: Across all adversarial events the platform has encountered — what was attempted, what was caught, what got through, and what does the accumulated record reveal about the attack surface, model failure modes, and gaps in the detection layer?**

### 1.2 Why This Service Exists

Every quality layer service produces adversarial intelligence as a byproduct of its primary function. IAS/SCUDO records what it blocked and what it did not. ERAS/LOGOS records reasoning sessions where adversarial content may have reached the context window. CVS/VERITAS records fabricated citations and corpus poisoning indicators. MAS/EIDOLON records high-risk media artifacts with manipulation signatures. ADS/CASSANDRA records anomalies that were later determined to have adversarial origins.

Without HADES, these records are siloed. IAS/SCUDO's adversarial corpus informs IAS/SCUDO's next catalog update. It does not inform CPS/APORIA's capability zone assessment. It does not inform MNEMOSYNE's institutional knowledge extraction. It does not provide the Research & Red Team with a unified picture of the platform's adversarial exposure over time.

HADES aggregates the adversarial record across all quality layer services into a single, restricted-access repository that functions as the platform's institutional memory of adversarial experience. This is the service the ERAS architecture document referenced from the beginning: adversarial reasoning records, indexed as a distinct HADES category, are the highest-value content for understanding model failure modes. They describe not just what failed but *how* the model reasoned when it was being deceived.

The name Hades is apt. This is not the discard pile — it is the underworld. Nothing that enters HADES is destroyed. Every adversarial event is preserved, structured, and made queryable. The underworld remembers everything.

### 1.3 The Original Design Placement

HADES was specified in the original THEMIS governance foundation alongside PCES, PGS, MOIRAI, MIMIR, FGTS/ALETHEIA, TVS/KAIROS, RQS/HERMES, KCS/ARGUS, and ERAS/LOGOS. It was referenced in the ERAS architecture as the designated repository for adversarial reasoning records. That it was not separately specified as a full service PRD until now reflects not a design omission but a documentation gap — HADES was always load-bearing infrastructure for the Research & Red Team's ability to improve the platform's adversarial robustness over time.

### 1.4 Design Principles

- **HADES contains the platform's most sensitive security content.** The adversarial record describes attack vectors, detection gaps, and failure modes. This is precisely the content an adversary would want. Access is restricted to the Research & Red Team, Platform Team security function, and IOB. No analyst or supervisor access. No service account access except for designated ingestion pipelines.
- **Ingestion is automatic; analysis is human.** HADES ingests adversarial events from all quality layer services automatically when they occur. Analysing what those events mean — what they reveal about the attack surface, what they imply for catalog or zone updates — is a Research & Red Team function.
- **The adversarial corpus is classified at the level of the most sensitive event it contains.** Events in HADES may describe adversarial techniques at multiple classification levels. The repository as a whole is classified at the highest classification of any event it contains.
- **HADES feeds improvement cycles, not operational decisions.** HADES does not surface real-time signals to analysts. It feeds IAS/SCUDO catalog updates, CPS/APORIA zone calibration, and MNEMOSYNE institutional knowledge — all improvement pipelines, not operational analytical sessions.
- **The adversarial reasoning record is the crown jewel.** Among all content types in HADES, the adversarial reasoning record from ERAS/LOGOS — the full chain-of-thought of the AI as it was being manipulated — is the most analytically valuable for understanding how adversarial techniques work against this specific model. It reveals the mechanism of failure, not just the fact of failure.

### 1.5 Explicit Out of Scope

- **Real-time alerting.** IAS/SCUDO, ADS/CASSANDRA, and SENTINEL handle real-time adversarial alerting. HADES is a retrospective repository.
- **Automated threat response.** HADES informs threat response improvements through the Research & Red Team analysis cycle. It does not trigger automated responses.
- **Analyst-facing adversarial intelligence.** Analysts see the effects of adversarial detection (blocks, flags, session alerts) but not the adversarial intelligence that drove those detections.

---

## 2. Core Responsibilities

### 2.1 Primary Function

HADES aggregates adversarial events from all quality layer services — IAS/SCUDO blocks and bypasses, ERAS/LOGOS adversarial reasoning records, CVS/VERITAS fabrication detections, MAS/EIDOLON high-risk media assessments, ADS/CASSANDRA adversarially-attributed anomalies, and WSF/LACHESIS adversarially-attributed signal fusions — into a unified, restricted-access adversarial intelligence repository that the Research & Red Team can query to understand the platform's adversarial exposure, identify systematic failure patterns, and produce the catalog updates and zone recalibrations that improve platform resilience.

### 2.2 Secondary Functions

- Adversarial pattern taxonomy: maintaining a structured taxonomy of adversarial technique types across all event categories
- Failure mode extraction: identifying systematic patterns in the adversarial record that indicate model failure modes (distinct from the technique that exploited them)
- IAS/SCUDO catalog feed: providing the Research & Red Team with structured adversarial pattern data for IAS/SCUDO threat catalog updates
- CPS/APORIA failure mode feed: providing model failure mode data for capability zone reassessment
- MNEMOSYNE feed: providing de-identified adversarial institutional knowledge nodes (what the platform has learned about adversarial exposure without revealing specific techniques)
- Retrospective session analysis: allowing the Research & Red Team to query whether a specific historical session contained adversarial content that was not detected at the time
- IOB adversarial intelligence reporting: producing periodic adversarial exposure summaries for IOB oversight review

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
AdversarialEvent:
  event_id:                uuid
  source_service:          IAS_SCUDO | ERAS_LOGOS | CVS_VERITAS | MAS_EIDOLON |
                           ADS_CASSANDRA | WSF_LACHESIS
  source_event_id:         uuid             # FK → source service event
  session_id:              uuid | null
  event_type:              AdversarialEventType
  technique_category:      str              # adversarial technique taxonomy entry
  severity:                CRITICAL | HIGH | MEDIUM | LOW
  outcome:                 BLOCKED | BYPASSED | PARTIALLY_BYPASSED | UNKNOWN
  content_hash:            str              # hash of adversarial content (not stored plain)
  analysis_notes:          str | null       # Research & Red Team annotation
  classification:          str
  ingested_at:             datetime

AdversarialEventType:
  # IAS/SCUDO sourced
  PROMPT_INJECTION_BLOCKED
  PROMPT_INJECTION_BYPASSED       # retrospectively identified bypass
  CORPUS_POISONING_DETECTED
  INDIRECT_INJECTION_BLOCKED
  # ERAS/LOGOS sourced
  ADVERSARIAL_REASONING_RECORD    # chain-of-thought during adversarial session
  REASONING_MANIPULATION_DETECTED # reasoning was manipulated but detected
  REASONING_MANIPULATION_BYPASSED # manipulation not detected at session time
  # CVS/VERITAS sourced
  FABRICATED_CITATION_DETECTED
  CORPUS_POISONING_INDICATOR
  # MAS/EIDOLON sourced
  SYNTHETIC_MEDIA_HIGH_RISK
  SYNTHETIC_MEDIA_CONFIRMED       # confirmed synthetic after initial assessment
  # ADS/CASSANDRA sourced
  ANOMALY_ADVERSARIAL_CONFIRMED
  # WSF/LACHESIS sourced
  ADVERSARIAL_SIGNAL_FUSION

AdversarialReasoningRecord:
  record_id:               uuid
  event_id:                uuid             # FK → AdversarialEvent (ADVERSARIAL_REASONING_RECORD)
  eras_capture_id:         uuid             # FK → ERAS/LOGOS ReasoningCapture
  adversarial_content_type:str              # what type of adversarial content was present
  detected:                bool             # was the adversarial manipulation detected?
  detection_service:       str | null       # which service detected it
  chain_of_thought_hash:   str              # SHA-256 of the chain-of-thought (not stored plain)
  manipulation_mechanism:  str | null       # Research & Red Team analysis: how it worked
  model_version:           str

TechniqueEntry:
  technique_id:            uuid
  technique_name:          str
  technique_category:      str              # taxonomy category
  first_observed:          datetime
  last_observed:           datetime
  event_count:             int
  bypass_rate:             float            # fraction of events with BYPASSED outcome
  description:             str
  ias_catalog_ref:         str | null       # IAS/SCUDO catalog entry if published

FailureMode:
  failure_id:              uuid
  model_version:           str
  domain:                  str | null
  claim_type:              str | null
  failure_description:     str
  evidence_event_ids:      [uuid]           # AdversarialEvent IDs supporting this finding
  first_documented:        datetime
  cps_zone_impact:         str | null       # recommended CPS zone recalibration
  remediation_status:      OPEN | ADDRESSED | ACCEPTED_RISK | MONITORING
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | AdversarialEvent, TechniqueEntry, FailureMode (metadata only; content hashes only) | Indefinite |
| Adversarial content store | Elasticsearch (air-gapped index) | Queryable adversarial content — highest classification store | Indefinite |
| Reasoning record index | Elasticsearch (air-gapped index) | AdversarialReasoningRecord — classified separately | Indefinite |
| Event store | MOIRAI | Signed ingestion events (metadata only; no adversarial content) | Indefinite |

**Critical storage note:** The adversarial content store and reasoning record index are maintained as air-gapped Elasticsearch indices — isolated from the main THEMIS Elasticsearch infrastructure. No service account that reaches HADES content can reach any other THEMIS service, and vice versa. Access is exclusively through a dedicated Research & Red Team terminal interface.

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| AdversarialEvent metadata | SECRET minimum | Research & Red Team and IOB only |
| Adversarial content (hashed) | Classification of source session | Air-gapped storage; no programmatic access |
| AdversarialReasoningRecord | Classification of source session | Separately classified; highest-value content |
| TechniqueEntry | SECRET minimum | Research & Red Team only |
| FailureMode | Controlled Unclassified (findings) | Platform Team security function and IOB |

### 3.4 Retention and Purge Policy

All HADES records are retained indefinitely. The adversarial record is a permanent intelligence asset. No record is purged without IOB authority, and purge of any MOIRAI-ingestion-attested event requires documenting the hash-chain gap. The assumption is that the adversarial corpus grows perpetually and its value compounds with age.

---

## 4. API Contract

HADES does not expose standard REST endpoints to the broader platform. All access is through a dedicated Research & Red Team interface, not through service accounts. The following describes the Research & Red Team query interface.

### 4.1 Research & Red Team Query Interface

```
# All endpoints require Research & Red Team token or IOB token.
# No service account access. No session token access.

GET /events
  Params:   source_service, event_type, outcome, severity, from, to
  Response: [AdversarialEvent] (metadata; content referenced by hash)

GET /events/{event_id}
  Response: AdversarialEvent with TechniqueEntry and linked FailureMode

GET /reasoning-records
  Params:   model_version, detected: bool, from, to
  Response: [AdversarialReasoningRecord]

GET /reasoning-records/{record_id}
  Response: AdversarialReasoningRecord with full manipulation mechanism annotation

GET /techniques
  Params:   category, min_event_count, bypass_rate_min
  Response: [TechniqueEntry] with bypass rates and IAS catalog linkage

GET /failure-modes
  Params:   model_version, domain, remediation_status
  Response: [FailureMode] with evidence count and CPS zone impact

POST /failure-modes/{failure_id}/annotate
  Auth:     Research & Red Team token
  Request:  { analysis: str, cps_zone_impact: str, remediation_status: str }
  Response: { failure_id: uuid, updated: bool }

GET /reports/exposure-summary?model_version={version}&from={date}&to={date}
  Auth:     Research & Red Team token | IOB token
  Response: {
    period:                { from, to },
    total_events:          int,
    by_source_service:     { service: count },
    by_outcome:            { BLOCKED: int, BYPASSED: int, PARTIAL: int },
    bypass_rate:           float,
    new_techniques:        int,
    open_failure_modes:    int,
    reasoning_records:     int,
    undetected_count:      int   # events where manipulation was not caught
  }

GET /health
  Auth:     Platform Team security function token
  Response: {
    status, dependencies: { moirai, eras, ias_ingest, cvs_ingest },
    total_events:          int,
    bypass_events:         int,
    reasoning_records:     int,
    last_ingestion:        datetime,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

HADES emits minimal metadata events to MOIRAI. Adversarial content does not appear in MOIRAI events — only the fact of ingestion.

```yaml
EventType:          HADES_EVENT_INGESTED
service_id:         "HADES"
session_id:         uuid | null
classification:     CONTROLLED_UNCLASSIFIED    # metadata only; no adversarial content
event_payload:
  event_id:               uuid
  source_service:         str
  event_type:             str
  outcome:                str
  severity:               str
  # No content hashes, no technique details in MOIRAI
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          HADES_FAILURE_MODE_DOCUMENTED
event_payload:
  failure_id:             uuid
  model_version:          str
  remediation_status:     str
  # No technique details
```

### 4.3 Ingestion Events Consumed

HADES consumes events from quality layer services through a dedicated, one-way ingestion pipeline — not through the standard MOIRAI event bus. The ingestion pipeline is write-only from the quality layer perspective: once an adversarial event is written to HADES, no quality layer service can read from HADES.

| Source service | Event type | HADES ingestion |
|---|---|---|
| IAS/SCUDO | `IAS_INPUT_BLOCKED`, `IAS_CHUNK_BLOCKED`, `IAS_ADVERSARIAL_SESSION_FLAGGED` | ALL blocked events; plus retrospective bypass identifications |
| ERAS/LOGOS | `ERAS_CAPTURE_CREATED` (flagged sessions) | Adversarial session captures only |
| CVS/VERITAS | `CVS_FABRICATION_DETECTED` | All fabrication detection events |
| MAS/EIDOLON | `MAS_HIGH_RISK_DETECTED`, `MAS_ASSESSMENT_COMPLETE` (HIGH risk) | All HIGH risk media assessments |
| ADS/CASSANDRA | `ADS_ANOMALY_CONFIRMED` (adversarially attributed) | Anomalies confirmed as adversarial |
| WSF/LACHESIS | `WSF_FUSION_ESTABLISHED` (adversarially attributed) | Adversarially attributed fusions |

---

## 5. Integration Map

### 5.1 Depends On (Ingestion Sources)

| Service | Epithet | What HADES receives | Ingest type |
|---|---|---|---|
| IAS/SCUDO | Adversarial Screening | All blocks; retrospective bypass events | One-way ingest |
| ERAS/LOGOS | Reasoning Audit | Adversarial reasoning records (flagged sessions) | One-way ingest |
| CVS/VERITAS | Source Corroboration | Fabrication detections | One-way ingest |
| MAS/EIDOLON | Media Authenticity | HIGH risk media assessments | One-way ingest |
| ADS/CASSANDRA | Anomaly Detection | Adversarially confirmed anomalies | One-way ingest |
| WSF/LACHESIS | Weak Signal Fusion | Adversarially attributed fusions | One-way ingest |
| MOIRAI | Provenance | Signed ingestion event emission | Async |

### 5.2 Feeds Into (Improvement Cycles — Research & Red Team mediated)

| Destination | What HADES provides | How |
|---|---|---|
| IAS/SCUDO threat catalog | New adversarial patterns for catalog update | Research & Red Team review → catalog submission |
| CPS/APORIA zone calibration | Model failure mode evidence | Research & Red Team review → zone evaluation submission |
| MNEMOSYNE | De-identified adversarial institutional knowledge | Research & Red Team review → knowledge node submission (de-identified) |
| IOB | Adversarial exposure summary reports | Direct query via IOB token |

**Critical design note:** HADES never directly writes to IAS/SCUDO, CPS/APORIA, or MNEMOSYNE. The Research & Red Team reads from HADES, analyses the patterns, and then submits catalog updates, evaluation submissions, and knowledge nodes through those services' normal governance workflows. The human analytical review is mandatory — HADES does not autonomously update the threat catalog.

### 5.3 PCES/AEGIS Integration

HADES does not integrate with PCES/AEGIS for session-scope enforcement because HADES is not session-facing. Access control is at the service infrastructure level: HADES's Elasticsearch indices and PostgreSQL schemas are accessible only from designated Research & Red Team terminals and IOB workstations. There are no REST endpoints exposed to the general THEMIS service mesh.

---

## 6. Non-Functional Requirements

### 6.1 Latency

HADES is not on the critical path for any analytical operation. No latency requirements on query responses (Research & Red Team analytical queries may take seconds to minutes). Ingestion latency: events should be ingested within 60 seconds of source event occurrence.

### 6.2 Availability and Durability

| Metric | Target |
|---|---|
| Ingestion availability | 99.5% — events that cannot be ingested are queued and replayed |
| Adversarial content durability | 99.9999% — the adversarial corpus must not be lost |
| Query availability | 99.0% — Research & Red Team analytical access |
| RTO | 60 minutes — not a production critical path service |

### 6.3 No Graceful Degradation for Access Control

If the access control infrastructure for HADES is unavailable, the service is unavailable. There is no fallback mode that opens access. The adversarial content in HADES is the most sensitive security content in the platform.

---

## 7. Security Model

### 7.1 Authentication

Research & Red Team tokens issued by a separate identity provider from the main THEMIS analyst identity provider. IOB tokens similarly separate. Platform Team security function tokens for health monitoring only. No analyst session tokens. No supervisor tokens. No general service account tokens.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Research & Red Team | Full read; failure mode annotation | Dedicated R&RT token |
| IOB | Exposure summary reports; failure mode read | IOB token |
| Platform Team security function | Health check only | Security function token |
| Ingestion pipeline | Write-only from designated source services | Dedicated ingest service account per source |
| NO other callers | — | No access |

### 7.3 Network Isolation

HADES Elasticsearch indices are deployed on a network segment isolated from the main THEMIS service mesh. The ingestion pipeline is a one-way write connector — it can deliver events to HADES but cannot receive responses (beyond acknowledgment). The Research & Red Team query interface is accessible only from designated terminals on a separate network segment. This is not a software access control — it is a network architecture requirement.

### 7.4 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/hades/signing-key` | 90 days |
| PostgreSQL credentials | `themis/hades/db-credentials` | 30 days |
| Elasticsearch credentials (air-gapped) | Managed separately from main Vault | 30 days |
| Ingest service account credentials | `themis/hades/ingest/{source}-key` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Ingestion pipeline backlog (source events not ingested promptly) | Low | P2 — adversarial events delayed in HADES | Ingestion lag monitoring | Event queue with 72h retention; replay on recovery |
| Air-gapped Elasticsearch index unavailable | Low | P2 — Research & Red Team cannot query adversarial content | Health check | PostgreSQL metadata still queryable; content access restored on recovery |
| Bypass event not identified retrospectively | Medium | P2 — HADES record is incomplete | Research & Red Team periodic adversarial session review | Quarterly Research & Red Team review of flagged sessions for retrospective bypass identification |

### 8.1 Known Design Risks

- **The bypass record is necessarily incomplete.** HADES records adversarial bypasses that are *retrospectively identified*. Events where adversarial manipulation succeeded and was never identified are not in HADES — they are in the main analytical record, undetected. The Research & Red Team's periodic retrospective review programme (Section 8: Failure modes) is the mechanism for identifying unrecognised bypasses, but it will never achieve complete coverage. This is an honest limitation: HADES knows what it knows it doesn't know, not what it doesn't know it doesn't know.
- **The adversarial reasoning record is also a model capability exposure.** The chain-of-thought records in HADES reveal how the model reasons when being manipulated — which is the most valuable information for the Research & Red Team to improve defences. It is also the most sensitive information for an adversary who wants to know how to attack the model more effectively. The air-gapped storage and restricted terminal access are the physical security controls; they must be treated as a physical security requirement, not a software configuration.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `hades.ingest.lag_minutes` | Gauge | `> 60` minutes | P2 |
| `hades.bypass_rate` | Gauge | `> 5%` of total events | P1 |
| `hades.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `hades.failure_modes.open_critical` | Gauge | `> 0` | P1 |

### 9.2 Health Check

```
GET /health
  Auth:     Platform Team security function token
  Response: {
    status:                  "healthy" | "degraded" | "unavailable",
    dependencies: {
      moirai:                "healthy" | "unavailable",
      postgresql:            "healthy" | "unavailable",
      elasticsearch_airgap:  "healthy" | "unavailable"
    },
    ingest_lag_minutes:      int,
    total_events:            int,
    bypass_event_count:      int,
    open_failure_modes:      int,
    last_event_hash:         str
  }
```

### 9.3 Log Schema

Logs for HADES are written to a dedicated, air-gapped log store — not the main THEMIS logging infrastructure. No adversarial content or technique detail appears in logs.

```json
{
  "timestamp":          "ISO-8601",
  "service":            "HADES",
  "event":              "EVENT_INGESTED | FAILURE_MODE_DOCUMENTED | QUERY_SERVED",
  "source_service":     "string",
  "event_type":         "string",
  "outcome":            "BLOCKED | BYPASSED | PARTIAL | UNKNOWN",
  "severity":           "string",
  "duration_ms":        0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/hades/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant (metadata only; no adversarial content in the chain)

### 10.2 What This Service Attests

The MOIRAI record for HADES proves that specific adversarial events were received from specific source services and ingested into the adversarial repository at specific times. An oversight body can verify from the MOIRAI chain that adversarial events were not suppressed or selectively excluded from the repository.

### 10.3 What This Service Cannot Prove

HADES attests what it was told about adversarial events. If a source service failed to report an adversarial event to HADES (either due to a bug, a deliberate omission, or a bypass that was never detected), there is no MOIRAI record of that event. The adversarial record is only as complete as the detection layer that feeds it.

---

## 11. Implementation Roadmap

### Phase 1 — Core Repository and IAS/SCUDO Ingestion (Weeks 29–46)

This phase deploys alongside IAS/SCUDO, establishing HADES as the repository for IAS adversarial detections from day one of adversarial screening.

- AdversarialEvent, TechniqueEntry, FailureMode schemas
- Air-gapped Elasticsearch indices (separate from main THEMIS infrastructure)
- IAS/SCUDO ingestion pipeline (one-way, write-only)
- CVS/VERITAS fabrication event ingestion
- MAS/EIDOLON high-risk event ingestion
- Basic Research & Red Team query interface
- MOIRAI ingestion event emission (metadata only)
- IOB exposure summary report endpoint

**Phase gate criterion:** IAS/SCUDO blocks appearing in HADES within 60 seconds of detection. Research & Red Team query interface accessible from designated terminals only. Air-gapped network isolation validated. IOB exposure report operational.

### Phase 2 — ERAS Adversarial Reasoning Records and ADS/CASSANDRA (Weeks 47–66)

This phase deploys alongside ERAS/LOGOS, adding the highest-value content type — adversarial reasoning records — to the repository.

- ERAS/LOGOS adversarial reasoning record ingestion (ADVERSARIAL_REASONING_RECORD events)
- AdversarialReasoningRecord schema and air-gapped indexing
- ADS/CASSANDRA adversarially-confirmed anomaly ingestion
- WSF/LACHESIS adversarially-attributed fusion ingestion
- Failure mode annotation workflow
- Research & Red Team bypass rate analysis tools
- Retrospective session analysis capability

**Phase gate criterion:** ERAS adversarial reasoning records appearing in HADES. Retrospective session review identifies previously unrecognised adversarial content in test sessions. Failure mode annotation workflow produces MOIRAI events. ARB and Cell Lead sign-off.

### Phase 3 — Improvement Cycle Integration and MNEMOSYNE Feed (Year 2+)

- Research & Red Team IAS catalog update workflow using HADES patterns
- CPS/APORIA failure mode evidence submission from HADES findings
- De-identified MNEMOSYNE knowledge node generation from HADES institutional knowledge
- Quarterly adversarial exposure briefing format for IOB

**Phase gate criterion:** At least one IAS/SCUDO catalog update traceable to a HADES finding. At least one CPS/APORIA zone recalibration informed by HADES failure mode data. IOB quarterly briefing delivered.

---

## 12. Policy Dependencies

No GC items gate HADES deployment. The Research & Red Team operational charter must specify:
- The cadence of retrospective session review (recommended: quarterly)
- The criteria for identifying retrospective bypasses
- The process for escalating failure modes to the ARB for CPS/APORIA zone recalibration
- The IOB reporting schedule for adversarial exposure summaries

---

## 13. Training and Analyst Guidance

HADES is not accessible to analysts or supervisors. There is no analyst-facing training module for HADES. The ATHENA training programme should note that the platform maintains a restricted adversarial intelligence repository — without describing its contents — so that analysts understand why certain detection events are handled with particular procedures.

Research & Red Team members who access HADES receive dedicated operational security briefings covering the physical access controls, handling procedures for adversarial content, and the protocols for translating HADES findings into catalog updates and zone submissions without revealing source techniques.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Air-gapped Elasticsearch operational complexity.** Operating a separate, air-gapped Elasticsearch cluster with one-way write connections from the main THEMIS infrastructure is operationally complex. The ingestion connector must be validated against adversarial use — an attacker who can manipulate the ingestion connector could potentially write false adversarial events to HADES or, worse, read from the main cluster through the connector. Resolution path: the ingestion connector architecture requires a dedicated security architecture review before Phase 1.

### 14.2 Operational Assumptions

- **Assumption 1: The Research & Red Team has bandwidth for quarterly retrospective review.** Retrospective review of flagged sessions to identify previously undetected bypasses is labour-intensive analytical work. If the Research & Red Team does not have bandwidth for this, the bypass record will be increasingly incomplete over time.
- **Assumption 2: Physical security controls for the air-gapped infrastructure are in place before Phase 1.** The air-gapped Elasticsearch indices and dedicated terminals require physical security controls (access-controlled rooms, device management, etc.). These are not THEMIS engineering tasks — they are facility and operations requirements that must be confirmed before Phase 1.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — service was specified in original design and referenced in ERAS architecture; formally documented here |

---

## Appendix A: Adversarial Event Type Reference

| Event Type | Source Service | What It Represents | Bypass Possible? |
|---|---|---|---|
| `PROMPT_INJECTION_BLOCKED` | IAS/SCUDO | Input injection caught before reaching model | No — blocked |
| `PROMPT_INJECTION_BYPASSED` | IAS/SCUDO | Injection not caught; retrospectively identified | Yes — bypassed |
| `CORPUS_POISONING_DETECTED` | IAS/SCUDO + CVS | Adversarially crafted corpus content identified | Depends on when detected |
| `INDIRECT_INJECTION_BLOCKED` | IAS/SCUDO | Injection embedded in retrieved content; caught | No — blocked |
| `ADVERSARIAL_REASONING_RECORD` | ERAS/LOGOS | Full reasoning chain during adversarial session | Either — most valuable in HADES |
| `REASONING_MANIPULATION_BYPASSED` | ERAS/LOGOS | Manipulation reached reasoning; not detected at time | Yes — bypassed |
| `FABRICATED_CITATION_DETECTED` | CVS/VERITAS | AI fabricated a source that doesn't exist | Depends on detection timing |
| `SYNTHETIC_MEDIA_HIGH_RISK` | MAS/EIDOLON | Media showing synthesis/manipulation indicators | Assessment uncertain |
| `SYNTHETIC_MEDIA_CONFIRMED` | MAS/EIDOLON | Synthetic origin confirmed after initial assessment | Was bypassed initially |
| `ANOMALY_ADVERSARIAL_CONFIRMED` | ADS/CASSANDRA | Behavioural anomaly confirmed as adversarially placed | Partially bypassed |

---

## Appendix D: Red Team Findings

*Pending — Phase 5 gate review. The ingestion connector architecture requires a dedicated security architecture red team review before Phase 1. This is separate from the standard Research & Red Team evaluation — it requires a security architecture team focused specifically on the one-way connector design.*
