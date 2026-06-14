# SCBS — Session Capability Bounding Service
### SENTINEL-CAP · *"The sentinel who enforces the capability ceiling — what this session may do and nothing more"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `SCBS` |
| **Epithet** | `SENTINEL-CAP` |
| **Full name** | Session Capability Bounding Service |
| **Namespace** | `themis-agent` |
| **Layer** | Agent-Native Infrastructure |
| **Build phase** | Phase 2–3 (Weeks 5–28) |
| **Build priority** | 16 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — bounds what AI agents may do within a session |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**SCBS/SENTINEL-CAP answers: Does this agent session remain within its authorized capability envelope — and if it is about to exceed that envelope, what happens?**

### 1.2 Why This Service Exists

As AI moves from generating outputs to taking actions — retrieving from live systems, writing to documents, executing multi-step research — the threat surface changes fundamentally. The critical risk is not a bad output. It is a bad action that cannot be undone, executed by an agent that was never explicitly authorized to perform it.

Without SCBS, agentic AI sessions operate without bounds: no spend cap, no resource scope, no time limit, no escalation path. An agent that runs indefinitely, consuming unbounded inference tokens and writing to any system it can reach, is an operational security failure regardless of how well-intentioned the task was. SCBS enforces the capability envelope that makes agentic AI governable.

The pre-call cost estimate is the design element that makes this meaningful rather than reactive. Before any state-changing action, the agent receives a budget estimate from SCBS. An action that would exceed the remaining budget is declined before it is attempted, not terminated mid-execution. Governance before action, not cleanup after.

### 1.3 Why This Service Is Sixteenth

SCBS requires PCES for session scope (an agent session is nested within an analyst session — the agent cannot access more than the analyst) and MOIRAI for event recording. It is Phase 2-3 because agentic capability in ATHENA is a Phase 2 feature — analysts do not use AI agents in Phase 1. SCBS must be operational before any agent capability is exposed.

### 1.4 Design Principles

- **No agent session without a capability envelope.** There is no agentic operation without an SCBS session. The envelope is created at agent session initialization and cannot be expanded mid-session.
- **Fails closed on bound exceedance.** When any bound is exceeded, the session is suspended and escalation is triggered. There is no graceful degradation mode that allows the session to continue past its authorized bounds.
- **Pre-call estimation, not post-call accounting.** SCBS exposes a pre-call cost estimate endpoint. Agents must call this before any state-changing action. An action that would exceed budget is declined before execution, not reversed after.
- **The analyst session is the ceiling, not the floor.** An agent session nested within an analyst session cannot exceed the analyst's PCES-granted privileges. The agent session envelope is always a subset of the analyst session privilege, never an extension of it.
- **Spend accounting creates a self-defeating gaming dynamic.** Unused capabilities that were requested but never exercised are flagged. This creates an incentive toward minimum viable capability requests rather than broad requests — the opposite of IAM's typical access sprawl tendency.

### 1.5 Explicit Out of Scope

- **Credential management.** SCBS bounds what the agent can do; CBS/BROKER manages the credentials that enable the agent to do it. SCBS does not issue or hold credentials.
- **State rollback.** RSS/ROLLBACK handles pre-action snapshots and rollback. SCBS triggers RSS to preserve rollback points; it does not execute them.
- **Capability surface definitions.** What operations are available on each service is defined in CBS's capability surface registry. SCBS bounds how much of that surface can be used, not what the surface contains.

---

## 2. Core Responsibilities

### 2.1 Primary Function

SCBS/SENTINEL-CAP creates and enforces capability envelopes for AI agent sessions — defining maximum inference spend, resource scope, environment designation, and session TTL at creation time; maintaining a live budget ledger per session; declining actions that would exceed bounds before they are attempted; triggering human escalation when bounds are approached or anomalous spend rates are detected; and recording all capability decisions to MOIRAI.

### 2.2 Secondary Functions

