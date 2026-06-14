# FGS — Financial Governance Service
### PLUTUS · *"Greek god of wealth and abundance — the steward who ensures resources flow to where they are needed and that those who draw from the common store are accountable for what they take"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `FGS` |
| **Epithet** | `PLUTUS` |
| **Full name** | Financial Governance Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 1–2 (Weeks 1–8) |
| **Build priority** | Deploys with Phase 1 — cost tracking must begin from session one |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Cross-cutting — governs the economics of inference capacity across the organisation |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**FGS/PLUTUS answers: How much inference capacity has been consumed — by whom, for what analytical purpose, against which budget — and does the tokenomics model ensure that this capacity is allocated, tracked, and attributed in ways that sustain the platform's operational funding model?**

### 1.2 Why This Service Exists

AI inference at the scale of an intelligence analytical community is not free. Every ATHENA session, every STOA research orchestration, every agent task consumes inference tokens that translate directly into API costs or on-premise compute costs. At 200 analysts running multiple sessions per day, this is a significant and variable operational cost that must be governed, attributed, and reported — or it becomes unmanageable.

Without FGS, the organisation cannot answer basic operational questions: How much did the HUMINT analysis team consume last quarter? Which requirement types are the most inference-intensive? Is the consumption growth consistent with the expected analytical workload? Are any analysts or sessions consuming anomalously large amounts relative to their output? The answers to these questions matter for budget forecasting, for operational planning, and for detecting misuse.

FGS is also the implementation layer for **tokenomics** — the economic model that governs how inference capacity is allocated, priced, and charged back across the organisation. Tokenomics is not about cryptocurrency. It is about building a rational, governable economic model for AI inference that prevents both overconsumption (teams consuming unlimited inference without accountability) and underconsumption (teams self-rationing so aggressively that they do not use ATHENA when they should).

### 1.3 Why This Service Is Phase 1

Cost tracking must begin from session one. If FGS is deployed six months after the platform is operational, the first six months of consumption data is irretrievably lost. Budget attribution for Month 1 cannot be reconstructed. The financial baseline that makes all future reporting meaningful does not exist.

FGS is a foundation service in the same sense as PCES and PGS: it governs something that must be governed from the beginning because the cost of not governing it from the beginning compounds with every session.

### 1.4 Tokenomics Design

The tokenomics model has three components:

**Allocation.** Each organisational unit (team, division, or requirement portfolio) receives a periodic token allocation — monthly or quarterly. The allocation reflects the analytical workload the organisation expects that unit to perform, adjusted by the interaction class distribution of their work (STOA multi-step research consumes more tokens than a single ATHENA query).

**Consumption tracking.** Every inference call is logged to FGS with its token count, interaction class, model version, session ID, analyst ID, and requirement context. Consumption is attributed in real time. An analyst approaching their team's allocation can see this in ATHENA before the allocation is exhausted.

**Reserve and escalation.** A platform-level reserve pool handles high-priority analytical work that exceeds team allocations. Reserve access requires supervisor approval and generates an escalation event to FGS. The reserve is finite; the escalation is accountable.

This is not a hard token ceiling that blocks analytical work — it is a governance model that makes consumption visible and attributed. The decision to allow a team to exceed its allocation is a human management decision, not a platform block. FGS surfaces the information; managers decide.

### 1.5 Design Principles

- **Every inference token has an attribution.** No session can be initiated without an FGS account to charge against. Unattributed consumption is a governance failure.
- **Cost data is management information, not analyst information.** Analysts see their own consumption. Supervisors see their team's consumption. Leadership sees the platform total. Individual analyst consumption data does not flow upward without appropriate authority.
- **Tokenomics models are policy artefacts, not technical configurations.** The allocation levels, reserve structure, and escalation thresholds are decisions owned by the organisation's leadership and IOB — not by the platform team. FGS implements whatever tokenomics model the organisation adopts.
- **FGS integrates with SCBS, not the reverse.** SCBS enforces per-session token bounds. FGS tracks organisational-level consumption. When SCBS initialises a session envelope, it queries FGS for the team's remaining allocation, ensuring session-level bounds are consistent with organisational budgets.
- **Cost attribution by interaction class enables informed budget decisions.** A leadership team that knows STOA multi-step research consumes 10× the tokens of a single analytical query can make informed decisions about which analytical workflows to invest in and for which requirement types.

### 1.6 Explicit Out of Scope

