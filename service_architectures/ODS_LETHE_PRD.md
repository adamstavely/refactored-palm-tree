# ODS — Out-of-Distribution Screening Service
### LETHE · *"Greek river of oblivion in Hades — those who drank its waters forgot everything; the territory beyond what the model has learned is the territory its patterns cannot reach"*
*THEMIS Platform · Service PRD · v1.0 · **PROPOSED — Pending ARB Approval***

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `ODS` |
| **Epithet** | `LETHE` |
| **Full name** | Out-of-Distribution Screening Service |
| **Namespace** | `themis-quality` (proposed) |
| **Layer** | Quality Layer — Competence Axis (proposed) |
| **Build phase** | Proposed — Year 3+ pending ARB approval |
| **Build priority** | Proposed — would be 25th service if approved |
| **Owner team** | THEMIS Platform Team |
| **Status** | **PROPOSED** |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | **Competence (proposed 4th axis)** — detects when queries or content fall outside the model's training distribution |

---

> **Proposal status note:** ODS/LETHE is a proposed extension complementing CPS/APORIA on the Competence Axis. Where CPS provides capability zone profiles derived from empirical benchmarks, ODS provides real-time distributional distance assessment — detecting when a specific query or content item falls far enough outside the model's training distribution that its learned patterns cannot reliably apply. Not part of the approved platform. Approval requires ARB endorsement of the OOD detection methodology and IOB approval of threshold policy.

---

## 1. The Distributional Distance Problem

### 1.1 The Limitation CPS/APORIA Does Not Fully Address

CPS/APORIA provides capability zone assessments at the domain × claim type level — empirically derived from evaluation benchmarks. This is a structural assessment: "the model is not reliably capable on capability assessment claims in technical programme domains."

What CPS/APORIA cannot address is the case-level distributional problem: within a domain and claim type where the model has demonstrated reasonable capability, there may be specific content items — specific technical specifications, specific proprietary terminology, specific classified programme details — that fall so far outside the model's training distribution that even within a Green-zone domain, the model's learned patterns do not apply to this specific item.

This is the out-of-distribution problem. The model was trained on a distribution of content. When it encounters content far from that distribution — content types it has rarely or never seen during training — its outputs become unreliable in ways that capability benchmarks on representative content will not detect, because the benchmark tests were drawn from the in-distribution population.

### 1.2 The Question This Service Answers

**ODS/LETHE answers: Is this specific query or content item far enough outside the model's training distribution that its learned patterns cannot be trusted to apply — even in a domain where the model has demonstrated general capability?**

### 1.3 The Complementary Relationship with CPS/APORIA

CPS/APORIA and ODS/LETHE address the Competence Axis from different angles:

| | CPS/APORIA | ODS/LETHE |
|---|---|---|
| **Scope** | Domain × claim type level | Specific query / content item level |
| **Method** | Empirical benchmark performance | Embedding-space distributional distance |
| **Timing** | Pre-session (zone lookup) | Real-time (per query/item) |
| **Output** | Green / Amber / Red zone | OOD distance score + threshold flag |
| **Requires** | Evaluation programme | Training distribution characterisation |

Both are needed. CPS without ODS misses in-distribution anomalies. ODS without CPS lacks the empirical grounding that makes zone assignments credible. They are designed to be deployed together, but each has independent value if the other is not yet available.

### 1.4 Design Principles

- **OOD detection operates in embedding space, not feature space.** Whether a query is out-of-distribution is determined by its proximity to the training distribution in the model's own representation space — not by surface features like vocabulary or topic. This is technically demanding but analytically correct.
- **OOD scores are continuous, not binary.** A query is not simply "in distribution" or "out of distribution" — it falls at some distance from the training distribution centroid. The analyst sees a continuous OOD score, not a binary flag.
- **OOD flags are advisory below a threshold, hard ceiling above.** A low OOD score produces no action. A moderate score elevates the model_dominance signal in UCS/TYCHE. A high score — above the IOB-approved threshold — applies a hard confidence ceiling equivalent to a CPS Red zone.
- **The training distribution characterisation must be updated per model version.** Each model version has a different training distribution. The OOD detection model must be re-trained or re-calibrated whenever MDS/KRONOS registers a new model version.

---

## 2. Core Responsibilities (Proposed)

### 2.1 Primary Function

ODS/LETHE computes real-time out-of-distribution scores for analyst queries and retrieved content items by measuring embedding-space distance from the current model version's training distribution characterisation, applies threshold-based signals to UCS/TYCHE for uncertainty characterisation, and publishes high-OOD events to MOIRAI for the provenance record.

### 2.2 Secondary Functions

