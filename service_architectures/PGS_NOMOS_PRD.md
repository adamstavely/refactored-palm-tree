# PGS — Policy & Guardrails Service
### NOMOS · *"Greek for 'law' / 'convention' — the rules that govern a community"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `PGS` |
| **Epithet** | `NOMOS` |
| **Full name** | Policy & Guardrails Service |
| **Namespace** | `themis-gates` |
| **Layer** | Safety Gates |
| **Build phase** | Phase 1–2 (Weeks 1–8) |
| **Build priority** | 3 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Cross-cutting — enforces analytic standards across all three axes |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**PGS/NOMOS answers: Does this interaction comply with analytic standards, platform policy, and output quality requirements?**

### 1.2 Why This Service Exists

Intelligence Community Directive 203 (Analytic Standards) requires that all IC analytic products meet defined standards for sourcing, objectivity, uncertainty expression, and distinction of judgments from underlying intelligence. AI-assisted analysis introduces a new failure mode: the AI generates fluent, confident output that violates these standards — asserting certainty without appropriate hedging, failing to distinguish AI-generated synthesis from retrieved evidence, or producing output that would not pass tradecraft review if written by a human analyst.

Without PGS, analysts have no systematic mechanism to catch standard violations before they enter finished intelligence products. Manual tradecraft review exists but is resource-constrained and happens downstream. PGS provides an automated first pass at the generation layer, before the analyst has acted on the output.

### 1.3 Why This Service Is Third

PGS requires PCES to have validated the session (so the policy context is correct for this analyst and compartment) and produces records that MOIRAI captures (so policy decisions are in the provenance chain). PGS enforces what PCES permits — an analyst authorized for a compartment still must comply with the analytic standards that apply to their work in that compartment.

### 1.4 Design Principles

- **Policy rules are version-controlled and IOB-approved.** Changes to screening thresholds, classification rules, or interaction type definitions require IOB sign-off. The platform does not modify its own policy rules.
- **Screening is advisory or blocking, never silent.** PGS never silently drops output. Every policy flag produces a visible signal to the analyst and a MOIRAI event. The analyst knows they are working under a flagged condition.
- **Policy version is logged per session.** An audit query on any session can retrieve the exact policy version that governed it. This prevents "the policy changed" from being a valid excuse for non-compliance.
- **PGS screens output, not intent.** PGS applies rules to what the AI produces. It does not infer what the analyst intended to produce. Intent is an analyst responsibility, not a platform determination.
- **Completeness scores from ERAS feed output screening thresholds.** When ERAS signals that a response's reasoning quality is low, PGS tightens its output screening threshold. This is a design coupling — PGS is the enforcement point for quality signals from ERAS.

### 1.5 Explicit Out of Scope

- **Substantive analytical review.** PGS checks structural compliance with standards, not whether the analytical conclusion is correct.
- **Classification authority.** PGS applies existing classification rules; it does not make original classification decisions.
- **Legal compliance determination.** PGS checks against configured policy rules. Whether an output creates legal liability is a legal question for general counsel, not a PGS determination.

---

## 2. Core Responsibilities

### 2.1 Primary Function

PGS/NOMOS evaluates every analyst-AI interaction against a version-controlled policy rule set and every AI output against configured analytic standards guardrails — generating a classification for the interaction type, a screening result for the output, and a set of flags for any policy violations — before the analyst sees the response. Policy rules are configured by the IOB and encoded in versioned PolicyVersion records. The version applied to every session is logged to MOIRAI.

### 2.2 Secondary Functions

- Interaction classification: classifying every session turn into a defined interaction class (RESEARCH, EVIDENCE_ANALYSIS, CAPABILITY_ASSESSMENT, etc.) for downstream calibration routing
- PII detection: screening analyst inputs and AI outputs for personally identifiable information
- Output screening threshold management: tightening screening thresholds when ERAS signals low-completeness reasoning
- Policy version management: maintaining the version history of policy rules and providing the active policy version to other services
- Tradecraft compliance reporting: aggregate compliance metrics for IOB reporting

### 2.3 What This Service Does Not Decide

