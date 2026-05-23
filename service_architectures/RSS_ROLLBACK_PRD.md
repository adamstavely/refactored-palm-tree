# RSS — Reversibility & State Snapshot Service
### ROLLBACK · *"The ability to undo what was done — the engineering guarantee that no agentic action is permanently irreversible within the retention window"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `RSS` |
| **Epithet** | `ROLLBACK` |
| **Full name** | Reversibility & State Snapshot Service |
| **Namespace** | `themis-agent` |
| **Layer** | Agent-Native Infrastructure |
| **Build phase** | Phase 3–4 (Weeks 9–28) |
| **Build priority** | 18 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — ensures every agentic write action is reversible within the retention window |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**RSS/ROLLBACK answers: If this agent action produced an unwanted state, can we restore what existed before it — and how?**

### 1.2 Why This Service Exists

An AI agent that takes an irreversible action that was not intended — overwriting an analytical draft, deleting a session record, modifying a source citation — has produced a harm that cannot be corrected. The harm may not be noticed immediately. By the time it is discovered, the action may be hours or days in the past.

RSS/ROLLBACK is the service that makes irreversibility impossible within a defined retention window. Before every write action, RSS captures a pre-action state snapshot. Within 72 hours, any snapshot can be restored with a single command. The agent can write; the analyst can undo.

This is not an afterthought safety net. It is a first-order design requirement for agentic deployment. An analyst who cannot undo an agent's actions cannot trust the agent to act autonomously on anything consequential. RSS is what makes autonomous action trustworthy.

### 1.3 Why This Service Is Eighteenth

RSS requires SCBS to be operational (agent sessions are created by SCBS; RSS receives write-action triggers from SCBS) and MOIRAI for snapshot records. It is Phase 3-4 rather than Phase 2-3 because snapshots are only meaningful if there is state to snapshot — which requires agent sessions (SCBS and CBS) to have been deployed and producing actions first.

### 1.4 Design Principles

- **Snapshot before write, always.** There is no write action through CBS/BROKER without a preceding RSS snapshot. The calling convention is enforced at the SCBS level: SCBS triggers RSS before notifying CBS to allow the write.
- **Selective rollback is preferred over session rollback.** Restoring a single action boundary is less disruptive than restoring the full session. RSS supports both, but surfaces selective action-boundary rollback as the primary path.
- **The retention window is a policy decision, not an engineering default.** 72 hours is the initial design; operational experience may change this. The retention window is configurable per environment designation (SANDBOX vs. PRODUCTION may differ).
- **Rollback is a human action, not an automated response.** RSS provides rollback capability; analysts and supervisors decide whether and when to use it. Automated rollback on anomaly detection is not in scope — that risks undoing valid actions based on false positives.
- **Snapshots are integrity-protected.** State snapshots include a SHA-256 hash of the pre-action state. Rollback validates the snapshot hash before applying it. A corrupted snapshot cannot be silently applied.

### 1.5 Explicit Out of Scope

- **Rollback of actions outside THEMIS.** RSS captures state snapshots for actions taken through CBS/BROKER within THEMIS. External system state changes that occur as a consequence of those actions but outside the THEMIS-proxied call path may not be capturable.
- **Version control for analytical documents.** DPS/CODEX manages document versioning. RSS manages pre-action state for in-session agentic write operations.
- **Automatic conflict resolution.** If state has been modified by another session between the snapshot and the rollback, RSS surfaces the conflict and requires human resolution.

---

## 2. Core Responsibilities

### 2.1 Primary Function

RSS/ROLLBACK captures pre-action state snapshots before every write operation in an agent session, maintains those snapshots within a configurable retention window (default 72h) with tiered storage by age, provides single-command rollback to any snapshot within the retention window, and surfaces available rollback points in the ATHENA reviewer interface for analyst and supervisor decision.

### 2.2 Secondary Functions

- Action-boundary rollback: restoring state to any single action boundary rather than full session rollback
- Snapshot integrity validation: verifying the hash of a snapshot before applying it
- Conflict detection: identifying state modifications by other sessions between snapshot and rollback time
- Tiered storage management: moving snapshots from HOT to WARM to COLD storage as they age
- Session rollback summary: at session close, providing a summary of all rollback points available and whether any were used
- Retention window enforcement: purging snapshots beyond the retention window

### 2.3 What This Service Does Not Decide

RSS captures snapshots and executes rollbacks when instructed. Whether a specific action warrants rollback, whether a session's actions should be fully reversed, and whether a conflict detected during rollback should be resolved in favour of the snapshot or the current state are human decisions. RSS provides the capability; humans exercise it.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
StateSnapshot:
  snapshot_id:             uuid
  session_id:              uuid
  envelope_id:             uuid             # FK → SCBS CapabilityEnvelope
  action_id:               uuid             # the action this snapshot precedes
  action_type:             str              # WRITE | DELETE | MODIFY | CREATE
  action_description:      str              # human-readable description of the action
  target_resource:         str              # what is being acted upon
  state_representation:    str              # structured diff or full state (compressed)
  state_hash:              str              # SHA-256 of state before action
  storage_tier:            HOT | WARM | COLD
  created_at:              datetime
  expires_at:              datetime         # created_at + retention_window
  rolled_back:             bool
  rolled_back_at:          datetime | null

