# FGTS — Feedback & Ground Truth Service
### ALETHEIA · *"Greek for 'truth' or 'disclosure' — the unconcealment of what is real"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `FGTS` |
| **Epithet** | `ALETHEIA` |
| **Full name** | Feedback & Ground Truth Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 5–6 (Weeks 29–46) |
| **Build priority** | 5 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust — accumulates and weights the ground truth that calibrates TCS/MIMIR |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**FGTS/ALETHEIA answers: How much does this verification outcome tell us about the AI's actual accuracy — and how much should it update the analyst's calibration?**

### 1.2 Why This Service Exists

Calibration without quality-weighted ground truth is not calibration — it is counting. If every verification action carries equal weight, a junior analyst rapidly clicking through the verification queue produces the same calibration update as a senior analyst who spent twenty minutes confirming a GRND claim against the source document. If performative verification is indistinguishable from genuine verification, the calibration system can be gamed trivially. If deadline pressure is ignored, a correction submitted at 2am under a Deadline-Critical flag is treated with the same confidence as a correction submitted in a standard analytical session.

FGTS/ALETHEIA exists because the ground truth corpus is the most sensitive artifact in the THEMIS platform. TCS/MIMIR's calibration posteriors are only as reliable as the corrections FGTS feeds them. A corrupted corpus produces corrupted calibration, which produces corrupted confidence signals, which mislead analysts who are relying on those signals as measurements of AI accuracy. FGTS is the service that protects TCS from that attack.

### 1.3 Why This Service Is Fifth

FGTS requires TCS to be deployed first (it feeds TCS, and TCS's gaming probability score is one of FGTS's weighting inputs — a circular dependency resolved by: TCS reads FGTS corrections; FGTS reads TCS gaming scores; the gaming score is computed at session close, after all corrections are already recorded). FGTS also requires MOIRAI (for the provenance chain), PCES (for analyst identity), and PGS (for pressure mode context at the time of verification). All four precede FGTS in the build sequence.

FGTS is Phase 5-6 rather than earlier because the verification data it accumulates is only meaningful after analysts have been using ATHENA for enough sessions to produce non-trivial verification queues. Deploying FGTS before the analyst workflow is established would produce a sparse corpus from an unrepresentative early-adopter population.

### 1.4 Design Principles

- **The corpus is write-once at the record level.** Individual CorrectionRecord entries are never modified. Corrections to a correction are new records that reference and supersede the original.
- **Weight is a property of the verification behavior, not the verification outcome.** A correction submitted with high dwell time and low gaming probability carries more weight than the same correction submitted rapidly under deadline pressure — regardless of whether the correction itself is CONFIRMED or MISREPRESENTS_SOURCE.
- **Supervisory confirmation is the highest-weight signal in the system.** A supervisor who reviews a correction and confirms it produces a correction weight that exceeds anything an analyst can produce through their own verification behavior alone.
- **Corpus versions are auditable and rollbackable.** If a corpus version is found to have been corrupted — by coordinated false corrections or gaming — the version can be rolled back to a prior clean state without losing the record of what was in the corrupted version.
- **The corpus is not the calibration.** FGTS accumulates corrections and weights them. TCS/MIMIR uses those weighted corrections to update Bayesian posteriors. The two services are coupled but distinct. FGTS does not perform calibration; it curates the training signal.

### 1.5 Explicit Out of Scope

- **Determining whether a claim is true.** FGTS records analyst verification outcomes. Whether the underlying claim was factually correct is determined by real-world outcomes (OFS/NEMESIS), not by FGTS.
- **Calibration computation.** FGTS produces weighted corrections and publishes them to TCS. TCS updates the Bayesian posteriors. The computation is TCS's responsibility.
- **Disciplinary records.** Gaming probability scores inform calibration weighting; they are not disciplinary records and do not appear in performance records without an explicit human decision.

---

## 2. Core Responsibilities

### 2.1 Primary Function

