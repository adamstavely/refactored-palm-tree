# MGS — MCP Gateway Service
### TERMINUS · *"Roman god of boundaries and borders — the deity who marked the limits of territory and presided over what crossed those boundaries; the fixed point through which all passage must go"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MGS` |
| **Epithet** | `TERMINUS` |
| **Full name** | MCP Gateway Service |
| **Namespace** | `themis-interaction` |
| **Layer** | Interaction Layer |
| **Build phase** | Phase 1–2 (Weeks 1–8) |
| **Build priority** | 21 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — governs access to and responses from external MCP tool servers |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MGS/TERMINUS answers: Has this MCP server call been authorized against the session's capability envelope, is the MCP server approved for this classification context, and does the response contain adversarial content before it reaches the context window?**

### 1.2 Why This Service Exists

MCP servers introduce a governance surface that nothing else in the platform addresses. An MCP server gives an AI agent — or a RAG retrieval pipeline — access to external tools and data: live database queries, collection system interfaces, partner data feeds, web services. Each of these is simultaneously a capability and a threat vector.

Without MGS/TERMINUS, every MCP call is ungoverned: no validation that the session is authorized to call this server, no logging of what was called and what was returned, no screening of the response for adversarial injection before it reaches the context window, and no enforcement of classification boundaries between the session's security level and the server's data sensitivity profile. A classified session calling an unclassified MCP server with classified content in the query is a potential spillage. An adversarially crafted MCP response embedded with injection instructions is a prompt injection attack at the tool layer.

MGS/TERMINUS is the single controlled point through which all MCP traffic passes. Everything that enters or leaves through MCP has been validated, logged, and screened.

### 1.3 Why This Service Is Twenty-First

MGS is Phase 1-2 because MCP access is a capability that may be requested from the first day of platform operation. If MGS is not operational, MCP access must be prohibited entirely — which is more restrictive than necessary. MGS must be operational before any MCP-enabled analytical session is permitted.

### 1.4 Design Principles

- **Single controlled ingress.** All MCP server calls from ATHENA, agents, and STOA pass through MGS. Direct MCP access that bypasses MGS is a policy violation and a security incident.
- **Classification boundary enforcement is absolute.** An MCP server approved at UNCLASSIFIED cannot receive queries containing classified content. A session cleared for SECRET cannot call a server authorised only for UNCLASSIFIED. Classification boundaries are enforced before each call; there is no override path.
- **Every MCP call is a MOIRAI event.** The analytical provenance record includes what MCP tools were called, what operations were performed, and whether the response passed IAS/SCUDO screening. Tool call provenance is part of the session provenance chain.
- **Registry entries require ARB approval.** Adding an MCP server to the registry requires both a security assessment from the Research & Red Team and ARB approval. The registry is not self-service.
- **MCP server versions are tracked like model versions.** A behavioural change in an MCP server is analytically equivalent to a model version change for the capabilities that use it. MDS/KRONOS is notified of MCP server version changes.

### 1.5 Explicit Out of Scope

- **MCP server development or hosting.** MGS governs access to MCP servers; it does not build or host them.
- **THEMIS platform service credential management.** CBS/BROKER handles THEMIS service credentials. MGS handles MCP server credentials.
- **Content management of MCP server responses.** MGS screens for adversarial content and logs responses; it does not modify or filter legitimate response content.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MGS/TERMINUS proxies all MCP server calls from ATHENA, agent sessions, and STOA — validating the requesting session's capability envelope (SCBS), checking the MCP server's registry entry for classification compatibility and approval status, logging the call to MOIRAI, screening the MCP response through IAS/SCUDO, enforcing classification boundaries on the query before it is transmitted, and blocking or passing the response to the calling context.

### 2.2 Secondary Functions

- MCP Registry management: CRUD operations on the MCP server registry with ARB approval workflow
- Security assessment scheduling: routing new MCP servers to the Research & Red Team for security evaluation before registry entry
- MCP server version tracking: detecting server version changes and notifying MDS/KRONOS
- Quota and rate limiting: enforcing per-session and per-server call rate limits
- Call latency and reliability monitoring: tracking MCP server health from the gateway's perspective
- Capability surface validation for SKS: confirming that tool configurations referenced in skills are accessible