PGS applies configured policy rules — it does not determine whether those rules are appropriate. Whether a specific output meets a broader definition of analytical quality, whether a policy rule should be waived for an edge case, whether a compliance flag warrants disciplinary action — these are human decisions owned by the analytic standards authority, not PGS determinations.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
PolicyVersion:
  version_id:            uuid
  version_string:        str               # semantic version e.g., "2.4.1"
  effective_from:        datetime
  effective_until:       datetime | null   # null if current active version
  iob_approval_ref:      str              # IOB decision reference
  approved_by:           str
  rules:                 [PolicyRule]
  screening_thresholds:  ScreeningThresholds

PolicyRule:
  rule_id:               uuid
  version_id:            uuid
  rule_type:             ANALYTIC_STANDARDS | PII | OUTPUT_QUALITY | INTERACTION_CLASS | EXPORT_CONTROL
  name:                  str
  description:           str
  condition:             str               # rule expression (domain-specific language)
  action:                BLOCK | FLAG | ANNOTATE | LOG
  severity:              P0 | P1 | P2 | P3
  applies_to:            INPUT | OUTPUT | SESSION_INIT
  interaction_classes:   [str] | null      # null = applies to all

ScreeningThresholds:
  default_completeness_floor:     float    # minimum ERAS completeness score for pass
  deadline_critical_adjustment:   float    # additional tightening under DC pressure
  eras_triggered_adjustment:      float    # tightening when ERAS flags low completeness

InteractionRecord:
  record_id:             uuid
  session_id:            uuid
  turn_id:               uuid
  policy_version_id:     uuid
  interaction_class:     str
  input_screening:       ScreeningResult
  output_screening:      ScreeningResult
  pii_detected:          bool
  pii_flags:             [PIIFlag]
  completeness_threshold_applied: float    # from ScreeningThresholds at time of screening
  timestamp:             datetime

ScreeningResult:
  result_id:             uuid
  passed:                bool
  flags:                 [PolicyFlag]
  blocking_rule_ids:     [uuid] | null     # rules that caused a BLOCK
  timestamp:             datetime

PolicyFlag:
  flag_id:               uuid
  rule_id:               uuid
  rule_name:             str
  description:           str
  severity:              P0 | P1 | P2 | P3
  action:                BLOCK | FLAG | ANNOTATE | LOG
  context_snippet:       str               # sanitised excerpt; no classified content in flag

PIIFlag:
  flag_id:               uuid
  pii_type:              str               # NAME | SSN | CONTACT | etc.
  location:              INPUT | OUTPUT
  sanitised_excerpt:     str
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | InteractionRecord, PolicyVersion, PolicyRule | Session + 7 years |
| Active policy cache | Redis | Active PolicyVersion and rules (hot path) | Policy TTL + invalidation |
| Event store | MOIRAI | Signed policy decision events | Indefinite |
| PII detection results | PostgreSQL (separate schema) | PIIFlag records — restricted access | Session + 7 years |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| InteractionRecord | Inherits session classification | Session compartment inherited |
| PolicyVersion/Rule | Controlled Unclassified | Accessible to platform team; IOB approval records are controlled |
| PIIFlag | Controlled Unclassified (PII records are themselves sensitive) | Separate access control; not visible to standard analyst |

### 3.4 Retention and Purge Policy

InteractionRecord and associated ScreeningResult records are retained for the session lifetime plus seven years. PolicyVersion records are retained indefinitely — historical policy versions must be queryable for any past session's audit. PII flags are retained for the session lifetime plus seven years with restricted access.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /screen/input
  Auth:     session token (PCES-validated)
  Request:  {
    session_id:     uuid,
    turn_id:        uuid,
    input_text:     str,
    interaction_context: str | null
  }
  Response: {
    result:         ScreeningResult,
    interaction_class: str,
    pii_detected:   bool,
    proceed:        bool     # false if any BLOCK rule fired
  }
  SLA: p99 < 200ms

