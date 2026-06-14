# PCS — Policymaker Communication Service
### IRIS · *"Greek goddess of the rainbow — the messenger who traveled between Olympus and the mortal world, translating between divine knowledge and mortal action without loss of meaning"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `PCS` |
| **Epithet** | `IRIS` |
| **Full name** | Policymaker Communication Service |
| **Namespace** | `themis-dissemination` |
| **Layer** | Intelligence Layer — Intelligence Cycle Completion |
| **Build phase** | Year 2 · Q3 (Addendum F) |
| **Build priority** | 22 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust + Origin — translates the platform's accountability metadata into actionable consumer intelligence |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**PCS/IRIS answers: How does the provenance, calibration, and uncertainty characterisation of a THEMIS-governed assessment reach a decision-maker in a form they can actually use to make decisions?**

### 1.2 Why This Service Exists

ATHENA is designed for analysts. Intelligence products from ATHENA go to policymakers. Policymakers do not understand calibrated confidence, source type badges, or cryptographic provenance chains. They receive finished assessments with no visibility into how confident to be, why the confidence level is what it is, what would change the assessment, or what kind of uncertainty they are facing.

The accountability chain currently ends at dissemination. Everything that happens after — how the policymaker interprets the assessment, what weight they give it, how they act on it — is outside the platform's governance. The purpose of intelligence analysis is to inform decisions. Governing the analytical process but not the point at which analysis meets decision leaves the most consequential moment ungoverned.

PCS/IRIS closes that gap. It translates — without loss — the technical provenance and calibration metadata of a THEMIS-governed assessment into a consumer package that decision-makers can read, interpret, and act on without knowing THEMIS exists.

### 1.3 The Translation Imperative

The word "translate" is intentional. PCS/IRIS does not simplify by discarding. It translates by finding the decision-maker's vocabulary for what the platform expresses in analytical vocabulary. The three-sentence confidence translation does not hide the UCS/TYCHE uncertainty decomposition — it expresses it in terms a policymaker can act on. The falsification indicators do not summarise the analytical record — they extract the operationally actionable element that tells the policymaker what to watch for. Nothing is lost; everything is re-expressed.

### 1.4 Design Principles

- **The falsification indicator is the most important element in the consumer package.** An assessment that says "we assess X with high confidence" tells the policymaker what to believe. A falsification indicator that says "this assessment would be called into question if [observable condition] is observed" tells the policymaker what to watch for. The second is operationally more valuable. The assessment is a snapshot; the falsification indicator is a monitoring framework.
- **The consumer package is generated from the platform record, not from analyst declaration.** The prior accuracy record, the key assumptions, and the uncertainty language are derived from MOIRAI events, UCS/TYCHE profiles, and ORACLE data — not from what the analyst writes in a disclosure field. The package is audit-trail-backed, not self-reported.
- **GC-6 owns the content policy; PCS owns the generation.** What appears in a consumer package, at what classification, with what modification authority — these are policy decisions owned by the IOB under GC-6. PCS implements the policy; it does not define it.
- **Disclosure format is configurable, not hardcoded.** Different oversight bodies, different dissemination channels, and different partner organisations require different disclosure formats. PCS supports jurisdiction-configurable formats without engineering changes — format configuration is a policy artifact, not a code change.

### 1.5 Explicit Out of Scope

- **Editorial review of finished intelligence products.** PCS generates the consumer package for a product it is given. It does not evaluate the analytical quality of the underlying assessment.
- **Dissemination channel management.** PCS generates packages for dissemination. DPS/CODEX records the dissemination event; the dissemination infrastructure routes the product. PCS does not manage distribution.
- **Oversight body reporting.** PCS generates consumer packages for decision-makers. The IOB reporting function uses a different interface for platform-level oversight.

---

## 2. Core Responsibilities

### 2.1 Primary Function