- Pre-call cost estimation: responding to agent pre-call estimate requests before state-changing actions
- Anomalous spend rate detection: identifying spend patterns that suggest the agent is operating outside its intended task scope
- Session summary generation: producing a structured session summary at session close for analyst and supervisor review
- CBS revocation trigger: notifying CBS/BROKER to revoke all handles when a session terminates for any reason
- RSS snapshot trigger: notifying RSS/ROLLBACK to create a pre-action snapshot before every write operation
- Unused capability reporting: tracking which capabilities were requested but not exercised for capability surface governance feedback

### 2.3 What This Service Does Not Decide

SCBS enforces the capability envelope that was defined at session creation. Whether the envelope is appropriate for the task, whether an escalation should be approved or the session terminated, and whether a session that approached its limits warrants supervisor review are human decisions. SCBS enforces; humans decide appropriateness.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
CapabilityEnvelope:
  envelope_id:             uuid
  agent_session_id:        uuid
  analyst_session_id:      uuid           # FK → PCES analyst session — agent inherits this scope
  declared_intent:         str            # what task this agent session is performing
  max_spend_tokens:        int            # maximum inference tokens for this session
  max_resource_scope:      [str]          # list of resource identifiers accessible
  environment_designation: SANDBOX | STAGING | PRODUCTION
  ttl_seconds:             int            # session time-to-live
  created_at:              datetime
  expires_at:              datetime       # created_at + ttl_seconds
  status:                  ACTIVE | SUSPENDED | TERMINATED | EXPIRED

SpendLedger:
  ledger_id:               uuid
  envelope_id:             uuid
  total_budget:            int            # = max_spend_tokens
  total_spent:             int
  remaining:               int
  spend_rate_current:      float          # tokens/second over last 60 seconds
  spend_rate_baseline:     float          # expected rate from declared intent
  anomaly_flag:            bool
  entries:                 [SpendEntry]

SpendEntry:
  entry_id:                uuid
  ledger_id:               uuid
  action_type:             INFERENCE | RETRIEVAL | WRITE | READ | MCP_CALL
  resource:                str
  tokens_consumed:         int
  estimated_tokens:        int            # from pre-call estimate
  timestamp:               datetime
  action_id:               uuid           # FK → the action that consumed this budget

EscalationEvent:
  escalation_id:           uuid
  envelope_id:             uuid
  trigger:                 SPEND_EXCEEDED | ANOMALOUS_RATE | TTL_EXCEEDED | SCOPE_VIOLATION | MANUAL
  trigger_detail:          str
  escalated_to:            str            # analyst or supervisor
  resolution:              APPROVED | TERMINATED | PENDING
  resolution_rationale:    str | null
  escalated_at:            datetime
  resolved_at:             datetime | null

SessionSummary:
  summary_id:              uuid
  envelope_id:             uuid
  total_actions:           int
  write_actions:           int
  total_spend:             int
  unused_capability_items: [str]          # capabilities requested but not exercised
  escalations:             int
  generated_at:            datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CapabilityEnvelope, SpendLedger, EscalationEvent, SessionSummary | Session + 7 years |
| Live ledger cache | Redis | SpendLedger (hot path — updated on every action) | Session TTL |
| Event store | MOIRAI | Signed capability and escalation events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| CapabilityEnvelope | Inherits analyst session classification | Session-compartmented |
| SpendLedger | Inherits analyst session classification | Session-compartmented |
| EscalationEvent | Inherits session classification | Supervisor and IOB access |

### 3.4 Retention and Purge Policy

All records retained for session lifetime plus seven years. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /sessions/create
  Auth:     ATHENA service account | analyst session token
  Request:  {
    analyst_session_id:    uuid,
    declared_intent:       str,
    max_spend_tokens:      int,
    resource_scope:        [str],
    environment:           str,
    ttl_seconds:           int
  }
  Response: {
    envelope_id:           uuid,
    agent_session_token:   str,     # scoped token for this agent session
    expires_at:            datetime
  }
  SLA: p99 < 300ms

