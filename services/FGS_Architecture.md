# FGS — Financial Governance Service
### PLUTUS · *"The god of wealth — who counts what AI costs"*
*Part of the THEMIS Platform · Architectural Addition · Build Priority: Phase 1*

---

## Design Philosophy

AI infrastructure at the scale of a law firm incurs significant operational costs. Without matter-level cost tracking, AI infrastructure becomes an undifferentiated overhead line that either gets absorbed into firm economics or allocated arbitrarily. More urgently: a large evidence review — millions of documents, thousands of retrieval queries, dozens of analyst sessions — can generate token volumes that are genuinely shocking if nobody is watching.

The Financial Governance Service answers three questions simultaneously: **Where did this AI spend go?** (attribution), **Is this matter about to run out of AI budget?** (governance), and **Is something anomalous happening?** (detection).

> **Build Placement:** Phase 1. The FGS attribution instrumentation must be live from the first governed AI interaction — retroactive cost reconstruction from logs is unreliable. Every turn that runs without FGS attribution is a turn that cannot be accurately priced.

---

## Three Capabilities

### 1. Cost Attribution

Every TurnCompleted event from the Provenance Service carries `input_token_count` and `output_token_count`. FGS subscribes to these events and applies a versioned rate card.

```yaml
CostRecord:
  turn_id:            uuid
  session_id:         uuid
  matter_id:          uuid
  client_id:          uuid
  analyst_id:         uuid
  model:              str
  input_tokens:       int
  output_tokens:      int
  rate_card_version:  semver
  input_cost:         decimal
  output_cost:        decimal
  total_cost:         decimal
  interaction_class:  str
  timestamp:          ISO8601
```

**Aggregations produced:** cost-per-turn, cost-per-session, cost-per-matter, cost-per-client, cost-per-interaction-class.

**Rate card versioning:** When model pricing changes, historical records retain the rate that applied at time of generation. Rate cards are versioned configuration — never retroactively repriced.

```yaml
RateCard:
  version:            semver
  effective_from:     ISO8601
  models:
    claude-sonnet-4:
      input_per_mtok:  float
      output_per_mtok: float
    [other models...]
```

### 2. Budget Governance

```yaml
MatterBudget:
  matter_id:          uuid
  budget_total:       decimal
  soft_ceiling_pct:   float    # default: 0.80
  hard_ceiling_pct:   float    # default: 1.00
  current_spend:      decimal
  status:             normal | soft_warning | hard_ceiling | authorized_over
  authorizations:     [BudgetAuthorisationEvent]
```

**Budget enforcement flow:**
```
Pre-flight check (< 5ms added to request path):
  FGS.check_budget(matter_id)
  → if current_spend / budget_total > soft_ceiling_pct:
      emit BudgetSoftWarning; proceed with warning
  → if current_spend / budget_total >= hard_ceiling_pct:
      emit BUDGET_EXCEEDED to HITL Hold Queue
      block generation until BudgetAuthorisationEvent received
```

**BudgetAuthorisationEvent:**
```yaml
  event_id:           uuid
  matter_id:          uuid
  authorised_by:      user_id    # supervising attorney
  co_authorised_by:   user_id    # practice group finance lead
  new_ceiling:        decimal
  justification:      str        # required
  timestamp:          ISO8601
```

### 3. Anomaly Detection

Statistical anomaly detection flags patterns that static ceilings cannot catch:

| Pattern | Detection Method | Alert |
|---|---|---|
| Session spending 10× matter average | Z-score against rolling matter session average | Immediate alert to matter supervisor |
| Single query > weekly matter total | Absolute threshold comparison | Immediate alert + generation hold |
| Retrieval loop without convergence | Turn count per session > configurable threshold | Alert + recommended stop |
| Model substitution cost spike | Unexpected model on turn record vs. matter policy | Alert to platform engineering |

---

## Billing Model Support

The FGS stores raw cost data separately from any markup logic so the billing model can change without reprocessing historical records.

| Billing Model | Description | Configuration |
|---|---|---|
| Disbursement | AI cost passed through at actual cost | `billing_model: passthrough` |
| Fee-based | Marked up at a fixed multiplier | `billing_model: markup, multiplier: 1.25` |
| Absorbed | Treated as firm overhead | `billing_model: absorbed` |

Configuration is **per-matter**, not firm-wide. Different matters may bill differently.

---

## Multi-Model Rate Cards

When multiple models are in use (different models for different interaction classes, or fine-tuned models from Phase 6 onward), the rate card is multi-dimensional:

```yaml
rate_key: "{model_version}:{interaction_class}:{matter_type}"
```

This allows differentiated pricing for high-complexity interaction classes without blending costs across matter types.

---

## Dashboards (via Intellect)

- **Matter cost view:** Running total, burn rate, projected end-of-matter cost, budget utilization bar
- **Firm-wide AI spend:** Total daily/weekly/monthly spend, top matters by cost, cost by interaction class
- **Anomaly alert feed:** Active anomaly alerts, resolution status, escalation history
- **Billing export:** Per-matter cost records in ABA UTBMS task code format for billing system integration

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | TurnCompleted events are the primary input; cost records attached to turn records |
| PGS / NOMOS | BUDGET_EXCEEDED condition feeds into HITL Hold Queue via PGS hard stop mechanism |
| INTELLECT | Cost dashboards, anomaly alert feed, billing export |
| Matter Management System | Budget envelopes pulled from matter management at matter intake; billing records exported back |

---

## Implementation Roadmap

### Phase 1 — Cost Attribution (Weeks 21–22, parallel with Phase 3)
- CostRecord schema; TurnCompleted event subscription
- Versioned rate card: initial model pricing
- Cost aggregations: per-turn, per-session, per-matter, per-client
- Matter cost dashboard in Intellect

### Phase 2 — Budget Governance & Anomaly Detection (Weeks 23–24)
- MatterBudget schema; soft and hard ceiling enforcement
- Pre-flight budget check at API gateway (< 5ms)
- BUDGET_EXCEEDED → HITL Hold Queue integration
- BudgetAuthorisationEvent: supervising attorney + finance lead sign-off
- Anomaly detection: session spike, query spike, convergence detection
- Multi-model rate card support

---

**Depends on:** PROV / MOIRAI, PGS / NOMOS (HITL integration)
**Feeds into:** INTELLECT (dashboards), Matter Management System (billing export), PGS / NOMOS (budget hard stop)
