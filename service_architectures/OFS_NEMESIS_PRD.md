# OFS — Outcome Feedback Service
### NEMESIS · *"Greek goddess of retributive justice — the force that corrects imbalance and ensures consequences follow actions; what was assessed must be reckoned against what actually happened"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `OFS` |
| **Epithet** | `NEMESIS` |
| **Full name** | Outcome Feedback Service |
| **Namespace** | `themis-core` |
| **Layer** | Core Infrastructure |
| **Build phase** | Year 2 · Q2 (Addendum F) |
| **Build priority** | 15 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency + Trust — closes the calibration loop from real-world outcomes |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**OFS/NEMESIS answers: Did the AI-assisted assessment turn out to be correct — and does that outcome update our understanding of how reliable AI assistance is in this domain?**

### 1.2 Why This Service Exists

Calibration in TCS/MIMIR updates from analyst verification behaviour: what the analyst checks, how they verify it, whether they correct the AI's output. This is necessary. It is not sufficient.

The highest-weight ground truth signal in intelligence analysis is not analyst verification. It is outcome: the adversary did or did not do what the assessment said they would. The programme was or was not in the development phase the assessment asserted. The capability was or was not demonstrated. Real-world events are the ultimate arbiter of analytical accuracy — and without OFS/NEMESIS, THEMIS has no mechanism to capture them.

Analyst verification is better than no verification. But it is bounded by the analyst's own accuracy and knowledge. An analyst who verifies a GRND claim against a source that is itself incorrect produces a CONFIRMED correction. The correction enters the corpus. The calibration posterior updates toward greater confidence. The confidence signal increases. The assessment is wrong, and the platform has increased its confidence in the type of error that produced it.

Real-world outcomes break this feedback loop. A DISCONFIRMED outcome — the adversary did not do what the assessment said — updates the calibration in the direction of greater epistemic humility regardless of how many analyst verifications pointed the other way. OFS/NEMESIS is the service that makes the calibration system accountable to reality, not just to analyst behaviour.

### 1.3 Why This Service Is Fifteenth and Year 2

OFS/NEMESIS requires a mature MOIRAI provenance graph to traverse (linking outcome events to prior assessments and claims), a mature FGTS/ALETHEIA to receive high-weight outcome corrections, and a mature ORACLE in the intelligence layer to receive outcome data for predictive modelling. Most importantly, OFS cannot produce meaningful outcomes until the platform has been operational long enough for analytical assessments to have been tested by real-world events. This is inherently a Year 2 service — it cannot exist before Year 1 assessments have outcomes.

The outcome data floor is also the critical design constraint. Most intelligence assessments never receive a clean CONFIRMED or DISCONFIRMED signal within a predictable timeframe. GC-5 must define what constitutes an outcome event before OFS is deployed — without this policy decision, OFS cannot be operationalised.

### 1.4 Design Principles

- **Outcomes are independent of analyst behaviour.** Whether the adversary did what the assessment said is not determined by how carefully the analyst verified sources. This is what makes outcome corrections substantially more valuable than verification corrections — they are independent of the analyst's own analytical accuracy.
- **Outcome classification is a policy decision, not a technical one.** GC-5 defines what constitutes CONFIRMED, DISCONFIRMED, AMBIGUOUS, and SUPERSEDED for intelligence calibration purposes. OFS implements that classification; it does not define it.
- **Partial accuracy is not failure.** An assessment that was right about capability but wrong about intent is not simply DISCONFIRMED. OFS supports multi-dimensional outcome records that capture which aspects of an assessment were confirmed and which were not.
- **The calibration loop compound.** OFS outcome corrections feed FGTS/ALETHEIA (improving calibration posteriors) and ORACLE (improving predictive models of what assessments look like when accurate). Both improve. The compound effect is what makes the intelligence layer genuinely smarter over time.

### 1.5 Explicit Out of Scope

- **Generating outcome data.** OFS ingests and classifies outcome events from external sources, analyst declarations, and KCS supersession events. It does not generate the outcomes themselves.
- **Determining attribution.** OFS classifies whether an outcome confirms or disconfirms an assessment. Whether the outcome was caused by the adversary's deliberate action versus accidental or third-party events is an analytical judgment, not an OFS determination.
- **Comprehensive coverage.** Most assessments will not receive OFS outcome records within the platform's operational timeline. OFS operates on the outcomes it can observe; it does not attempt to force classification of unobservable outcomes.