POST /sessions/{envelope_id}/estimate
  Auth:     agent session token
  Request:  {
    action_type:           str,
    resource:              str,
    estimated_tokens:      int
  }
  Response: {
    approved:              bool,
    remaining_after:       int,
    budget_utilisation:    float,
    warning:               str | null   # if approaching limit
  }
  SLA: p99 < 50ms

POST /sessions/{envelope_id}/debit
  Auth:     agent session token
  Request:  {
    action_id:             uuid,
    action_type:           str,
    resource:              str,
    tokens_consumed:       int
  }
  Response: {
    accepted:              bool,
    remaining:             int,
    suspended:             bool      # true if this debit triggered suspension
  }
  SLA: p99 < 100ms

GET /sessions/{envelope_id}/budget
  Auth:     agent session token | analyst session token | supervisor token
  Response: SpendLedger (current state)
  SLA: p99 < 50ms (from Redis)

POST /sessions/{envelope_id}/terminate
  Auth:     analyst session token | supervisor token | SCBS internal (TTL expiry)
  Request:  { reason: str }
  Response: {
    envelope_id:           uuid,
    session_summary:       SessionSummary,
    cbs_handles_revoked:   int
  }

GET /sessions/{envelope_id}/summary
  Auth:     analyst session token | supervisor token | IOB token
  Response: SessionSummary

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis },
    active_sessions:       int,
    suspended_sessions:    int,
    anomaly_flags_active:  int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          SCBS_SESSION_CREATED
service_id:         "SCBS"
session_id:         uuid
classification:     str
event_payload:
  envelope_id:            uuid
  analyst_session_id:     uuid
  declared_intent:        str
  max_spend_tokens:       int
  environment:            str
  ttl_seconds:            int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          SCBS_SPEND_EXCEEDED
event_payload:
  envelope_id:            uuid
  limit_type:             SPEND | SCOPE | TTL
  value_at_trigger:       int
  max_allowed:            int

EventType:          SCBS_ANOMALOUS_RATE
event_payload:
  envelope_id:            uuid
  current_rate:           float
  baseline_rate:          float
  ratio:                  float

EventType:          SCBS_SESSION_TERMINATED
event_payload:
  envelope_id:            uuid
  termination_reason:     str
  total_spend:            int
  write_actions:          int
  escalations:            int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `SCBS_SESSION_CREATED` | Agent session initialized | MOIRAI, CBS/BROKER (enable handle issuance) |
| `SCBS_SPEND_EXCEEDED` | Any limit breached | MOIRAI, analyst notification, supervisor alert |
| `SCBS_SESSION_TERMINATED` | Session ends for any reason | MOIRAI, CBS/BROKER (revoke all handles), RSS/ROLLBACK (preserve snapshots) |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| PCES/AEGIS | Classification Enforcement | Analyst session scope inherited by agent session | Sync | Agent session creation blocked |
| MOIRAI | Provenance | Signed capability events | Async event | Events buffered; session still enforced |

### 5.2 Feeds Into

| Service | Epithet | What SCBS provides | How |
|---|---|---|---|
| CBS/BROKER | Credential Broker | Session token for handle issuance; termination signal for handle revocation | API + `SCBS_SESSION_TERMINATED` event |
| RSS/ROLLBACK | State Snapshots | Pre-write-action snapshot trigger | API call before each write action |
| MGS/TERMINUS | MCP Gateway | Session capability envelope validation on MCP calls | API |
| MOIRAI | Provenance | All capability events | Signed events |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Agent session envelope is a strict subset of the analyst session privilege. SCBS validates the analyst session token at agent session creation and caps the agent's resource scope to what the analyst's PCES grant allows.
- **Failure behavior:** PCES unavailable → agent session creation blocked. Active agent sessions continue with the scope established at creation (already validated).

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Pre-call estimate | 10ms | 30ms | 50ms |
| Spend debit | 20ms | 50ms | 100ms |
| Session creation | 100ms | 200ms | 300ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.9% — SCBS unavailability stops all agentic operations |
| MOIRAI event durability | 99.999% |
| RTO | 5 minutes |
| RPO | 0 (Redis ledger state is the authoritative live state) |

