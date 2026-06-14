# ADS — Anomaly Detection Service
### CASSANDRA · *"Greek prophetess cursed to speak truth that would never be believed — the one who saw what others missed and could not make them act on it; the warning that arrives too early to be credited"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `ADS` |
| **Epithet** | `CASSANDRA` |
| **Full name** | Anomaly Detection Service |
| **Namespace** | `themis-warning` |
| **Layer** | Intelligence Layer — Warning |
| **Build phase** | Year 3 |
| **Build priority** | 11 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — detects when entity behaviour deviates from baseline in analytically significant ways |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**ADS/CASSANDRA answers: Is this entity behaving differently from its established pattern — and if so, is the deviation analytically significant enough to warrant urgent analytical attention?**

### 1.2 Why This Service Exists

Significant intelligence developments often announce themselves through subtle behavioural changes before they become unmistakably obvious. A programme that is approaching a critical test changes its communications patterns, its facility activity, its personnel movements — days or weeks before the test occurs. By the time the test is confirmed, the analytical community has already missed the warning window.

Without CASSANDRA, the analytical community relies on the analyst's memory of prior entity behaviour to notice deviation. With CASSANDRA, behavioural baselines are maintained systematically, deviations are detected automatically across multiple collection types, and significant anomalies are surfaced to analysts before they have compiled enough evidence to notice them manually.

The name Cassandra is not ironic — it is honest. Anomaly detection systems are frequently right about anomalies and ignored because the detection arrives before conventional analytical process has confirmed the underlying cause. The design problem is not technical (detecting anomalies is solvable) but institutional (ensuring detected anomalies are acted upon). CASSANDRA produces structured anomaly records that are actionable, not just alarms.

### 1.3 Design Principles

- **Baselines require operational history.** A meaningful behavioural baseline requires sufficient observational history. CASSANDRA's Year 3 deployment is deliberately timed to allow baseline accumulation from the first two years of corpus ingestion. Anomaly detection against an immature baseline produces false positives at rates that create alert fatigue and destroy the service's analytical credibility.
- **Multi-source corroboration elevates anomaly significance.** A single-source behavioural deviation may be collection noise. The same deviation corroborated across multiple collection types is analytically significant. CASSANDRA weights multi-source anomalies substantially higher than single-source detections.
- **The absence of expected behaviour is a signal.** Unusual absence — an entity that routinely produces collection but goes dark — is as analytically significant as unusual presence. CASSANDRA tracks absence as actively as deviation.
- **Anomaly records are actionable, not just alarms.** Every anomaly record includes a structured description of the deviation, its historical precedent, a recommended analytical response, and linkage to the leading indicators and LACHESIS weak signals that may be related.

---

## 2. Core Responsibilities

### 2.1 Primary Function

ADS/CASSANDRA maintains behavioural baselines for entities in the OGS entity graph, monitors incoming corpus additions for deviations from those baselines, produces structured anomaly records for significant deviations (including multi-source corroboration scores and historical precedent linkage), and publishes anomaly signals to SENTINEL for strategic warning synthesis and to PYTHIA for anticipatory analyst surfacing.

### 2.2 Secondary Functions

- Baseline maintenance: updating baselines as new collection accumulates
- Unusual absence monitoring: detecting when entities that routinely generate collection go dark
- Multi-source corroboration: computing the correlation score when the same anomaly appears across multiple collection types
- Anomaly lifecycle tracking: following anomalies from detection through confirmation or dismissal
- Historical precedent search: matching detected anomaly patterns to historical precedents in the corpus
- LACHESIS coordination: sharing anomaly signals with LACHESIS for weak signal fusion

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
BehaviouralBaseline:
  baseline_id:             uuid
  entity_id:               uuid             # FK → OGS
  domain:                  str
  collection_method:       str
  baseline_type:           FREQUENCY | VOLUME | PATTERN | TEMPORAL | GEOGRAPHIC
  baseline_parameters:     {}              # statistical model parameters
  established_from:        datetime
  established_to:          datetime | null  # null = current
  data_point_count:        int
  confidence:              float
  last_updated:            datetime