- Training distribution characterisation: maintaining a compressed representation of each model version's training distribution in embedding space
- Content-item OOD scoring: for retrieved corpus items, computing OOD scores that surface alongside validity scores
- IAS/SCUDO coordination: high OOD content items are elevated for adversarial screening (OOD content may indicate adversarially crafted inputs designed to exploit distributional blind spots)
- OOD pattern tracking: logging OOD detections by domain to identify systematic distributional gaps that should inform future model training or collection priorities
- Temporal OOD: content items with creation dates after model training cutoff have inherent temporal OOD properties — surfacing these as a distinct signal type distinct from TVS/KAIROS source validity

---

## 3. Data Architecture (Proposed)

### 3.1 Primary Data Models

```yaml
DistributionModel:
  model_id:                uuid
  model_version:           str              # FK → MDS/KRONOS ModelVersion
  training_distribution:   str              # reference to compressed distribution representation
  embedding_dimension:     int
  centroid_refs:           [str]            # cluster centroid references
  characterised_at:        datetime
  sample_size:             int              # training examples used to characterise
  coverage_domains:        [str]            # domains well-represented in training

OodRecord:
  record_id:               uuid
  session_id:              uuid
  turn_id:                 uuid
  content_type:            QUERY | RETRIEVED_CHUNK | MCP_RESPONSE
  content_hash:            str
  ood_score:               float            # 0.0 (in-distribution) – 1.0 (far OOD)
  distance_percentile:     float            # percentile of training distribution distances
  signal_level:            ADVISORY | ELEVATED | CEILING_APPLIED
  model_version:           str
  temporal_ood:            bool             # true if creation date > training cutoff
  flagged_at:              datetime

OodSignal:
  signal_id:               uuid
  record_id:               uuid
  signal_type:             UCS_MODEL_ELEVATION | HARD_CEILING | IAS_ELEVATION
  threshold_triggered:     float
  action_taken:            str
  timestamp:               datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Distribution models | Object storage | Compressed distribution representations | Per model version |
| OOD records | PostgreSQL | OodRecord, OodSignal | Session + 2 years |
| Distribution cache | Redis | Active centroid model for inference | Model version TTL |
| Event store | MOIRAI | Signed OOD events | Indefinite |

---

## 4. API Contract (Proposed)

### 4.1 Endpoints

```
POST /score/query
  Auth:     inference gateway service account
  Request:  {
    session_id:            uuid,
    turn_id:               uuid,
    query_embedding:       [float],         # query embedding vector
    model_version:         str
  }
  Response: {
    record_id:             uuid,
    ood_score:             float,
    signal_level:          str,
    ucs_model_elevation:   bool,
    ceiling_applied:       bool,
    ceiling_value:         float | null
  }
  SLA: p99 < 50ms (latency-critical; runs on every query)

POST /score/content
  Auth:     retrieval gateway service account
  Request:  {
    session_id:            uuid,
    turn_id:               uuid,
    chunk_id:              str,
    content_embedding:     [float],
    creation_date:         datetime | null
  }
  Response: {
    record_id:             uuid,
    ood_score:             float,
    signal_level:          str,
    temporal_ood:          bool
  }
  SLA: p99 < 50ms

GET /distribution/{model_version}/coverage
  Auth:     Research & Red Team | IOB token
  Response: {
    model_version:         str,
    sample_size:           int,
    coverage_domains:      [str],
    uncovered_domains:     [str] | null
  }

POST /distribution/{model_version}
  Auth:     Research & Red Team token
  Request:  {
    training_distribution: str,            # reference to distribution artefact
    centroid_refs:         [str],
    coverage_domains:      [str],
    sample_size:           int
  }
  Response: { model_id: uuid }