POST /screen/output
  Auth:     service account (inference gateway)
  Request:  {
    session_id:     uuid,
    turn_id:        uuid,
    output_text:    str,
    eras_completeness: float | null    # from ERAS if available
  }
  Response: {
    result:         ScreeningResult,
    proceed:        bool,
    flags:          [PolicyFlag]
  }
  SLA: p99 < 300ms

GET /policy/current
  Auth:     any service account
  Response: PolicyVersion (rules omitted unless supervisor token)
  SLA: p99 < 50ms (cached)

GET /policy/{version_id}
  Auth:     supervisor token | IOB token
  Response: PolicyVersion (full including rules)

GET /session/{session_id}/compliance
  Auth:     supervisor token | IOB token
  Response: {
    session_id:          uuid,
    policy_version_id:   uuid,
    interaction_records: [InteractionRecord],
    aggregate_flags:     { P0: int, P1: int, P2: int, P3: int },
    pii_detected:        bool
  }

GET /audit/compliance-summary?from={date}&to={date}&class={class}
  Auth:     IOB token
  Response: {
    period:              { from, to },
    total_interactions:  int,
    pass_rate:           float,
    by_interaction_class: [{ class, pass_rate, flag_counts }],
    by_rule:             [{ rule_id, name, flag_count }]
  }

GET /health
  Response: { status, dependencies: { moirai, pces, redis }, policy_version: str }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          PGS_INPUT_SCREENED
service_id:         "PGS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  interaction_class:  str
  passed:             bool
  flag_count:         int
  blocking_flag_count:int
  policy_version_id:  uuid
  pii_detected:       bool
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          PGS_OUTPUT_BLOCKED
event_payload:
  blocking_rule_ids: [uuid]
  flag_count:        int
  eras_completeness: float | null

EventType:          PGS_POLICY_VERSION_CHANGED
event_payload:
  old_version_id:    uuid
  new_version_id:    uuid
  approved_by:       str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `PGS_INPUT_SCREENED` | Every session turn input | MOIRAI, TCS/MIMIR (interaction class for calibration routing) |
| `PGS_OUTPUT_BLOCKED` | Output screening produces BLOCK | MOIRAI, ATHENA (block notification to analyst) |
| `PGS_OUTPUT_FLAGGED` | Output screening produces FLAG | MOIRAI, ATHENA (flag notification) |
| `PGS_POLICY_VERSION_CHANGED` | New policy version activated | MOIRAI, all services (cache invalidation) |
| `PGS_PII_DETECTED` | PII found in input or output | MOIRAI, session audit record |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| ERAS/LOGOS | `ERAS_CAPTURE_CREATED` | Updates output screening threshold for the session based on completeness score |
| PCES/AEGIS | `PCES_SESSION_GRANTED` | Loads applicable policy rules for the session's compartment and interaction type |
| IOB (via admin API) | Policy version activation | Triggers `PGS_POLICY_VERSION_CHANGED` and cache invalidation |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed event emission | Async event | Events buffered; screening decisions still enforced |
| PCES/AEGIS | Classification Enforcement | Session token validation; session compartment context | Sync | Screening proceeds; PCES unavailability noted in record |
| ERAS/LOGOS | Reasoning Audit | Completeness score for output screening threshold | Async event | Screening uses default threshold; no ERAS adjustment |

### 5.2 Feeds Into

