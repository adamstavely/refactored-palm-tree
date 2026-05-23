# CBS — Credential Broker Service
### BROKER · *"The intermediary who holds what is valuable and issues only what is needed — never the key itself, only access shaped to the task"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `CBS` |
| **Epithet** | `BROKER` |
| **Full name** | Credential Broker Service |
| **Namespace** | `themis-agent` |
| **Layer** | Agent-Native Infrastructure |
| **Build phase** | Phase 2–3 (Weeks 5–28) |
| **Build priority** | 17 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — governs what credentials and capabilities AI agents may access |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**CBS/BROKER answers: Has this agent session been granted the minimum viable scoped capability it needs — without ever receiving raw credentials that would allow it to act beyond that scope?**

### 1.2 Why This Service Exists

The fundamental security problem in agentic AI is credential exposure. An AI agent that holds raw credentials — API keys, passwords, service tokens — has the full capability of those credentials. If the agent's prompt is manipulated via injection, if the agent executes a task it was not intended to perform, or if the agent session is compromised, the attacker inherits everything those credentials can do.

CBS/BROKER eliminates raw credential exposure entirely. Agents never hold credentials. They hold time-limited, operation-scoped handles that CBS issues and that CBS proxies. A handle issued for "read-only query of the HUMINT corpus" cannot be used to write to it. A handle issued for one session cannot be used after that session ends. A handle that is revoked — immediately on session termination or SCBS escalation — is worthless.

The proxy model is the load-bearing architectural element. CBS doesn't just issue handles; it proxies every credentialed call through itself. CBS logs every call, validates it against the handle's scope, and refuses calls outside that scope. An agent cannot "forget" its handle restrictions when making a call — the restriction is enforced by the service processing the call, not by the agent respecting it.

### 1.3 Why This Service Is Seventeenth

CBS requires SCBS to be operational (it issues handles within SCBS-validated session envelopes) and MOIRAI for call logging. It is Phase 2-3 because it is the credential layer for agentic capability — no agent credential access should occur before CBS is deployed.

### 1.4 Design Principles

- **Raw credentials never reach agents.** The invariant: no credential ever leaves CBS in readable form toward an agent session. Agents receive handle identifiers; CBS holds the credential and proxies calls.
- **Handles are scoped to operations, not resources.** "Read HUMINT corpus" and "Write analytical draft" are operations. "HUMINT system" is a resource. Handles are scoped to the specific operations the agent is authorized to perform within the resource — not to the resource itself.
- **Revocation is immediate and global.** When SCBS terminates a session, CBS revokes all handles for that session within 100ms. There is no grace period. A revoked handle cannot be used for any subsequent call.
- **API owners define capability surfaces.** The operations available on a capability surface are defined and attested by the team that owns the API, not by a central IAM team. API owners know what their API can do. Consumers declare intent. CBS maps intent to minimum viable handles automatically.
- **Unused capabilities are tracked and penalised in spend accounting.** Handles issued but never exercised appear in the session summary's unused capability report. This creates an incentive against over-requesting handles — the opposite of traditional access sprawl.

### 1.5 Explicit Out of Scope

- **Session capability bounding.** CBS issues handles within SCBS-defined envelopes; it does not enforce spend limits. SCBS is the bounding service.
- **State snapshot management.** CBS logs calls; RSS/ROLLBACK manages pre-action snapshots.
- **MCP server management.** MGS/TERMINUS manages the MCP server registry and proxies MCP-specific calls. CBS handles THEMIS platform service credentials; MGS handles MCP server credentials.

---

## 2. Core Responsibilities

### 2.1 Primary Function

CBS/BROKER holds all THEMIS platform service credentials in a secrets vault, issues time-limited, operation-scoped handles to agent sessions based on their declared intent and SCBS-validated envelope, proxies all credentialed service calls from agents through itself enforcing operation scope, revokes all handles for a session immediately on session termination or SCBS escalation, and rotates underlying credentials on schedule without disrupting active sessions. Every credentialed call is logged to MOIRAI.

### 2.2 Secondary Functions