### 2.3 What This Service Does Not Decide

MGS enforces registry-based access policy. Whether a specific MCP server should be added to the registry, whether a security risk in a server warrants revocation, and whether a classification boundary exception should be granted are human decisions requiring ARB approval. MGS enforces; ARB decides.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
McpRegistryEntry:
  entry_id:                uuid
  server_name:             str              # unique stable identifier
  server_url:              str
  current_version:         str
  data_sensitivity_level:  str              # classification ceiling of data this server touches
  approved_clearances:     [str]            # clearance levels approved to call this server
  approved_compartments:   [str] | null     # null = any compartment within approved clearances
  capabilities:            [str]            # operations this server exposes
  arb_approval_ref:        str
  security_assessment_id:  uuid
  status:                  PENDING | APPROVED | DEPRECATED | REVOKED
  registered_at:           datetime
  last_version_check:      datetime

McpSecurityAssessment:
  assessment_id:           uuid
  server_name:             str
  assessed_by:             str              # Research & Red Team reference
  injection_test_passed:   bool
  classification_audit_passed: bool
  capability_boundary_notes:str
  known_risks:             [str]
  assessment_date:         datetime
  next_review_date:        datetime

McpCallRecord:
  call_id:                 uuid
  session_id:              uuid
  envelope_id:             uuid | null      # FK → SCBS (null for non-agent sessions)
  server_name:             str
  operation:               str
  request_content_hash:    str              # SHA-256 of request (classification boundary check)
  request_classification:  str              # classification of the query content
  response_screening:      PASSED | BLOCKED | FAILED
  response_content_hash:   str | null       # SHA-256 of response if passed
  ias_screening_decision_id:uuid | null
  classification_boundary_violated:bool
  duration_ms:             int
  timestamp:               datetime

McpServerVersion:
  version_id:              uuid
  server_name:             str
  version_string:          str
  detected_at:             datetime
  prior_version:           str | null
  behaviour_change_detected:bool
  mds_notified:            bool
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | McpRegistryEntry, McpSecurityAssessment, McpCallRecord, McpServerVersion | Indefinite |
| Registry cache | Redis | Active APPROVED registry entries (hot path for call validation) | 1h TTL + invalidation |
| Event store | MOIRAI | Signed call and registry events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| McpRegistryEntry | Controlled Unclassified | Platform-wide; ARB access |
| McpCallRecord | Inherits session classification | Session-compartmented; IOB access |
| McpSecurityAssessment | Controlled Unclassified | Research & Red Team and ARB |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /gateway/call
  Auth:     ATHENA service account | SCBS agent session token | STOA service account
  Request:  {
    session_id:            uuid,
    envelope_id:           uuid | null,     # null for non-agent sessions
    server_name:           str,
    operation:             str,
    request_payload:       {},
    request_classification:str              # caller declares classification of query content
  }
  Response: {
    call_id:               uuid,
    passed:                bool,
    response_payload:      {} | null,       # null if blocked
    block_reason:          str | null,
    ias_decision_id:       uuid | null
  }
  SLA: p99 < 1000ms + server latency

GET /registry
  Auth:     any service account
  Response: [{ server_name, capabilities, data_sensitivity_level, status }]
  SLA: p99 < 100ms (from Redis cache)

GET /registry/{server_name}
  Auth:     any service account
  Response: McpRegistryEntry with McpSecurityAssessment

POST /registry
  Auth:     ARB approval token
  Request:  {
    server_name:           str,
    server_url:            str,
    data_sensitivity_level:str,
    approved_clearances:   [str],
    arb_approval_ref:      str,
    security_assessment:   McpSecurityAssessment
  }
  Response: { entry_id: uuid }

DELETE /registry/{server_name}
  Auth:     ARB token (emergency revocation)
  Request:  { reason: str }
  Response: { revoked_at: datetime, active_calls_terminated: int }
  SLA: p99 < 1000ms (must terminate active calls)

GET /sessions/{session_id}/call-log
  Auth:     supervisor token | IOB token
  Response: [McpCallRecord]

POST /registry/{server_name}/validate-config
  Auth:     SKS service account
  Request:  { operations: [str] }
  Response: { valid: bool, unavailable_operations: [str] }