---

## 2. Core Responsibilities

### 2.1 Primary Function

OFS/NEMESIS ingests real-world outcome events from collection reporting, watchlist matches from KCS/ARGUS, and analyst declarations — traverses the MOIRAI provenance graph to identify every assessment, claim, and session that bears on each outcome event — classifies the relationship between the outcome and prior assessments as CONFIRMED, DISCONFIRMED, AMBIGUOUS, or SUPERSEDED — and publishes outcome records to FGTS/ALETHEIA as high-weight ground truth corrections and to ORACLE as outcome data for predictive model training.

### 2.2 Secondary Functions

- Multi-dimensional outcome classification: separately classifying outcome confirmation for different aspects of the assessment (capability, intent, timing, magnitude)
- Outcome confidence weighting: producing outcome records at different confidence levels depending on the quality of the evidence establishing the outcome
- Analyst-declared outcomes: pathway for analysts to declare outcomes for assessments where they have domain knowledge of what actually happened
- Supervisor-confirmed outcomes: highest-confidence pathway requiring supervisor confirmation before outcome is published to FGTS/ALETHEIA
- ORACLE integration: structured outcome events consumed by the intelligence layer ORACLE service for predictive model training
- Outcome summary reporting: IOB-facing reports on assessment accuracy by domain, interaction class, and analyst pool

### 2.3 What This Service Does Not Decide

OFS classifies outcomes against assessments. Whether the outcome data is sufficient to warrant re-evaluation of current assessments, whether an analyst whose assessments are consistently DISCONFIRMED warrants additional supervision, and whether a pattern of DISCONFIRMED outcomes indicates a systemic analytical failure requiring review are human decisions owned by supervisors, the IOB, and the analytic standards authority.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
OutcomeEvent:
  event_id:              uuid
  event_type:            COLLECTION_REPORT | WATCHLIST_MATCH | ANALYST_DECLARATION | SUPERVISOR_DECLARATION | IOB_DETERMINATION
  domain:                str
  description:           str              # plain language description of what happened
  evidence_quality:      HIGH | MEDIUM | LOW
  event_timestamp:       datetime         # when the real-world event occurred
  recorded_at:           datetime         # when OFS received this event
  source_ref:            str | null       # collection report reference, if applicable

OutcomeRecord:
  record_id:             uuid
  outcome_event_id:      uuid
  session_id:            uuid             # session whose assessment is being evaluated
  turn_id:               uuid | null
  claim_id:              uuid | null
  classification:        OutcomeClassification
  dimensions:            [DimensionOutcome]   # multi-dimensional classification
  confidence:            HIGH | MEDIUM | LOW
  basis:                 str              # why this classification was assigned
  analyst_acknowledged:  bool
  supervisor_confirmed:  bool
  fgts_correction_weight:float            # weight assigned when published to FGTS
  oracle_event_published:bool
  published_at:          datetime | null

OutcomeClassification:
  CONFIRMED              # assessment's central claim borne out by events
  DISCONFIRMED           # events contradict the claim
  AMBIGUOUS              # events consistent with claim but do not confirm it
  SUPERSEDED             # world changed; claim's truth value undeterminable
  PARTIALLY_CONFIRMED    # some dimensions confirmed, others not

DimensionOutcome:
  dimension_id:          uuid
  record_id:             uuid
  dimension:             CAPABILITY | INTENT | TIMING | MAGNITUDE | ATTRIBUTION
  classification:        CONFIRMED | DISCONFIRMED | AMBIGUOUS | NOT_ASSESSED
  confidence:            HIGH | MEDIUM | LOW
  basis:                 str

OutcomeCorrection:
  correction_id:         uuid
  record_id:             uuid
  fgts_correction_id:    uuid | null      # FK → FGTS CorrectionRecord on acceptance
  domain:                str
  claim_type:            str
  model_version:         str
  correction_weight:     float            # 0.95 for confirmed/disconfirmed; 0.5 for ambiguous
  submitted_to_fgts:     bool
  oracle_notified:       bool
  submitted_at:          datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | OutcomeEvent, OutcomeRecord, DimensionOutcome, OutcomeCorrection | Indefinite |
