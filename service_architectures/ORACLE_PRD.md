# ORACLE — Outcome Intelligence Service
### ORACLE · *"Greek for the voice of divine foreknowledge — not prophecy but probability; what the patterns of the past reveal about the shape of the future"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `ORACLE` |
| **Epithet** | `ORACLE` |
| **Full name** | Outcome Intelligence Service |
| **Namespace** | `themis-research` |
| **Layer** | Intelligence Layer — Research |
| **Build phase** | Year 3 |
| **Build priority** | 7 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust + Currency — learns from historical outcomes to calibrate future assessment accuracy |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**ORACLE answers: Given what we know about this analytical requirement — its domain, claim types, collection coverage, and analytical approach — what does the historical outcome record predict about where this assessment is likely to be accurate, where it is likely to be wrong, and what would most improve its reliability?**

### 1.2 Why This Service Exists

TCS/MIMIR calibrates analyst reliance on AI in a domain by tracking whether the analyst's verification behaviour is accurate. This is a within-session and within-analyst calibration. It does not address a different question: independently of how carefully this analyst verified the AI's claims, are assessments of this type historically accurate?

These are different questions. An analyst who verifies claims carefully using imperfect methods can be well-calibrated but systematically wrong about a specific claim type. The analyst's calibration reflects their verification quality; ORACLE's calibration reflects the historical accuracy of assessments of this type, regardless of who produced them.

ORACLE builds predictive models from OFS/NEMESIS outcome data: across all requirements with classified outcomes, what conditions predict CONFIRMED versus DISCONFIRMED? What claim types in what domains with what collection coverage patterns reliably produce accurate assessments? What analytical approaches have historically been associated with confirmed outcomes for requirements of this type?

These predictions inform STOA's sub-question routing, TCS/MIMIR's confidence signals, and directly answer the question that MIRROR's historical analogies can only approximate: not "what did we do on similar requirements" but "what should we do, given the historical accuracy patterns?"

### 1.3 The Data Floor Is Non-Negotiable

ORACLE requires 200 requirements with OFS/NEMESIS-classified outcomes before any of its models produce meaningful predictions. At a realistic estimate of 40 analytical requirements per year processed through THEMIS with classifiable outcomes (20% observability of the 200 requirements per year), this is a 25-year data floor at current scale — unless the IC community accepts AMBIGUOUS outcomes as partial signals and broadens what constitutes a classifiable outcome.

Year 3 deployment is the minimum realistic target given Year 1 platform deployment and Year 2 outcome accumulation. ORACLE's initial models will be underpowered by any standard statistical measure. This is acknowledged explicitly in every ORACLE output: the prediction is accompanied by the evidence count and a clear statement of model maturity.

The honest framing is that ORACLE is a Year 3+ service that improves continuously as outcome data accumulates. It does not suddenly become useful at 200 requirements — it becomes incrementally more useful as each new outcome is added. But it becomes misleadingly confident below a minimum threshold.

### 1.4 Design Principles

- **Every prediction includes its evidence base.** An ORACLE prediction accompanied by "based on 12 similar requirements with outcome data" is a different object than one accompanied by "based on 340 similar requirements." Both are surfaced; the analyst decides how much weight to give them.
- **Predictions are probabilistic, not determinative.** ORACLE produces accuracy probability estimates, risk factor identification, and approach recommendations. It does not produce guaranteed outcomes.
- **ORACLE learns from its own predictions.** When an ORACLE-informed STOA session produces an OFS/NEMESIS outcome, that outcome feeds back into ORACLE's training data. The model self-improves.
- **Systematic bias in the outcome record propagates to ORACLE.** If OFS/NEMESIS consistently classifies similar outcomes differently (due to D&D, measurement difficulties, or classifier inconsistency), ORACLE will learn that inconsistency as signal. The IOB knowledge quality review that applies to MNEMOSYNE applies equally here.

### 1.5 Explicit Out of Scope

- **Making analytical conclusions.** ORACLE predicts accuracy likelihood for assessment types. The assessment itself is the analyst's responsibility.
- **Replacing TCS/MIMIR.** ORACLE provides assessment-type accuracy signals; TCS/MIMIR provides analyst-reliance calibration. Both are needed; neither replaces the other.
- **Predicting adversary behaviour.** ORACLE predicts assessment accuracy, not world events.

---

## 2. Core Responsibilities

### 2.1 Primary Function

