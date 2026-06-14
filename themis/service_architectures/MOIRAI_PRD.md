# Provenance — AI Content Provenance Service
### MOIRAI · *"Greek for the Fates — those who weave the thread of destiny, recording what was and cannot be undone"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MOIRAI` |
| **Epithet** | `MOIRAI` |
| **Full name** | AI Content Provenance Service |
| **Namespace** | `themis-core` |
| **Layer** | Core Infrastructure |
| **Build phase** | Phase 3–4 (Weeks 9–28) |
| **Build priority** | 2 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Cross-cutting — the cryptographic backbone for all three axes |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MOIRAI answers: What happened in this analytical session, in what order, and can we prove that record has not been altered?**

### 1.2 Why This Service Exists

Every governance claim THEMIS makes rests on MOIRAI. The Origin axis claims to prove where AI content came from — MOIRAI holds the provenance events. The Currency axis claims to track when sources became stale — MOIRAI holds the invalidation events. The Trust axis claims to measure calibration over time — MOIRAI holds the correction and calibration events. Without a tamper-evident event chain, every governance claim is an assertion. With it, every governance claim is a verifiable fact.

The distinction between an audit log and an evidence record is cryptographic. An audit log records what happened; a determined insider with database access can alter it. MOIRAI's hash chain means that any retroactive modification breaks the chain from the alteration point forward and is detectable without access to the original. This is the difference between "we have records" and "our records can be independently verified."

### 1.3 Why This Service Is Second

MOIRAI is the first Core Infrastructure service because every other service writes to it. A service that emits MOIRAI events before MOIRAI exists produces orphaned events that cannot be incorporated into the chain. The correct sequence: PCES/AEGIS gates sessions (Phase 1-2), MOIRAI captures what those sessions contain (Phase 3-4), and all subsequent services write into the chain MOIRAI maintains.

### 1.4 Design Principles

- **Append-only, always.** No record is ever modified or deleted. The provenance chain is a write-once structure. Corrections do not overwrite — they add new events referencing the corrected event.
- **The chain proves non-modification, not accuracy.** MOIRAI proves the record has not been altered since writing. It does not prove the record accurately reflects what actually occurred at write time. This limitation must be stated explicitly in all oversight communications.
- **Per-service signing, not per-platform.** Each service has its own Vault signing key. A compromised signing key affects only that service's events — not the full chain.
- **The provenance graph is the analytical artifact.** The event ledger is the evidence; the Neo4j graph is the analytical interface. The ledger is authoritative; the graph is derived.
- **External anchoring is not optional.** RFC 3161 timestamp anchoring every 24 hours provides an independently verifiable reference point. Without external anchoring, the chain can be reconstructed by a party with control of both the signing keys and the event store.

### 1.5 Explicit Out of Scope

- **Real-time event streaming to consumers.** MOIRAI is a write target and a query target. Consumers that need real-time event notification use Kafka; MOIRAI does not push to consumers.
- **Business logic.** MOIRAI stores what other services write. It does not interpret, classify, or act on event content.
- **Access control decisions.** MOIRAI records access decisions made by PCES. It does not enforce them.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MOIRAI maintains a cryptographically tamper-evident event ledger for all analytically significant events across the THEMIS platform, and a provenance graph that makes those events queryable by session, turn, claim, source, analyst, and relationship. Every event is signed with a per-service Vault key, includes the SHA-256 hash of the prior event in its chain, and is anchored to an external RFC 3161 timestamp authority every 24 hours. Any retroactive modification of any event breaks the hash chain from that point forward and is detectable without access to the original event store.

### 2.2 Secondary Functions

- Provenance graph maintenance: Neo4j graph of sessions, turns, claims, sources, analysts and their relationships
- Cross-event querying: reconstructing the full provenance for any claim, session, or source
- Oversight audit package generation: producing a cryptographic chain audit certificate suitable for submission to oversight bodies
- Event chain validation: endpoint for verifying chain integrity across a date range or session set
- Blast radius analysis: given a source invalidation, traversing the graph to identify all dependent claims and sessions