- Capability surface registry: maintaining the registry of what operations each THEMIS service exposes for agent consumption, with API owner attestation
- Intent-to-handle mapping: automatically mapping an agent session's declared intent to the minimum viable set of handles needed
- Handle audit trail: producing per-session logs of every credentialed call, what operation was performed, and what was returned (metadata only, not content)
- Concurrent session handle isolation: ensuring handles issued to one agent session cannot be used by another session
- Credential rotation: rotating underlying credentials on a schedule without invalidating active handles (the handle continues to work; CBS uses the new credential when proxying the call)

### 2.3 What This Service Does Not Decide

CBS issues handles based on the session's declared intent mapped against the capability surface registry. Whether a specific handle combination is appropriate for a specific analytical task, whether a capability surface should be expanded, and whether an agent session that exercised a particular handle warrants review are human decisions. CBS enforces the handle grants; humans define what handle grants are appropriate.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
CapabilityHandle:
  handle_id:               uuid
  handle_ref:              str              # opaque reference given to agent; not the credential
  envelope_id:             uuid             # FK → SCBS CapabilityEnvelope
  service_name:            str
  surface_version:         str
  operations_allowed:      [str]            # specific operations; never "all"
  credential_vault_path:   str              # Vault path to the underlying credential
  issued_at:               datetime
  expires_at:              datetime         # session TTL or shorter
  revoked:                 bool
  revoked_at:              datetime | null
  revocation_reason:       str | null
  call_count:              int
  last_used:               datetime | null

CapabilitySurface:
  surface_id:              uuid
  service_name:            str
  version:                 str
  operations:              [OperationDefinition]
  owner_team:              str
  owner_attestation:       str              # attestation statement from API owner
  arb_approval_ref:        str | null       # ARB approval for surface additions
  effective_from:          datetime
  active:                  bool

OperationDefinition:
  operation_id:            uuid
  surface_id:              uuid
  name:                    str
  description:             str
  scope:                   str              # minimum permission needed
  idempotent:              bool             # whether this operation is reversible
  requires_context:        [str]            # session context prerequisites
  max_frequency:           int | null       # calls per minute; null = unlimited

CallRecord:
  call_id:                 uuid
  handle_id:               uuid
  envelope_id:             uuid
  service_name:            str
  operation:               str
  request_metadata_hash:   str              # SHA-256 of request metadata (not content)
  response_status:         int
  duration_ms:             int
  timestamp:               datetime

IntentHandleMapping:
  mapping_id:              uuid
  intent_pattern:          str              # pattern matched against declared intent
  handle_grants:           [{ service_name: str, operations: [str] }]
  maintained_by:           str              # who maintains this mapping
  version:                 str
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CapabilityHandle, CapabilitySurface, OperationDefinition, CallRecord | Session + 7 years |
| Active handle cache | Redis | Active CapabilityHandle records (hot path for validation) | Handle TTL |
| Credential store | Vault | Underlying credentials (never in PostgreSQL) | Per rotation policy |
| Event store | MOIRAI | Signed call and handle events | Indefinite |

*Critical: Underlying credentials are stored exclusively in Vault. PostgreSQL stores only the Vault path reference, never the credential value.*

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| CapabilityHandle | Inherits agent session classification | Session-compartmented |
| CallRecord | Inherits session classification | Session-compartmented; IOB access |
| CapabilitySurface | Controlled Unclassified | Platform-wide; API owners and ARB |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /handles/request
  Auth:     SCBS agent session token
  Request:  {
    envelope_id:           uuid,
    service_name:          str,
    operations_needed:     [str]
  }
  Response: {
    handle_ref:            str,           # opaque reference; agent never sees the credential
    operations_granted:    [str],
    expires_at:            datetime
  }
  SLA: p99 < 200ms

POST /calls/proxy
  Auth:     handle_ref (agent presents this, not a credential)
  Request:  {
    handle_ref:            str,
    operation:             str,
    request_payload:       {}             # operation-specific payload
  }
  Response: {
    status:                int,
    response_payload:      {},
    call_id:               uuid           # for audit trail
  }
  SLA: p99 < 500ms + underlying service latency

