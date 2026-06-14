# IAS — Inference Adversarial Screening Service
### SCUDO · *"Italian for 'shield' — the defensive cover carried into battle"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `IAS` |
| **Epithet** | `SCUDO` |
| **Full name** | Inference Adversarial Screening Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 5–6 (Weeks 29–46) |
| **Build priority** | 10 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — screens against adversarial manipulation of the analytical workflow |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**IAS/SCUDO answers: Does this input or retrieved content contain adversarial instructions, manipulation attempts, or indicators of corpus poisoning designed to compromise the analytical workflow?**

### 1.2 Why This Service Exists

Foreign intelligence services know that the IC uses AI-assisted analysis. They are designing collection and influence operations specifically against AI analytical workflows:

**Corpus poisoning**: seeding open-source and third-party reporting with fabricated intelligence crafted to be ingested into analytical AI systems and retrieved in support of specific analytical conclusions.

**Prompt injection**: embedding adversarial instructions in documents likely to be retrieved in AI-assisted workflows — instructions that, when retrieved into the context window, manipulate the AI's behaviour.

**Adversarial competence probing**: querying the AI on known capability gaps to generate confident confabulations, then introducing those confabulations into the analytical record as AI-sourced intelligence.

**Anchoring exploitation**: submitting initial queries designed to anchor the AI's subsequent responses toward predetermined conclusions, exploiting the model's tendency to build on early context.

None of these attack vectors are visible to analysts looking at the AI's output. The AI responds with the same fluency whether it is working with clean intelligence or adversarially manipulated intelligence. IAS/SCUDO is the service that screens for adversarial signals before they reach the context window.

### 1.3 Why This Service Is Tenth

IAS requires PGS to have established the interaction classification taxonomy (adversarial screening rules are interaction-class-specific), CVS to be operational (fabrication patterns from CVS inform IAS corpus poisoning detection), and MOIRAI to record adversarial screening events. Phase 5-6 allows the platform to accumulate interaction data before adversarial baseline models are trained.

### 1.4 Design Principles

- **Two-layer screening: input and retrieval.** Adversarial attacks occur at two points. Input screening happens before the query reaches the retrieval system. Retrieval screening happens after retrieval, before retrieved content enters the context window. Both layers are required.
- **Adversarial patterns evolve.** The threat catalog is not static. The Research & Red Team maintains and updates it. Pattern updates do not require service redeployment — the catalog is versioned and hot-reloadable.
- **Under Deadline-Critical conditions, screening tightens.** Deadline periods are when adversarial operations intensify and when analytical deliberation degrades. IAS escalates its screening sensitivity by one tier under DC conditions.
- **Screening blocks, not filters.** When IAS detects a pattern above the threshold, it blocks the request and surfaces the reason. It does not silently modify or filter content — analysts must know they are working under screening constraints.
- **The MCP response surface is an extension of the input surface.** MGS/TERMINUS routes all MCP server responses through IAS screening before they enter the context window. External tool calls are an injection vector equivalent to retrieved documents.

### 1.5 Explicit Out of Scope

- **Classification of adversarial intent or attribution.** IAS detects patterns; it does not attribute them to specific adversaries or campaigns.
- **Screening analyst output (finished intelligence).** IAS screens inputs to the AI and content retrieved into the context. It does not screen the analyst's finished intelligence products — that is PGS/NOMOS's remit.
- **Network-level security.** IAS operates at the content layer; network security is infrastructure.

---

## 2. Core Responsibilities

### 2.1 Primary Function

IAS/SCUDO screens analyst inputs before they reach the retrieval system and screens retrieved content before it enters the AI context window — applying a versioned threat catalog of adversarial patterns covering prompt injection, corpus poisoning indicators, persona hijacking, anchoring exploitation, and adversarial competence probing — blocking detected adversarial content and surfacing the detection to the analyst with a plain-language explanation. All screening decisions are recorded in MOIRAI.

### 2.2 Secondary Functions

- MCP response screening: extending input screening to MCP server responses via MGS/TERMINUS integration
- Threat catalog maintenance: versioned threat pattern library with hot-reload; Research & Red Team owns catalog updates
- Corpus poisoning signal feed: adversarial patterns detected in retrieval provided to CVS for fabrication rate monitoring
- Deadline-Critical escalation: automatic screening sensitivity increase under DC pressure modes
- Adversarial session pattern detection: detecting coordinated adversarial attempts across multiple turns in a session (not just individual inputs)
- Baseline model training feed: accumulating adversarial detection signals for Research & Red Team model improvement