PCS/IRIS generates a structured consumer package at document dissemination — triggered by a `DPS_DOCUMENT_DISSEMINATED` event — that translates the THEMIS session provenance and calibration metadata into: a plain-language source basis summary, a three-sentence confidence translation from UCS/TYCHE, the key analytical assumptions extracted from ATHENA Intention Gate records, observable falsification indicators, a prior accuracy record from ORACLE/MIRROR, and a cryptographic provenance certificate readable to non-technical audiences.

### 2.2 Secondary Functions

- Briefing deck summary generation: for high-priority assessments requiring policymaker briefings, generating a scenario-space summary drawing on TRS/CHRONOS outputs where available
- Consumer package format configuration: maintaining jurisdiction-configurable format templates (no engineering changes required to add a format)
- Package modification audit: recording any modifications to a consumer package between generation and delivery, with the identity of who modified it
- Prior accuracy record: querying ORACLE/MIRROR for historical accuracy on similar assessments and translating this into a consumer-facing record (e.g., "Assessments of this type from this analytical community have been confirmed at X% in recent years")

### 2.3 What This Service Does Not Decide

PCS generates the consumer package from the available metadata. Whether the package accurately represents the assessment's epistemic status, whether additional caveats should be added, and whether a specific disclosure format is appropriate for a specific recipient are decisions owned by the analyst, supervisor, and dissemination authority. Modification of a generated package requires explicit documentation (who modified, what changed, why).

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ConsumerPackage:
  package_id:              uuid
  document_id:             uuid             # FK → DPS/CODEX DocumentRecord
  version_id:              uuid             # FK → DPS/CODEX DocumentVersion
  dissemination_id:        uuid             # FK → DPS/CODEX DisseminationRecord
  generated_at:            datetime
  format:                  str              # jurisdiction/policy format identifier
  classification:          str              # may be lower than underlying document
  components:              PackageComponents
  provenance_certificate:  ProvenanceCertificate
  modified:                bool
  modifications:           [PackageModification]
  status:                  GENERATED | DELIVERED | RECALLED

PackageComponents:
  source_basis_summary:    str              # plain-language source characteristics
  confidence_translation:  ConfidenceTranslation
  key_assumptions:         [str]            # from Intention Gate records in MOIRAI
  falsification_indicators:[FalsificationIndicator]
  prior_accuracy_record:   PriorAccuracyRecord | null
  briefing_summary:        str | null       # only for high-priority assessments with TRS data

ConfidenceTranslation:
  sentence_1_assessment:   str             # "We assess [X] with [high/medium/low] confidence."
  sentence_2_uncertainty:  str             # "The primary uncertainty is [type + plain language]."
  sentence_3_reducibility: str             # "Confidence would [change] if [observable condition]."
  uncertainty_type:        str             # from UCS/TYCHE primary_uncertainty_type
  aleatory_language:       str | null      # if aleatory: "the adversary may not have decided"
  epistemic_language:      str | null      # if epistemic: "better collection could clarify"
  model_language:          str | null      # if model: "this claim type is at the boundary of AI reliability"

FalsificationIndicator:
  indicator_id:            uuid
  description:             str             # observable event that would call assessment into question
  observability:           COLLECTION_OBSERVABLE | OPEN_SOURCE | REQUIRES_REPORTING
  lead_time:               str | null      # how quickly this would be observable
  source:                  str             # derived from analyst Intention Gate or ERAS reasoning

PriorAccuracyRecord:
  similar_assessment_count:int
  confirmed_rate:          float
  disconfirmed_rate:       float
  time_period:             str             # e.g., "recent 3 years"
  domain_match:            str             # how closely these match the current assessment
  oracle_basis:            bool           # true if ORACLE data; false if MIRROR pattern only

ProvenanceCertificate:
  certificate_id:          uuid
  session_count:           int
  claim_count:             int
  verification_rate:       float
  model_versions_used:     [str]
  ai_disclosure_text:      str             # plain-language: "This assessment was produced with AI assistance..."
  moirai_chain_ref:        str             # MOIRAI event hash chain reference
  chain_valid:             bool
  generated_at:            datetime