RollbackRecord:
  rollback_id:             uuid
  session_id:              uuid
  snapshot_id:             uuid             # snapshot restored to
  rollback_type:           FULL_SESSION | ACTION_BOUNDARY
  initiated_by:            str              # analyst ID or supervisor ID hash
  actions_reversed:        [uuid]           # action_ids that were reversed
  conflict_detected:       bool
  conflict_description:    str | null
  status:                  SUCCESS | PARTIAL | FAILED | CONFLICT_PENDING
  started_at:              datetime
  completed_at:            datetime | null

RetentionPolicy:
  policy_id:               uuid
  environment:             SANDBOX | STAGING | PRODUCTION
  retention_hours:         int              # default 72
  hot_tier_hours:          int              # default 4
  warm_tier_hours:         int              # default 24
  cold_tier_hours:         int              # default 44 (remainder to 72)
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Snapshot index | PostgreSQL | StateSnapshot metadata and hashes | Session + 7 days post-expiry |
| HOT storage | Redis | Snapshots < 4h old (fast retrieval for recent rollbacks) | 4h TTL |
| WARM storage | PostgreSQL (JSONB) | Snapshots 4–24h old | 24h from creation |
| COLD storage | Object storage (S3-compatible) | Snapshots 24–72h old | 72h from creation |
| Rollback records | PostgreSQL | RollbackRecord with full audit | Session + 7 years |
| Event store | MOIRAI | Signed snapshot and rollback events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| StateSnapshot | Inherits agent session classification | Session-compartmented; may contain classified state |
| RollbackRecord | Inherits session classification | Supervisor and IOB access |

### 3.4 Retention and Purge Policy

State snapshots are purged after the retention window (72h default). Snapshot metadata and hashes retained for session lifetime plus seven years. RollbackRecord retained indefinitely — rollback history is a permanent accountability record. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /snapshots
  Auth:     SCBS service account (triggered before every write)
  Request:  {
    session_id:            uuid,
    envelope_id:           uuid,
    action_id:             uuid,
    action_type:           str,
    action_description:    str,
    target_resource:       str,
    state_representation:  str,       # pre-action state; compressed
    state_hash:            str
  }
  Response: {
    snapshot_id:           uuid,
    storage_tier:          str,
    expires_at:            datetime
  }
  SLA: p99 < 200ms

GET /sessions/{session_id}/rollback-points
  Auth:     analyst session token | supervisor token
  Response: [
    {
      snapshot_id:         uuid,
      action_description:  str,
      action_type:         str,
      created_at:          datetime,
      expires_at:          datetime,
      storage_tier:        str
    }
  ]
  SLA: p99 < 300ms

POST /rollback
  Auth:     analyst session token | supervisor token
  Request:  {
    snapshot_id:           uuid,
    rollback_type:         FULL_SESSION | ACTION_BOUNDARY,
    rationale:             str
  }
  Response: {
    rollback_id:           uuid,
    status:                str,
    actions_reversed:      [uuid],
    conflict_detected:     bool,
    conflict_description:  str | null
  }
  SLA: p99 < 5000ms

GET /rollback/{rollback_id}
  Auth:     analyst session token | supervisor token | IOB token
  Response: RollbackRecord

GET /sessions/{session_id}/rollback-history
  Auth:     supervisor token | IOB token
  Response: [RollbackRecord]

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis, object_storage },
    active_snapshots:      int,
    hot_storage_count:     int,
    cold_storage_count:    int,
    rollbacks_24h:         int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          RSS_SNAPSHOT_CREATED
service_id:         "RSS"
session_id:         uuid
classification:     str
event_payload:
  snapshot_id:            uuid
  action_id:              uuid
  action_type:            str
  action_description:     str
  state_hash:             str
  expires_at:             datetime
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          RSS_ROLLBACK_EXECUTED
event_payload:
  rollback_id:            uuid
  snapshot_id:            uuid
  rollback_type:          str
  actions_reversed:       int
  conflict_detected:      bool
  status:                 str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `RSS_SNAPSHOT_CREATED` | Before every write action | MOIRAI |
| `RSS_ROLLBACK_EXECUTED` | Rollback completed | MOIRAI, analyst notification, supervisor alert if PARTIAL or CONFLICT |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| SCBS/SENTINEL-CAP | Session Bounding | Write-action triggers; session termination signal | Sync trigger + Async event | If RSS unavailable, write action blocked (SCBS enforces pre-snapshot requirement) |
| MOIRAI | Provenance | Signed snapshot and rollback events | Async event | Events buffered; snapshots still created |
| PCES/AEGIS | Classification Enforcement | Session compartment on snapshot access | Sync | Proceeds with cached session scope |