FGTS/ALETHEIA accumulates analyst verification outcomes from the ATHENA verification queue, applies a five-factor weighting scheme to each correction to determine its contribution to the calibration corpus, maintains a versioned ground truth corpus with rollback capability, and publishes weighted corrections to TCS/MIMIR for posterior updates. It also maintains the supervisory confirmation pathway — where a supervisor explicitly confirms or overrides an analyst's verification outcome, producing the highest-weight signal in the calibration system.

### 2.2 Secondary Functions

- Corpus versioning: creating named corpus snapshots with quality metrics at configurable intervals
- Corpus quality monitoring: tracking aggregate correction rates, source type distributions, and weighting factor distributions for anomaly detection
- Supervisory override management: routing flagged corrections for supervisory review and recording supervisor decisions
- OFS/NEMESIS integration: receiving real-world outcome data and converting it to high-weight corrections that bypass the standard five-factor weighting
- Human review queue: surfacing low-confidence corrections for human review before they enter the corpus
- Corpus rollback: reverting the active corpus to a prior version when contamination is detected

### 2.3 What This Service Does Not Decide

FGTS determines correction weight. Whether a specific analyst's verification behavior warrants management attention, whether a corpus version should be rolled back, and whether a real-world outcome was correctly classified as CONFIRMED or DISCONFIRMED are human decisions owned by supervisors, the IOB, and the analytic standards authority respectively.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
CorrectionRecord:
  correction_id:           uuid
  session_id:              uuid               # FK → MOIRAI session
  turn_id:                 uuid               # FK → MOIRAI turn
  claim_id:                uuid               # FK → MOIRAI claim
  analyst_id:              str                # hashed for non-IOB contexts
  action:                  CorrectionAction
  source_type:             str                # GRND | PARAM | SYNTH | TRANSCRIPT | etc.
  domain:                  str
  claim_type:              str
  model_version:           str
  pressure_mode:           STANDARD | ELEVATED | DEADLINE_CRITICAL
  weighting_factors:       WeightingFactors
  computed_weight:         float              # 0.0–1.0 composite weight
  corpus_version_id:       uuid               # corpus version this correction entered
  superseded_by:           uuid | null        # if a later correction supersedes this one
  supervisory_reviewed:    bool
  supervisory_outcome:     CONFIRMED | OVERRIDDEN | PENDING | null
  supervisor_id_hash:      str | null
  created_at:              datetime

CorrectionAction:
  # Positive outcomes (claim is accurate):
  CONFIRMED                   # verified against source
  INDEPENDENTLY_VERIFIED      # verified via independent collection
  # Negative outcomes (claim is problematic):
  MISREPRESENTS_SOURCE        # source exists but AI characterised it inaccurately
  SOURCE_NOT_FOUND            # GRND claim; cited source does not exist
  COULD_NOT_VERIFY            # honest acknowledgment; no verification path available
  PARAM_NOT_SUPPORTABLE       # PARAM claim; independent collection did not support it
  SYNTH_INFERENCE_UNSUPPORTED # SYNTH claim; synthesis conclusion not supported by sources
  OCR_ERROR                   # OCR transcription inaccuracy confirmed
  MEDIA_AUTHENTICITY_CONCERN  # MAS/EIDOLON concern confirmed by analyst

WeightingFactors:
  source_dwell_time_score:     float    # 0.0–1.0; based on time spent on source before action
  pressure_mode_factor:        float    # 1.0 (STANDARD) | 0.7 (ELEVATED) | 0.4 (DC)
  gaming_probability_factor:   float    # 1.0 - gaming_probability from TCS
  supervisory_confirmation:    float    # 0.0 (no review) | 0.8 (confirmed) | 1.0 (supervisor-initiated)
  verification_completeness:   float    # fraction of verification queue items completed this session
  # Note: weights are configurable; defaults above are initial values pending empirical validation

CorpusVersion:
  version_id:                uuid
  version_string:            str           # semantic version e.g., "1.4.2"
  created_at:                datetime
  record_count:              int
  weighted_record_count:     float         # sum of all correction weights
  source_type_distribution:  {}            # { GRND: float, PARAM: float, ... }
  domain_distribution:       {}
  quality_score:             float         # aggregate quality metric
  active:                    bool
  deactivated_at:            datetime | null
  deactivation_reason:       str | null    # ROLLBACK | SUPERSEDED | CONTAMINATION