| Event store | MOIRAI | Signed outcome events | Indefinite |
| Outcome analytics | Elasticsearch | Outcome distribution analytics for IOB reporting | 5 years |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| OutcomeEvent | Inherits classification of the underlying outcome | Compartment-gated; outcomes may be classified above the original assessment |
| OutcomeRecord | Inherits the higher of outcome or session classification | Compartment-gated |
| OutcomeCorrection | Controlled Unclassified (weights and metadata only) | Platform-wide calibration data |

### 3.4 Retention and Purge Policy

OutcomeEvent and OutcomeRecord retained indefinitely — the historical outcome record is the most valuable calibration asset in the platform. OutcomeCorrection records retained indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /outcomes/events
  Auth:     KCS service account | IOB token | supervisor token
  Request:  {
    event_type:          str,
    domain:              str,
    description:         str,
    evidence_quality:    str,
    event_timestamp:     datetime,
    source_ref:          str | null
  }
  Response: {
    event_id:            uuid,
    assessments_linked:  int,          # number of prior assessments linked to this event
    records_created:     int
  }
  SLA: p99 < 2000ms (provenance graph traversal)

POST /outcomes/events/{event_id}/classify
  Auth:     IOB token | supervisor token
  Request:  {
    session_id:          uuid,
    claim_id:            uuid | null,
    classification:      str,
    dimensions:          [{ dimension: str, classification: str, confidence: str, basis: str }],
    confidence:          str,
    basis:               str
  }
  Response: {
    record_id:           uuid,
    fgts_correction_weight: float,
    ready_for_publishing:bool
  }

POST /outcomes/records/{record_id}/confirm
  Auth:     supervisor token
  Request:  { supervisor_rationale: str }
  Response: {
    record_id:           uuid,
    fgts_correction_id:  uuid,
    oracle_notified:     bool
  }

POST /outcomes/declare
  Auth:     analyst session token
  Request:  {
    session_id:          uuid,
    claim_id:            uuid | null,
    outcome:             str,
    basis:               str,
    evidence_quality:    str
  }
  Response: {
    record_id:           uuid,
    requires_supervisor: bool         # analyst declarations require supervisor confirmation
  }

GET /outcomes/session/{session_id}
  Auth:     session token | supervisor token | IOB token
  Response: {
    session_id:          uuid,
    outcome_records:     [OutcomeRecord],
    overall_accuracy:    float | null,   # null if insufficient confirmed/disconfirmed
    by_dimension:        [{ dimension, confirmed: int, disconfirmed: int }]
  }

GET /audit/accuracy-summary?domain={domain}&from={date}&to={date}
  Auth:     IOB token
  Response: {
    period:              { from, to },
    domain:              str,
    records_with_outcomes:int,
    confirmed_rate:      float,
    disconfirmed_rate:   float,
    ambiguous_rate:      float,
    by_interaction_class:[{ class, confirmed_rate, record_count }],
    by_claim_type:       [{ type, confirmed_rate, record_count }]
  }