### 5.2 Feeds Into

| Service | Epithet | What RSS provides | How |
|---|---|---|---|
| ATHENA | Interface | Available rollback points in session reviewer panel | API |
| IOB Reporting | Oversight | Rollback history; conflict records | Audit endpoints |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Snapshot creation | 50ms | 100ms | 200ms |
| Rollback point listing | 50ms | 150ms | 300ms |
| Rollback execution | 500ms | 2000ms | 5000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — RSS unavailability blocks all agent write actions (SCBS requires pre-snapshot) |
| Object storage durability | 99.9999% (snapshots must not be lost) |
| RTO | 15 minutes |

### 6.3 Graceful Degradation

RSS follows a write-blocking policy when unavailable: if RSS cannot create a snapshot before a write action, the write action is blocked by SCBS. This is a deliberate design choice — write actions without pre-action snapshots are not permitted in the PRODUCTION environment. In SANDBOX environments, the write-blocking policy is configurable.

---

## 7. Security Model

### 7.1 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| SCBS (write trigger) | `POST /snapshots` | SCBS service account |
| Analyst (own session) | Rollback points; rollback execution for own sessions | Session token |
| Supervisor | Rollback points and history for their team | Supervisor token |
| IOB | Full rollback history | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/rss/signing-key` | 90 days |
| PostgreSQL credentials | `themis/rss/db-credentials` | 30 days |
| Object storage credentials | `themis/rss/object-storage-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Snapshot storage full or slow | Low | P1 — write actions blocked if HOT storage unavailable | Storage capacity monitoring | Automatic tier migration; capacity alerts |
| Conflict on rollback (state changed by another session) | Medium | P2 — rollback cannot safely proceed | Conflict detection in rollback logic | Surface conflict; require human resolution; do not auto-resolve |
| Snapshot hash mismatch on rollback | Very low | P1 — snapshot corrupted; rollback unsafe | Hash validation before apply | Reject rollback; alert; snapshot flagged as corrupted in MOIRAI |

### 8.1 Known Design Risks

- **External state side-effects may not be capturable.** An agent action that triggers a chain of external events (sending an email, updating an external database, triggering a collection request) may be partially undone by RSS but the external consequences cannot be recalled. RSS captures THEMIS platform state; it does not have authority over external systems. Resolution path: agents should be designed to defer irreversible external actions to human confirmation; RSS covers platform-internal state.

---

## 9. Observability

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `rss.snapshot.latency_p99` | Histogram | `> 200ms for 5m` | P1 |
| `rss.rollback.conflict_rate` | Gauge | `> 5%` of rollbacks | P2 |
| `rss.storage.hot_capacity_pct` | Gauge | `> 80%` | P1 |
| `rss.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/rss/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every pre-action snapshot and every rollback execution is permanently recorded. The state_hash in the snapshot proves the snapshot content was not altered after creation. An oversight body can verify that a rollback actually restored the state that was captured.

---

## 11. Implementation Roadmap

### Phase 1 — Snapshot Creation and HOT/WARM Storage (Weeks 9–16)

- StateSnapshot schema with HOT (Redis) and WARM (PostgreSQL JSONB) tiers
- `POST /snapshots` endpoint and SCBS write-trigger integration
- Hash validation on creation
- `GET /sessions/{id}/rollback-points` endpoint
- Basic rollback execution (action boundary only initially)
- MOIRAI event emission

**Phase gate criterion:** Every agent write action produces a snapshot within 200ms. Rollback-points listing shows all active snapshots. Simple action-boundary rollback executes correctly on test cases.

### Phase 2 — COLD Storage, Full Session Rollback, and Conflict Detection (Weeks 17–28)

- COLD storage tier (object storage) with automatic tier migration
- Full session rollback
- Conflict detection and human resolution surface
- Snapshot hash mismatch handling
- Retention window enforcement and purge
- Supervisor and IOB audit endpoints

**Phase gate criterion:** COLD storage tier operational. Full session rollback reverses all actions in test session. Conflict detected when state changes between snapshot and rollback. Purge executes at 72h. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate RSS. The retention window (72h default) is a platform operational policy owned by the AI Trust Cell. Environment-specific retention policies (SANDBOX vs. PRODUCTION) may differ.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In the ATHENA agent session reviewer panel, a rollback timeline shows all available rollback points for the current session, with action descriptions and timestamps. Each point shows the action it precedes and how long it remains available. A "Rollback to here" button initiates rollback for any point within the retention window.

### 13.2 What the Analyst Should Do

Review the agent's action log before marking a session complete. If any action produced unexpected results, use the rollback timeline to restore state before that action. If a conflict is detected during rollback (the target state has been modified by another session), do not force the rollback — escalate to your supervisor for resolution.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 3 gate review.*