| Service | Epithet | What PGS provides | How |
|---|---|---|---|
| TCS/MIMIR | Trust Calibration | Interaction class for calibration routing | MOIRAI event → TCS consumer |
| ERAS/LOGOS | Reasoning Audit | Completeness threshold applied; policy version | API |
| ATHENA | Interface | Block/flag notifications; interaction class for session context | API response |
| MGS/TERMINUS | MCP Gateway | Interaction class for MCP access scope determination | API |
| IOB Reporting | Oversight | Aggregate compliance metrics; per-session compliance records | Audit query endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Session token validated on all data-returning endpoints.
- **Compartment inheritance:** InteractionRecord inherits session classification. Policy rules are not compartment-restricted (they apply platform-wide) but their application to a specific session is classified at the session level.
- **Failure behavior:** PCES unavailability → screening proceeds using the session context already established at session initialization (cached in Redis). New session init is blocked by PCES.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Input screening | 30ms | 100ms | 200ms |
| Output screening | 50ms | 150ms | 300ms |
| Policy version lookup (cached) | 2ms | 5ms | 50ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Screening requests/second | 100 (50 analysts × 2 screen calls per turn) |
| Policy version lookups/second | 500 (called by multiple services per turn) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — PGS unavailability degrades policy compliance logging but should not stop sessions |
| MOIRAI event durability | 99.999% |
| RTO | 10 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Events buffered; screening decisions still enforced | No analyst-facing impact; provenance gap logged |
| PCES | Screening proceeds using cached session context; new session init blocked elsewhere | No impact on active sessions |
| ERAS (completeness signal) | Output screening uses default threshold | Screening is less precise; alert fires |
| Redis (policy cache) | Fallback to PostgreSQL policy lookup (higher latency) | Screening latency increases; alert fires |

---

## 7. Security Model

### 7.1 Authentication

Input screening is called by the inference gateway service account. Output screening is called by the same. Policy management endpoints require IOB or supervisor tokens. Compliance audit endpoints require IOB tokens.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Inference gateway | `/screen/input`, `/screen/output` | Service account |
| Any service | `GET /policy/current` (redacted) | Service account |
| Supervisor | Compliance records for their team; full policy version | Supervisor token |
| IOB | All endpoints including audit summary | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/pgs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/pgs/db-credentials` | 30 days |
| Redis credentials | `themis/pgs/redis-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

Policy rule bypass via input crafting is the primary threat: adversarially constructed inputs designed to avoid PII detection or standards screening. Mitigation: PII detection uses multiple layers (pattern matching, entity recognition, semantic classification); screening rules are updated by the red team when bypass techniques are identified. Policy version tampering is mitigated by IOB sign-off requirements and MOIRAI event logging on all version changes.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Policy rule false positives (valid output blocked) | Medium | P2 — analyst friction | Analyst complaint rate; manual review of blocked outputs | Rule sensitivity tuning; analyst appeal mechanism |
| PII false negatives (PII not detected) | Low | P1 — PII in analytical record | Retrospective PII audit | Multilayer detection; regular red team testing |
| Policy version cache inconsistency | Low | P2 — stale policy applied to new session | Version mismatch monitoring | Cache TTL + event-driven invalidation on version change |

### 8.1 Known Design Risks