GET /health
  Response: {
    status, dependencies: { moirai, redis },
    active_model_version:  str,
    ood_flags_24h:         int,
    ceiling_applied_24h:   int,
    inference_latency_p99: float,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          ODS_CEILING_APPLIED
service_id:         "ODS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  record_id:              uuid
  content_type:           str
  ood_score:              float
  ceiling_value:          float
  model_version:          str
  temporal_ood:           bool
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          ODS_ELEVATED_SIGNAL
event_payload:
  record_id:              uuid
  ood_score:              float
  signal_level:           str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `ODS_CEILING_APPLIED` | OOD score exceeds hard ceiling threshold | MOIRAI, UCS/TYCHE (ceiling applied), ATHENA (ceiling indicator) |
| `ODS_ELEVATED_SIGNAL` | OOD score exceeds advisory threshold | MOIRAI, UCS/TYCHE (model_dominance elevation) |

---

## 5. Integration Map (Proposed)

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MDS/KRONOS | Model Drift | Model version change triggers distribution model update | Async event | Operating on stale distribution model; alert P1 |
| MOIRAI | Provenance | Signed OOD events | Async event | Events buffered |

### 5.2 Feeds Into (Proposed)

| Service | Epithet | What ODS provides | How |
|---|---|---|---|
| UCS/TYCHE | Uncertainty | model_dominance elevation and confidence ceiling | `POST /score/query` response |
| IAS/SCUDO | Adversarial | High OOD content flagged for elevated screening | API signal |
| ATHENA | Interface | OOD indicator in session (advisory and ceiling) | API |
| IOB Reporting | Oversight | OOD pattern summary by domain | API |

---

## 6. Non-Functional Requirements (Proposed)

**The latency requirement is the primary design constraint.** ODS scores every query and every retrieved content item. At 50 concurrent analysts with 10 retrieved chunks per turn, this is 500+ OOD scoring operations per turn, all on the critical path between query and inference. The p99 < 50ms target is aggressive and requires the distribution model to be fully loaded in memory (Redis) with vector distance computation optimised for production throughput.

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Query OOD scoring | 5ms | 20ms | 50ms |
| Content OOD scoring | 5ms | 20ms | 50ms |

---

## 7. Known Design Risks (Proposed)

### 7.1 Training Distribution Characterisation Is Technically Hard

Accurately characterising a large language model's training distribution in a form that enables real-time embedding-space distance computation is a non-trivial ML engineering problem. It requires:
- Access to model internals (or a proxy model) for embedding computation
- Sufficient training data representation to build a meaningful distribution characterisation
- A distance metric that correlates with actual model performance degradation on OOD content

The Research & Red Team must validate that the OOD score correlates with actual accuracy degradation before this service is operationalised. A high OOD score that does not predict performance degradation is analytically useless and creates false ceiling applications. Resolution path: pilot study validating OOD score vs. CPS capability zone correlation on a labelled test set before ARB proposal is submitted.

### 7.2 The 50ms Latency Target May Not Be Achievable at Scale

The p99 < 50ms target requires embedding computation and vector distance operations in under 50 milliseconds per scoring call, under concurrent load. This is achievable with optimised vector similarity libraries and pre-loaded distribution models, but has not been validated under realistic load conditions. If the latency target cannot be met, the service would need to run asynchronously (scoring after inference, flagging retrospectively) rather than on the critical path. This changes the design significantly — retrospective flagging means the AI has already used the OOD content before the flag is applied. Resolution path: latency benchmarking must be completed before ARB approval.

### 7.3 Temporal OOD Is a Distinct but Overlapping Signal

Content created after the model's training cutoff has inherent distributional distance from the training data — the model has not seen it. This temporal OOD signal overlaps with TVS/KAIROS source validity but is not identical: a stale source (TVS) may be in-distribution; a current source about a newly emerging programme area may be OOD. The design distinguishes `temporal_ood: bool` from the main OOD score to avoid conflating the two signals. Analyst training must address both signal types clearly.

---

## 8. Proposal Requirements for ARB Approval

1. **Technical feasibility validation.** Latency benchmarking demonstrating p99 < 50ms under realistic concurrent load before ARB submission.

2. **OOD score validity study.** Research & Red Team pilot study demonstrating that OOD scores correlate with actual model accuracy degradation on a labelled test set. Without this, the service has no demonstrated analytical value.

3. **Distribution characterisation methodology.** A specific technical specification of how the training distribution is characterised and how it will be updated per model version, reviewed by the Research & Red Team.

4. **IOB threshold policy.** The IOB must define the OOD score threshold at which a hard confidence ceiling is applied and the ceiling value — this is a policy decision with direct analyst impact requiring IOB endorsement.

5. **CPS/APORIA relationship.** ODS is more valuable when CPS/APORIA is also operational — they cover different aspects of the Competence Axis. ARB should consider the two proposals together.

---

## 9. Implementation Roadmap (Proposed — Contingent on ARB Approval)

### Phase 1 — Distribution Characterisation and Advisory Scoring

- DistributionModel schema and deployment
- Real-time query and content scoring endpoints
- Advisory OOD signal to UCS/TYCHE
- MOIRAI events
- Latency validation under load

### Phase 2 — Hard Ceiling, IAS Integration, and Temporal OOD

- Hard ceiling enforcement via UCS/TYCHE (IOB threshold policy required)
- IAS/SCUDO elevated screening for high-OOD content
- Temporal OOD signal differentiation
- Distribution model update pipeline for MDS/KRONOS model version changes
- IOB pattern reporting

---

## 10. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| ARB approval | Full OOD screening service approval | Phase 1 begins |
| IOB threshold policy | OOD score threshold for hard ceiling application and ceiling value | Phase 2 ceiling enforcement |
| Research & Red Team programme | OOD validity study and distribution characterisation methodology | ARB proposal submission |

---

## 11. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial proposal PRD — Addendum D Competence Axis specification |

## Appendix D: Red Team Findings

*No red team assessment conducted. The Research & Red Team OOD validity study (Section 8.2) must be completed before ARB review. That study will serve as the initial red team assessment for the service.*