### 2.3 What This Service Does Not Decide

MOIRAI records what other services determine. Whether an access decision was correct, whether a claim was accurate, whether an analyst's behavior was appropriate — none of these are MOIRAI's determination. MOIRAI is the record; the determination belongs to the service that wrote the event or the human authority reviewing the record.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ProvenanceEvent:
  event_id:              uuid
  event_type:            str              # e.g., PCES_SESSION_GRANTED, TCS_CALIBRATION_UPDATE
  service_id:            str              # issuing service code
  session_id:            uuid
  turn_id:               uuid | null      # null for session-level events
  classification:        str              # classification floor of event content
  payload:               {}               # service-specific event data
  prev_event_hash:       str              # SHA-256 of prior event in this session's stream
  event_hash:            str              # SHA-256 of this event (computed at write)
  signature:             str              # HMAC-SHA256 with per-service Vault key
  tsa_anchor_ref:        str | null       # RFC 3161 anchor ID (set every 24h)
  timestamp:             datetime

TsaAnchor:
  anchor_id:             uuid
  anchor_timestamp:      datetime         # time of RFC 3161 request
  covered_event_ids:     [uuid]           # all events anchored in this batch
  tsa_token:             str              # RFC 3161 token from timestamp authority
  tsa_authority:         str              # identity of timestamp authority used

# Neo4j Graph Nodes (managed by MOIRAI)
Session:
  session_id:            uuid
  analyst_id:            str              # hashed in non-IOB contexts
  start_time:            datetime
  end_time:              datetime | null
  classification:        str
  requirement_id:        str | null

Turn:
  turn_id:               uuid
  session_id:            uuid
  sequence_number:       int
  timestamp:             datetime
  model_version:         str

Claim:
  claim_id:              uuid
  turn_id:               uuid
  text_hash:             str              # SHA-256 of claim text (not stored plain)
  claim_type:            str
  source_type:           GRND | PARAM | SYNTH | TRANSCRIPT | VIDEO | AUDIO | IMAGE | OCR | MEMORY

Source:
  source_id:             uuid
  chunk_id:              str              # retrieval chunk identifier
  source_ref:            str             # source document reference
  ingested_at:           datetime
  validity_score:        float           # TVS score at time of use

Analyst:
  analyst_id_hash:       str             # SHA-256 — not stored plain in graph
  calibration_domain:    [str]           # for graph traversal; no PII