- **Per-session token enforcement.** SCBS/SENTINEL-CAP handles per-session token bounds. FGS tracks organisational consumption; it does not enforce session-level limits.
- **Inference API billing.** FGS tracks consumption in tokens. Translation to dollars/costs in specific API contracts or on-premise compute costs is a finance system function that FGS supports with data but does not execute.
- **Individual analyst performance management.** FGS provides consumption data. Using that data for performance evaluation is a management decision outside the platform's governance scope.

---

## 2. Core Responsibilities

### 2.1 Primary Function

FGS/PLUTUS tracks every inference token consumed across all THEMIS sessions and agent tasks — attributing consumption to the analyst, team, session, requirement, interaction class, and model version — implements the organisational tokenomics model (allocation, consumption tracking, reserve access, escalation) — and provides consumption reporting at analyst, team, and platform levels to support budget forecasting, operational planning, and anomaly detection.

### 2.2 Secondary Functions

- Allocation management: maintaining and updating organisational token allocations per period
- Reserve pool governance: processing reserve access requests with supervisor approval workflow
- Consumption anomaly detection: flagging sessions or analysts with anomalous consumption patterns relative to their interaction class and workload
- Budget forecasting: projecting consumption forward based on historical patterns and workload trends
- Interaction class cost profiling: computing the per-turn token consumption profile for each interaction class (STOA vs. single query vs. agent task)
- Model version cost tracking: tracking whether model version changes produce consumption changes (a more capable model may consume more tokens per equivalent analytical task)
- Integration with SCBS: providing team allocation context to SCBS session envelope initialisation

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
TokenAccount:
  account_id:              uuid
  account_type:            TEAM | REQUIREMENT | DIVISION | RESERVE | ANALYST
  account_name:            str
  period:                  str              # e.g., "2026-Q1"
  period_allocation:       int              # tokens allocated for this period
  consumed:                int              # tokens consumed to date in period
  remaining:               int              # period_allocation - consumed
  reserve_draws:           int              # tokens drawn from reserve this period
  status:                  ACTIVE | APPROACHING | EXHAUSTED | SUSPENDED
  managed_by:              str              # supervisor or finance authority

TokenTransaction:
  transaction_id:          uuid
  account_id:              uuid
  session_id:              uuid
  turn_id:                 uuid | null
  analyst_id_hash:         str
  interaction_class:       str
  model_version:           str
  tokens_consumed:         int
  tokens_estimated:        int              # pre-call estimate from SCBS
  transaction_type:        SESSION | AGENT_TASK | STOA | BACKGROUND_SERVICE
  requirement_ref:         str | null
  timestamp:               datetime

AllocationPeriod:
  period_id:               uuid
  period_string:           str              # "2026-Q1"
  start_date:              datetime
  end_date:                datetime
  total_allocated:         int              # sum of all team allocations
  reserve_pool:            int              # platform reserve tokens
  tokenomics_model_version:str
  approved_by:             str             # leadership / IOB

ReserveRequest:
  request_id:              uuid
  requesting_account_id:   uuid
  analyst_id_hash:         str
  tokens_requested:        int
  justification:           str
  approved_by:             str | null
  status:                  PENDING | APPROVED | DENIED
  tokens_granted:          int | null
  request_at:              datetime
  resolved_at:             datetime | null

ConsumptionAnomaly:
  anomaly_id:              uuid
  account_id:              uuid
  session_id:              uuid | null
  anomaly_type:            EXCESS_CONSUMPTION | UNUSUAL_PATTERN | RESERVE_OVERUSE | ZERO_CONSUMPTION
  severity:                HIGH | MEDIUM | LOW
  description:             str
  tokens_involved:         int
  detected_at:             datetime
  reviewed:                bool

TokenomicsModel:
  model_id:                uuid
  version:                 str
  allocation_basis:        str             # how allocations are determined
  reserve_fraction:        float           # fraction of total allocation held as reserve
  escalation_threshold:    float           # utilisation % that triggers alert
  interaction_class_weights:{ class: float }  # relative token weight per interaction class
  approved_by:             str
  effective_from:          datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | TokenTransaction, TokenAccount, AllocationPeriod, ReserveRequest | Indefinite |
