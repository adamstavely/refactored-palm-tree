# TCS — Trust Calibration Service
### MIMIR · *"Norse god of wisdom"*
*Part of the THEMIS Platform · Core Infrastructure · Build Priority: 4*

---

## Design Philosophy

The Trust Calibration Service measures whether analysts are relying on AI appropriately relative to its actual performance. An AI that is wrong 15% of the time and an AI where analysts catch 15% of its errors are very different systems — but both look identical without calibration measurement.

TCS does not measure whether analysts *like* AI outputs. It measures whether their trust decisions correspond to AI accuracy. An analyst who always accepts AI outputs is not necessarily well-calibrated — they are well-calibrated only if the AI is also consistently accurate. An analyst who always overrides AI is not under-relying if the AI is genuinely unreliable for their interaction class.

---

## Core Measurement Model

### Reliance-Accuracy Index (RAI)

```
RAI = (corrections_made / AI_outputs_reviewed)
      normalized against AI system Elo rating

RAI = 0.0  → analyst never relies on AI (always overrides)
RAI = 1.0  → analyst always relies on AI (never corrects)
RAI = 0.5  → balanced reliance

Calibration Error = 1 - |RAI - Elo_normalized|
  → 1.0 = perfectly calibrated
  → 0.0 = maximally miscalibrated
```

### Elo-Based AI Performance Rating

AI system Elo ratings are established per interaction class × matter type. The Elo system treats each AI output as a "match" — a correction by an analyst is a "loss," acceptance (without subsequent revision) is a "win."

```yaml
EloRecord:
  system_id:          uuid
  interaction_class:  str
  matter_type:        str
  current_rating:     float      # initialized at 1200; updates on each correction event
  rating_history:     [(ISO8601, rating)]
  total_interactions: int
  correction_rate:    float
  last_updated:       ISO8601
```

---

## Calibration Score Computation

```yaml
AnalystCalibrationRecord:
  analyst_id:          uuid
  interaction_class:   str
  matter_type:         str
  rai_score:           float
  calibration_error:   float     # 1 - |RAI - Elo_normalized|
  calibration_score:   float     # composite per analyst/class/matter-type
  trend:               [(ISO8601, score)]   # 5-sprint rolling window
  status:              calibrated | over_relying | under_relying
  flag_threshold_over: 0.80      # RAI > this → over-reliance flag
  flag_threshold_under: 0.20     # RAI < this → under-reliance flag
  intervention_routed: bool
  last_updated:        ISO8601
```

---

## Event Subscriptions

```yaml
# From Provenance Service
TurnCompleted:
  → Extract interaction_class (from PGS classification on turn record)
  → Extract analyst_id, session_id, matter_type
  → Initialize or update EloRecord for this interaction class

# From FGTS / ALETHEIA (primary calibration signal)
AIPerformanceUpdate:
  turn_id:            uuid
  interaction_class:  str
  matter_type:        str
  outcome:            corrected | accepted
  error_category:     factual_error | outdated | irrelevant | citation_error | style | other
  severity:           float        # edit_distance from FGTS
  analyst_id:         uuid
  analyst_weight:     float        # from FGTS AnalystSignalRecord

→ Update AI system Elo rating for interaction_class × matter_type
→ Update analyst RAI for this interaction
→ Recompute calibration_error and calibration_score

# From ERAS / LOGOS (additional calibration signal)
UnsupportedClaimAccepted:
  turn_id:            uuid
  analyst_id:         uuid
  unsupported_claim_count: int

→ Weight this as additional over-reliance signal
```

---

## Calibration Intervention Routing

| Status | Calibration Score | Intervention |
|---|---|---|
| `calibrated` | > 0.75 | No intervention; positive reinforcement if trending up |
| `over_relying` | 0.50–0.75 | Soft nudge: guidance surfaced in analyst tool; flagged to knowledge management |
| `over_relying` | < 0.50 | Hard flag: escalated to supervising attorney; scheduled review |
| `under_relying` | 0.50–0.75 | Soft nudge: AI quality context surfaced to analyst |
| `under_relying` | < 0.50 | Hard flag: may indicate systematic distrust of AI; practice group review |

Intervention routing integrates with HITL — severe calibration flags can trigger a supervised review session before the analyst's next AI interaction on a high-risk interaction class.

---

## Analyst Dashboard

Surfaces in the Intellect platform (not as a standalone THEMIS app):

- Current calibration score per interaction class with 5-sprint trend sparkline
- Over-relying / Under-relying / Calibrated status badge
- Comparison to cohort average for same interaction class and matter type
- Recent corrections that drove score changes
- Intervention history

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | TurnCompleted events initialize calibration tracking; calibration scores attached to turn records |
| PGS / NOMOS | Interaction class from PGS feeds TCS scoring; high-risk interaction classes receive elevated calibration scrutiny |
| FGTS / ALETHEIA | Primary calibration signal source: AIPerformanceUpdate events drive Elo and RAI updates |
| ERAS / LOGOS | Unsupported claim acceptance rates feed as additional over-reliance signal |
| HADES | Surface 5 (calibration boundary) probes test TCS scoring model under adversarial conditions |
| INTELLECT | Calibration dashboard; analyst trend visualization; cohort comparison |

---

## Implementation Roadmap

### Phase 1 — RAI Scoring & Elo Baseline (Weeks 19–20)
- TCS service deployed; RAI formula implemented
- AI system Elo rating: baseline from first 500 governed interactions per interaction class
- Calibration Error Score computation
- Analyst calibration dashboard in Intellect
- Over-reliance and under-reliance flag thresholds configured per practice group
- TurnCompleted event subscription from Provenance Service

### Phase 2 — FGTS Integration & Interventions (after FGTS Sprint 13)
- AIPerformanceUpdate event feed from FGTS
- Elo rating updates from real firm-specific corrections
- Calibration intervention routing: knowledge management review escalation
- ERAS unsupported claim rate as additional calibration signal
- Cohort-level analysis: systematic over/under-reliance detection

---

**Depends on:** PROV / MOIRAI, PGS / NOMOS
**Feeds into:** FGTS / ALETHEIA (receives AIPerformanceUpdates), ERAS / LOGOS (reasoning quality baseline), INTELLECT