### 2.3 What This Service Does Not Decide

IAS detects adversarial patterns above a configured threshold. Whether a specific detection is a true positive, whether the analyst may proceed despite a detection, and whether the pattern indicates an active collection threat are human decisions. IAS blocks and surfaces; an analyst can request supervisor override for ambiguous cases.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ScreeningDecision:
  decision_id:           uuid
  session_id:            uuid
  turn_id:               uuid
  screening_type:        INPUT | RETRIEVAL | MCP_RESPONSE
  content_hash:          str              # SHA-256 of screened content (not stored plain)
  decision:              PASS | BLOCK | ESCALATE
  confidence:            float
  patterns_detected:     [PatternMatch]
  threat_catalog_version:str
  pressure_mode:         str
  sensitivity_tier:      STANDARD | ELEVATED | HIGH  # standard + DC escalation
  timestamp:             datetime

PatternMatch:
  match_id:              uuid
  pattern_id:            uuid             # FK → ThreatPattern
  pattern_type:          str
  confidence:            float
  location:              INPUT | RETRIEVED_CHUNK | MCP_RESPONSE
  context_snippet:       str              # sanitised, no classified content

ThreatPattern:
  pattern_id:            uuid
  pattern_type:          PROMPT_INJECTION | CORPUS_POISONING | PERSONA_HIJACKING | ANCHORING_EXPLOITATION | COMPETENCE_PROBING | INDIRECT_INJECTION | ADVERSARIAL_FRAMING
  name:                  str
  description:           str
  detection_method:      PATTERN_MATCH | SEMANTIC | ML_CLASSIFIER | HEURISTIC
  sensitivity_threshold: { STANDARD: float, ELEVATED: float, HIGH: float }
  catalog_version:       str
  created_by:            str             # Research & Red Team reference
  effective_from:        datetime

CatalogVersion:
  version_id:            uuid
  version_string:        str
  patterns:              [ThreatPattern]
  published_at:          datetime
  published_by:          str
  change_summary:        str

AdversarialSessionFlag:
  flag_id:               uuid
  session_id:            uuid
  flag_type:             ESCALATING_PATTERNS | CROSS_TURN_INJECTION | COORDINATED_ATTEMPT
  turn_count_in_pattern: int
  confidence:            float
  flagged_at:            datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | ScreeningDecision, PatternMatch, AdversarialSessionFlag | Session + 7 years |
| Threat catalog store | PostgreSQL | ThreatPattern, CatalogVersion | Indefinite |
| Catalog cache | Redis | Active catalog version (hot-loaded) | Catalog TTL |
| Event store | MOIRAI | Signed screening events | Indefinite |
| Adversarial pattern corpus | Elasticsearch (restricted) | Sanitised pattern examples for Research & Red Team | 5 years |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ScreeningDecision | Inherits session classification | Session-compartmented |
| PatternMatch | Controlled Unclassified (pattern descriptions only) | Research & Red Team access for full details |
| ThreatPattern | Controlled Unclassified (catalog details are sensitive) | Platform team + Research & Red Team access |
| Adversarial pattern corpus | SECRET (examples may be classified) | Research & Red Team restricted access |

### 3.4 Retention and Purge Policy

ScreeningDecision and PatternMatch records retained for session lifetime plus seven years. ThreatPattern and CatalogVersion records retained indefinitely. Adversarial pattern corpus retained for five years. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /screen/input
  Auth:     inference gateway service account
  Request:  {
    session_id:          uuid,
    turn_id:             uuid,
    input_text:          str,
    pressure_mode:       str
  }
  Response: {
    decision_id:         uuid,
    decision:            PASS | BLOCK | ESCALATE,
    patterns_detected:   int,
    confidence:          float,
    block_reason:        str | null,    # plain language; no pattern details
    proceed:             bool
  }
  SLA: p99 < 200ms