OutcomeCorrection:
  outcome_id:               uuid
  correction_id:            uuid           # the correction record this outcome updates
  source:                   OFS_NEMESIS | SUPERVISOR | IOB
  outcome_classification:   CONFIRMED | DISCONFIRMED | AMBIGUOUS | SUPERSEDED
  outcome_weight:           float          # fixed high weight; bypasses 5-factor weighting
  source_event_id:          uuid           # FK → OFS/NEMESIS outcome event
  recorded_at:              datetime

HumanReviewQueueItem:
  item_id:                  uuid
  correction_id:            uuid
  reason:                   str            # why this was routed to human review
  status:                   PENDING | REVIEWED | CLOSED
  reviewer_id_hash:         str | null
  reviewed_at:              datetime | null
  reviewer_decision:        ACCEPT | REJECT | MODIFY | null
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CorrectionRecord, CorpusVersion, OutcomeCorrection, HumanReviewQueue | Indefinite |
| Corpus index | Elasticsearch | Full-text search and aggregate queries on the correction corpus | Mirrors primary |
| Event store | MOIRAI | Signed correction and corpus events | Indefinite |
| Corpus snapshot store | Object storage (S3-compatible) | CorpusVersion snapshots for rollback | Indefinite |

The corpus is append-only at the record level. Rollback does not delete records — it deactivates the affected corpus version and reactivates a prior clean version. The deactivated records remain in PostgreSQL as an audit trail of what was in the contaminated version.

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| CorrectionRecord | Inherits session classification | Claim-level content is session-compartmented |
| WeightingFactors | Controlled Unclassified | Analyst-specific; supervisor and IOB access |
| CorpusVersion metadata | Controlled Unclassified | Quality metrics are platform-level; accessible to Research & Red Team |
| OutcomeCorrection | Inherits outcome source classification | OFS/NEMESIS outcomes may inherit session classification |

### 3.4 Retention and Purge Policy

CorrectionRecord entries are retained indefinitely — the corpus history is the calibration history, and longitudinal analysis of corpus evolution is required for research. CorpusVersion snapshots in object storage are retained indefinitely (rollback capability). OFS/NEMESIS outcome corrections are retained indefinitely. HumanReviewQueue items are retained for session lifetime plus seven years.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /corrections
  Auth:     ATHENA service account
  Request:  {
    session_id:      uuid,
    turn_id:         uuid,
    claim_id:        uuid,
    analyst_id:      str,
    action:          CorrectionAction,
    source_type:     str,
    domain:          str,
    claim_type:      str,
    model_version:   str,
    pressure_mode:   str,
    dwell_seconds:   int          # from ATHENA UI telemetry
  }
  Response: {
    correction_id:   uuid,
    computed_weight: float,
    weighting_factors: WeightingFactors,
    routed_to_review: bool
  }
  SLA: p99 < 300ms

POST /corrections/supervisor-confirm
  Auth:     supervisor token
  Request:  {
    correction_id:   uuid,
    decision:        CONFIRMED | OVERRIDDEN,
    override_action: CorrectionAction | null,   # required if OVERRIDDEN
    rationale:       str
  }
  Response: {
    correction_id:   uuid,
    new_weight:      float,
    tcs_updated:     bool
  }
  SLA: p99 < 500ms

POST /corrections/outcome
  Auth:     OFS/NEMESIS service account
  Request:  {
    correction_id:   uuid,
    outcome:         CONFIRMED | DISCONFIRMED | AMBIGUOUS | SUPERSEDED,
    source_event_id: uuid
  }
  Response: { outcome_id: uuid, weight_applied: float }

GET /corpus/current
  Auth:     TCS service account | supervisor token | IOB token
  Response: CorpusVersion (metadata only; records not returned in bulk)

GET /corpus/{version_id}/quality
  Auth:     supervisor token | IOB token | Research & Red Team
  Response: {
    version:               CorpusVersion,
    weighting_distribution: { factor: { mean, stddev } },
    action_distribution:   { action: count },
    domain_coverage:       [{ domain, claim_type, n_records }],
    anomaly_flags:         [str]
  }