GET /health
  Response: {
    status, dependencies: { moirai, pces, ias, scbs, redis },
    approved_servers:      int,
    calls_proxied_24h:     int,
    block_rate_24h:        float,
    boundary_violations_24h:int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          MGS_CALL_PROXIED
service_id:         "MGS"
session_id:         uuid
classification:     str
event_payload:
  call_id:                uuid
  server_name:            str
  operation:              str
  response_screening:     str
  classification_boundary_violated:bool
  duration_ms:            int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          MGS_CALL_BLOCKED
event_payload:
  server_name:            str
  block_reason:           UNREGISTERED | CLEARANCE_MISMATCH | CLASSIFICATION_BOUNDARY | IAS_BLOCKED | RATE_LIMITED
  request_classification: str

EventType:          MGS_SERVER_VERSION_CHANGED
event_payload:
  server_name:            str
  prior_version:          str
  new_version:            str
  behaviour_change:       bool

EventType:          MGS_REGISTRY_ENTRY_REVOKED
event_payload:
  server_name:            str
  reason:                 str
  active_calls_at_revocation:int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `MGS_CALL_PROXIED` | Every proxied MCP call | MOIRAI, SCBS (spend debit) |
| `MGS_CALL_BLOCKED` | Any call blocked | MOIRAI, ATHENA (analyst notification), supervisor alert if BOUNDARY_VIOLATION |
| `MGS_SERVER_VERSION_CHANGED` | MCP server version change detected | MOIRAI, MDS/KRONOS |
| `MGS_REGISTRY_ENTRY_REVOKED` | Emergency revocation | MOIRAI, all active sessions using that server |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| SCBS/SENTINEL-CAP | `SCBS_SESSION_TERMINATED` | Terminates any in-flight calls for that session |
| IAS/SCUDO | MCP response screening result | Applied to response before returning to caller |
| ARB (admin) | Server approval | Creates or updates McpRegistryEntry |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| SCBS/SENTINEL-CAP | Session Bounding | Session capability envelope validation | Sync | Call blocked if SCBS unavailable for agent sessions; non-agent sessions proceed with session token |
| IAS/SCUDO | Adversarial Screening | MCP response screening before delivery | Sync | Response blocked if IAS unavailable; alert fires |
| PCES/AEGIS | Classification Enforcement | Session classification for boundary checking | Sync | Call blocked if PCES unavailable |
| MOIRAI | Provenance | Signed call events | Async event | Events buffered; calls still proxied |

### 5.2 Feeds Into

| Service | Epithet | What MGS provides | How |
|---|---|---|---|
| ATHENA / agents / STOA | All callers | Proxied MCP responses (screened and logged) | API |
| MDS/KRONOS | Model Drift | MCP server version change events | `MGS_SERVER_VERSION_CHANGED` event |
| SKS/DAEDALUS | Skill Registry | Tool configuration validation | `POST /registry/{server}/validate-config` |
| IOB Reporting | Oversight | Call log; boundary violation report | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Classification boundary check on every call. Session classification verified; if query content is classified above the MCP server's `data_sensitivity_level`, the call is blocked with `CLASSIFICATION_BOUNDARY` reason.
- **Failure behavior:** PCES unavailable → classification boundary check cannot be performed → call blocked for safety. Agent sessions may continue with cached capability envelope for non-classification-sensitive operations only.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Gateway call overhead (ex. server latency) | 20ms | 60ms | 100ms |
| Registry lookup (cached) | 2ms | 5ms | 50ms |
| Emergency revocation | 200ms | 500ms | 1000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.9% — MGS unavailability stops all MCP-backed operations |
| MOIRAI event durability | 99.999% |
| RTO | 5 minutes |

### 6.3 Graceful Degradation

MGS follows a fail-closed policy for classification boundary enforcement and IAS screening. If either dependency is unavailable, calls are blocked rather than proceeding without enforcement. Registry lookups fall back to cached entries when Redis is unavailable.

---

## 7. Security Model

### 7.1 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ATHENA / STOA / agents | `POST /gateway/call` for approved servers within session scope | Session token |
| ARB | Registry management; emergency revocation | ARB token |
| Research & Red Team | Security assessment management | Research team token |
| SKS | Tool configuration validation | Service account |
| Supervisor / IOB | Call log audit | Supervisor / IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MCP server credentials | `themis/mgs/servers/{server_name}/credential` | Per-server policy |
| MOIRAI signing key | `themis/mgs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/mgs/db-credentials` | 30 days |
| Redis credentials | `themis/mgs/redis-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Adversarial injection in MCP response (missed by IAS) | Low | P1 — injection reaches context window | Research & Red Team evaluation; IAS false negative rate monitoring | Conservative IAS screening; MCP server security assessment includes injection testing |
| Classification boundary violation (query content classified above server level) | Medium | P1 — potential data spillage | `MGS_CALL_BLOCKED` with BOUNDARY_VIOLATION reason | Hard block; supervisor notification; IOB reporting |
| MCP server revocation needed immediately (compromise detected) | Low | P0 — active sessions may have already called compromised server | Security incident detection | Emergency revocation endpoint; < 1s termination of active calls |

### 8.1 Known Design Risks

- **Classification boundary enforcement depends on accurate self-classification by the caller.** The caller declares the classification of their query content. MGS enforces this against the server's sensitivity level. If the caller understates classification, the boundary check may not prevent spillage. Resolution path: PGS/NOMOS screens all content and annotates its classification; MGS uses PGS classification annotation rather than caller declaration where available.
- **MCP server security assessments have a half-life.** A server that passes security assessment today may introduce a vulnerability in a future version. The version change detection and MDS/KRONOS notification is the detection mechanism; the response requires human decision (re-assess or revoke). The response time matters operationally.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `mgs.gateway.latency_p99` | Histogram | `> 100ms overhead for 5m` | P1 |
| `mgs.block_rate` | Gauge | Spike > 5x baseline | P2 |
| `mgs.boundary_violations` | Counter | `> 0` in 1h | P0 |
| `mgs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `mgs.server.version_age_max_days` | Gauge | `> 30` (server assessment may be stale) | P2 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/mgs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every MCP call — server, operation, classification boundary check result, IAS screening result — is permanently recorded. Tool call provenance is part of the session provenance chain. An oversight body can reconstruct exactly what external tools were called in any session and whether those calls were appropriately screened.

---

## 11. Implementation Roadmap

### Phase 1 — Gateway, Registry, and Classification Enforcement (Weeks 1–4)

- McpRegistryEntry schema and registry management endpoints
- `POST /gateway/call` with registry validation and classification boundary enforcement
- IAS/SCUDO response screening integration
- MOIRAI event emission for all calls
- SCBS capability envelope validation for agent sessions
- Redis registry cache

**Phase gate criterion:** All MCP calls pass through gateway. Classification boundary enforcement blocks cross-level calls in test. IAS screening applied to every response. MOIRAI events produced for all calls.

### Phase 2 — Version Tracking, Security Assessment Workflow, and Audit (Weeks 5–8)

- MCP server version detection and change notification to MDS/KRONOS
- Research & Red Team security assessment workflow
- Emergency revocation endpoint
- SKS tool configuration validation
- Supervisor and IOB audit endpoints

**Phase gate criterion:** Version change detected and MDS/KRONOS notified within 60 seconds of detection. Emergency revocation terminates active calls within 1 second. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate MGS. The ARB approval requirement for registry entries is an AI Trust Cell governance policy, not a GC policy decision.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In ATHENA session configuration, the analyst sees the list of approved MCP servers available for their classification level. If an MCP call is blocked during a session, ATHENA shows: "MCP call to [server] was blocked. Reason: [UNREGISTERED | CLEARANCE_MISMATCH | IAS_BLOCKED]. Session continues without this tool." Classification boundary violations are highlighted in red with a supervisor notification.

### 13.2 What the Analyst Should Do

Do not attempt to work around an MCP blocking by reformulating the call or using an external tool to replicate the blocked capability. Blocks are enforcement decisions. If a server you need is not in the registry, request its addition through the standard ARB process — contact the platform team with the server details and intended use case.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 1 gate review. Research & Red Team security assessment required for each MCP server before registry entry.*