POST /screen/retrieval
  Auth:     retrieval gateway service account
  Request:  {
    session_id:          uuid,
    turn_id:             uuid,
    chunks:              [{ chunk_id, source_id, content_hash, content_preview }],
    pressure_mode:       str
  }
  Response: {
    decision_id:         uuid,
    chunks_passed:       [str],         # chunk IDs cleared for context assembly
    chunks_blocked:      [{ chunk_id, reason }],
    overall_decision:    PASS | PARTIAL_BLOCK | FULL_BLOCK
  }
  SLA: p99 < 500ms

POST /screen/mcp-response
  Auth:     MGS/TERMINUS service account
  Request:  {
    session_id:          uuid,
    mcp_server_name:     str,
    response_hash:       str,
    response_preview:    str,           # partial content for screening (not full response)
    pressure_mode:       str
  }
  Response: {
    decision:            PASS | BLOCK,
    confidence:          float,
    block_reason:        str | null
  }
  SLA: p99 < 200ms

GET /catalog/current
  Auth:     any service account
  Response: { version_string: str, pattern_count: int, published_at: datetime }

GET /session/{session_id}/screening-summary
  Auth:     session token | supervisor token
  Response: {
    blocks:              int,
    escalations:         int,
    patterns_detected:   int,
    adversarial_flags:   [AdversarialSessionFlag]
  }

GET /audit/threat-report?from={date}&to={date}
  Auth:     Research & Red Team token | IOB token
  Response: {
    period:              { from, to },
    total_screenings:    int,
    block_rate:          float,
    by_pattern_type:     [{ type, detection_count, block_rate }],
    adversarial_sessions:int
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis },
    active_catalog_version: str,
    screenings_today:    int,
    block_rate_24h:      float,
    last_event_hash:     str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          IAS_INPUT_BLOCKED
service_id:         "IAS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  decision_id:            uuid
  screening_type:         str
  patterns_detected:      int
  dominant_pattern_type:  str
  confidence:             float
  pressure_mode:          str
  sensitivity_tier:       str
  catalog_version:        str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          IAS_CHUNK_BLOCKED
event_payload:
  decision_id:            uuid
  chunks_blocked:         int
  dominant_pattern_type:  str

EventType:          IAS_ADVERSARIAL_SESSION_FLAGGED
event_payload:
  session_id:             uuid
  flag_type:              str
  confidence:             float
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `IAS_INPUT_BLOCKED` | Any input blocked | MOIRAI, ATHENA (analyst notification), supervisor alert if repeated |
| `IAS_CHUNK_BLOCKED` | Retrieved chunks blocked | MOIRAI, RQS (blocked chunks affect coverage assessment) |
| `IAS_ADVERSARIAL_SESSION_FLAGGED` | Cross-turn pattern detected | MOIRAI, supervisor notification, Research & Red Team alert |
| `IAS_CATALOG_UPDATED` | New threat catalog version deployed | MOIRAI |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| CVS/VERITAS | `CVS_FABRICATION_DETECTED` | Updates corpus poisoning pattern confidence for similar sources |
| PGS/NOMOS | `PGS_INPUT_SCREENED` | Interaction class used to apply class-specific screening rules |
| MGS/TERMINUS | MCP response for screening | Runs `POST /screen/mcp-response` |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed screening events | Async event | Events buffered; screening still enforced |
| PCES/AEGIS | Classification Enforcement | Session token validation | Sync | Screening proceeds with cached session context |
| CVS/VERITAS | Source Corroboration | Fabrication patterns for corpus poisoning detection | Async event | Corpus poisoning detection operates without CVS signal |

### 5.2 Feeds Into

| Service | Epithet | What IAS provides | How |
|---|---|---|---|
| ATHENA | Interface | Block notifications with plain-language reason | API response |
| MGS/TERMINUS | MCP Gateway | MCP response screening | `POST /screen/mcp-response` API |
| RQS/HERMES | Retrieval Quality | Blocked chunk information for coverage reassessment | MOIRAI event |
| Research & Red Team | Threat Analysis | Adversarial pattern corpus for model improvement | Adversarial corpus feed |
| IOB Reporting | Oversight | Threat report; adversarial session flags | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Session token validated on analyst-facing summary endpoint. Service-to-service screening calls use service accounts.
- **Threat catalog classification:** The threat catalog is Controlled Unclassified; pattern details (specific signatures) are restricted to the Research & Red Team. Analysts see plain-language block reasons, not pattern signatures.
- **Failure behavior:** PCES unavailable → screening proceeds with cached session context; analyst-facing endpoints unavailable.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Input screening | 30ms | 80ms | 200ms |
| Retrieval screening (per batch) | 100ms | 300ms | 500ms |
| MCP response screening | 30ms | 80ms | 200ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Input screenings/second | 100 |
| Chunk screening operations/second | 500 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — IAS unavailability means adversarial screening is suspended |
| MOIRAI event durability | 99.999% |
| RTO | 5 minutes — security service; rapid recovery required |
| RPO | 0 minutes |