GET /health
  Response: {
    status, dependencies: { moirai, fgts, oracle, pces },
    outcome_events_30d:  int,
    records_confirmed:   int,
    records_pending:     int,
    last_event_hash:     str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          OFS_OUTCOME_CLASSIFIED
service_id:         "OFS"
session_id:         uuid
classification:     str
event_payload:
  record_id:              uuid
  outcome_event_id:       uuid
  claim_id:               uuid | null
  classification:         str
  confidence:             str
  supervisor_confirmed:   bool
  fgts_correction_weight: float
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          OFS_ASSESSMENT_DISCONFIRMED
event_payload:
  session_id:             uuid
  domain:                 str
  claim_type:             str
  dimensions_disconfirmed:[str]
  evidence_quality:       str

EventType:          OFS_CORRECTION_PUBLISHED
event_payload:
  record_id:              uuid
  fgts_correction_id:     uuid
  correction_weight:      float
  oracle_notified:        bool
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `OFS_OUTCOME_CLASSIFIED` | Outcome record created and confirmed | MOIRAI, FGTS/ALETHEIA (correction), ORACLE, TVS/KAIROS (supporting sources) |
| `OFS_ASSESSMENT_DISCONFIRMED` | Classification = DISCONFIRMED (confirmed) | MOIRAI, KCS/ARGUS (triggers source supersession review), supervisor notification |
| `OFS_CORRECTION_PUBLISHED` | Outcome correction submitted to FGTS | MOIRAI, TCS/MIMIR (indirect via FGTS update) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| KCS/ARGUS | `KCS_SOURCE_SUPERSEDED` | Generates candidate outcome event for assessments using that source |
| TVS/KAIROS | `TVS_SOURCE_INVALIDATED` | Triggers review of assessments built on invalidated source |
| DPS/CODEX | `DPS_DOCUMENT_DISSEMINATED` | Registers disseminated assessment for outcome tracking |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Provenance graph traversal to link outcomes to assessments; signed events | Sync query + Async event | Events buffered; provenance traversal unavailable — outcome events queue until recovery |
| FGTS/ALETHEIA | Ground Truth | Receives high-weight outcome corrections | Async event | Corrections queued; calibration update deferred |
| PCES/AEGIS | Classification Enforcement | Outcome and record compartment enforcement | Sync | Proceeds with cached session scope |

### 5.2 Feeds Into

| Service | Epithet | What OFS provides | How |
|---|---|---|---|
| FGTS/ALETHEIA | Ground Truth | High-weight outcome corrections (weight: 0.95 for CONFIRMED/DISCONFIRMED) | `OFS_CORRECTION_PUBLISHED` event |
| ORACLE | Outcome Intelligence | Structured outcome events for predictive model training | `OFS_OUTCOME_CLASSIFIED` event |
| TVS/KAIROS | Temporal Validity | DISCONFIRMED outcomes trigger source supersession review | `OFS_ASSESSMENT_DISCONFIRMED` event |
| KCS/ARGUS | Knowledge Currency | Outcome data informs model knowledge currency calibration | API + events |
| TCS/MIMIR | Trust Calibration | Indirect (via FGTS corrections) | Via FGTS update cycle |
| IOB Reporting | Oversight | Assessment accuracy summaries by domain and analyst pool | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Outcome events and records may be classified at or above the original assessment. All outcome endpoints validate that the caller has access to the classification level of both the outcome event and the session being evaluated.
- **Cross-classification outcomes:** An outcome that is classified above the original assessment can be used to update calibration (via FGTS) without revealing the classified outcome data to the analyst whose session is being evaluated. The correction weight is published; the outcome basis is not.
- **Failure behavior:** PCES unavailable → outcome classification endpoints blocked; outcome events can still be ingested (write path continues).

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Outcome event ingestion | 500ms | 1500ms | 2000ms |
| Provenance traversal (assessment linkage) | 1000ms | 3000ms | 5000ms |
| Outcome classification | 200ms | 500ms | 1000ms |
| Correction publishing to FGTS | 300ms | 800ms | 1500ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Outcome events/day | 20 (low frequency by nature — real-world outcomes are sparse) |
| Records classified/day | 50 |
| Corrections published to FGTS/day | 30 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — OFS unavailability suspends outcome feedback but does not affect sessions |
| MOIRAI event durability | 99.999% |
| RTO | 30 minutes |
| RPO | 1 hour |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI provenance graph | Outcome events ingested but assessment linkage deferred until recovery | No analyst impact; outcome records queue |
| FGTS/ALETHEIA | Outcome corrections queued; calibration updates deferred | No analyst-facing impact |
| ORACLE | Outcome events queued; intelligence layer predictive models not updated | No analyst-facing impact; Year 2 capability |

---

## 7. Security Model

### 7.1 Authentication

Outcome event ingestion requires KCS service account, supervisor token, or IOB token. Analyst outcome declarations require session token plus supervisor confirmation. Classification requires supervisor or IOB token. FGTS correction publishing is service-internal.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| KCS/ARGUS | `POST /outcomes/events` | Service account |
| Supervisor | Event ingestion; classification; confirmation; analyst declarations review | Supervisor token |
| Analyst (own session) | `POST /outcomes/declare` (requires supervisor confirmation) | Session token |
| IOB | All endpoints including accuracy audit | IOB token |
| ORACLE | Outcome event consumption | Service account |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/ofs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/ofs/db-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Outcome classification intractable (GC-5 edge cases) | High | P2 — outcomes cannot be classified; remain AMBIGUOUS | Classification rate monitoring | GC-5 must provide edge case guidance before OFS deployment |
| Low outcome event volume (most assessments never observed) | High (structural) | P2 — calibration loop rarely closes from outcomes | Outcome event rate tracking | Design constraint acknowledged; OFS supplements verification, not replaces it |
| Cross-classification outcome cannot be shared with analyst | Medium | P2 — analyst unaware calibration was updated from classified outcome | Analyst calibration update notification (without basis) | Correction weight published; basis restricted; notification: "Your calibration was updated from an outcome record" |

### 8.1 Known Design Risks

- **The outcome classification problem may be structurally intractable for most assessments.** GC-5 asks for a definition of "confirmed" and "disconfirmed." In practice, most IC assessments never receive a clean confirmed/disconfirmed signal: outcomes are partial, delayed, classified above the assessment level, or contaminated by successful adversary D&D. If ORACLE needs 200 requirements with outcome data and only 20% of assessments produce classifiable outcomes, the data floor requires 1,000 assessments — which may take years. The design acknowledges this as a structural constraint, not an engineering problem.
- **Analyst-declared outcomes may introduce motivated reasoning.** An analyst who declares an outcome for their own assessment has an incentive to classify it as CONFIRMED. The supervisor confirmation requirement is the mitigation — all analyst-declared outcomes require supervisor confirmation before entering FGTS. But this adds a supervision burden that may reduce the volume of analyst-declared outcomes.
- **Real-world outcomes may be ambiguous even when they appear clear.** An adversary who successfully executed a D&D campaign may cause an assessment that was analytically correct to appear DISCONFIRMED (because the analyst correctly assessed the adversary's true capability, but the operation was designed to make it look like the capability was absent). OFS has no mechanism to distinguish a genuinely wrong assessment from an assessment that was right and was D&D'd. This is acknowledged as a design limitation.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `ofs.outcome.events_30d` | Gauge | `< 5` (outcome feed may be blocked) | P2 |
| `ofs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `ofs.classification.pending_count` | Gauge | `> 50` (classification backlog) | P2 |
| `ofs.fgts.correction.failure_rate` | Counter | `> 1%` | P1 |
| `ofs.provenance_traversal.latency_p99` | Histogram | `> 5000ms for 1h` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, fgts, pces },
  outcome_events_30d:         int,
  records_pending_classification: int,
  corrections_published_30d:  int,
  last_event_hash:            str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":           "ISO-8601",
  "service":             "OFS/NEMESIS",
  "event":               "OUTCOME_RECEIVED | OUTCOME_CLASSIFIED | CORRECTION_PUBLISHED | DISCONFIRMED_ALERT",
  "event_id":            "uuid | null",
  "record_id":           "uuid | null",
  "session_id":          "uuid | null",
  "classification":      "CONFIRMED | DISCONFIRMED | AMBIGUOUS | SUPERSEDED",
  "confidence":          "HIGH | MEDIUM | LOW",
  "correction_weight":   0.0,
  "duration_ms":         0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/ofs/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant
- **Prev_event_hash source:** Prior OFS event in the platform-wide outcome event stream

### 10.2 What This Service Attests

The MOIRAI record for OFS proves that specific assessments were evaluated against specific real-world outcomes, that outcomes were classified by an authorised party, and that the classification has not been altered since recording. The correction weight published to FGTS is permanently recorded alongside the outcome classification. An oversight body can verify that the calibration corrections were appropriate given the outcome evidence.

### 10.3 What This Service Cannot Prove

OFS proves the outcome was classified as presented. It does not prove the outcome was correctly determined. A CONFIRMED classification that was based on mistaken outcome evidence produces an unbreakable record of a correct assessment that was actually wrong. OFS attests the classification process was followed; it does not attest the underlying outcome data was accurate.

---

## 11. Implementation Roadmap

### Phase 1 — Outcome Event Ingestion and Assessment Linkage (Year 2, Weeks 1–8)

- OutcomeEvent, OutcomeRecord, DimensionOutcome schemas
- `POST /outcomes/events` endpoint and MOIRAI provenance traversal for assessment linkage
- Outcome classification endpoint (manual, by IOB and supervisors)
- `OFS_OUTCOME_CLASSIFIED` and `OFS_ASSESSMENT_DISCONFIRMED` MOIRAI events
- Supervisor confirmation pathway
- FGTS/ALETHEIA outcome correction publishing
- Analyst declaration pathway with supervisor confirmation requirement

**Phase gate criterion:** Outcome event triggers MOIRAI provenance traversal and links to prior assessments within 5 seconds (p95). Confirmed outcomes publish to FGTS within 60 seconds of supervisor confirmation. MOIRAI events chained correctly.

### Phase 2 — Oracle Integration, Audit Interface, and Analyst Declaration (Year 2, Weeks 9–16)

- ORACLE integration: outcome events published to intelligence layer ORACLE service
- Multi-dimensional outcome classification (capability, intent, timing, magnitude, attribution dimensions)
- IOB accuracy audit endpoints
- KCS/ARGUS supersession event → candidate outcome pipeline
- Cross-classification outcome handling (outcome classified above session level)
- Outcome summary reporting for IOB monthly report

**Phase gate criterion:** ORACLE receives outcome events and confirms consumption. Multi-dimensional classification produces meaningful dimension-level accuracy metrics. IOB accuracy audit shows confirmed/disconfirmed rates by domain. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-5 | Definition of "confirmed" and "disconfirmed" for intelligence calibration purposes — including edge cases: partial accuracy (capability correct, intent wrong); assessments overtaken by events; outcomes classified above the assessment level; outcomes where D&D may have contaminated the evaluation. | OFS deployment — without GC-5, outcome classification cannot proceed consistently |

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

When an outcome record is created for a session the analyst contributed to, they receive a notification in ATHENA: "An outcome has been recorded for an assessment you contributed to. [Domain]. [Classification without basis if basis is restricted]. Your calibration has been updated." Analysts can view outcome records for their own sessions (basis may be restricted based on classification). The outcome summary is available in the analyst's calibration dashboard.

### 13.2 What the Analyst Should Do

Review outcome records for your sessions when they appear. The outcome classification and any dimension-level detail that is accessible provides valuable feedback on analytical accuracy. If an analyst believes an outcome was incorrectly classified, they can raise a review request through their supervisor — the outcome classification process has an appeal pathway through the IOB.

### 13.3 What the Signal Does Not Mean

A DISCONFIRMED outcome does not mean the analyst made an error. A correct assessment built on good analytical process can be DISCONFIRMED by successful adversary D&D, by the adversary genuinely changing course, or by the outcome itself being ambiguous. CONFIRMED does not mean the analyst's methodology was sound — a lucky guess is CONFIRMED just as a well-reasoned assessment is. The calibration update reflects the AI's accuracy, not the analyst's analytical quality in a general sense.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Provenance traversal performance at scale.** Linking an outcome event to all prior assessments requires traversing the MOIRAI Neo4j provenance graph. As the graph grows to millions of nodes, this traversal may become slow. Resolution path: pre-compute potential outcome links for high-priority sessions on a schedule; on-demand traversal for ad hoc outcome events.

### 14.2 Research Dependencies

- **GC-5 is a blocking dependency.** OFS cannot be meaningfully deployed without a policy definition of what constitutes a classifiable outcome. The technical service can be built before GC-5 is resolved; it cannot be operated. GC-5 must be initiated and resolved before Year 2 Q2 deployment target.
- **Outcome data volume is structurally limited.** ORACLE's predictive model requires 200 requirements with outcome data. At 20% outcome observability rate, this requires 1,000 analysed requirements. At 40 requirements per year processed through THEMIS with outcome events, this is a 25-year data floor. A realistic Year 3 target for ORACLE usefulness requires accepting lower outcome data requirements and broader data sources. Resolution path: Research & Red Team to design ORACLE's minimum viable outcome dataset before Year 2.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — incorporates Addendum F outcome feedback service design |

---

## Appendix A: Outcome Classification Reference

Per GC-5 (pending — required before deployment):

| Classification | Definition | FGTS Correction Weight | Notes |
|---|---|---|---|
| CONFIRMED | Assessment's central claim borne out by observable events | 0.95 | Strongest calibration signal |
| DISCONFIRMED | Observable events contradict the assessment's central claim | 0.95 | Strongest calibration signal (negative direction) |
| AMBIGUOUS | Events consistent with but not confirming the claim | 0.50 | Moderate calibration signal |
| SUPERSEDED | World changed in a way that makes truth value undeterminable | TVS update only | No calibration update; TVS/KAIROS validity update |
| PARTIALLY_CONFIRMED | Some dimensions confirmed; others not | Weighted average of confirmed/disconfirmed dimensions | Multi-dimensional classification required |

*Note: These weights and definitions are subject to revision pending GC-5 policy determination. The table above reflects the design intent; the policy authority for these values is the IOB.*

---

## Appendix D: Red Team Findings
*Pending — Year 2 Q1 gate review. GC-5 policy resolution is a prerequisite for red team to assess outcome classification logic.*