### 6.3 Graceful Degradation

SCBS follows a **fail-closed policy**. If SCBS is unavailable, no new agent sessions are created and active sessions are suspended. There is no mode that allows agentic operations to proceed without capability enforcement.

---

## 7. Security Model

### 7.1 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ATHENA / analyst | Session creation | Analyst session token |
| Agent (within session) | Estimate, debit, budget read | Agent session token (issued by SCBS at creation) |
| Supervisor | Session summary; termination | Supervisor token |
| IOB | Full read | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/scbs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/scbs/db-credentials` | 30 days |
| Redis credentials | `themis/scbs/redis-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Agent bypasses pre-call estimate | Medium | P1 — spend may exceed limit mid-action | Debit without preceding estimate flag | Agent framework must enforce estimate-before-write as a calling convention |
| Anomalous rate threshold miscalibrated | Medium | P2 — excessive false escalations | Escalation rate monitoring | Threshold configurable; calibrated from operational data |
| Redis ledger state lost (Redis restart) | Low | P1 — spend ledger reset to zero mid-session | Redis persistence configuration | Redis configured with AOF persistence; PostgreSQL as backup source of truth |

### 8.1 Known Design Risks

- **Declared intent to capability mapping is not automated.** The agent declares intent (e.g., "summarise collection reports on Topic X") and the analyst or system sets the capability envelope. There is no automated mapping from intent to minimum viable capability. If the envelope is set too broadly (which is the path of least resistance), SCBS enforces a bound that is not minimum viable. Resolution path: CBS capability surface governance creates the incentive structure against over-requesting (unused capabilities are flagged), but this requires operational maturity.

---

## 9. Observability

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `scbs.session.debit.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `scbs.session.exceeded_rate` | Gauge | `> 5%` of sessions | P2 |
| `scbs.session.anomaly_rate` | Gauge | `> 2%` of sessions | P2 |
| `scbs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/scbs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every agent session's capability envelope, every spend event, every bound exceedance, and every termination are permanently recorded. An oversight body can reconstruct exactly what any agent session was authorized to do and whether it stayed within bounds.

---

## 11. Implementation Roadmap

### Phase 1 — Envelope Creation and Spend Enforcement (Weeks 5–16)

- CapabilityEnvelope and SpendLedger schemas
- Session creation, estimate, debit, and budget endpoints
- Redis live ledger with PostgreSQL backup
- MOIRAI event emission
- CBS/BROKER session token coordination
- TTL enforcement

**Phase gate criterion:** Every agent session operates within a MOIRAI-attested capability envelope. Pre-call estimate responds correctly. Spend exceedance suspends session within 1 second.

### Phase 2 — Anomaly Detection, Escalation, and Session Summary (Weeks 17–28)

- Anomalous spend rate detection
- Human escalation workflow
- SessionSummary generation at close
- RSS/ROLLBACK snapshot trigger integration
- Unused capability reporting
- Supervisor review interface

**Phase gate criterion:** Anomalous rate detected in test simulations. Escalation workflow produces MOIRAI event and analyst notification. SessionSummary includes unused capability report. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate SCBS. The capability envelope configuration — what spend limits and resource scopes are appropriate for different task types — is an operational policy established by the AI Trust Cell operating model, not a GC policy decision.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

When creating an agent session in ATHENA, the analyst specifies the declared intent and sees a capability envelope configuration form pre-populated with suggested bounds based on the intent type. An active session shows a live spend gauge in the ATHENA session header. If the session approaches its limit or triggers an escalation, the analyst receives an in-session notification.

### 13.2 What the Analyst Should Do

Set minimum viable capability envelopes. The spend limit should be enough for the intended task with reasonable headroom — not the platform maximum. If the agent is suspended for exceeding bounds, investigate whether the task was as scoped as intended before requesting a supervisor override.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 2 gate review.*
