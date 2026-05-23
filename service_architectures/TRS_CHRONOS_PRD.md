# TRS — Temporal Reasoning Service
### CHRONOS · *"Greek personification of sequential time — not just duration but the ordered unfolding of events; he who holds the thread of sequence and can project where it leads"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `TRS` |
| **Epithet** | `CHRONOS` |
| **Full name** | Temporal Reasoning Service |
| **Namespace** | `themis-warning` |
| **Layer** | Intelligence Layer — Warning |
| **Build phase** | Year 3 (Addendum F) |
| **Build priority** | 10 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — models where analytical subjects are heading, not just where they are |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**TRS/CHRONOS answers: Given what we know about this entity or situation, what trajectory is it on — and what is the range of plausible future states, at what timelines, with what observable precursors?**

### 1.2 Why This Service Exists

TVS/KAIROS tracks whether intelligence sources are still current. That is a backward-looking question: is what we collected then still valid now? TRS/CHRONOS is the forward-looking question: where is the subject of our intelligence heading?

Decision-makers do not act on static intelligence. They act on intelligence that tells them what will be true at the decision point — which is always in the future. An assessment that accurately characterises today's adversary capability without projecting where that capability will be at the decision point is less useful than an assessment that reaches the decision-maker precisely when it is needed.

TRS/CHRONOS provides trajectory modelling, leading indicator monitoring, and scenario space generation — the three analytical functions required to move from "here is what we know" to "here is where this is going and what would tell us which way."

This service was specified in Addendum F. Distinct from TVS/KAIROS (source currency) and distinct from ORACLE (outcome prediction from historical patterns) — TRS reasons forward from current evidence, not backward from historical outcomes.

### 1.3 Design Principles

- **Trajectories are probabilistic, not deterministic.** TRS produces scenario spaces with probability distributions, not single-point predictions. An assessment that presents a single future state as certain is analytically dishonest regardless of how confident the model is.
- **Leading indicators are the actionable output.** A trajectory projection tells the decision-maker where things are heading. A leading indicator tells them what to watch — what observable event would confirm or disconfirm the projected trajectory. Leading indicators are what intelligence collection can task against.
- **GC-8 governs forecast products.** Temporal projections are a distinct analytical product type from current intelligence assessments. GC-8 defines the governance framework for forecast products including disclosure requirements and dissemination authority.
- **Temporal uncertainty is a distinct UCS/TYCHE component.** Long-horizon projections for inherently probabilistic subjects have high temporal uncertainty. TRS adds `temporal_horizon_dominance` to UCS/TYCHE profiles, making temporal uncertainty visible alongside aleatory, epistemic, and model uncertainty.

### 1.4 Explicit Out of Scope

- **Real-time forecasting.** TRS produces analytical trajectory models from corpus evidence; it does not receive real-time collection data feeds.
- **Predicting specific adversary decisions.** TRS models observable trajectories (capability development rate, programme stage progression, force posture changes). Decision-making is inherently aleatory and is handled by UCS/TYCHE's aleatory uncertainty characterisation, not TRS trajectory models.
- **Long-range forecasting beyond 3–5 years.** The further the projection horizon, the wider the confidence interval until the projection becomes analytically meaningless. TRS caps projection horizons at configurable limits by domain.

---

## 2. Core Responsibilities

### 2.1 Primary Function

TRS/CHRONOS builds trajectory models for analytical subjects from time-series data in the corpus, identifies leading indicators whose presence or absence would confirm or disconfirm projected trajectories, generates a structured scenario space with probability distributions and observable conditions for each scenario, and feeds the temporal uncertainty component to UCS/TYCHE for incorporation into assessment uncertainty profiles.

### 2.2 Secondary Functions

- Programme stage inference: for entities at known programme development stages, inferring likely timeline to next stage
- Historical pattern matching: identifying similar historical programmes and their development trajectories
- Leading indicator status monitoring: tracking whether active leading indicators are emerging, present, or absent
- Scenario probability updating: revising scenario probabilities as new evidence arrives or leading indicators change status
- GC-8 compliant forecast product generation: producing temporally-scoped assessments in the format required by GC-8
- Briefing summary for PCS/IRIS: contributing the scenario space summary to PCS/IRIS consumer packages for high-priority assessments

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
TrajectoryModel:
  model_id:                uuid
  entity_id:               uuid             # FK → OGS entity
  domain:                  str
  variable:                str              # what is being modelled (e.g., "capability maturity")
  data_points:             [DataPoint]
  model_type:              LINEAR | EXPONENTIAL | SIGMOID | STEP | CYCLICAL
  current_estimate:        float
  trajectory_vector:       float            # direction and rate of change
  projection_horizon_days: int
  confidence_at_horizon:   float
  last_updated:            datetime

DataPoint:
  point_id:                uuid
  model_id:                uuid
  timestamp:               datetime
  value:                   float
  source_id:               str              # corpus source ID
  confidence:              float

Scenario:
  scenario_id:             uuid
  model_id:                uuid
  name:                    str
  probability:             float            # 0.0–1.0; scenario probabilities sum to <= 1.0
  conditions:              [str]            # what must be true for this scenario
  projected_state:         str              # plain language: what this scenario looks like
  timeline:                str              # when this would occur
  observable_indicators:   [uuid]           # FK → LeadingIndicator IDs that would confirm this
  last_probability_update: datetime

LeadingIndicator:
  indicator_id:            uuid
  domain:                  str
  description:             str
  historical_lead_days:    int              # how many days before event this typically appears
  historical_evidence_count:int
  current_status:          ABSENT | EMERGING | PRESENT | STRONG | UNKNOWN
  last_status_update:      datetime
  associated_scenarios:    [uuid]           # scenario_ids this indicator is relevant to
  source_refs:             [str]            # corpus sources informing current status