### 6.4 Graceful Degradation

IAS follows a **fail-safe-closed policy** in most degradation scenarios. If IAS is unavailable, the platform operator must decide whether to suspend analytical sessions or proceed without adversarial screening. There is no silent fallback that bypasses screening without explicit operator decision.

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| Redis (catalog cache) | Screening proceeds from PostgreSQL catalog (higher latency); alert | Latency increase |
| MOIRAI | Events buffered; screening still enforced | No analyst impact; provenance gap logged |
| CVS pattern feed | Corpus poisoning detection operates without fabrication signal | Reduced corpus poisoning sensitivity |

---

## 7. Security Model

### 7.1 Authentication

All screening endpoints use service accounts. Analyst-facing summary endpoint uses session token. Threat catalog management requires Research & Red Team token with IOB notification. Audit endpoints require IOB token.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Inference gateway | All screening endpoints | Service account |
| MGS/TERMINUS | `POST /screen/mcp-response` | Service account |
| Analyst session | Session screening summary (own sessions) | Session token |
| Research & Red Team | Catalog management; adversarial corpus; threat report | Research team token |
| IOB | Full audit and threat reports | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/ias/signing-key` | 90 days |
| PostgreSQL credentials | `themis/ias/db-credentials` | 30 days |
| Redis credentials | `themis/ias/redis-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

**Catalog bypass through novel patterns**: an adversary who knows the threat catalog can craft injections that avoid the current patterns. Mitigation: catalog is not publicly accessible; Research & Red Team uses classified adversarial testing. Semantic detection is harder to bypass than pattern matching — the ML classifier component is the most robust layer against novel patterns.

**Screening timing attacks**: an adversary who can measure IAS response latency may infer whether a pattern was detected based on response time differences. Mitigation: screening responses are padded to a constant minimum response time.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| False positive (legitimate content blocked) | Medium | P2 — analyst friction; legitimate intelligence blocked | Analyst complaint rate on blocks | Conservative initial thresholds; analyst override with supervisor approval |
| Novel injection technique evading all detection layers | Low | P1 — adversarial content reaches context window | Research & Red Team red team testing | Semantic layer as catch-all; Research & Red Team ongoing adversarial testing |
| Catalog update introducing regression (blocking legitimate content) | Low | P2 — elevated false positive rate | Block rate spike monitoring | Catalog updates tested on historical clean sessions before deployment |

### 8.1 Known Design Risks

- **The threat catalog will always lag adversary capability.** This is a structural limitation. Adversary patterns evolve; catalog updates are reactive. The Research & Red Team must run continuous red team exercises to discover novel patterns before they are operationally deployed against THEMIS. The semantic ML classifier is the primary defense against unseen patterns.
- **ML classifier performance on low-resource attack types.** The ML classifier for adversarial pattern detection requires training data. Novel attack types with few examples will have low classifier confidence. The Research & Red Team must prioritise developing training data for emerging attack vectors. Initial deployment may require conservative classifier thresholds that produce more false positives — a known trade-off.
- **Deadline-Critical escalation may produce operational friction.** Increasing screening sensitivity under DC conditions means higher false positive rates exactly when analysts are under the most time pressure. The Research & Red Team must validate that DC-escalated thresholds are set to produce acceptable false positive rates in simulated DC conditions before Phase 2 deployment.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `ias.screening.latency_p99` | Histogram | `> 200ms for 5m` (input); `> 500ms for 5m` (retrieval) | P1 |
| `ias.block_rate` | Gauge | Spike > 5x baseline over 10m | P2 |
| `ias.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `ias.adversarial_session_rate` | Gauge | `> 1% of sessions` | P1 |
| `ias.catalog.age_hours` | Gauge | `> 720h` (30 days without catalog update) | P2 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces, redis },
  active_catalog_version: str,
  catalog_age_hours:      int,
  block_rate_24h:         float,
  screenings_today:       int,
  last_event_hash:        str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":         "ISO-8601",
  "service":           "IAS/SCUDO",
  "event":             "INPUT_PASSED | INPUT_BLOCKED | CHUNK_BLOCKED | SESSION_FLAGGED",
  "decision_id":       "uuid",
  "session_id":        "uuid",
  "screening_type":    "INPUT | RETRIEVAL | MCP_RESPONSE",
  "decision":          "PASS | BLOCK | ESCALATE",
  "patterns_detected": 0,
  "pressure_mode":     "STANDARD | ELEVATED | DEADLINE_CRITICAL",
  "duration_ms":       0
}
```

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/ias/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every adversarial screening decision — what was screened, what was detected, what was blocked — is permanently recorded. The full adversarial screening history for any session is available to the Research & Red Team and IOB for threat analysis and platform audit.