POST /corpus/rollback
  Auth:     IOB token (dual authorization required)
  Request:  {
    target_version_id:   uuid,
    reason:              str,
    authorization_ref:   str       # second authorization reference
  }
  Response: {
    rolled_back_from:   str,
    rolled_back_to:     str,
    records_deactivated:int
  }

GET /review-queue
  Auth:     supervisor token
  Response: [HumanReviewQueueItem]

GET /health
  Response: {
    status, dependencies: { moirai, tcs, pces, elasticsearch },
    active_corpus_version: str,
    pending_review_count:  int,
    last_event_hash:       str,
    corpus_record_count:   int
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          FGTS_CORRECTION_WEIGHTED
service_id:         "FGTS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  correction_id:          uuid
  claim_id:               uuid
  action:                 str
  source_type:            str
  domain:                 str
  claim_type:             str
  computed_weight:        float
  pressure_mode:          str
  gaming_probability_factor: float
  supervisory_confirmation:  float
  routed_to_review:       bool
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          FGTS_SUPERVISOR_CONFIRMED
event_payload:
  correction_id:          uuid
  supervisor_id_hash:     str
  decision:               CONFIRMED | OVERRIDDEN
  weight_delta:           float     # change in weight from supervisor action

EventType:          FGTS_CORPUS_VERSION_CREATED
event_payload:
  version_id:             uuid
  version_string:         str
  record_count:           int
  quality_score:          float

EventType:          FGTS_CORPUS_ROLLBACK
event_payload:
  rolled_back_from:       str
  rolled_back_to:         str
  records_deactivated:    int
  reason:                 str
  authorized_by:          str

EventType:          FGTS_OUTCOME_CORRECTION
event_payload:
  correction_id:          uuid
  outcome:                str
  weight_applied:         float
  source:                 str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `FGTS_CORRECTION_WEIGHTED` | Every correction submitted via ATHENA | MOIRAI, TCS/MIMIR (triggers posterior update) |
| `FGTS_SUPERVISOR_CONFIRMED` | Supervisor confirms or overrides a correction | MOIRAI, TCS/MIMIR (re-weights the correction) |
| `FGTS_CORPUS_VERSION_CREATED` | New corpus version snapshot | MOIRAI, IOB reporting |
| `FGTS_CORPUS_ROLLBACK` | Corpus rollback executed | MOIRAI, all services (calibration signals may change), IOB alert |
| `FGTS_OUTCOME_CORRECTION` | OFS/NEMESIS outcome received | MOIRAI, TCS/MIMIR (high-weight posterior update) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| TCS/MIMIR | `TCS_GAMING_DETECTED` | Applies lower gaming_probability_factor to that analyst's pending corrections |
| OFS/NEMESIS | `OFS_OUTCOME_CLASSIFIED` | Creates OutcomeCorrection record; publishes to TCS as high-weight update |
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | Flags corrections from prior model version for corpus version boundary |
| PCES/AEGIS | `PCES_SESSION_GRANTED` | Records pressure mode context available at session start |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| TCS/MIMIR | Trust Calibration | Gaming probability score for weighting | Sync query at correction submission | Gaming factor defaults to 0.5 (neutral); correction still accepted |
| MOIRAI | Provenance | Signed event emission; correction records linked to provenance chain | Async event | Events buffered; corrections still recorded in PostgreSQL |
| PCES/AEGIS | Classification Enforcement | Session token validation; analyst identity | Sync | Correction rejected; analyst must re-authenticate |
| OFS/NEMESIS | Outcome Feedback | Real-world outcomes for high-weight corrections | Async event | No outcome corrections received; standard corpus continues |

### 5.2 Feeds Into

| Service | Epithet | What FGTS provides | How |
|---|---|---|---|
| TCS/MIMIR | Trust Calibration | Weighted corrections for Bayesian posterior updates | `FGTS_CORRECTION_WEIGHTED` event → TCS consumer |
| IOB Reporting | Oversight | Corpus quality metrics; rollback history; supervisor confirmation rates | Audit query endpoints |
| Research & Red Team | Quality Analysis | Corpus quality distribution; weighting factor analytics | Corpus quality endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** `POST /corrections` validates session token. Correction is attributed to the authenticated analyst.
- **Compartment inheritance:** CorrectionRecord inherits the classification of the session and claim it references. The corpus is internally partitioned by classification — unclassified corrections do not inform calibration for classified claim types.
- **Failure behavior:** PCES unavailable → corrections cannot be authenticated → correction submission returns `503 PCES_UNAVAILABLE`. No corrections accepted without valid session authentication.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Correction submission (`POST /corrections`) | 80ms | 200ms | 300ms |
| Supervisor confirmation | 100ms | 300ms | 500ms |
| Outcome correction | 100ms | 300ms | 500ms |
| Corpus quality query | 500ms | 2000ms | 5000ms |
| Corpus rollback | 5s | 30s | 120s |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Correction submissions/second | 50 (peak verification queue activity) |
| Supervisor confirmations/day | 100 |
| Outcome corrections/day | 20 (low frequency — depends on OFS/NEMESIS) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — FGTS unavailability suspends calibration updates but does not stop sessions |
| MOIRAI event durability | 99.999% |
| Corpus snapshot durability | 99.9999% (rollback targets must not be lost) |
| RTO | 15 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| TCS/MIMIR | Gaming factor defaults to 0.5 for new corrections. Posterior updates queue. Alert. | Corrections still accepted; calibration updates deferred |
| MOIRAI | Events buffered; corrections recorded in PostgreSQL. Provenance gap logged. | No analyst-facing impact |
| Elasticsearch index | Corpus query endpoints unavailable. Corrections still recorded in PostgreSQL. Alert. | Supervisory corpus quality dashboard unavailable |
| OFS/NEMESIS | No outcome corrections received. Standard corpus continues. | Calibration loop from real-world outcomes suspended |

---

## 7. Security Model

### 7.1 Authentication

ATHENA submits corrections via service account. Individual correction records are authenticated to the analyst via the session token embedded in the submission. Supervisory confirmation requires supervisor token. Corpus rollback requires IOB token plus a second-authorization reference (dual-person integrity for the most sensitive operation in the platform).

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ATHENA service | `POST /corrections` | Service account |
| TCS/MIMIR | Gaming probability query | Service account |
| OFS/NEMESIS | `POST /corrections/outcome` | Service account |
| Supervisor | `/corrections/supervisor-confirm`; review queue; corpus quality (team scope) | Supervisor token |
| IOB | All endpoints including corpus rollback | IOB token + second authorization |
| Research & Red Team | Corpus quality analytics (de-identified) | Research team token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/fgts/signing-key` | 90 days |
| PostgreSQL credentials | `themis/fgts/db-credentials` | 30 days |
| Elasticsearch credentials | `themis/fgts/es-credentials` | 30 days |
| Object storage credentials | `themis/fgts/snapshot-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

**Coordinated corpus poisoning** is the primary threat: multiple analysts with good gaming scores submitting systematically false corrections in a specific domain to bias calibration. The five-factor weighting provides defense in depth: a coordinated attack requires gaming detection scores to be low (which is observable over time), correction patterns that avoid triggering anomaly monitoring, and either no supervisory review or a compromised supervisor. The IOB monthly report is the final detection layer — aggregate corpus anomalies that individual gaming scores might not catch.

**Single-analyst gaming** is the secondary threat: an individual analyst performing rapid, low-effort verification to build calibration data quickly. The source dwell time score, verification completeness factor, and gaming probability score from TCS together make this expensive to execute without detection.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Weighting factor miscalibration (factors produce wrong weights) | Medium | P1 — calibration systematically biased | Research & Red Team corpus quality audit | Factor weights are configurable; tuning requires pre-registered empirical criteria |
| Corpus contamination (coordinated false corrections) | Low | P1 — calibration signals in affected domain unreliable | IOB aggregate monitoring; corpus quality anomaly detection | Rollback capability; dual-authorization requirement; IOB monthly review |
| Supervisory bottleneck (review queue grows faster than supervisors review) | Medium | P2 — high-weight confirmations delayed | Queue depth monitoring | Alert at queue depth > 50; supervisory workload planning |
| Object storage failure (snapshot unavailable for rollback) | Low | P0 for rollback operations | Snapshot integrity check | Multi-region object storage; integrity verification after each snapshot |

### 8.1 Known Design Risks

- **Weighting factor weights are not empirically validated.** The five factors and their relative contributions to the composite weight are theoretically motivated but not empirically calibrated. The initial weights (source dwell: 0.3, pressure mode: 0.2, gaming: 0.25, supervisory: 0.15, completeness: 0.1) are best estimates. These must be treated as priors to be updated as outcome data from OFS/NEMESIS accumulates. The Research and Red Team must run a pre-registered calibration study of the weighting model within the first year of operation.
- **Source dwell time is a proxy, not a measurement.** Dwell time (seconds spent on source document in ATHENA before submitting a verification action) is a behavioral signal, not direct evidence of genuine verification. A sophisticated evader who knows the dwell time threshold can simply keep the source open for the required time without actually reading it. Mitigation: dwell time is one of five factors; it does not dominate the composite. The gaming probability score from TCS provides a complementary signal that considers behavioral patterns across the session.
- **The supervisory confirmation bottleneck may create perverse incentives.** If supervisory confirmation carries the highest weight and supervisors have limited bandwidth, analysts with frequent supervisory interaction will accumulate calibration faster than isolated analysts. This could create a two-tier calibration system. Mitigation: supervisory confirmation enhances weight but is not required for corrections to enter the corpus — the base five-factor weighting is sufficient for standard corrections.
- **Rollback removes calibration signal, not just bad corrections.** Rolling back a corpus version removes all corrections in that version, including genuine corrections that happened to be in the same version as the contaminated ones. The rollback is a blunt instrument. Mitigation: corpus versions are created frequently (configurable, default: weekly) to limit the amount of genuine signal lost in a rollback. The deactivated records are retained for manual review and selective re-entry.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `fgts.correction.latency_p99` | Histogram | `> 300ms for 5m` | P1 |
| `fgts.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `fgts.corpus.weight_mean` | Gauge | `< 0.3 sustained over 1h` (low quality signal) | P2 |
| `fgts.review_queue.depth` | Gauge | `> 50` | P2 |
| `fgts.gaming_factor.mean` | Gauge | `< 0.5 sustained over 1h` (high gaming signal) | P1 |
| `fgts.corpus.version_age_hours` | Gauge | `> 200h` (> 1 week without new version) | P2 |
| `fgts.snapshot.integrity_check` | Gauge | Failure | P0 |

### 9.2 Health Check

```
GET /health
Response: {
  status:                 "healthy" | "degraded" | "unavailable",
  dependencies: {
    moirai:               "healthy" | "unavailable",
    tcs:                  "healthy" | "unavailable",
    pces:                 "healthy" | "unavailable",
    elasticsearch:        "healthy" | "degraded" | "unavailable",
    object_storage:       "healthy" | "unavailable"
  },
  active_corpus_version:  str,
  corpus_record_count:    int,
  pending_review_count:   int,
  last_snapshot_at:       datetime,
  last_event_hash:        str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":             "ISO-8601",
  "service":               "FGTS/ALETHEIA",
  "level":                 "INFO | WARN | ERROR",
  "event":                 "CORRECTION_WEIGHTED | SUPERVISOR_CONFIRMED | OUTCOME_RECEIVED | CORPUS_ROLLBACK | SNAPSHOT_CREATED",
  "correction_id":         "uuid | null",
  "session_id":            "uuid | null",
  "corpus_version":        "string",
  "duration_ms":           0,
  "fields": {
    "computed_weight":     0.0,
    "action":              "string",
    "source_type":         "string",
    "pressure_mode":       "string",
    "gaming_factor":       0.0,
    "routed_to_review":    false
  }
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/fgts/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant
- **Prev_event_hash source:** Prior FGTS event in the platform-wide quality event stream

### 10.2 What This Service Attests

The MOIRAI record for FGTS proves that a specific analyst submitted a specific verification outcome for a specific claim at a specific time, with a specific computed weight based on measurable behavioral signals, and that this record has not been altered since it was written. An oversight body can query FGTS events to reconstruct the complete correction history and weighting rationale for any calibration update.

The corpus rollback event proves that a specific version was deactivated at a specific time for a specific reason, with dual-person authorization. The full audit trail of what was in the deactivated version is preserved.

### 10.3 What This Service Cannot Prove

FGTS proves the correction was submitted and weighted as recorded. It does not prove the analyst's verification was genuine — dwell time is a behavioral proxy, not evidence of comprehension. It does not prove the correction was accurate. Whether CONFIRMED corrections were actually correct is determined by OFS/NEMESIS outcome data, not by FGTS.

---

## 11. Implementation Roadmap

### Phase 1 — Correction Intake and Weighting (Weeks 29–36)

- CorrectionRecord schema and `POST /corrections` endpoint
- Five-factor weighting implementation (initial weights as configured defaults)
- MOIRAI event emission: `FGTS_CORRECTION_WEIGHTED`
- TCS/MIMIR integration: weighted correction published to TCS after each submission
- Basic corpus version management (manual snapshot creation)
- ATHENA integration: verification queue correction submission

**Phase gate criterion:** Every ATHENA verification action produces a CorrectionRecord with a computed weight. Weights are non-trivially different across verification behaviors (dwell time variation, pressure mode variation). TCS receives weighted corrections and updates posteriors.

### Phase 2 — Supervisory Confirmation, Corpus Versioning, and Rollback (Weeks 37–46)

- Supervisory confirmation endpoint and human review queue
- Automated corpus versioning on configurable schedule
- Corpus snapshot to object storage with integrity verification
- Rollback endpoint with dual-authorization
- OFS/NEMESIS outcome integration: `POST /corrections/outcome`
- Corpus quality monitoring endpoints
- IOB audit endpoints

**Phase gate criterion:** Supervisory confirmation demonstrably increases correction weight and triggers TCS posterior update. Corpus rollback executes successfully in test. OFS/NEMESIS outcome corrections enter corpus at higher weight than standard corrections. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-5 | Definition of "confirmed" and "disconfirmed" for calibration purposes | Required before OFS/NEMESIS outcome integration in Phase 2 |

*Note: The five-factor weighting scheme and corpus version rollback authority are platform design decisions, not policy decisions requiring GC items. The IOB's role is oversight of the corpus, not approval of the weighting methodology. However, any proposal to modify the weighting factors after initial deployment requires Research and Red Team pre-registration and IOB notification.*

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In the ATHENA verification queue, each claim shows:
- A dwell time indicator (a progress bar that fills as the analyst spends time on the source — not a timer, a signal to the analyst that the platform measures engagement)
- The verification action options with plain-language descriptions
- The current pressure mode in the session header

After submitting a verification action, ATHENA shows a brief confirmation: "Verification recorded. Your calibration in [domain] is being updated."

### 13.2 What the Analyst Should Do

Verification is not compliance theater. The calibration system only becomes useful if corrections reflect genuine verification — a correction submitted after actually reading the source and confirming the AI's characterization is worth more than one submitted after a glance. This is in the analyst's interest: a calibration model built on genuine verification produces confidence signals the analyst can actually rely on. Performative verification degrades the analyst's own calibration.

When the source document is a PARAM claim with no specific source to check: the correct action is `COULD_NOT_VERIFY` or `INDEPENDENTLY_VERIFIED` if independent collection was used. Selecting `CONFIRMED` for a PARAM claim misrepresents the verification action and will have a lower weight in the calibration corpus.

### 13.3 What the Signal Does Not Mean

A high computed weight on a correction does not mean the correction is factually correct — it means the behavioral signals suggest genuine verification occurred. A low weight does not mean the correction is wrong — it means the behavioral signals suggest the verification may have been less rigorous. The weighting is a quality signal about the verification process, not a determination about the underlying claim.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Optimal corpus versioning cadence.** Too frequent (daily) produces many small versions, increasing rollback granularity but reducing version meaningfulness. Too infrequent (monthly) means a contaminated month is a large rollback. Default of weekly is a reasonable starting point pending operational experience. Resolution path: review after six months of operation.
- **Q2: Threshold for routing corrections to human review.** The current design routes corrections with gaming probability > 0.7 to the supervisor review queue. This threshold is configurable and has not been empirically validated. Resolution path: Research and Red Team analysis of false positive rate at 0.7, 0.8, and 0.9 after three months of operation.

### 14.2 Research Dependencies

- **Weighting factor empirical calibration.** The five factors and their weights are the most important research question for FGTS. The Research and Red Team must design a pre-registered study comparing calibration accuracy under different weighting schemes before the initial weights are treated as production-validated. Target: six months of operation, then weighting review. The OFS/NEMESIS outcome data is the ground truth for this study — FGTS cannot validate its own weights without real-world outcome data.

### 14.3 Operational Assumptions

- **Assumption 1: ATHENA can measure source dwell time accurately.** The dwell time score requires the ATHENA UI to track how long the analyst has the source document open before submitting a verification action. This is a UI instrumentation requirement. If ATHENA cannot instrument this reliably (e.g., analyst opens source in a separate window), dwell time defaults to a neutral value and the factor is set to 0.5.
- **Assumption 2: Supervisors have bandwidth for the review queue.** The supervisory confirmation pathway is the highest-weight signal in the system. If supervisors do not review the queue, the high-weight signal is never produced, and calibration relies entirely on analyst self-reported verification. The supervisory review cadence must be specified in the AI Trust Cell operating model before Phase 2 deployment.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

---

## Appendix A: Five-Factor Weighting Reference

| Factor | Signal | Default weight | Range |
|---|---|---|---|
| Source dwell time | Seconds on source document before verification action | 0.30 | 0.0–1.0 per factor |
| Pressure mode | Standard / Elevated / Deadline-Critical | 0.20 | 1.0 / 0.7 / 0.4 per mode |
| Gaming probability | 1.0 − TCS gaming probability score | 0.25 | 0.0–1.0 |
| Supervisory confirmation | No review / Confirmed / Supervisor-initiated | 0.15 | 0.0 / 0.8 / 1.0 |
| Verification completeness | Fraction of verification queue completed this session | 0.10 | 0.0–1.0 |

*Composite weight = weighted sum, normalised to 0.0–1.0. Factor weights are configurable and subject to Research & Red Team calibration review.*

*Outcome corrections from OFS/NEMESIS bypass the five-factor formula and are assigned a fixed weight of 0.95 — substantially higher than any five-factor combination can produce.*

---

## Appendix C: Calibration and Outcome Data

### What FGTS contributes to calibration

Every weighted correction is published to TCS/MIMIR immediately after submission. TCS uses the correction weight as the update magnitude for the Bayesian posterior: high-weight corrections move the posterior more than low-weight corrections. A supervisory-confirmed correction from a senior analyst with long source dwell time under standard conditions produces a posterior update roughly 8× as large as a rapid low-gaming-probability correction under Deadline-Critical conditions.

### How outcome data from OFS/NEMESIS affects FGTS

OFS/NEMESIS real-world outcomes are the most valuable calibration signal in the system. They are independent of analyst behavior — whether the adversary did or did not do what the assessment said they would is not affected by how carefully the analyst verified the AI's source citations. When OFS/NEMESIS classifies an outcome as CONFIRMED or DISCONFIRMED, FGTS creates an OutcomeCorrection at weight 0.95 and publishes it to TCS. These are rarer than analyst corrections (depending on OFS/NEMESIS data availability) but carry disproportionate calibration weight. The Research and Red Team calibration study (Section 14.2) will use outcome corrections as the ground truth for validating the five-factor weighting scheme.

---

## Appendix D: Red Team Findings

*Pending red team evaluation — scheduled for Phase 5 gate review.*