DELETE /handles/{handle_ref}
  Auth:     SCBS termination event | supervisor token
  Response: { revoked: bool, revoked_at: datetime }
  SLA: p99 < 100ms

GET /handles/{envelope_id}
  Auth:     analyst session token | supervisor token | IOB token
  Response: [CapabilityHandle]            # all handles for this session

GET /surfaces
  Auth:     SCBS service account | developer token
  Response: [CapabilitySurface] (operations omitted unless API owner token)

POST /surfaces
  Auth:     ARB approval token + API owner token
  Request:  CapabilitySurface (without surface_id)
  Response: { surface_id: uuid }

GET /sessions/{envelope_id}/call-log
  Auth:     supervisor token | IOB token
  Response: [CallRecord]

GET /health
  Response: {
    status, dependencies: { moirai, vault, pces, redis },
    active_handles:        int,
    calls_proxied_24h:     int,
    credential_rotation_age_max_days: int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          CBS_HANDLE_ISSUED
service_id:         "CBS"
session_id:         uuid
classification:     str
event_payload:
  handle_id:              uuid
  envelope_id:            uuid
  service_name:           str
  operations_granted:     [str]
  expires_at:             datetime
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          CBS_HANDLE_REVOKED
event_payload:
  handle_id:              uuid
  revocation_reason:      str
  calls_made:             int

EventType:          CBS_CALL_PROXIED
event_payload:
  handle_id:              uuid
  service_name:           str
  operation:              str
  response_status:        int
  duration_ms:            int

EventType:          CBS_SCOPE_VIOLATION
event_payload:
  handle_id:              uuid
  attempted_operation:    str
  allowed_operations:     [str]
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `CBS_HANDLE_ISSUED` | Handle issued to agent | MOIRAI |
| `CBS_HANDLE_REVOKED` | SCBS termination or supervisor action | MOIRAI, alert if revocation_reason=SCOPE_VIOLATION |
| `CBS_CALL_PROXIED` | Every credentialed call | MOIRAI (audit trail) |
| `CBS_SCOPE_VIOLATION` | Agent attempts operation outside handle scope | MOIRAI, SCBS escalation trigger, supervisor alert |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| SCBS/SENTINEL-CAP | `SCBS_SESSION_TERMINATED` | Revokes all handles for that envelope immediately |
| SCBS/SENTINEL-CAP | `SCBS_SPEND_EXCEEDED` | Suspends new calls on active handles; escalation pending |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| SCBS/SENTINEL-CAP | Session Bounding | Session token validation; termination signals | Sync + Async event | New handle requests blocked if SCBS unavailable |
| Vault | N/A | Credential storage and retrieval | Sync | All proxy calls fail; alert P0 |
| MOIRAI | Provenance | Signed handle and call events | Async event | Events buffered; proxy calls still serviced |
| PCES/AEGIS | Classification Enforcement | Handle scope validation against session compartment | Sync | Handle requests blocked |

### 5.2 Feeds Into

| Service | Epithet | What CBS provides | How |
|---|---|---|---|
| All agent-accessible services | All | Proxied credentialed calls | `POST /calls/proxy` |
| MGS/TERMINUS | MCP Gateway | Platform credential handles for MCP-backed operations | API |
| IOB Reporting | Oversight | Full call log audit | Audit endpoint |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Handle issuance | 50ms | 150ms | 200ms |
| Proxy call overhead | 10ms | 30ms | 50ms (add to underlying service latency) |
| Handle revocation | 20ms | 60ms | 100ms |
| Vault credential fetch | 10ms | 30ms | 50ms (cached per session) |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.9% — CBS unavailability stops all agentic credentialed operations |
| Vault availability | 99.99% (Vault is CBS's critical dependency) |
| RTO | 5 minutes |

### 6.3 Graceful Degradation

CBS fails closed. If CBS is unavailable, all credentialed agent operations are suspended. If Vault is unavailable, CBS cannot retrieve credentials and all proxy calls fail. There is no fallback that bypasses credential brokering.

---

## 7. Security Model

### 7.1 Authentication

Agents use the handle_ref (an opaque token issued by CBS) for proxy calls — not credentials, not service tokens. Handle_refs are signed by CBS and include the envelope_id and handle_id for validation. They cannot be forged.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| SCBS agent session | Handle request (within envelope scope); proxy calls | Agent session token → handle_ref |
| SCBS (termination) | `DELETE /handles/{ref}` for that session | SCBS_SESSION_TERMINATED event |
| API owner | Capability surface management | API owner token |
| ARB | Surface approval | ARB token |
| Supervisor / IOB | Call log audit; handle review | Supervisor / IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| All service credentials | `themis/cbs/services/{service_name}/credential` | Per-service rotation policy (30–90 days) |
| MOIRAI signing key | `themis/cbs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/cbs/db-credentials` | 30 days |
| Redis credentials | `themis/cbs/redis-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Scope violation (agent attempts operation outside handle scope) | Low | P1 — indicates prompt injection or agent misbehaviour | `CBS_SCOPE_VIOLATION` event | Immediate SCBS escalation; analyst notification; IAS/SCUDO adversarial flag |
| Vault latency spike (credential fetch slow) | Low | P1 — all proxy calls slow | Vault health monitoring | Credentials cached per session at handle issuance; Vault miss only on first call |
| Handle not revoked on session termination | Very low | P0 — handle orphaned after session end | Handle TTL enforcement; SCBS termination event | Handles have TTL = session TTL; even without revocation event, handle expires |

### 8.1 Known Design Risks

- **Intent-to-handle mapping is the weakest governance link.** The IntentHandleMapping — which operations are granted for which declared intents — is maintained by the AI Trust Cell. If this mapping is too broad, agents receive more capability than they need. If it is too narrow, agents cannot complete legitimate tasks. This mapping needs continuous refinement as agentic task patterns become clear in production. Resolution path: unused capability reporting from session summaries provides empirical data on which handle grants were exercised; mapping refinement is a quarterly operational task.

---

## 9. Observability

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `cbs.proxy.latency_p99` | Histogram | `> 50ms overhead for 5m` | P1 |
| `cbs.scope_violations` | Counter | `> 0` | P1 |
| `cbs.handle.revocation_latency_p99` | Histogram | `> 100ms for 5m` | P0 |
| `cbs.vault.latency_p99` | Histogram | `> 50ms for 5m` | P1 |
| `cbs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/cbs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every handle issued, every credentialed call proxied, and every scope violation attempted is permanently recorded. An oversight body can reconstruct the complete credential access history for any agent session.

---

## 11. Implementation Roadmap

### Phase 1 — Handle Issuance and Proxy (Weeks 5–16)

- CapabilityHandle and CapabilitySurface schemas
- Handle request, proxy call, and revocation endpoints
- Vault credential integration with per-session caching
- SCBS termination event → handle revocation
- MOIRAI event emission for all handle and call events
- Scope violation detection and SCBS escalation

**Phase gate criterion:** Agent sessions can request and use handles. Proxy calls log to MOIRAI. Handle revocation completes within 100ms of SCBS termination event. Scope violation produces MOIRAI event and escalation.

### Phase 2 — Capability Surface Registry, Intent Mapping, and Audit (Weeks 17–28)

- Capability surface registry with API owner attestation
- Intent-to-handle mapping implementation
- Unused capability reporting in session summary
- ARB surface approval workflow
- Supervisor and IOB call log audit endpoints
- Credential rotation without active handle disruption

**Phase gate criterion:** API owner attestation workflow operational. Intent mapping produces minimum viable handle sets for test scenarios. Credential rotation completes without active handle errors. ARB sign-off.

---

## 12. Policy Dependencies

No GC items gate CBS. The capability surface governance process (API owner attestation + ARB approval) is an AI Trust Cell operational policy, not a GC policy decision.

---

## 13. Training and Analyst Guidance

Analysts do not interact with CBS directly. CBS is an infrastructure service. Analysts see the effect of CBS through: handle grants listed in the agent session panel (what operations the agent has been granted), scope violations surfaced as session alerts, and session summary reports showing which handles were exercised vs. unused.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 2 gate review.*