AnomalyRecord:
  anomaly_id:              uuid
  entity_id:               uuid
  domain:                  str
  anomaly_type:            BEHAVIOURAL_DEVIATION | UNUSUAL_ABSENCE | PATTERN_BREAK |
                           MULTI_SOURCE_CORROBORATION | SEQUENCE_ANOMALY
  severity:                CRITICAL | HIGH | MEDIUM | LOW
  deviation_score:         float            # standard deviations from baseline
  corroboration_score:     float            # multi-source agreement score; 0.0–1.0
  description:             str
  historical_precedent:    str | null       # what similar anomalies historically preceded
  recommended_response:    str
  evidence_refs:           [str]            # corpus source IDs
  related_weak_signals:    [uuid]           # FK → LACHESIS WeakSignal IDs
  first_detected:          datetime
  last_confirmed:          datetime
  status:                  ACTIVE | CONFIRMED | DISMISSED | EXPLAINED
  dismissed_reason:        str | null
  resolution:              str | null
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Baseline store | PostgreSQL | BehaviouralBaseline records | Indefinite |
| Anomaly store | PostgreSQL | AnomalyRecord | Indefinite |
| Baseline cache | Redis | Active baselines for frequently monitored entities | 4h TTL |
| Event store | MOIRAI | Signed anomaly events | Indefinite |

---

## 4. API Contract

### 4.1 Endpoints

```
GET /anomalies/active
  Auth:     analyst session token | supervisor token
  Params:   domain: str, severity: str, entity_id: uuid
  Response: [AnomalyRecord]
  SLA: p99 < 300ms

GET /anomalies/{anomaly_id}
  Auth:     analyst session token | supervisor token
  Response: AnomalyRecord with full detail

POST /anomalies/{anomaly_id}/status
  Auth:     analyst session token | supervisor token
  Request:  { status: DISMISSED | CONFIRMED | EXPLAINED, reason: str }
  Response: { anomaly_id: uuid, updated: bool }

GET /baselines/{entity_id}
  Auth:     analyst session token | supervisor token
  Response: [BehaviouralBaseline]

GET /health
  Response: {
    status, dependencies: { moirai, ogs, pces, redis },
    active_anomalies:      int,
    critical_anomalies:    int,
    baselines_established: int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          ADS_ANOMALY_DETECTED
service_id:         "ADS"
session_id:         null
classification:     str
event_payload:
  anomaly_id:             uuid
  entity_id:              uuid
  anomaly_type:           str
  severity:               str
  deviation_score:        float
  corroboration_score:    float
prev_event_hash:    str
signature:          str
timestamp:          datetime
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `ADS_ANOMALY_DETECTED` | New anomaly above threshold | MOIRAI, PYTHIA (active session alert), SENTINEL, LACHESIS |
| `ADS_ANOMALY_CONFIRMED` | Analyst confirms anomaly | MOIRAI, SENTINEL (elevated confidence) |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| OGS/YGGDRASIL | Ontology | Entity identification for baseline assignment | Sync query | Domain-level baselines only |
| MOIRAI | Provenance | Corpus additions as baseline data points | Async event stream | Baselines not updated |

### 5.2 Feeds Into

| Service | Epithet | What ADS provides | How |
|---|---|---|---|
| SENTINEL | Strategic Warning | Anomaly signals for warning synthesis | Event |
| PYTHIA | Anticipatory | Anomaly alerts for active sessions | Event |
| LACHESIS | Weak Signal Fusion | Anomaly records as strong signals for fusion | API + Event |

---

## 6. Known Design Risks

- **Baseline quality degrades for infrequently reported entities.** An entity that appears in collection only occasionally does not have a meaningful frequency baseline. Baseline confidence must be surfaced alongside anomaly severity — a CRITICAL anomaly against an immature baseline is less credible than a HIGH anomaly against a mature baseline.
- **False positive rate is the primary operational risk.** A CASSANDRA that produces many false positives will be ignored — the Cassandra problem, operationally. Resolution path: minimum baseline data_point_count before anomaly detection activates (configurable per collection method); conservative initial deviation thresholds with Research & Red Team calibration against known historical anomaly events.

---

## 7. Implementation Roadmap

### Phase 1 — Baseline Establishment and Anomaly Detection (Year 3, Weeks 1–12)

- BehaviouralBaseline and AnomalyRecord schemas
- Baseline establishment from Year 1–2 corpus accumulation
- Frequency and volume baseline types initially
- Multi-source corroboration scoring
- Unusual absence detection
- MOIRAI events; PYTHIA and SENTINEL integration

**Phase gate criterion:** Baselines established for entities with sufficient historical data (> 20 data points per collection method). Anomaly detection produces meaningful deviation scores on test cases with known historical anomalies.

### Phase 2 — Pattern and Sequence Baselines, LACHESIS Integration (Year 3, Weeks 13–24)

- Pattern and temporal baseline types
- LACHESIS weak signal linkage
- Historical precedent search
- Analyst feedback interface (CONFIRMED / DISMISSED / EXPLAINED)

**Phase gate criterion:** Analyst feedback updates anomaly status. LACHESIS receives anomaly signals and correctly links to related weak signals. ARB sign-off.

---

## 8. Policy Dependencies

No GC items gate ADS. Anomaly disclosure to analysts is governed by the same classification constraints as the underlying collection — no additional policy required.

---

## 9. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 3 gate review.*
