# PCES — Compartment & Classification Enforcement Service
### AEGIS · *"Greek for 'shield' — the protective aegis of Zeus"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `PCES` |
| **Epithet** | `AEGIS` |
| **Full name** | Compartment & Classification Enforcement Service |
| **Namespace** | `themis-gates` |
| **Layer** | Safety Gates |
| **Build phase** | Phase 1–2 (Weeks 1–8) |
| **Build priority** | 1 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — governs what intelligence the analyst may access |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**PCES/AEGIS answers: Is this analyst authorized to access this intelligence in this session, right now?**

### 1.2 Why This Service Exists

Without compartment enforcement, a retrieval-augmented AI system will surface any intelligence its corpus contains, regardless of the analyst's clearance, compartment access, or conflict of interest status. A junior analyst without SCI access asking about a compartmented program receives the same response as a cleared analyst. An analyst with a conflict of interest on a matter receives the same AI assistance as one without. Both failure modes are operational security failures and may constitute unauthorized disclosure.

PCES/AEGIS is the technical implementation of need-to-know in an AI-assisted analytical environment. It is not an approximation of access control — it is the authoritative enforcement point. Every request that touches source intelligence or session context passes through PCES before anything else happens.

### 1.3 Why This Service Is First

PCES is the first service in every request path because no downstream governance is meaningful if the wrong analyst is accessing the wrong intelligence. A provenance record of an unauthorized access is worse than no record — it documents the violation. The ordering is not architectural preference; it is a security requirement. PCES unavailability is the only condition under which the platform fails with zero analyst-facing degradation: if PCES is unreachable, no session proceeds.

### 1.4 Design Principles

- **Fails closed, always.** PCES unavailability means no session proceeds. There is no fallback mode that allows access without enforcement.
- **Compartment decisions are session-scoped.** Privilege grants are issued per session and expire with the session. There are no persistent privilege tokens that survive session termination.
- **Conflict of interest detection is mandatory, not optional.** CoI checks run on every session initialization. A session that cannot complete CoI detection does not open.
- **Classification log is an evidence record, not an audit log.** The classification log exists to satisfy oversight requirements, not to enable debugging.
- **PCES does not interpret policy.** PCES enforces the access control decisions encoded in the analyst's clearance and compartment records. Policy questions (should this analyst have access?) are resolved upstream by the appropriate authority, not by PCES at enforcement time.

### 1.5 Explicit Out of Scope

- **Granting or revoking clearances.** PCES enforces clearances it is given; it does not manage the clearance system.
- **Determining whether intelligence should be classified at a given level.** PCES enforces existing classification; it does not apply original classification.
- **Coalition partner handling constraints.** CLS/PROTEUS extends PCES for coalition intelligence handling; PCES provides the base compartment enforcement layer.
- **Policy interpretation for novel compartment combinations.** PCES applies configured rules; novel policy questions escalate to the appropriate authority.

---

## 2. Core Responsibilities

### 2.1 Primary Function

PCES/AEGIS validates every analytical session initialization and every subsequent retrieval or inference request against the analyst's current clearance, compartment access, special controls, conflict of interest status, and collection authority — and either grants a scoped session privilege set, partially restricts it, or denies session creation. It generates a signed privilege annotation for each granted session that downstream services use to scope their own behavior, and it maintains a classification log of all content decisions for oversight review.

### 2.2 Secondary Functions

- Conflict of interest detection: checks analyst identity against matter relationships and flags or blocks where CoI policy applies
- Collection authority validation: confirms the analyst holds the collection authority required for the session's requirement type
- Classification log generation: records all content classification decisions in a queryable, oversight-submissible format
- Session privilege refresh: re-validates privileges mid-session when session scope changes (e.g., analyst accesses a new corpus segment)
- Scope restriction enforcement: when partial access is granted (PARTIAL decision), provides downstream services with the specific compartments that are accessible

### 2.3 What This Service Does Not Decide