| Consumption cache | Redis | Real-time account balances (hot path for SCBS integration) | Period TTL |
| Analytics store | Elasticsearch | TokenTransaction indexed for complex consumption queries | 5 years |
| Event store | MOIRAI | Signed consumption and reserve events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| TokenTransaction | Controlled Unclassified | Analyst-level data restricted per authorization tier |
| TokenAccount balances | Controlled Unclassified | Own account (analyst); team accounts (supervisor); all (IOB/leadership) |
| ConsumptionAnomaly | Controlled Unclassified | Supervisor and IOB access |
| TokenomicsModel | Controlled Unclassified | Leadership, IOB, platform team |

### 3.4 Retention and Purge Policy

TokenTransaction records retained indefinitely — consumption history is an operational intelligence asset for forecasting and audit. AllocationPeriod records retained indefinitely. ReserveRequest records retained indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
GET /accounts/{account_id}/balance
  Auth:     analyst session token (own account) | supervisor token (team) | IOB token (all)
  Response: {
    account_id:            uuid,
    account_name:          str,
    period:                str,
    period_allocation:     int,
    consumed:              int,
    remaining:             int,
    utilisation_pct:       float,
    status:                str,
    reserve_draws:         int
  }
  SLA: p99 < 50ms (from Redis cache)

POST /transactions
  Auth:     inference gateway service account | SCBS service account
  Request:  {
    account_id:            uuid,
    session_id:            uuid,
    turn_id:               uuid | null,
    analyst_id_hash:       str,
    interaction_class:     str,
    model_version:         str,
    tokens_consumed:       int,
    tokens_estimated:      int,
    transaction_type:      str,
    requirement_ref:       str | null
  }
  Response: {
    transaction_id:        uuid,
    account_balance_after: int,
    utilisation_pct:       float,
    alert_generated:       bool
  }
  SLA: p99 < 100ms

GET /accounts/{account_id}/query-balance
  Auth:     SCBS service account
  Response: {
    remaining:             int,
    status:                str,
    session_budget_advisory:int   # suggested per-session budget given remaining
  }
  SLA: p99 < 50ms  # Called by SCBS at session initialization

POST /reserve/request
  Auth:     analyst session token | supervisor token
  Request:  {
    account_id:            uuid,
    tokens_requested:      int,
    justification:         str
  }
  Response: { request_id: uuid, status: PENDING }

POST /reserve/requests/{request_id}/approve
  Auth:     supervisor token | IOB token
  Request:  { tokens_granted: int, approval_note: str }
  Response: { request_id: uuid, status: APPROVED, account_balance_after: int }

GET /reports/consumption
  Auth:     supervisor token | leadership token | IOB token
  Params:   account_id: uuid | null, period: str, group_by: str
  Response: {
    period:                str,
    total_consumed:        int,
    total_allocated:       int,
    utilisation_pct:       float,
    by_interaction_class:  [{ class, consumed, pct }],
    by_model_version:      [{ version, consumed }],
    daily_trend:           [{ date, consumed }]
  }

GET /reports/forecast
  Auth:     leadership token | IOB token
  Params:   period: str
  Response: {
    projected_period_consumption:int,
    confidence_interval:   { low, high },
    projection_basis:      str,
    reserve_adequacy:      str
  }

GET /anomalies
  Auth:     supervisor token | IOB token
  Params:   status: str, severity: str
  Response: [ConsumptionAnomaly]

GET /tokenomics/current
  Auth:     any service account
  Response: TokenomicsModel (current active version)

POST /tokenomics
  Auth:     IOB token (tokenomics model changes require IOB approval)
  Request:  TokenomicsModel
  Response: { model_id: uuid, effective_from: datetime }