ORACLE builds and maintains predictive models from OFS/NEMESIS outcome data — learning what conditions predict assessment accuracy versus inaccuracy for each combination of domain, claim type, collection coverage pattern, and analytical approach — and surfaces these predictions to STOA for sub-question routing, to TCS/MIMIR for analytical-type confidence adjustment, and to TIS/DIKE for collection effectiveness guidance.

### 2.2 Secondary Functions

- Risk factor identification: identifying the specific conditions associated with historical DISCONFIRMED outcomes (the analytical risks for this type of assessment)
- Approach recommendation: identifying which analytical approaches and collection methods are associated with historically confirmed outcomes
- Accuracy trend monitoring: tracking whether assessment accuracy in a domain is improving or degrading over time
- Model maturity reporting: explicitly reporting evidence count and model confidence for every prediction
- Prediction audit: recording every prediction and the eventual outcome, enabling ORACLE to track its own accuracy as a predictor

### 2.3 What This Service Does Not Decide

ORACLE produces accuracy probability estimates and risk factors. Whether those probabilities are meaningful given the evidence base, whether a specific risk factor applies to the current requirement, and how aggressively to revise an analytical approach in response to ORACLE's recommendations are analyst decisions. ORACLE calibrates; humans decide.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
OutcomePattern:
  pattern_id:              uuid
  domain:                  str
  claim_type:              str
  collection_coverage:     str              # HIGH | MEDIUM | LOW | SPARSE
  analytical_approach:     str | null       # SKS skill type or STOA decomposition type
  sample_size:             int
  confirmed_rate:          float
  disconfirmed_rate:       float
  ambiguous_rate:          float
  risk_factors:            [str]            # conditions associated with DISCONFIRMED outcomes
  approach_recommendations:[str]            # approaches associated with CONFIRMED outcomes
  model_version:           str
  trained_at:              datetime
  freshness:               CURRENT | AGING | STALE

AccuracyPrediction:
  prediction_id:           uuid
  session_id:              uuid
  research_id:             uuid | null      # FK → STOA ResearchSession
  domain:                  str
  claim_types:             [str]
  collection_coverage:     str
  predicted_accuracy:      float            # 0.0–1.0 probability of CONFIRMED outcome
  evidence_count:          int
  model_maturity:          SUFFICIENT | MARGINAL | INSUFFICIENT
  risk_factors:            [str]
  approach_recommendations:[str]
  created_at:              datetime
  eventual_outcome:        str | null       # populated when OFS/NEMESIS classifies

ModelVersion:
  model_id:                uuid
  domain:                  str
  version_string:          str
  training_outcome_count:  int
  accuracy_on_holdout:     float | null     # ORACLE's own prediction accuracy
  trained_at:              datetime
  active:                  bool
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Pattern store | PostgreSQL | OutcomePattern, AccuracyPrediction, ModelVersion | Indefinite |
| Model store | Object storage | Trained ML model artifacts | Per version |
| Event store | MOIRAI | Signed prediction and training events | Indefinite |
| Prediction cache | Redis | Recent predictions for active sessions | 24h TTL |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /predictions/generate
  Auth:     STOA service account | TCS service account | analyst session token
  Request:  {
    session_id:            uuid,
    research_id:           uuid | null,
    domain:                str,
    claim_types:           [str],
    collection_coverage:   str
  }
  Response: {
    prediction_id:         uuid,
    predicted_accuracy:    float,
    evidence_count:        int,
    model_maturity:        str,
    risk_factors:          [str],
    approach_recommendations:[str]
  }
  SLA: p99 < 500ms

GET /patterns/{domain}
  Auth:     analyst session token | STOA service account
  Response: [OutcomePattern]             # for this domain

GET /patterns/{domain}/{claim_type}
  Auth:     analyst session token
  Response: OutcomePattern with full risk factor and recommendation detail

GET /model-maturity
  Auth:     platform operator | IOB token
  Response: {
    by_domain: [{ domain, outcome_count, model_maturity, last_trained_at }]
  }

GET /predictions/{prediction_id}/outcome
  Auth:     OFS/NEMESIS service account | IOB token
  Response: AccuracyPrediction (with eventual_outcome if populated)