PackageModification:
  modification_id:         uuid
  package_id:              uuid
  modified_by:             str
  component_modified:      str
  original_content_hash:   str
  rationale:               str
  modified_at:             datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | ConsumerPackage, PackageModification, ProvenanceCertificate | Indefinite |
| Format template store | PostgreSQL | Configurable format templates per jurisdiction/policy | Per version |
| Event store | MOIRAI | Signed package generation and modification events | Indefinite |
| Package cache | Redis | Generated packages (24h TTL; high-priority packages may be pre-generated) | 24h |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ConsumerPackage | Configurable — may be lower than underlying document | Per GC-6 content policy |
| ProvenanceCertificate | Configurable | Per GC-6 |
| PackageModification | Inherits package classification | IOB access |

### 3.4 Retention and Purge Policy

ConsumerPackage and ProvenanceCertificate retained indefinitely — the consumer-facing disclosure record is a permanent accountability artefact. PackageModification retained indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /packages/generate
  Auth:     DPS/CODEX service account
  Request:  {
    document_id:           uuid,
    version_id:            uuid,
    dissemination_id:      uuid,
    format:                str,
    target_classification: str | null    # if package should be lower classification
  }
  Response: {
    package_id:            uuid,
    status:                GENERATED,
    components_available:  [str]         # which components were successfully generated
  }
  SLA: p99 < 3000ms

GET /packages/{package_id}
  Auth:     dissemination authority token | supervisor token | IOB token
  Response: ConsumerPackage

GET /packages/{package_id}/certificate
  Auth:     dissemination authority token | IOB token
  Response: ProvenanceCertificate

PATCH /packages/{package_id}
  Auth:     dissemination authority token (GC-6 defines who may modify)
  Request:  {
    component:             str,
    new_content:           str,
    rationale:             str
  }
  Response: { modification_id: uuid, package_id: uuid }

GET /packages/{package_id}/modifications
  Auth:     supervisor token | IOB token
  Response: [PackageModification]

GET /formats
  Auth:     any service account
  Response: [{ format_id: str, description: str, jurisdiction: str }]

POST /formats
  Auth:     IOB token (format configuration is policy; requires IOB for new formats)
  Request:  { format_id: str, template: {} }
  Response: { format_id: str }

GET /audit/packages?from={date}&to={date}
  Auth:     IOB token
  Response: {
    period:                { from, to },
    packages_generated:    int,
    modified_count:        int,
    modification_rate:     float,
    by_format:             [{ format, count }]
  }