GET /health
  Response: {
    status, dependencies: { moirai, redis, postgresql },
    current_period:        str,
    platform_utilisation:  float,
    anomalies_active:      int,
    pending_reserve_requests:int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          FGS_ALLOCATION_PERIOD_OPENED
service_id:         "FGS"
session_id:         null
classification:     CONTROLLED_UNCLASSIFIED
event_payload:
  period_id:              uuid
  period_string:          str
  total_allocated:        int
  reserve_pool:           int
  tokenomics_model:       str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          FGS_ACCOUNT_APPROACHING_LIMIT
event_payload:
  account_id:             uuid
  account_name:           str
  utilisation_pct:        float
  remaining_tokens:       int
  period:                 str

EventType:          FGS_RESERVE_APPROVED
event_payload:
  request_id:             uuid
  account_id:             uuid
  tokens_granted:         int
  approved_by:            str

EventType:          FGS_ANOMALY_DETECTED
event_payload:
  anomaly_id:             uuid
  account_id:             uuid
  anomaly_type:           str
  severity:               str
  tokens_involved:        int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `FGS_ACCOUNT_APPROACHING_LIMIT` | Account reaches 80% utilisation | MOIRAI, supervisor notification, ATHENA banner |
| `FGS_RESERVE_APPROVED` | Reserve request approved | MOIRAI, analyst notification |
| `FGS_ANOMALY_DETECTED` | Consumption anomaly detected | MOIRAI, supervisor notification |
| `FGS_ALLOCATION_PERIOD_OPENED` | New period begins | MOIRAI, all account reset |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| SCBS/SENTINEL-CAP | `SCBS_SESSION_CREATED` | Creates transaction baseline; queries team balance |
| SCBS/SENTINEL-CAP | `SCBS_SESSION_TERMINATED` | Final spend reconciliation; closes session transaction |
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | Updates model cost profile; alerts if new model is significantly more expensive |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed consumption and reserve events | Async event | Events buffered; tracking continues |
| PCES/AEGIS | Classification | Session account association validation | Sync | Proceeds with cached session context |

### 5.2 Feeds Into

| Service | Epithet | What FGS provides | How |
|---|---|---|---|
| SCBS/SENTINEL-CAP | Session Bounding | Team balance advisory at session init | API |
| ATHENA | Interface | Account balance and utilisation display | API |
| Leadership / IOB | Governance | Consumption reports and forecasts | API |
| Finance system | External | Raw consumption data for cost attribution | API + periodic export |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Account balance (cached) | 5ms | 15ms | 50ms |
| Transaction record | 20ms | 50ms | 100ms |
| SCBS query balance | 5ms | 15ms | 50ms |
| Consumption report | 500ms | 2000ms | 5000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — FGS unavailability means consumption tracking gaps; sessions proceed without cost attribution |
| Redis balance cache durability | Transaction write succeeds to PostgreSQL before Redis update |
| RTO | 15 minutes |
| RPO | 0 minutes (every transaction writes to PostgreSQL immediately) |

### 6.3 Graceful Degradation

| Dependency unavailable | Behavior | Impact |
|---|---|---|
| Redis (balance cache) | Balance reads from PostgreSQL (higher latency); SCBS uses conservative advisory | Session initialization slower |
| MOIRAI | Events buffered; transactions still recorded in PostgreSQL | Provenance gap logged |

---

## 7. Security Model

### 7.1 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst (own) | Own account balance; own transaction history | Session token |
| Supervisor | Team account balances; team transaction history; reserve approvals | Supervisor token |
| Leadership / IOB | All accounts; all reports; tokenomics management | Leadership / IOB token |
| Inference gateway / SCBS | Transaction posting; balance queries | Service account |
| Finance system | Periodic export | Finance service account |

### 7.2 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/fgs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/fgs/db-credentials` | 30 days |
| Redis credentials | `themis/fgs/redis-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Balance cache/PostgreSQL divergence (crash during transaction) | Low | P2 — account balance temporarily inaccurate | Periodic reconciliation job | Dual-write: PostgreSQL first, Redis after; reconcile on startup |
| Anomaly detection false positive (legitimate high-consumption session flagged) | Medium | P2 — supervisor alert fatigue | Anomaly dismissal rate | Anomaly model trained on interaction class baselines; STOA sessions not flagged as anomalous |
| Reserve pool exhausted | Low | P1 — high-priority work cannot access reserve | Real-time reserve balance monitoring | Alert at 50% reserve consumption; leadership notification at 80% |

### 8.1 Known Design Risks

- **The tokenomics model is politically sensitive.** Allocating inference tokens across teams creates winners and losers in a resource-constrained environment. The tokenomics model — who gets how many tokens, what justifies a reserve draw — will generate organisational conflict that is not a platform engineering problem. FGS implements whatever model is approved; but the approval process and the governance of that model will require active leadership engagement that the platform team cannot provide.
- **Consumption data enables monitoring of analyst workload that was not previously possible.** Leadership now has data on how many sessions analysts run, how long they take, and what interaction classes they use. This is operationally valuable for planning but may be perceived as surveillance. The policy governing what management decisions consumption data may inform must be established before Phase 1 deployment — ideally in the IOB charter.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `fgs.transaction.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `fgs.platform.utilisation_pct` | Gauge | `> 85%` of period allocation | P1 |
| `fgs.reserve.utilisation_pct` | Gauge | `> 50%` | P2 |
| `fgs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `fgs.anomalies.unreviewed_count` | Gauge | `> 10` | P2 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/fgs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every inference token consumed is permanently recorded with attribution to analyst, session, team, and requirement. The complete consumption history is available for audit and financial reconciliation. Tokenomics model changes are MOIRAI-attested with IOB approval references.

---

## 11. Tokenomics Reference

### 11.1 Standard Allocation Basis

| Unit | Allocation basis | Review cadence |
|---|---|---|
| Analytical team | Historical consumption × growth factor + planned new requirements | Quarterly |
| Reserve pool | 15% of total platform allocation | Quarterly |
| Individual analyst | No hard cap — managed through team account | Continuous |

### 11.2 Interaction Class Cost Profile (Initial Estimates)

| Interaction Class | Relative token weight | Basis |
|---|---|---|
| Single ATHENA query | 1.0× | Baseline |
| Evidence analysis | 1.5× | Longer context |
| Capability assessment | 2.0× | Multiple retrieval + synthesis |
| STOA research (per sub-question) | 3.0× | Skill invocation + synthesis |
| Agent task (per action) | 1.5× | Inference + tool call |
| STOA full research session (5 sub-questions) | 12–18× | End-to-end orchestration |

*Weights are initial estimates to be calibrated from operational data after Phase 1.*

### 11.3 Reserve Access Criteria (Proposed)

Reserve access is appropriate when:
- A high-priority analytical requirement has time-critical collection or assessment needs
- An analytical team has consumed its allocation due to unexpected workload (not due to inefficiency)
- A SENTINEL strategic warning generates high-priority follow-on analytical work

Reserve access is not appropriate for:
- Routine analytical work that should have been in the team's allocation
- Correction of inefficient analytical workflows (the fix is workflow improvement, not reserve access)
- Individual analyst preferences that fall outside their team's analytical scope

---

## 12. Implementation Roadmap

### Phase 1 — Transaction Recording and Account Management (Weeks 1–4)

- TokenAccount, TokenTransaction, AllocationPeriod schemas
- Transaction recording endpoint (inference gateway integration)
- SCBS balance query endpoint
- Basic account balance read endpoints (analyst, supervisor, leadership views)
- Redis balance cache
- Period allocation setup for Phase 1 deployment
- MOIRAI event emission

**Phase gate criterion:** Every session turn produces a TokenTransaction record attributed to the correct account. SCBS receives balance advisory at session initialization. Account balance readable at all authorization tiers. First allocation period opened with MOIRAI event.

### Phase 2 — Reserve Governance, Anomaly Detection, and Tokenomics Reporting (Weeks 5–8)

- Reserve pool and reserve request workflow
- Consumption anomaly detection (by interaction class baseline)
- Consumption reports and forecast endpoint
- Interaction class cost profile calibration
- Tokenomics model governance endpoint (IOB approval workflow)
- Finance system export
- ATHENA balance display integration

**Phase gate criterion:** Reserve request workflow produces MOIRAI events. Anomaly detection fires on test excess-consumption sessions. Leadership consumption report accessible. IOB approves initial tokenomics model. ARB and Cell Lead sign-off.

---

## 13. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| Tokenomics model approval | IOB-approved allocation basis, reserve fraction, escalation thresholds, and interaction class weights | Phase 1 allocation period cannot open without an approved tokenomics model |
| Consumption data governance policy | What management decisions consumption data may inform; what analyst privacy protections apply | Phase 1 — must be established before analyst consumption data is collected |

---

## 14. Training and Analyst Guidance

### 14.1 What the Analyst Sees

In the ATHENA session header, a token indicator shows the team account's current utilisation: green (< 70%), amber (70–85%), red (> 85%). Hovering shows the remaining allocation for the period. When an account reaches 80%, an ATHENA notification appears: "Your team's token allocation for this period is at 80%. Contact your supervisor if additional allocation is needed."

### 14.2 What the Analyst Should Do

Use STOA research orchestration for complex multi-faceted requirements; use single ATHENA sessions for focused evidence queries. Reserve access is not a routine option — it requires supervisor approval and a documented justification. If your team's allocation is frequently exhausted, the correct response is to request an allocation review in the next planning cycle, not to draw repeatedly from the reserve.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — service specified in original THEMIS governance foundation |

## Appendix D: Red Team Findings
*Pending — Phase 1 gate review.*
