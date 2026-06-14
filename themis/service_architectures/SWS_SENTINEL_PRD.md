# SWS — Strategic Warning Synthesis Service
### SENTINEL · *"The guard who maintains continuous watch and synthesises what all others report — the final station where dispersed warning signals become a coherent alarm"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `SWS` |
| **Epithet** | `SENTINEL` |
| **Full name** | Strategic Warning Synthesis Service |
| **Namespace** | `themis-warning` |
| **Layer** | Intelligence Layer — Warning |
| **Build phase** | Year 3–4 |
| **Build priority** | 14 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — synthesises all warning signals into coherent strategic warning products |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**SWS/SENTINEL answers: Taking all active signals from across the warning layer — TRS trajectories, CASSANDRA anomalies, LACHESIS weak signal fusions, CRF cross-requirement contradictions — is there a coherent strategic warning message emerging that warrants escalation to senior analytical leadership?**

### 1.2 Why This Service Exists

The warning layer produces individual signals: CHRONOS produces trajectory projections, CASSANDRA produces anomaly records, LACHESIS produces weak signal fusions, JANUS produces cross-requirement contradictions. Each of these is a partial view. No individual signal constitutes a strategic warning.

SENTINEL is the synthesis layer. It monitors all incoming warning signals simultaneously, assesses whether they constitute a coherent pattern pointing toward a significant intelligence development, and when the aggregate evidence crosses a threshold, produces a structured strategic warning for escalation to senior analytical leadership and the decision-making chain.

The key design insight is that strategic warning is not the loudest individual signal — it is the coherent pattern across multiple independent signal types. A trajectory model suggesting programme advancement plus anomalous facility activity plus unusual communications absence plus a LACHESIS fusion matching a known pre-test precursor pattern is more than the sum of its parts. SENTINEL exists to see that sum.

### 1.3 Design Principles

- **No warning without minimum multi-source corroboration.** A strategic warning from SENTINEL requires signals from at least two distinct warning layer services. A single service — however confident — is not strategic warning; it is an analytical assessment that should route through normal ATHENA channels.
- **Warning confidence is explicitly graded.** SENTINEL warnings carry a confidence level that reflects both the strength of individual signals and their mutual corroboration. LOW confidence warnings inform; HIGH confidence warnings escalate.
- **Warning lifecycle is managed, not disposable.** A strategic warning is created, acknowledged, monitored, and closed — not issued and forgotten. SENTINEL tracks what happened to every warning: whether it was acknowledged, whether it was confirmed, whether it was a false positive.
- **False positives are not failures.** A SENTINEL warning that is later determined to be a false positive has still served its function — it prompted analytical attention at a moment when the evidence warranted it. The design problem is not eliminating false positives but ensuring the false positive rate does not produce alert fatigue that causes real warnings to be dismissed.
- **SENTINEL is last because it requires all warning services.** Year 3-4 deployment reflects the genuine dependency: SENTINEL can only synthesise signals from services that exist. Building SENTINEL before CASSANDRA, LACHESIS, and JANUS are operational would produce a synthesis layer with nothing to synthesise.

### 1.4 Explicit Out of Scope

- **Producing finished intelligence.** SENTINEL produces structured warning advisories that prompt senior analytical leadership to commission finished intelligence. It does not replace finished analytical products.
- **Disseminating warnings externally.** SENTINEL produces warnings for internal analytical leadership escalation. PCS/IRIS handles external dissemination. SENTINEL feeds PCS/IRIS for high-priority warning products.
- **Tasking collection autonomously.** SENTINEL warnings may identify collection gaps that ARGUS-LACUNA surfaces to TIS/DIKE. SENTINEL does not task collection directly.

---

## 2. Core Responsibilities

### 2.1 Primary Function

SWS/SENTINEL monitors incoming signals from TRS/CHRONOS, ADS/CASSANDRA, WSF/LACHESIS, and CRF/JANUS — continuously computing a warning score per domain and entity based on signal type, strength, recency, and mutual corroboration — and produces structured StrategicWarning records when the aggregate score crosses configurable thresholds, routing these for senior analytical leadership acknowledgment and lifecycle tracking.

### 2.2 Secondary Functions