```

**Neo4j Relationships:**

| Relationship | From → To | Meaning |
|---|---|---|
| `HAS_TURN` | Session → Turn | Session contains this turn |
| `CONTAINS_CLAIM` | Turn → Claim | Turn produced this claim |
| `DERIVED_FROM` | Claim → Source | Claim cites this source |
| `CONTRIBUTED_TO` | Analyst → Session | Analyst participated in this session |
| `SUPERSEDES` | Claim → Claim | Correction event; new claim supersedes prior |
| `INVALIDATED_BY` | Source → ProvenanceEvent | Source was invalidated by this event |
| `BLAST_RADIUS` | ProvenanceEvent → [Claim] | Invalidation event affects these claims |

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Event ledger | PostgreSQL (append-only) | Authoritative ProvenanceEvent record | Indefinite |
| Provenance graph | Neo4j | Queryable relationship graph | Indefinite |
| Event write buffer | Kafka | High-throughput event ingestion; async write to ledger and graph | 72h buffer |
| TSA anchor store | PostgreSQL | TsaAnchor records and tokens | Indefinite |
| Chain validation cache | Redis | Pre-computed chain integrity results for recent sessions | 24h TTL |

The PostgreSQL event ledger is the authoritative store. The Neo4j graph is derived from the ledger and can be rebuilt from ledger events. If the graph and ledger diverge, the ledger is authoritative.

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ProvenanceEvent payload | Inherits from event classification field | Queried only with matching PCES clearance |
| Neo4j graph nodes | Inherits session classification | Cross-session graph traversal requires IOB token |
| Analyst identity in graph | Hashed | Plain identity available only to IOB with explicit authority |
| TSA tokens | Unclassified (external) | Stored without classification restriction |

### 3.4 Retention and Purge Policy

No ProvenanceEvent is ever deleted or modified. The event ledger is append-only by database constraint. Purge of any event would break the hash chain from that point forward, which is itself evidence of tampering. Requests to purge individual events are escalated to IOB — the appropriate response is not purge but a new event noting the correction or context, referencing the original event.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /events
  Auth:     service account (per-service)
  Request:  ProvenanceEvent (without event_hash — MOIRAI computes it)
  Response: {
    event_id:         uuid,
    event_hash:       str,       # SHA-256 computed by MOIRAI
    chain_position:   int,       # position in session's event chain
    tsa_pending:      bool       # true if this event will be in next TSA anchor batch
  }
  SLA: p99 < 300ms

GET /events/{event_id}
  Auth:     session token | supervisor token | IOB token
  Response: ProvenanceEvent (payload redacted based on caller clearance)

GET /session/{session_id}/chain
  Auth:     supervisor token | IOB token
  Request:  { verify_integrity: bool }
  Response: {
    events:           [ProvenanceEvent],
    chain_valid:      bool,
    broken_at:        uuid | null,    # event_id where chain breaks, if broken
    tsa_coverage:     float           # proportion of events covered by TSA anchors
  }

GET /claim/{claim_id}/provenance
  Auth:     session token (with PCES clearance for that claim's session)
  Response: {
    claim:            Claim,
    turn:             Turn,
    session:          Session,
    sources:          [Source],
    events:           [ProvenanceEvent]  # events referencing this claim
  }

POST /blast-radius
  Auth:     TVS service account | IOB token
  Request:  { source_id: uuid, invalidation_event_id: uuid }
  Response: {
    affected_claims:  [uuid],
    affected_sessions:[uuid],
    affected_analysts:[str]     # hashed unless IOB token
  }

GET /audit/certificate/{session_id}
  Auth:     IOB token
  Response: {
    session_id:       uuid,
    certificate:      str,     # oversight-submissible chain audit certificate
    chain_valid:      bool,
    tsa_coverage:     float,
    event_count:      int,
    generated_at:     datetime
  }

GET /health
  Response: {
    status, dependencies: { postgresql, neo4j, kafka, tsa_authority },
    ledger_lag_ms: int,     # Kafka → PostgreSQL write lag
    graph_lag_ms: int,      # Kafka → Neo4j write lag
    last_tsa_anchor: datetime
  }
```

### 4.2 MOIRAI Event Schema

MOIRAI does not emit events to itself — it is the event store. MOIRAI emits a single internal event for TSA anchoring:

```yaml
EventType:          MOIRAI_TSA_ANCHOR_CREATED
service_id:         "MOIRAI"
session_id:         null           # anchor is platform-level, not session-level
payload:
  anchor_id:              uuid
  covered_event_count:    int
  tsa_authority:          str
  anchor_timestamp:       datetime
prev_event_hash:    str            # hash of prior anchor event
signature:          str
timestamp:          datetime
```

### 4.3 Consumed Events

MOIRAI consumes events from all other services via Kafka. The Kafka topic structure:

| Topic | Producer services | Consumer |
|---|---|---|
| `themis.events.session` | PCES, PGS | MOIRAI ledger writer |
| `themis.events.quality` | FGTS, TVS, RQS, CVS, IAS, MAS, MDS, UCS | MOIRAI ledger writer |
| `themis.events.calibration` | TCS, ERAS | MOIRAI ledger writer |
| `themis.events.agent` | SCBS, CBS, RSS | MOIRAI ledger writer |
| `themis.events.interaction` | PRS, SKS, MGS | MOIRAI ledger writer |
| `themis.events.intelligence` | OFS, all intel layer services | MOIRAI ledger writer |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| Kafka | N/A | Event ingestion from all services | Async stream | Events buffered in Kafka; ledger and graph lag increases. No events lost. |
| External TSA | N/A | RFC 3161 timestamp anchoring | Sync (24h batch) | Anchoring deferred; alert fired; events still written to ledger |
| Vault | N/A | Per-service signing key validation (MOIRAI validates other services' signatures) | Sync | Signature validation unavailable; events accepted unverified with flag; alert P0 |

### 5.2 Feeds Into

Every service in the platform reads from MOIRAI for audit and provenance queries. The primary consumers:

| Service | Epithet | What MOIRAI provides | How |
|---|---|---|---|
| ALL services | All | Event chain integrity for audit queries | API query |
| ERAS/LOGOS | Reasoning Audit | Turn-level provenance graph for reasoning record linkage | API query |
| TVS/KAIROS | Temporal Validity | Blast radius traversal for source invalidation propagation | `/blast-radius` endpoint |
| OFS/NEMESIS | Outcome Feedback | Session and claim provenance for outcome linkage | API query |
| IOB Reporting | Oversight | Chain audit certificates, session provenance | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Every data-returning MOIRAI endpoint validates the session token against PCES before returning payload content. Event metadata (event_id, timestamp, event_type) is less restricted; payload content is PCES-gated.
- **Compartment inheritance:** ProvenanceEvent payloads inherit the classification specified in the `classification` field, which is set by the writing service based on the session classification at write time.
- **Failure behavior:** PCES unavailability → payload content returns `PCES_UNAVAILABLE`; event metadata still returned. Write path is not PCES-gated (events are written by services, not analysts).

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Event write (`POST /events`) | 50ms | 150ms | 300ms |
| Event read (`GET /events/{id}`) | 10ms | 30ms | 100ms |
| Session chain (`GET /session/{id}/chain`) | 200ms | 800ms | 2000ms |
| Blast radius (`POST /blast-radius`) | 500ms | 2000ms | 5000ms |
| Audit certificate | 1000ms | 3000ms | 10000ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Events written/second | 500 (50 concurrent analysts × 10 events/turn × 1 turn/second peak) |
| Event reads/second | 200 |
| Chain integrity validations/hour | 100 (audit batch operations) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.9% — MOIRAI unavailability creates provenance gaps in all active sessions |
| Event durability | 99.9999% — events in Kafka buffer survive MOIRAI outages |
| RTO | 10 minutes |
| RPO | 0 minutes — Kafka buffer ensures no events lost |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| PostgreSQL (ledger) | Events accumulate in Kafka buffer. Reads unavailable. Write path queued. | No audit or provenance queries available |
| Neo4j (graph) | Ledger writes continue. Graph queries unavailable. Graph rebuilt from ledger on recovery. | Provenance graph queries unavailable; audit certificates unavailable |
| TSA authority | Anchoring deferred; events written to ledger without anchor. Alert P1. | No analyst-facing impact; audit certificate coverage degrades |
| Kafka | Events written synchronously to PostgreSQL (higher latency, lower throughput). Alert P0. | Event write latency increases significantly |
| Vault (signature validation) | Events accepted with signature validation flagged as unavailable. Alert P0. | No analyst-facing impact; chain integrity claims degraded |

---

## 7. Security Model

### 7.1 Authentication

Service accounts authenticate with per-service credentials issued by Vault. Each service has a unique signing key pair. MOIRAI validates that the signature on an incoming event matches the signing key registered for the service_id in that event. Service accounts cannot forge events from other services.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Any THEMIS service | `POST /events` — write only | Service account credential |
| Analyst session | `GET /claim/{id}/provenance` for accessible sessions | Session token (PCES-gated) |
| Supervisor | Session chain queries for their team | Supervisor token |
| IOB / Oversight | Full read including audit certificates | IOB token |
| TVS/KAIROS | `/blast-radius` | TVS service account |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/moirai/signing-key` | 90 days |
| PostgreSQL credentials | `themis/moirai/db-credentials` | 30 days |
| Neo4j credentials | `themis/moirai/neo4j-credentials` | 30 days |
| Kafka credentials | `themis/moirai/kafka-credentials` | 30 days |
| Per-service signing keys (all services) | `themis/{service}/signing-key` | 90 days (rotated by MOIRAI key manager) |

### 7.4 Adversarial Threat Surface

The primary threat to MOIRAI is an insider attacker with simultaneous access to the signing keys and the PostgreSQL event ledger, who could overwrite events and recompute the hash chain. Mitigations: Vault access to signing keys is logged and audited; key access requires dual authorization for production keys; the RFC 3161 TSA anchor is external and cannot be forged retroactively. Secondary threat: Kafka topic tampering. Mitigation: Kafka topic ACLs restrict write access to registered service accounts only.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Hash chain break detected | Very low | P0 — security incident | `/session/chain` validation endpoint | Incident response: isolate affected sessions; security authority notification |
| Neo4j graph drift from ledger | Low | P1 — graph queries return stale results | Periodic ledger-to-graph reconciliation | Automated reconciliation job; graph rebuilt from ledger |
| TSA anchor failure (authority unavailable) | Low | P2 — audit certificate coverage degrades | TSA anchor age monitoring | Fallback to secondary TSA authority; alert |
| Kafka consumer lag (ledger write delay) | Medium | P2 — events written with delay | Consumer lag monitoring | Auto-scale Kafka consumers; alert at lag > 10s |

### 8.1 Known Design Risks

- **Hash chain proves non-modification, not accuracy.** A service that writes a correctly-signed but factually incorrect event (e.g., PCES records a grant decision that was not actually made) produces an unbreakable chain that proves the false record was not subsequently altered. This is a design constraint, not a fixable bug. Mitigations: PCES decision logic is independently audited; dual-write validation for high-stakes events.
- **Neo4j at scale.** The provenance graph grows indefinitely. Neo4j performance on large graphs with complex traversal queries (blast radius, multi-hop provenance) will degrade over time without active management. Resolution path: query performance benchmarking at 1M, 10M, and 100M nodes; partitioning strategy for older session graphs.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `moirai.event.write.latency_p99` | Histogram | `> 300ms for 5m` | P1 |
| `moirai.kafka.consumer.lag` | Gauge | `> 10s sustained` | P0 |
| `moirai.chain.break_count` | Counter | `> 0` | P0 |
| `moirai.tsa.anchor_age_hours` | Gauge | `> 26h` (> 2h past schedule) | P1 |
| `moirai.neo4j.query.latency_p99` | Histogram | `> 2000ms for 5m` | P1 |
| `moirai.vault.signature_validation.failure_rate` | Counter | `> 0.01%` | P0 |

### 9.2 Health Check

```
GET /health
Response: {
  status:              "healthy" | "degraded" | "unavailable",
  dependencies: {
    postgresql:        "healthy" | "unavailable",
    neo4j:             "healthy" | "degraded" | "unavailable",
    kafka:             "healthy" | "unavailable",
    vault:             "healthy" | "unavailable",
    tsa_authority:     "healthy" | "unavailable"
  },
  kafka_consumer_lag_ms: int,
  neo4j_graph_lag_ms:    int,
  last_tsa_anchor:       datetime,
  chain_integrity:       "verified" | "unverified" | "broken"
}
```

### 9.3 Log Schema

```json
{
  "timestamp":       "ISO-8601",
  "service":         "MOIRAI",
  "level":           "INFO | WARN | ERROR",
  "event":           "EVENT_WRITTEN | CHAIN_VALIDATED | CHAIN_BREAK_DETECTED | TSA_ANCHOR_CREATED",
  "event_id":        "uuid | null",
  "session_id":      "uuid | null",
  "service_id":      "writing service code",
  "duration_ms":     0,
  "fields": {
    "chain_position": 0,
    "signature_valid": true,
    "tsa_anchored": false
  }
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

MOIRAI is the chain — it does not sign events itself for the cross-service chain. Instead:
- Each service signs its own events with `themis/{service}/signing-key`
- MOIRAI computes the `event_hash` (SHA-256 of the full event including the writing service's signature)
- The `event_hash` becomes the `prev_event_hash` for the next event in that session's stream
- MOIRAI signs the TSA anchor events with `themis/moirai/signing-key`

### 10.2 What This Service Attests

MOIRAI's chain proves: a sequence of signed events occurred in a specific order, each event references its predecessor by hash, and as of the most recent TSA anchor, the chain from the anchor backward has not been modified. An oversight body can independently verify chain integrity using the TSA tokens without trusting any THEMIS component.

### 10.3 What This Service Cannot Prove

The chain does not prove that events accurately reflect the real-world actions they describe. A service that writes a correctly-signed false event produces an unbreakable, independently-verifiable record of that false claim. Independent validation of the events' accuracy requires examining the service that wrote them, not MOIRAI.

---

## 11. Implementation Roadmap

### Phase 1 — Event Ledger Foundation (Weeks 9–16)

- PostgreSQL event ledger schema (ProvenanceEvent, append-only constraint)
- Kafka topic structure and consumer group configuration
- `POST /events` write endpoint with hash chaining
- Per-service signing key validation via Vault
- Basic `GET /events/{id}` and `GET /session/{id}/chain` read endpoints
- Chain integrity validation endpoint

**Phase gate criterion:** Every PCES and PGS event is written to MOIRAI with an unbroken hash chain. Chain validation endpoint confirms integrity. Hash chain break detection alert fires in test.

### Phase 2 — Graph, Blast Radius, and TSA Anchoring (Weeks 17–28)

- Neo4j provenance graph schema and population from event stream
- All Neo4j relationships: HAS_TURN, CONTAINS_CLAIM, DERIVED_FROM, etc.
- `/claim/{id}/provenance` endpoint
- `/blast-radius` endpoint for TVS/KAIROS
- RFC 3161 TSA anchoring (24-hour batch)
- Audit certificate generation endpoint
- IOB query endpoints

**Phase gate criterion:** Blast radius traversal returns affected claims for a test source invalidation in < 5 seconds. TSA anchor covers 100% of events older than 26 hours. Audit certificate accepted by IOB in test review. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| None | No GC items gate MOIRAI deployment. | — |

*Note: MOIRAI's audit certificate format is configurable. The content of what constitutes an adequate audit package for oversight submission is a policy decision owned by the IOB, not MOIRAI.*

---

## 13. Training and Analyst Guidance

MOIRAI is not directly analyst-facing in ATHENA. Analysts interact with MOIRAI indirectly through the provenance indicators surfaced by other services (source type badges, the Context Window Inspector, the session manifest). Supervisors and IOB members access MOIRAI directly through the audit endpoints.

**Supervisor training note:** When a supervisor reviews a session's chain audit certificate, the certificate proves the record was not altered — not that the underlying analytical work was sound. Chain validity is a necessary condition for accountability, not a sufficient condition for analytical quality.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Neo4j performance at operational scale.** The graph will grow to tens of millions of nodes within the first two years of operation. Query performance for multi-hop traversals (blast radius, multi-session provenance) must be benchmarked at this scale before Phase 2 is declared complete. Resolution path: load testing with synthetic data at 10M and 100M nodes before Phase 2 gate.

### 14.2 Operational Assumptions

- **Assumption 1: Kafka is available as platform infrastructure.** MOIRAI's write path assumes a managed Kafka cluster. If Kafka is not available, MOIRAI's write path must synchronously write to PostgreSQL, which reduces throughput significantly and changes the latency profile.
- **Assumption 2: An RFC 3161 timestamp authority is accessible from the deployment environment.** In air-gapped or high-security environments, access to external TSA services may not be permitted. If so, an internal TSA must be provisioned — this is a prerequisite deployment task, not a MOIRAI engineering task.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

---

## Appendix D: Red Team Findings

*Pending red team evaluation — scheduled for Phase 3 gate review.*