PCES enforces the access control state as it finds it. Whether an analyst should have access to a compartment is a decision owned by the appropriate security authority. Whether a conflict of interest should be waived is a decision owned by the relevant approving official. PCES records decisions and enforces them; it does not make them.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
SessionPrivilege:
  privilege_id:            uuid
  session_id:              uuid               # FK → MOIRAI session record
  analyst_id:              str
  clearance_level:         str                # e.g., TS/SCI
  granted_compartments:    [str]              # compartments accessible in this session
  denied_compartments:     [str]              # compartments requested but denied
  special_controls:        [str]              # NOFORN, ORCON, PROPIN, HCS, etc.
  collection_authorities:  [str]              # authorized collection types
  coi_flags:               [str]              # active conflict of interest flags
  scope_restriction:       str | null         # free-text restriction note
  decision:                GRANTED | PARTIAL | DENIED
  valid_from:              datetime
  valid_until:             datetime           # session expiry
  issued_by:               str               # PCES service instance

CompartmentDecision:
  decision_id:             uuid
  session_id:              uuid
  turn_id:                 uuid | null        # null for session-level decisions
  request_type:            RETRIEVAL | INFERENCE | EXPORT | SESSION_INIT
  requested_compartments:  [str]
  analyst_clearance:       [str]
  decision:                GRANTED | DENIED | PARTIAL
  granted_compartments:    [str]
  denial_reasons:          [str]
  coi_flags_active:        [str]
  policy_version:          str               # PGS/NOMOS policy version at decision time
  timestamp:               datetime

ClassificationLog:
  log_id:                  uuid
  session_id:              uuid
  content_hash:            str               # SHA-256 of content being classified
  content_type:            SOURCE | RESPONSE | EXPORT
  assigned_level:          str
  assigned_compartments:   [str]
  special_controls:        [str]
  basis:                   str               # classification determination basis
  analyst_id:              str
  timestamp:               datetime

ConflictOfInterest:
  coi_id:                  uuid
  analyst_id:              str
  matter_id:               str
  coi_type:                str               # FINANCIAL | PERSONAL | PRIOR_INVOLVEMENT | etc.
  detected_at:             datetime
  status:                  ACTIVE | WAIVED | EXPIRED
  waiver_authority:        str | null
  waiver_expires:          datetime | null
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CompartmentDecision, ClassificationLog, ConflictOfInterest records | Session + 7 years |
| Session cache | Redis | Active SessionPrivilege records (hot path for mid-session validation) | Session TTL |
| Event store | MOIRAI (signed events) | Immutable record of all gate decisions | Indefinite |
| CoI registry | PostgreSQL | ConflictOfInterest table, separately access-controlled | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| SessionPrivilege | Inherits session classification | Contains clearance metadata — itself controlled |
| CompartmentDecision | Classification of session | Accessible only to IOB and security authority |
| ClassificationLog | Classification of content logged | Full controls inherited |
| ConflictOfInterest | Controlled Unclassified | Separate access — not available to general analysts |

### 3.4 Retention and Purge Policy

Classification logs and compartment decisions are retained for the session lifetime plus seven years. ConflictOfInterest records are retained indefinitely. Session privilege records in Redis expire with the session TTL. MOIRAI-signed PCES events are retained indefinitely and cannot be purged.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /session/initialize
  Auth:     analyst credential (pre-authenticated)
  Request:  {
    analyst_id:           str,
    requirement_id:       str,
    requested_compartments: [str],
    session_type:         RESEARCH | ANALYSIS | REVIEW
  }
  Response: {
    session_id:           uuid,
    privilege:            SessionPrivilege,
    decision:             GRANTED | PARTIAL | DENIED,
    denial_reasons:       [str] | null,
    coi_flags:            [str]
  }
  SLA: p99 < 500ms

POST /request/validate
  Auth:     session token (active SessionPrivilege)
  Request:  {
    session_id:           uuid,
    request_type:         RETRIEVAL | INFERENCE | EXPORT,
    compartments_touched: [str],
    content_hashes:       [str]
  }
  Response: {
    decision:             GRANTED | DENIED | PARTIAL,
    granted_compartments: [str],
    denial_reasons:       [str] | null
  }
  SLA: p99 < 100ms  # called on every retrieval chunk — must be fast

GET /session/{session_id}/privilege
  Auth:     session token | supervisor token | IOB token
  Response: SessionPrivilege

GET /classification-log/{session_id}
  Auth:     IOB token | security authority token
  Response: [ClassificationLog]