- Signal ingestion and normalisation: receiving signals from all warning layer services and normalising them into a common warning signal schema
- Warning score computation: maintaining a continuously updated warning score per domain/entity from active signals
- Warning deduplication: suppressing duplicate warnings when the same event is signalled by multiple services
- Warning lifecycle management: tracking from GENERATED through ACKNOWLEDGED through CONFIRMED or CLOSED
- False positive tracking: recording warning outcomes for calibration
- PCS/IRIS escalation: when warnings reach HIGH confidence, triggering PCS/IRIS consumer package generation for senior leadership dissemination

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
WarningSignal:
  signal_id:               uuid
  source_service:          CHRONOS | CASSANDRA | LACHESIS | JANUS
  source_ref_id:           uuid
  domain:                  str
  entity_ids:              [uuid]
  signal_type:             str
  strength:                float
  corroboration_score:     float            # multi-source agreement
  received_at:             datetime
  expires_at:              datetime | null

WarningScore:
  score_id:                uuid
  domain:                  str
  entity_id:               uuid | null
  current_score:           float            # 0.0–1.0 rolling aggregate
  contributing_signals:    int
  source_diversity:        int              # number of distinct warning services contributing
  score_trend:             RISING | STABLE | FALLING
  last_updated:            datetime

StrategicWarning:
  warning_id:              uuid
  domain:                  str
  subject_entity_ids:      [uuid]
  warning_type:            str              # what the warning is about
  confidence:              HIGH | MEDIUM | LOW
  contributing_signals:    {
    chronos_trajectories:  [uuid],
    cassandra_anomalies:   [uuid],
    lachesis_fusions:      [uuid],
    janus_contradictions:  [uuid]
  }
  warning_narrative:       str             # synthesised plain-language warning
  recommended_actions:     [str]
  collection_gaps:         [uuid]          # FK → ARGUS-LACUNA gaps directly relevant
  dissemination_priority:  IMMEDIATE | URGENT | ROUTINE
  generated_at:            datetime
  status:                  GENERATED | ACKNOWLEDGED | ACTIONED | CONFIRMED | CLOSED | FALSE_POSITIVE
  acknowledged_by:         str | null
  acknowledged_at:         datetime | null
  outcome:                 str | null       # eventual classification once resolved
  pcs_iris_triggered:      bool
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Warning store | PostgreSQL | StrategicWarning, WarningScore, WarningSignal | Indefinite |
| Score cache | Redis | Rolling WarningScore per domain/entity | 1h TTL |
| Event store | MOIRAI | Signed warning events | Indefinite |

---

## 4. API Contract

### 4.1 Endpoints