GET /health
  Response: {
    status, dependencies: { moirai, ucs_tyche, dps, oracle, redis },
    packages_generated_24h:int,
    generation_error_rate: float,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          PCS_PACKAGE_GENERATED
service_id:         "PCS"
session_id:         null             # package is document-level
classification:     str
event_payload:
  package_id:             uuid
  document_id:            uuid
  dissemination_id:       uuid
  format:                 str
  components_generated:   [str]
  chain_valid:            bool
  verification_rate:      float
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          PCS_PACKAGE_MODIFIED
event_payload:
  package_id:             uuid
  component:              str
  modified_by:            str
  rationale:              str

EventType:          PCS_PACKAGE_RECALLED
event_payload:
  package_id:             uuid
  reason:                 str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `PCS_PACKAGE_GENERATED` | Package generated at dissemination | MOIRAI, DPS/CODEX (dissemination record update) |
| `PCS_PACKAGE_MODIFIED` | Consumer package modified before delivery | MOIRAI, supervisor notification, IOB alert |
| `PCS_PACKAGE_RECALLED` | Package recalled after delivery | MOIRAI, P0 alert |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| DPS/CODEX | `DPS_DOCUMENT_DISSEMINATED` | Triggers package generation |
| UCS/TYCHE | Profile data (via API) | Drives ConfidenceTranslation generation |
| ORACLE/MIRROR | Historical accuracy data (via API) | Drives PriorAccuracyRecord generation |
| TRS/CHRONOS | Scenario outputs (via API, if available) | Drives briefing_summary generation for high-priority assessments |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| DPS/CODEX | Document Provenance | Dissemination trigger; session and claim provenance | Event trigger + Sync query | Package generation cannot start without DPS dissemination event |
| UCS/TYCHE | Uncertainty Characterization | Uncertainty profile for ConfidenceTranslation | Sync query | Confidence translation uses generic language; alert |
| MOIRAI | Provenance | Session and turn data for key assumptions; chain validation for certificate | Sync query + Async event | Key assumptions not generated; certificate shows chain_valid=false |
| ORACLE / MIRROR | Outcome Intelligence | Historical accuracy data for PriorAccuracyRecord | Sync query (Year 2+) | PriorAccuracyRecord omitted from package; noted explicitly |
| TRS/CHRONOS | Temporal Reasoning | Scenario space for briefing summary | Sync query (optional) | Briefing summary omitted; package still generated |

### 5.2 Feeds Into

| Service | Epithet | What PCS provides | How |
|---|---|---|---|
| Dissemination system | N/A | Consumer package delivered with intelligence product | API output |
| DPS/CODEX | Document Provenance | Package ID linked to dissemination record | API |
| IOB Reporting | Oversight | Package modification audit; generation statistics | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Package classification may be lower than the underlying document (per GC-6). PCES validates that the package classification is within the dissemination authority's authority.
- **Compartment handling:** Consumer packages do not contain source names or compartmented content — they contain plain-language source characteristics. The package classification is set by policy (GC-6), not by the underlying document's full classification.
- **Failure behavior:** PCES unavailable → package generation proceeds but package classification defaults to the underlying document classification (conservative).

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Package generation (full) | 1000ms | 2000ms | 3000ms |
| Certificate generation | 200ms | 600ms | 1000ms |
| Package retrieval (cached) | 20ms | 50ms | 100ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Package generations/hour | 50 |
| Package retrievals/hour | 200 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — PCS unavailability means consumer packages are not generated at dissemination |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Decision-maker impact |
|---|---|---|
| UCS/TYCHE | Generic confidence language substituted; alert | Package produced with reduced precision in confidence translation |
| ORACLE/MIRROR | PriorAccuracyRecord omitted; package notes absence | Policymaker does not receive historical accuracy context |
| MOIRAI (chain query) | Certificate shows chain_valid=unknown | Provenance certificate carries explicit caveat |

---

## 7. Security Model

### 7.1 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| DPS/CODEX | Package generation trigger | Service account |
| Dissemination authority | Package retrieval; modification (per GC-6) | Authority token |
| Supervisor | Package review | Supervisor token |
| IOB | Full access including modifications and audit | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/pcs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/pcs/db-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Falsification indicators too vague (not operationally useful) | Medium | P2 — policymaker cannot act on them | Package quality review by analytic standards authority | Falsification indicators reviewed quarterly; feedback loop to ERAS reasoning design |
| Package modification without documented rationale | Low | P1 — accountability record gap | PackageModification.rationale validation | Rationale field required; cannot be blank; IOB notified on any modification |
| Key assumptions missing (Intention Gate not completed) | Medium | P2 — package cannot accurately represent analytical assumptions | Key_assumptions empty count monitoring | ATHENA workflow enforces Intention Gate completion; empty set noted explicitly in package |

### 8.1 Known Design Risks

- **The quality of the consumer package is bounded by the quality of the underlying metadata.** A session with minimal Intention Gate engagement produces limited key assumptions. A session with no UCS/TYCHE resolution produces a generic confidence translation. PCS translates what it has; if the underlying analytical record is thin, the consumer package will reflect that. This is the correct behaviour — the package is an honest translation, not an improvement on the underlying work. The risk is that policymakers receive packages of variable quality that they interpret as platform variation rather than analytical quality variation.
- **Package modification is the highest-risk operation.** A consumer package that is modified between generation and delivery may no longer accurately represent the MOIRAI-attested assessment. GC-6 must specify strict constraints on what can be modified, by whom, and with what documentation. The `PCS_PACKAGE_MODIFIED` event and IOB notification on any modification are the safeguards.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `pcs.generation.latency_p99` | Histogram | `> 3000ms for 5m` | P1 |
| `pcs.package.modification_rate` | Gauge | `> 5%` of packages | P1 |
| `pcs.package.missing_assumptions_rate` | Gauge | `> 20%` of packages | P2 |
| `pcs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/pcs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every consumer package generation event — what was generated, at what classification, with what components — is permanently recorded. Package modifications are MOIRAI-attested with the modifier's identity and rationale. A policymaker who receives a consumer package can verify from the MOIRAI chain that the provenance certificate reflects an unaltered session record.
- **What it cannot prove:** The consumer package accurately represents the analytical judgment. It accurately represents the MOIRAI record — but if the session record itself was thin, the package will honestly reflect a thin record.

---

## 11. Implementation Roadmap

### Phase 1 — Core Package Generation (Year 2, Weeks 17–24)

- ConsumerPackage, PackageComponents, ProvenanceCertificate schemas
- Package generation triggered by `DPS_DOCUMENT_DISSEMINATED` event
- Source basis summary (from DPS session provenance)
- Confidence translation from UCS/TYCHE (three sentences)
- Key assumptions from MOIRAI Intention Gate records
- Provenance certificate with chain validation
- Basic format template support

**Phase gate criterion:** Package generated within 3 seconds of DPS dissemination event. Confidence translation produced for test assessments with UCS/TYCHE profiles. Provenance certificate chain valid flag correct. GC-6 policy reviewed and format templates configured.

### Phase 2 — Falsification Indicators, Prior Accuracy, and Format Library (Year 2, Weeks 25–32)

- Falsification indicator extraction from ERAS reasoning records and Intention Gate
- PriorAccuracyRecord from ORACLE/MIRROR historical data
- Package modification workflow with IOB notification
- Package recall mechanism
- Format template library expansion (per GC-6 jurisdiction list)
- IOB audit endpoints

**Phase gate criterion:** Falsification indicators generated for test assessments. PriorAccuracyRecord populated from ORACLE data (requires ORACLE to have sufficient outcome data). IOB audit endpoint operational. GC-6 content policy implemented. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-6 | Consumer package content policy: what appears in a package, its classification relative to the underlying product, who may modify it, modification documentation requirements, and which jurisdictions/formats are supported | PCS deployment — without GC-6, package content, classification, and modification policy are undefined |

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

At the point of document dissemination, ATHENA shows a preview of the consumer package that will accompany the product. The analyst can review the generated falsification indicators, key assumptions, and confidence translation before the product is disseminated. If the analyst believes any element is inaccurate, they must raise it with their supervisor (modification goes through the documented modification workflow) — they cannot self-modify the package.

### 13.2 What the Analyst Should Do

Engage genuinely with the Intention Gate during the session — this is where key assumptions come from. Engage genuinely with UCS/TYCHE uncertainty characterisation — this is what drives the three-sentence confidence translation. The quality of the consumer package the policymaker receives depends on the quality of the analytical metadata the analyst generates.

### 13.3 What the Signal Does Not Mean

The consumer package is not a summary of the finished intelligence product — it is a translation of the analytical metadata. The policymaker must still read the underlying product. The consumer package tells them how to read it — how confident to be, what would change it, what has historically been accurate in similar assessments.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Falsification indicator quality.** The falsification indicators are extracted from ERAS reasoning captures and Intention Gate records. The quality of these inputs varies. If the analyst did not explicitly consider falsification conditions during the session, the indicators will be thin. Resolution path: Research & Red Team to develop prompt templates specifically designed to elicit falsification thinking; these are the key PRS prompts for assessment interaction classes.

### 14.2 Operational Assumptions

- **Assumption 1: GC-6 is resolved before Year 2 Q3.** GC-6 defines what appears in a consumer package and at what classification. Without this, PCS cannot be deployed. GC-6 must be initiated by Year 2 Q1 at the latest to allow IOB deliberation time before Q3 deployment.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — incorporates Addendum F policymaker communication service design |

## Appendix D: Red Team Findings
*Pending — Year 2 Q2 gate review. GC-6 policy resolution is a prerequisite.*