GET /health
  Response: { status, dependencies: { moirai, coi_registry }, last_event_hash }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          PCES_SESSION_GRANTED
service_id:         "PCES"
session_id:         uuid
classification:     str
event_payload:
  analyst_id:             str
  decision:               GRANTED | PARTIAL | DENIED
  granted_compartments:   [str]
  coi_flags:              [str]
  policy_version:         str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          PCES_REQUEST_DENIED
event_payload:
  request_type:           str
  compartments_requested: [str]
  denial_reasons:         [str]
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `PCES_SESSION_GRANTED` | Session initialization with any decision | MOIRAI, PGS/NOMOS, all downstream services |
| `PCES_SESSION_DENIED` | Session denied outright | MOIRAI, security authority alerting |
| `PCES_REQUEST_DENIED` | Mid-session request denied | MOIRAI, ATHENA (surfaces to analyst) |
| `PCES_COI_FLAGGED` | Active CoI detected on session init | MOIRAI, supervisor notification |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| External clearance system | `CLEARANCE_UPDATED` | Invalidates cached SessionPrivilege for affected analyst |
| External CoI registry | `COI_STATUS_CHANGED` | Updates ConflictOfInterest record; may terminate active sessions |
| MDS/KRONOS | `MODEL_VERSION_CHANGED` | No direct action; model version logged in subsequent decisions |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed event emission; session record creation | Async event | Events buffered locally; replayed on recovery. Session creation proceeds. |
| External clearance system | N/A | Source of analyst clearance and compartment data | Sync | Session denied with CLEARANCE_UNAVAILABLE; no fallback access |
| External CoI registry | N/A | Source of conflict of interest data | Sync | Session denied with COI_CHECK_UNAVAILABLE; no fallback access |

### 5.2 Feeds Into

| Service | Epithet | What PCES provides | How |
|---|---|---|---|
| ALL services | All | SessionPrivilege — every service validates the session token against PCES before returning data | Sync query on every request |
| PGS/NOMOS | Analytic Standards | Policy version context; session privilege scope for policy rule application | MOIRAI event |
| CLS/PROTEUS | Coalition Liaison | Base compartment enforcement that CLS extends for coalition handling | API extension |
| MOIRAI | Provenance | Session initialization events as the root of each session's event chain | MOIRAI event |

### 5.3 PCES/AEGIS Integration

PCES is the enforcement point. Every other service's integration map includes PCES because every other service must validate session tokens against PCES before returning data. PCES does not integrate with itself.

The integration pattern for all other services:
- **Enforcement point:** Validate session token on every data-returning endpoint
- **Failure behavior:** Return `503 PCES_UNAVAILABLE` — never degrade access control
- **Cache policy:** SessionPrivilege may be cached in Redis with a TTL not exceeding the session TTL. Cached privilege must be invalidated on `PCES_SESSION_REVOKED` events.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Session initialization | 100ms | 300ms | 500ms |
| Per-request validation (`/request/validate`) | 5ms | 20ms | 100ms |
| MOIRAI event emission | 50ms | 150ms | 300ms |

The per-request validation target is aggressive because it is called on every retrieval chunk. A 100ms p99 on this endpoint adds up to seconds of latency per session if retrieval returns many chunks. Redis caching of the active SessionPrivilege is required to meet this target.

### 6.2 Throughput

| Metric | Target |
|---|---|
| Session initializations/second | 10 |
| Per-request validations/second | 500 (50 concurrent analysts × 10 validation calls/turn) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.9% — PCES unavailability stops all platform activity |
| MOIRAI event durability | 99.999% |
| RTO | 5 minutes |
| RPO | 0 minutes (no session decisions may be lost) |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Events buffered; session decisions still enforced from PostgreSQL. | None during outage; provenance gap logged |
| External clearance system | Session denied with explicit `CLEARANCE_UNAVAILABLE` reason | Analysts cannot open new sessions. Active sessions continue (privilege already granted). |
| External CoI registry | Session denied with `COI_CHECK_UNAVAILABLE` | Same as above |
| Redis session cache | Falls back to PostgreSQL lookup (higher latency) | Per-request validation latency increases; alert fires |