```
GET /warnings/active
  Auth:     supervisor token | senior analyst token | IOB token
  Params:   confidence: str, domain: str, priority: str
  Response: [StrategicWarning]
  SLA: p99 < 300ms

GET /warnings/{warning_id}
  Auth:     supervisor token | senior analyst | IOB token
  Response: StrategicWarning with full signal detail

POST /warnings/{warning_id}/acknowledge
  Auth:     senior analyst token | supervisor token
  Request:  { acknowledged_by: str, action_notes: str }
  Response: { warning_id: uuid, status: ACKNOWLEDGED }

POST /warnings/{warning_id}/close
  Auth:     supervisor token | IOB token
  Request:  { outcome: CONFIRMED | FALSE_POSITIVE | SUPERSEDED, resolution: str }
  Response: { warning_id: uuid, status: CLOSED }

GET /scores/{domain}
  Auth:     analyst session token | supervisor token
  Response: [WarningScore]               # current scores for this domain

GET /health
  Response: {
    status, dependencies: { moirai, chronos, cassandra, lachesis, janus, pces, redis },
    active_warnings:       int,
    high_confidence:       int,
    mean_score_by_domain:  [{ domain, score }],
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          SWS_WARNING_GENERATED
service_id:         "SWS"
session_id:         null
classification:     str
event_payload:
  warning_id:             uuid
  domain:                 str
  confidence:             str
  priority:               str
  source_diversity:       int
  contributing_signal_count:int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          SWS_WARNING_ACKNOWLEDGED
event_payload:
  warning_id:             uuid
  acknowledged_by:        str

EventType:          SWS_WARNING_CLOSED
event_payload:
  warning_id:             uuid
  outcome:                str
  resolution:             str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `SWS_WARNING_GENERATED` | Warning threshold crossed | MOIRAI, senior analyst notification, PCS/IRIS if IMMEDIATE priority |
| `SWS_WARNING_ACKNOWLEDGED` | Warning acknowledged | MOIRAI |
| `SWS_WARNING_CLOSED` | Warning resolved | MOIRAI, false positive tracking |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| TRS/CHRONOS | `TRS_SCENARIO_PROBABILITY_REVISED` | Updates WarningScore for affected domain/entity |
| ADS/CASSANDRA | `ADS_ANOMALY_DETECTED` | Ingests as WarningSignal; updates score |
| WSF/LACHESIS | `WSF_FUSION_ESTABLISHED` | Ingests fusion as WarningSignal; updates score |
| CRF/JANUS | `CRF_CONNECTION_DETECTED` | CONTRADICTION type triggers score update |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| TRS/CHRONOS | Temporal | Trajectory signals | Async event | Temporal signals not included; warning score incomplete |
| ADS/CASSANDRA | Anomaly | Anomaly signals | Async event | Anomaly signals not included |
| WSF/LACHESIS | Weak Signal | Fusion signals | Async event | Fusion signals not included |
| CRF/JANUS | Cross-Req | Contradiction signals | Async event | Cross-req signals not included |
| MOIRAI | Provenance | Signed warning events | Async event | Events buffered |

### 5.2 Feeds Into

| Service | Epithet | What SENTINEL provides | How |
|---|---|---|---|
| PCS/IRIS | Communication | High-confidence warnings for consumer package | `SWS_WARNING_GENERATED` event |
| Senior analysts | Management | Warning advisories for leadership | Notification |
| IOB Reporting | Oversight | Warning lifecycle and outcome records | API |

---

## 6. Known Design Risks

- **Alert fatigue from over-sensitive thresholds.** If SENTINEL's warning thresholds are set too low, it produces warnings that are frequently false positives. Senior analysts who receive many false positives will begin ignoring warnings — the Cassandra problem at the warning synthesis layer. Resolution path: conservative initial thresholds requiring HIGH source_diversity (signals from at least 3 distinct services); threshold adjustment only after 6 months of operational data.
- **SENTINEL's effectiveness is bounded by its input services.** If CASSANDRA has poor baselines (Year 3 entry-level quality), LACHESIS has few validated patterns, and JANUS has limited cross-requirement data, SENTINEL will produce low-confidence warnings that add little value over existing manual analytical processes. SENTINEL's value compounds as the warning layer matures — the Year 4 version will be substantially more powerful than the Year 3 version.

---

## 7. Implementation Roadmap

### Phase 1 — Signal Ingestion and Warning Score (Year 3, Weeks 13–24)

- WarningSignal, WarningScore, StrategicWarning schemas
- Signal ingestion from all warning layer services
- Rolling warning score computation per domain/entity
- Warning generation when score crosses threshold
- Senior analyst notification
- MOIRAI events

**Phase gate criterion:** Warning score correctly aggregates signals from at least 3 warning services. Warning generated correctly when test threshold crossed. Senior analyst notification delivered.

### Phase 2 — Lifecycle Management, PCS/IRIS Integration, and False Positive Tracking (Year 4, Weeks 1–12)

- Warning acknowledgment and closure workflow
- False positive tracking and outcome recording
- PCS/IRIS escalation for IMMEDIATE priority warnings
- IOB warning lifecycle reporting
- Threshold calibration from operational data

**Phase gate criterion:** Warning lifecycle tracks from GENERATED through ACKNOWLEDGED through CLOSED correctly. False positive rate tracked. PCS/IRIS receives escalation for IMMEDIATE warnings. ARB and Cell Lead sign-off.

---

## 8. Policy Dependencies

No GC items gate SENTINEL. The dissemination authority for IMMEDIATE priority strategic warnings is an operational policy owned by senior analytical leadership — SENTINEL notifies; leadership decides dissemination.

---

## 9. Training and Analyst Guidance

### 9.1 What Analysts See

SENTINEL warnings appear in a dedicated warning panel visible to senior analysts and supervisors. Each warning shows the domain, confidence level, priority, and a one-paragraph narrative. Expanding shows the full signal detail including which warning services contributed and the specific signals from each.

### 9.2 What Analysts Should Do

IMMEDIATE priority warnings should be acknowledged within 4 hours. Acknowledgment does not confirm the warning — it confirms receipt and initiates analytical review. After review, the warning is either actioned (analytical work commissioned or in progress) or closed with a resolution (CONFIRMED, FALSE_POSITIVE, or SUPERSEDED). All HIGH confidence warnings that are not actioned must be closed with documented rationale.

---

## 10. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 3 gate review.*