---

## 11. Implementation Roadmap

### Phase 1 — Input and Retrieval Screening (Weeks 29–36)

- ScreeningDecision and PatternMatch schemas
- Threat catalog v1.0: pattern-matching rules covering OWASP LLM Top 10 base patterns, plus IC-specific prompt injection and corpus poisoning patterns (Research & Red Team to specify)
- `POST /screen/input` and `POST /screen/retrieval` endpoints
- MOIRAI event emission
- DC escalation: screening sensitivity increase under Deadline-Critical
- ATHENA block notification integration

**Phase gate criterion:** Every ATHENA session input is screened before retrieval. Every retrieved chunk is screened before context assembly. Known test injection patterns are blocked with > 95% recall. MOIRAI events produced for all blocks.

### Phase 2 — MCP Screening, ML Classifier, and Adversarial Session Detection (Weeks 37–46)

- `POST /screen/mcp-response` endpoint and MGS/TERMINUS integration
- ML classifier component for semantic adversarial detection (Research & Red Team provides training data)
- Adversarial session pattern detection (cross-turn injection coordination)
- Research & Red Team threat report and adversarial corpus access
- IOB audit endpoints
- Threat catalog hot-reload mechanism

**Phase gate criterion:** MCP response screening operational. ML classifier demonstrates improvement over pattern-matching alone on Research & Red Team test set. Cross-turn adversarial session detection fires on test coordinated injection scenario. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate IAS deployment. Threat catalog content is owned by the Research & Red Team; changes require Research & Red Team sign-off and IOB notification, not IOB approval.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

When IAS blocks an input, the analyst sees: "Your query was blocked because it contains content that may compromise analytical integrity. [Plain-language reason]. If you believe this is an error, contact your supervisor for an override." When a retrieved chunk is blocked, the analyst sees: "One or more retrieved sources were blocked by adversarial screening. Your session continues with the remaining sources." The analyst is never shown the specific pattern that triggered the block.

### 13.2 What the Analyst Should Do

Input blocked: do not attempt to reformulate the query to replicate the blocked content. If the block appears erroneous, request supervisor override. Do not share the specific input that was blocked — it may contain adversarial content that should not be reproduced. Chunk blocked: note that some retrieved sources were unavailable in your analytical record. The coverage assessment may be lower than usual.

### 13.3 What the Signal Does Not Mean

A block does not mean the analyst has done something wrong — adversarial content embedded in retrieved documents is not the analyst's fault. A block does not mean the session is compromised — it means the platform prevented potential adversarial content from reaching the context window.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: IC-specific threat catalog content.** The base catalog covers OWASP LLM Top 10 patterns. IC-specific corpus poisoning patterns, adversarial SIGINT/HUMINT report structures, and adversary-specific injection techniques require domain expertise from the Research & Red Team. This content must be specified before Phase 1 deployment. Resolution path: Research & Red Team to produce IC-specific threat pattern specifications as a Phase 1 dependency.

### 14.2 Operational Assumptions

- **Assumption 1: The ML classifier has sufficient training data at Phase 2 deployment.** If the Research & Red Team has not produced sufficient adversarial training examples, the ML classifier will not improve screening coverage over pattern matching alone. Resolution path: Research & Red Team adversarial data collection begins before Phase 1 to have training data ready for Phase 2.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 5 gate review. Research & Red Team must produce IC-specific threat catalog before Phase 1.*