---

## 7. Security Model

### 7.1 Authentication

Session initialization requires analyst credentials validated by the platform identity provider (pre-PCES — PCES is not the identity provider). Mid-session validation requires a session token issued by PCES at session initialization. All other endpoints require supervisor, IOB, or security authority tokens.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst | Session initialization; session token for mid-session validation | Pre-authenticated credential |
| Service (calling /request/validate) | Validation of active session tokens only | Service account |
| Supervisor | `/session/{id}/privilege` for their team | Supervisor role token |
| IOB / Security authority | Full read including classification log, CoI records | IOB token |
| Agent session (SCBS-bounded) | No direct PCES endpoints — agents inherit the analyst session privilege | Analyst session token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/pces/signing-key` | 90 days |
| PostgreSQL credentials | `themis/pces/db-credentials` | 30 days |
| Redis credentials | `themis/pces/redis-credentials` | 30 days |
| Clearance system API key | `themis/pces/clearance-api-key` | 90 days |
| CoI registry API key | `themis/pces/coi-api-key` | 90 days |

### 7.4 Adversarial Threat Surface

The primary threat to PCES is privilege escalation via session token forgery or replay. All session tokens are cryptographically signed (PCES signing key in Vault), include a session expiry, and are invalidated on session termination. Replay attacks are mitigated by session TTL and single-use validation tokens for export operations. CoI bypass is the secondary threat — an analyst with an active CoI who attempts to open a session while the CoI registry is degraded is denied, not allowed through. No degraded-mode access control.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Clearance system unavailable | Medium | P0 — no new sessions open | `/health` dependency check | Alert immediately; operations leadership notified; active sessions unaffected |
| Redis cache eviction during session | Low | P1 — validation latency spike | p99 latency > 100ms sustained | Automatic fallback to PostgreSQL; Redis capacity alert |
| MOIRAI event queue backlog | Low | P2 — provenance gap | Queue depth > 500 events | Auto-scale consumer; alert |
| CoI record staleness (cached out of sync) | Low | P1 — incorrect CoI enforcement | CoI registry sync lag monitoring | TTL-based cache invalidation; max cache age 60 seconds |

### 8.1 Known Design Risks

- **External dependency on clearance system is a single point of failure for session creation.** If the clearance system is down, no new analytical sessions can open. Mitigation: the clearance system must have its own HA architecture and SLA that exceeds PCES's. This is an external dependency — PCES cannot solve it.
- **Per-request validation at 500 rps is latency-sensitive.** At scale, the Redis cache must be warmed for every active session. Cache misses on the validation hot path will degrade platform latency significantly. Load testing must validate the cache hit rate under realistic concurrency.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `pces.session.init.latency_p99` | Histogram | `> 500ms for 5m` | P1 |
| `pces.request.validate.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `pces.session.denied_rate` | Gauge | `> 20% over 10m` (baseline spike) | P2 |
| `pces.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `pces.clearance_system.availability` | Gauge | `< 100%` | P0 |
| `pces.coi_registry.availability` | Gauge | `< 100%` | P0 |
| `pces.cache.miss_rate` | Gauge | `> 5% on /request/validate` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status:              "healthy" | "degraded" | "unavailable",
  dependencies: {
    moirai:            "healthy" | "unavailable",
    clearance_system:  "healthy" | "unavailable",
    coi_registry:      "healthy" | "unavailable",
    redis:             "healthy" | "degraded" | "unavailable"
  },
  moirai_sync:         boolean,
  last_event_hash:     string,
  active_sessions:     int,
  cache_hit_rate:      float
}
```

### 9.3 Log Schema

```json
{
  "timestamp":        "ISO-8601",
  "service":          "PCES/AEGIS",
  "level":            "INFO | WARN | ERROR",
  "event":            "SESSION_GRANTED | SESSION_DENIED | REQUEST_DENIED | COI_FLAGGED",
  "session_id":       "uuid | null",
  "analyst_id":       "redacted-in-non-IOB-logs",
  "decision":         "GRANTED | PARTIAL | DENIED",
  "classification":   "string",
  "duration_ms":      0,
  "fields": {
    "denial_reasons": [],
    "coi_flags":      [],
    "compartments_granted": []
  }
}
```