TemporalUncertaintyProfile:
  profile_id:              uuid
  model_id:                uuid
  session_id:              uuid
  temporal_horizon_dominance: high | medium | low
  horizon_days:            int
  confidence_interval:     { lower: float, upper: float }
  basis:                   str              # why this temporal uncertainty level
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | TrajectoryModel, Scenario, LeadingIndicator | Indefinite |
| Time-series store | PostgreSQL (timescaleDB extension) | DataPoint records | Indefinite |
| Event store | MOIRAI | Signed trajectory and scenario events | Indefinite |
| Model cache | Redis | Active trajectory models (frequently queried) | 4h TTL |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /trajectories
  Auth:     ATHENA service account | analyst session token
  Request:  {
    entity_id:             uuid,
    domain:                str,
    variable:              str,
    projection_horizon_days:int
  }
  Response: {
    model_id:              uuid,
    current_estimate:      float,
    trajectory_vector:     float,
    scenarios:             [Scenario],
    leading_indicators:    [LeadingIndicator],
    confidence_at_horizon: float
  }
  SLA: p99 < 3000ms

GET /trajectories/{model_id}
  Auth:     analyst session token | supervisor token
  Response: TrajectoryModel with Scenario and LeadingIndicator arrays

GET /trajectories/{model_id}/scenarios
  Auth:     analyst session token
  Response: [Scenario] ordered by probability descending

PATCH /trajectories/{model_id}/update-indicator
  Auth:     analyst session token (analyst can update indicator status from new evidence)
  Request:  {
    indicator_id:          uuid,
    new_status:            str,
    evidence_ref:          str
  }
  Response: {
    indicator_id:          uuid,
    scenario_probability_updates: [{ scenario_id, old_prob, new_prob }]
  }

POST /temporal-uncertainty
  Auth:     UCS service account
  Request:  {
    session_id:            uuid,
    model_id:              uuid,
    horizon_days:          int
  }
  Response: TemporalUncertaintyProfile

GET /forecast-products/{model_id}
  Auth:     dissemination authority (GC-8 compliant access)
  Response: GC-8 formatted forecast product

GET /health
  Response: {
    status, dependencies: { moirai, ogs, pces, redis },
    active_models:         int,
    models_updated_24h:    int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          TRS_TRAJECTORY_CREATED
service_id:         "TRS"
session_id:         uuid
classification:     str
event_payload:
  model_id:               uuid
  entity_id:              uuid
  domain:                 str
  scenario_count:         int
  projection_horizon_days:int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          TRS_SCENARIO_PROBABILITY_REVISED
event_payload:
  model_id:               uuid
  scenario_id:            uuid
  old_probability:        float
  new_probability:        float
  trigger:                str
```

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| OGS/YGGDRASIL | Ontology | Entity identification; historical pattern matching | Sync query | Trajectory built without entity context |
| MOIRAI | Provenance | Signed trajectory events | Async event | Events buffered |
| PCES/AEGIS | Classification | Session scope | Sync | Cached session scope |

### 5.2 Feeds Into

| Service | Epithet | What TRS provides | How |
|---|---|---|---|
| UCS/TYCHE | Uncertainty | temporal_horizon_dominance component | API |
| PCS/IRIS | Communication | Scenario space for consumer package briefing summary | API |
| SENTINEL | Strategic Warning | Trajectory signals for warning synthesis | `TRS_SCENARIO_PROBABILITY_REVISED` event |
| ATHENA | Interface | Trajectory and scenario visualisation | API |

---

## 6. Non-Functional Requirements

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Trajectory model creation | 500ms | 2000ms | 3000ms |
| Scenario retrieval (cached) | 20ms | 60ms | 200ms |
| Probability revision | 100ms | 300ms | 500ms |

---

## 7. Known Design Risks

- **The trajectory models are only as good as the time-series data in the corpus.** Many analytical subjects do not have the longitudinal corpus coverage needed to fit a meaningful trajectory. A model fit to three data points is statistically weak regardless of what algorithm is used. TRS must explicitly label model data quality alongside the model outputs.
- **Scenario probabilities are not frequentist probabilities.** They are analyst-informed estimates calibrated against historical programme patterns. This distinction is analytically important — a "40% probability" in a TRS scenario means something different from a statistical frequency estimate and must be clearly communicated in GC-8 forecast products.

---

## 8. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-8 | Forecast product governance — disclosure requirements, dissemination authority, temporal scope limits, and probability expression format for TRS-derived analytical products | Phase 2 forecast product generation |

---

## 9. Implementation Roadmap

### Phase 1 — Trajectory Modelling and Scenario Space (Year 3, Weeks 1–12)

- TrajectoryModel, Scenario, LeadingIndicator schemas
- Trajectory model fitting from corpus time-series data
- Scenario space generation (3–5 scenarios with initial probabilities)
- Leading indicator status tracking
- UCS/TYCHE temporal_horizon_dominance integration
- MOIRAI event emission

**Phase gate criterion:** Trajectory models produce scenario spaces for test entities with known trajectories. Temporal_horizon_dominance fed to UCS/TYCHE correctly. Leading indicator status updates revise scenario probabilities.

### Phase 2 — GC-8 Compliance, PCS/IRIS Integration, and SENTINEL Feed (Year 3, Weeks 13–24)

- GC-8 compliant forecast product generation
- PCS/IRIS briefing summary integration
- SENTINEL feed for strategic warning synthesis
- Historical pattern matching against MIRROR requirement profiles

**Phase gate criterion:** GC-8 forecast product format validated. SENTINEL receives trajectory signals. ARB and Cell Lead sign-off.

---

## 10. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD — Addendum F specification implemented |

## Appendix D: Red Team Findings
*Pending — Year 3 gate review.*