- **Interaction classification accuracy affects TCS calibration routing.** If PGS misclassifies an interaction type, TCS/MIMIR routes the calibration update to the wrong cell. This is a compounding error — miscalibrated calibration is difficult to detect and correct. Mitigation: interaction classification benchmarking against a labelled test set of historical interactions before Phase 2.
- **ERAS completeness coupling may be circular.** PGS tightens screening when ERAS signals low completeness. But ERAS completeness depends on the quality of the reasoning capture, which depends on the system prompt configured by PGS. If a low-quality system prompt produces both low ERAS completeness and low-quality outputs, the feedback loop between PGS and ERAS amplifies the problem rather than catching it. Resolution path: ERAS completeness thresholds are set conservatively at launch; threshold adjustment is a Research and Red Team task with pre-registered criteria.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `pgs.screen.latency_p99` | Histogram | `> 300ms for 5m` | P1 |
| `pgs.output.block_rate` | Gauge | Spike > 5x baseline over 10m | P2 |
| `pgs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `pgs.policy.version_mismatch_rate` | Counter | `> 0` | P1 |
| `pgs.pii.detection_rate` | Gauge | Spike > 3x baseline | P2 |

### 9.2 Health Check

```
GET /health
Response: {
  status:             "healthy" | "degraded" | "unavailable",
  dependencies: {
    moirai:           "healthy" | "unavailable",
    pces:             "healthy" | "unavailable",
    redis:            "healthy" | "unavailable"
  },
  active_policy_version: str,
  moirai_sync:        boolean,
  last_event_hash:    string
}
```

### 9.3 Log Schema

```json
{
  "timestamp":        "ISO-8601",
  "service":          "PGS/NOMOS",
  "level":            "INFO | WARN | ERROR",
  "event":            "INPUT_SCREENED | OUTPUT_BLOCKED | OUTPUT_FLAGGED | POLICY_CHANGED",
  "session_id":       "uuid",
  "turn_id":          "uuid",
  "policy_version":   "string",
  "interaction_class":"string",
  "duration_ms":      0,
  "fields": {
    "passed":         true,
    "flag_count":     0,
    "blocking_count": 0,
    "pii_detected":   false
  }
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/pgs/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant
- **Prev_event_hash source:** Prior PGS event in the same session's event stream

### 10.2 What This Service Attests

The MOIRAI record for PGS proves that specific policy rules (from a specific, IOB-approved policy version) were applied to every interaction in every session, and that the screening decisions have not been altered since they were recorded. The policy version logged per session is the chain of accountability for standards compliance.

### 10.3 What This Service Cannot Prove

PGS proves policy was applied. It does not prove the policy was the right policy, that the policy rules were correctly configured, or that passing PGS screening implies the output met the spirit of analytic standards as opposed to their letter.

---

## 11. Implementation Roadmap

### Phase 1 — Core Screening (Weeks 1–4)

- PolicyVersion schema and initial policy rule set (baseline ICD 203 rules)
- Input screening endpoint with interaction classification
- Output screening endpoint with basic standards compliance rules
- PII detection (pattern-based, first pass)
- MOIRAI event emission
- Active policy version cache in Redis

**Phase gate criterion:** Every session turn produces a PGS screening record. Policy version is logged per session in MOIRAI. Output blocking fires correctly on test violation cases.

### Phase 2 — ERAS Integration, IOB Interface, and Audit (Weeks 5–8)

- ERAS completeness signal integration for adaptive output screening threshold
- IOB policy management endpoints (version approval workflow)
- Compliance audit query endpoints
- Multilayer PII detection (semantic + pattern)
- Interaction classification benchmarking and tuning

**Phase gate criterion:** ERAS completeness signal demonstrably tightens output screening threshold. IOB can query per-session compliance records. Interaction classification accuracy ≥ 85% on test set. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-3 | Query-type authorization taxonomy | Required before Phase 1 classifier training — interaction class taxonomy must be agreed |

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

When PGS blocks an output, the analyst sees a red banner in ATHENA: "This response was blocked by analytic standards screening. [Plain-language reason]. The session has been logged." When PGS flags an output (not blocks), the analyst sees an amber flag with the specific standard that was flagged and a prompt to review before proceeding.

### 13.2 What the Analyst Should Do

For a blocked output: do not attempt to work around the block by reformulating the query to produce the same output. The block is a standards compliance determination. The analyst should note the block in their analytical record and, if the block appears erroneous, escalate to their supervisor.

For a flagged output: review the flagged element before incorporating the output into any work product. The flag identifies a specific standards concern — address it explicitly in any work product that uses the flagged content.

### 13.3 What the Signal Does Not Mean

A PGS pass does not mean the output is analytically sound. PGS checks structural compliance with standards; it does not evaluate the quality of the analysis. An output can pass all PGS screens and still be analytically wrong.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Interaction classification accuracy on novel interaction types.** The initial interaction class taxonomy covers RESEARCH, EVIDENCE_ANALYSIS, CAPABILITY_ASSESSMENT, SYNTHESIS, and REVIEW. Novel interaction patterns that don't fit these classes will be misclassified, producing incorrect calibration routing in TCS. Resolution path: taxonomy is extensible; new classes added via IOB policy update (same governance process as other policy changes).

### 14.2 Operational Assumptions

- **Assumption 1: IOB has bandwidth to approve policy rule changes on a reasonable timeline.** If IOB approval for policy changes takes months, the platform cannot adapt to new analytical standards requirements or emerging adversarial patterns quickly enough. Resolution path: establish a tiered approval process — minor threshold adjustments may have delegated approval, major rule additions require full IOB review.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

---

## Appendix D: Red Team Findings

*Pending red team evaluation — scheduled for Phase 1 gate review.*