GET /health
  Response: {
    status, dependencies: { moirai, ofs_nemesis, mirror },
    total_outcome_patterns:int,
    domains_with_sufficient_data:int,
    predictions_24h:       int,
    last_training_run:     datetime,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          ORACLE_PREDICTION_GENERATED
service_id:         "ORACLE"
session_id:         uuid
classification:     CONTROLLED_UNCLASSIFIED
event_payload:
  prediction_id:          uuid
  domain:                 str
  predicted_accuracy:     float
  evidence_count:         int
  model_maturity:         str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          ORACLE_MODEL_TRAINED
event_payload:
  domain:                 str
  training_outcome_count: int
  model_version:          str
  accuracy_on_holdout:    float | null
```

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| OFS/NEMESIS | `OFS_OUTCOME_CLASSIFIED` | Adds to training data; triggers model retraining if threshold met |
| STOA | `STOA_RESEARCH_COMPLETE` | Research context used for prediction linkage |
| MIRROR | Profile data (via API) | MIRROR profiles augment ORACLE training data |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| OFS/NEMESIS | Outcome Feedback | Primary training data source | Async event + batch read | No new predictions until OFS data available |
| MIRROR | Requirement Similarity | Profile data to augment training | API | Training proceeds with reduced feature set |
| MOIRAI | Provenance | Signed prediction and training events | Async event | Events buffered |

### 5.2 Feeds Into

| Service | Epithet | What ORACLE provides | How |
|---|---|---|---|
| STOA | Research Orchestration | Sub-question accuracy predictions and approach recommendations | API |
| TCS/MIMIR | Trust Calibration | Assessment-type accuracy adjustment (Year 3+ capability) | API |
| TIS/DIKE | Tasking Integration | Collection effectiveness predictions | API |
| IOB Reporting | Oversight | Model maturity status; prediction accuracy tracking | API |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Prediction generation (cached model) | 50ms | 200ms | 500ms |
| Model training (scheduled, async) | 10 min | 30 min | 60 min |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% |
| RTO | 30 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| STOA / TCS | Prediction generation | Service account |
| Analyst session | Pattern lookup by domain | Session token |
| OFS/NEMESIS | Outcome linkage to predictions | Service account |
| IOB | Full access including model maturity | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Model trained on biased outcome data | Medium | P1 — ORACLE reinforces systematic errors | IOB model quality review | model_maturity = INSUFFICIENT below threshold; explicit bias flag in IOB reporting |
| Data floor not reached for years | High | P2 — ORACLE useless initially | Domain maturity monitoring | model_maturity explicitly surfaced; ORACLE predictions labelled INSUFFICIENT until threshold |

### 8.1 Known Design Risks

- **The outcome observability problem may keep ORACLE perpetually data-starved.** This is the honest assessment. Most IC intelligence assessments do not receive clean CONFIRMED/DISCONFIRMED signals within any predictable timeframe. The 200-requirement threshold at 20% observability requires 1,000 analysed requirements — which may be 5-10 years at realistic throughput. Resolution path: Research & Red Team must investigate whether AMBIGUOUS outcomes can be partially used as training data, and whether outcomes from adjacent programmes or domains can inform models for data-scarce areas.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `oracle.model.maturity_insufficient_domains` | Gauge | Any production domain | P2 |
| `oracle.prediction.evidence_count_mean` | Gauge | `< 20` across domains | P2 |
| `oracle.moirai.emit.failure_rate` | Counter | `> 0.1%` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/oracle/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every prediction generated and every model training run is permanently recorded. An oversight body can verify that predictions were based on the evidence count and model version recorded, and can track ORACLE's own prediction accuracy over time.

---

## 11. Implementation Roadmap

### Phase 1 — Training Pipeline and Pattern Store (Year 3, Weeks 1–12)

- OutcomePattern schema from OFS/NEMESIS data
- Basic accuracy prediction from outcome patterns (statistical, not ML initially)
- Model maturity tracking and INSUFFICIENT labelling
- Prediction endpoint with evidence count and maturity
- STOA and TCS integration

**Phase gate criterion:** Predictions generated with explicit evidence count and maturity labels. Domains with < 20 outcomes labelled INSUFFICIENT. STOA receives predictions.

### Phase 2 — ML Model, Self-Improvement, and IOB Reporting (Year 3, Weeks 13–24)

- ML model training pipeline (replacing statistical model when data sufficient)
- ORACLE self-accuracy tracking (prediction vs. eventual outcome)
- Model versioning and retraining on new outcome data
- IOB model maturity reporting

**Phase gate criterion:** ML model produces better accuracy than statistical baseline on holdout set. Self-accuracy tracking shows ORACLE's prediction accuracy rate. ARB sign-off.

---

## 12. Policy Dependencies

No GC items gate ORACLE. The minimum evidence threshold and model maturity classifications are Research & Red Team operational decisions.

---

## 13. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD — honest Year 3 data floor acknowledged |

## Appendix D: Red Team Findings
*Pending — Year 3 gate review.*