*Note: analyst_id is redacted in standard logs. Full analyst identity available only in IOB-access audit logs.*

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/pces/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — PCES events are the root of each session's event chain
- **Prev_event_hash source:** For session initialization, prev_event_hash is the hash of the prior session's final event for this analyst (providing cross-session continuity). For mid-session events, prev_event_hash is the prior PCES event in this session.

### 10.2 What This Service Attests

The MOIRAI record for PCES proves that at session initialization, a specific analyst with a specific clearance was granted or denied access to a specific set of compartments, and that the access decision has not been altered since it was recorded. An oversight body can query PCES events to reconstruct the complete access history for any analyst or any session.

### 10.3 What This Service Cannot Prove

The record proves the decision was made and recorded accurately. It does not prove the analyst's clearance records were accurate at the time the decision was made — if the upstream clearance system contained inaccurate data, PCES enforced that inaccuracy. It does not prove the analyst only used the access they were granted.

---

## 11. Implementation Roadmap

### Phase 1 — Core Enforcement (Weeks 1–4)

- Session initialization endpoint with clearance validation
- Per-request validation endpoint with Redis session cache
- Basic compartment decision logic (exact-match clearance and compartment)
- MOIRAI event emission: `PCES_SESSION_GRANTED`, `PCES_SESSION_DENIED`
- PostgreSQL schema: CompartmentDecision, SessionPrivilege
- Health check endpoint

**Phase gate criterion:** Every session initialization produces a MOIRAI event with a signed compartment decision. Per-request validation p99 < 100ms under 50 concurrent analysts. No session proceeds without PCES validation.

### Phase 2 — CoI, Classification Log, and Scope Restriction (Weeks 5–8)

- Conflict of interest detection integrated with CoI registry
- `PCES_COI_FLAGGED` event emission
- Classification log generation for all content decisions
- PARTIAL decision support with scope restriction enforcement
- Special controls enforcement (NOFORN, ORCON, PROPIN)
- Collection authority validation
- IOB query endpoints: classification log, privilege history

**Phase gate criterion:** CoI detection produces MOIRAI events. Classification log queryable by IOB token. PARTIAL decisions enforce scope restriction in downstream retrieval. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-3 | Query-type authorization taxonomy | Required before Phase 1 classifier training |
| GC-4 | Within-requirement role-tier access policy | Required before Phase 1 tier deployment |

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

If PCES denies a session, the analyst sees a session initialization failure with a plain-language reason (e.g., "You do not have access to the compartments required for this requirement" or "A conflict of interest has been flagged for this matter. Contact your supervisor."). If PCES grants partial access, the analyst sees a banner in ATHENA: "Your access to this requirement is restricted. Some intelligence may not be retrievable in this session."

### 13.2 What the Analyst Should Do

A denied session means the analyst should not proceed. They should contact their supervisor or security officer. They should not attempt to work around the restriction using external tools — doing so may constitute unauthorized access.

A partial access flag means the analyst should note the restriction in their analytical record. Any assessment produced under restricted access should be caveated accordingly.

### 13.3 What the Signal Does Not Mean

A CoI flag does not mean the analyst has done anything wrong. It means the system has detected a relationship that policy requires to be reviewed before access is granted. The analyst should contact their supervisor — the CoI may be waivable.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Compartment enumeration latency from clearance system.** The clearance system API must return the full compartment list for an analyst within the session initialization SLA. If the clearance system is slow, session initialization will exceed the 500ms p99 target. Resolution path: negotiate SLA with clearance system owners before Phase 1 begins.

### 14.2 Operational Assumptions

- **Assumption 1: A machine-readable clearance system API exists.** PCES requires a programmatic interface to the clearance records system. If no such API exists, it must be built before PCES Phase 1. This is a pre-Phase 1 dependency, not an engineering task for the AI Trust Cell.
- **Assumption 2: The CoI registry is queryable in real time.** If CoI records exist only in a manual or periodic-batch system, PCES cannot enforce CoI at session initialization. Resolution path: confirm registry API availability before Phase 1.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

---

## Appendix D: Red Team Findings

*Pending red team evaluation — scheduled for Phase 1 gate review.*
