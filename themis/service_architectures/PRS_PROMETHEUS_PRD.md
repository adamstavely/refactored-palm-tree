# PRS — Prompt Repository Service
### PROMETHEUS · *"Greek Titan who gave fire to humans — the enabling capability that transforms raw potential into productive action"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `PRS` |
| **Epithet** | `PROMETHEUS` |
| **Full name** | Prompt Repository Service |
| **Namespace** | `themis-interaction` |
| **Layer** | Interaction Layer |
| **Build phase** | Phase 1–2 (Weeks 1–8) |
| **Build priority** | 19 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — governs the analytical methodology encoded in prompts |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**PRS/PROMETHEUS answers: Is this prompt versioned, tested against analytic standards, and approved for this analytical context — and when the model changes, does it need to be re-evaluated?**

### 1.2 Why This Service Exists

Prompts encode analytical methodology. A prompt for "assess adversary technical capability" embeds assumptions about what evidence to weight, what uncertainty to express, how to structure the assessment, and what to caveat. These embedded methodological choices are analytically consequential — and without a governed repository, they proliferate without testing, vary without version control, and become invisible to oversight.

The failure mode without PRS is predictable: senior analysts develop effective prompts and share them informally by message. Six months later there are forty unversioned variants of the same prompt, no one knows which ones have been tested, and the MOIRAI provenance record shows "the analyst used a prompt" but cannot reproduce which one. The accountability chain has a gap exactly where it matters most — in the methodology that shaped how the AI approached the analytical question.

PRS/PROMETHEUS closes this gap. Every prompt used in ATHENA has a version hash. Every session turn records which prompt version produced it. The analytical methodology is part of the provenance record.

The mutation hint mapping is the operational design insight from production engineering experience: when a prompt fails at the response validation layer (IAS blocks output, CVS finds a citation error, PGS flags a standards violation), the retry should inject a targeted correction — not re-run the same prompt. PRS stores the failure_mode→mutation_hint mappings for each prompt. Retries are intelligent, not repetitive.

### 1.3 Why This Service Is Nineteenth

PRS is Phase 1-2 because the informal prompt economy begins from day one. If PRS is not operational when analysts start using ATHENA, prompts will proliferate without governance and retrofitting version control onto an existing uncontrolled corpus is much harder than building governance from the start. PRS must be the first Interaction Layer service.

### 1.4 Design Principles

- **Every prompt in use has a MOIRAI-attested version hash.** There is no unversioned prompt use in ATHENA. A prompt without a PRS version record cannot be used.
- **Testing is mandatory before shared promotion.** A prompt can be used personally without testing. It cannot be promoted to team, organization, or platform tier without passing the testing pipeline.
- **Model changes trigger re-evaluation.** A prompt tested against model v3.1 has not been tested against model v3.2. MDS/KRONOS model version changes trigger re-evaluation flags on all shared prompts. Prompts that pass re-evaluation are recertified; prompts that fail are downgraded until re-evaluated.
- **Failure mode mutation hints are first-class content.** Each prompt version carries a mapping from common failure modes (SCHEMA_VIOLATION, CONSTRAINT_VIOLATION, STANDARDS_VIOLATION) to targeted correction hints. This is the design element that makes retries effective rather than repetitive.
- **Analytic standards authority owns shared promotion approval.** Promotion to organization or platform tier requires the analytic standards authority to attest the prompt complies with ICD 203 and applicable analytic standards. The platform team manages the technical infrastructure; the analytic standards authority controls what methodology reaches the shared library.

### 1.5 Explicit Out of Scope

- **Prompt execution.** PRS stores and governs prompts. ATHENA and the inference gateway execute them.
- **Skill composition.** SKS/DAEDALUS composes prompts with tools and methodology definitions. PRS provides the prompt component; SKS provides the composition.
- **MCP tool configuration.** MGS/TERMINUS governs MCP access; PRS governs prompt content.

---

## 2. Core Responsibilities

### 2.1 Primary Function

PRS/PROMETHEUS maintains a versioned library of analytical prompts — from personal drafts through organisation-wide approved prompts — with mandatory testing pipeline for shared promotion, version hash logging to MOIRAI on every use, re-evaluation triggering on model version changes, and failure-mode-to-mutation-hint mappings for intelligent retry handling.

### 2.2 Secondary Functions

- Testing pipeline orchestration: coordinating calibration testing, ICD 203 compliance screening through PGS/NOMOS, and adversarial robustness evaluation through IAS/SCUDO
- Session prompt record: recording which prompt version was used in each turn for MOIRAI provenance
- Re-evaluation management: tracking which prompts need re-evaluation following model version changes and routing them for re-testing
- Prompt analytics: aggregate statistics on prompt performance by interaction class for Research & Red Team analysis
- Deprecation management: marking prompts as deprecated when they underperform or are superseded, with controlled migration paths

### 2.3 What This Service Does Not Decide

PRS governs prompt versions and testing. Whether a specific prompt should be used for a specific analytical task, whether a prompt that passes technical testing is analytically appropriate for a specific domain, and whether a deprecated prompt may still be used in exceptional circumstances are analytical and management decisions.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
PromptVersion:
  version_id:              uuid
  prompt_code:             str              # stable identifier across versions (e.g., "cap-assess-v")
  version_hash:            str              # SHA-256 of prompt content
  version_number:          int
  content:                 str              # the prompt text
  sharing_tier:            PERSONAL | TEAM | ORGANIZATION | PLATFORM
  interaction_class:       str | null       # PGS interaction class this prompt targets
  claim_types:             [str]            # claim types this prompt is designed for
  failure_mode_hints:      [FailureModeHint]
  approved_by:             str | null       # analytic standards authority reference
  test_results:            PromptTestResult | null
  model_version_certified: str | null       # model version the test was run against
  needs_re_evaluation:     bool
  created_by:              str              # analyst ID hash
  created_at:              datetime
  status:                  DRAFT | TESTING | CERTIFIED | DEPRECATED

FailureModeHint:
  mode:                    str              # failure mode identifier (from IAS/SCUDO, PGS, CVS)
  hint:                    str              # the correction injected on retry
  # Example: mode="STANDARDS_VIOLATION", hint="Re-read the analytic standards constraints.
  #           Your response must explicitly caveat uncertainty for all analytical judgments."

PromptTestResult:
  result_id:               uuid
  version_id:              uuid
  model_version:           str
  calibration_score:       float            # accuracy on known test cases
  standards_compliance:    bool             # PGS/NOMOS ICD 203 compliance
  adversarial_score:       float            # IAS/SCUDO robustness score
  citation_accuracy:       float | null     # CVS accuracy on test citations
  overall_passed:          bool
  tested_by:               str              # who ran the test (Research & Red Team or automated)
  tested_at:               datetime
  notes:                   str | null

PromptSessionRecord:
  record_id:               uuid
  session_id:              uuid
  turn_id:                 uuid
  version_id:              uuid
  version_hash:            str              # hash logged to MOIRAI for provenance
  used_at:                 datetime
  retry_count:             int              # how many retries this prompt needed
  failure_modes_triggered: [str]           # failure modes that triggered mutation hints
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | PromptVersion, PromptTestResult, PromptSessionRecord | Indefinite |
| Catalog cache | Redis | Active CERTIFIED prompts by interaction_class (hot path for ATHENA selection) | 1h TTL + invalidation |
| Event store | MOIRAI | Signed prompt use events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| PromptVersion (content) | Controlled Unclassified | Platform-wide for CERTIFIED; tier-restricted for lower |
| PromptSessionRecord | Inherits session classification | Session-compartmented |
| PromptTestResult | Controlled Unclassified | Research & Red Team access |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /prompts
  Auth:     analyst session token
  Request:  {
    prompt_code:           str,
    content:               str,
    interaction_class:     str | null,
    claim_types:           [str],
    failure_mode_hints:    [FailureModeHint]
  }
  Response: { version_id: uuid, version_hash: str }

POST /prompts/{version_id}/test
  Auth:     Research & Red Team token | platform operator
  Request:  { model_version: str }
  Response: { test_id: uuid, estimated_completion: str }

POST /prompts/{version_id}/promote
  Auth:     analytic standards authority token (for ORGANIZATION/PLATFORM)
             analyst supervisor token (for TEAM)
  Request:  { target_tier: str, approval_ref: str | null }
  Response: { version_id: uuid, new_tier: str }

GET /prompts/catalog
  Auth:     analyst session token
  Params:   interaction_class: str, sharing_tier: str (min)
  Response: [{ version_id, prompt_code, version_hash, interaction_class, claim_types, status }]
  SLA: p99 < 100ms (from Redis cache)

GET /prompts/{version_id}
  Auth:     session token (must have access to tier)
  Response: PromptVersion (content only for PERSONAL/TEAM/own; abstracted for ORGANIZATION+)

POST /sessions/record-prompt
  Auth:     ATHENA service account
  Request:  { session_id, turn_id, version_id }
  Response: { record_id: uuid }
  SLA: p99 < 100ms

GET /prompts/{version_id}/hint/{failure_mode}
  Auth:     inference gateway service account
  Response: { hint: str }
  SLA: p99 < 50ms (for retry logic)

GET /prompts/re-evaluation-needed
  Auth:     platform operator | Research & Red Team
  Response: [{ version_id, sharing_tier, last_model_version_tested, current_model_version }]

GET /health
  Response: {
    status, dependencies: { moirai, pcs, redis },
    certified_prompts:     int,
    needs_re_evaluation:   int,
    active_catalog_version:str,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          PRS_PROMPT_USED
service_id:         "PRS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  version_id:             uuid
  version_hash:           str
  prompt_code:            str
  sharing_tier:           str
  interaction_class:      str | null
  retry_count:            int
  failure_modes_triggered:[str]
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          PRS_PROMPT_CERTIFIED
event_payload:
  version_id:             uuid
  model_version_certified:str
  calibration_score:      float
  standards_compliant:    bool

EventType:          PRS_PROMPT_DEPRECATED
event_payload:
  version_id:             uuid
  reason:                 str
  successor_version_id:   uuid | null
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `PRS_PROMPT_USED` | Every session turn that uses a prompt | MOIRAI (provenance), TCS/MIMIR (prompt quality signal) |
| `PRS_PROMPT_CERTIFIED` | Prompt passes testing pipeline | MOIRAI, catalog cache refresh |
| `PRS_RE_EVALUATION_NEEDED` | MDS/KRONOS model version change | MOIRAI, Research & Red Team alert |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | Flags all CERTIFIED prompts `needs_re_evaluation = true` |
| PGS/NOMOS | `PGS_OUTPUT_BLOCKED` | Records failure mode in PromptSessionRecord; mutation hint surfaced on retry |
| IAS/SCUDO | `IAS_INPUT_BLOCKED` | Records adversarial failure mode; mutation hint surfaced on retry |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed prompt use events | Async event | Events buffered; session record still created |
| PGS/NOMOS | Analytic Standards | Standards compliance testing in pipeline | Sync (during test) | Testing deferred; prompt remains DRAFT |
| IAS/SCUDO | Adversarial Screening | Adversarial robustness scoring | Sync (during test) | Testing deferred |
| MDS/KRONOS | Model Drift | Model version change triggers | Async event | Re-evaluation flagging deferred |

### 5.2 Feeds Into

| Service | Epithet | What PRS provides | How |
|---|---|---|---|
| ATHENA | Interface | Prompt catalog for session configuration; version hash for provenance | API |
| SKS/DAEDALUS | Skill Registry | Prompt versions used in skills | API reference |
| Inference gateway | N/A | Mutation hints for intelligent retry | `GET /prompts/{id}/hint/{mode}` |
| MOIRAI | Provenance | Version hash per session turn | `PRS_PROMPT_USED` event |
| IOB Reporting | Oversight | Prompt certification status; re-evaluation backlog | Audit endpoints |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Catalog lookup (cached) | 5ms | 20ms | 100ms |
| Session record | 20ms | 60ms | 100ms |
| Mutation hint fetch | 5ms | 15ms | 50ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — PRS unavailability means prompt selection falls back to last cached catalog |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst (own) | PERSONAL tier; browse TEAM+ catalog | Session token |
| Supervisor | TEAM promotion approval | Supervisor token |
| Analytic standards authority | ORGANIZATION/PLATFORM promotion | Authority token |
| Research & Red Team | Testing pipeline execution; full catalog | Research team token |
| Inference gateway | Mutation hint fetch; session record | Service account |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Re-evaluation backlog grows (model version change, many prompts to re-test) | Medium | P2 — certified prompts used with stale model version | Re-evaluation backlog size monitoring | Automated re-test pipeline with Research & Red Team capacity planning |
| Prompt certification gaming (testing with easy cases) | Low | P2 — certified prompt underperforms on hard cases | Calibration score distribution monitoring | Test set managed by Research & Red Team; test cases are not visible to prompt authors |

### 8.1 Known Design Risks

- **The testing pipeline requires Research & Red Team bandwidth that may not be available at scale.** As the prompt library grows, the volume of prompts needing testing (especially after model version changes) may exceed Research & Red Team capacity. Resolution path: automated testing for calibration and standards compliance; human Research & Red Team required only for adversarial robustness (the hardest component). Automated testing covers the majority of the pipeline.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `prs.catalog.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `prs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `prs.re_evaluation.backlog` | Gauge | `> 20` prompts needing re-evaluation | P2 |
| `prs.mutation_hint.success_rate` | Gauge | `< 50%` (hints not reducing retries) | P2 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/prs/signing-key`
- **Chain participation:** Yes
- **What it attests:** The version hash of every prompt used in every analytical session is permanently recorded. An oversight body can reconstruct the exact prompt methodology used in any session. The provenance record includes not just what the AI said but how it was prompted to reason.

---

## 11. Implementation Roadmap

### Phase 1 — Versioning and Session Record (Weeks 1–4)

- PromptVersion schema with version_hash and failure_mode_hints
- Prompt creation, listing, and session record endpoints
- Redis catalog cache for CERTIFIED prompts
- MOIRAI event emission: `PRS_PROMPT_USED`
- ATHENA session configuration integration (prompt selection)

**Phase gate criterion:** Every ATHENA session turn produces a PromptSessionRecord with a MOIRAI-attested version hash. Mutation hints served correctly to inference gateway.

### Phase 2 — Testing Pipeline, Promotion, and Re-Evaluation (Weeks 5–8)

- Testing pipeline: calibration scoring, PGS standards compliance, IAS robustness scoring
- Promotion workflow with analytic standards authority approval for ORGANIZATION/PLATFORM tier
- MDS/KRONOS re-evaluation trigger integration
- Re-evaluation backlog management
- Deprecation workflow

**Phase gate criterion:** Testing pipeline produces pass/fail for test prompt set. Promotion workflow logs to MOIRAI. Re-evaluation flags set correctly on model version change. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate PRS. Shared promotion approval (ORGANIZATION/PLATFORM tier) is an analytic standards authority policy responsibility, not a GC policy decision.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In ATHENA session configuration, the analyst selects from a catalog of approved prompts for the interaction class. Prompts show: sharing tier, interaction class, claim types, certification status, and model version certified against. A NEEDS_RE_EVALUATION indicator shows when a certified prompt was tested against an older model version. Personal draft prompts show a "Not tested" indicator.

### 13.2 What the Analyst Should Do

Use CERTIFIED prompts from the shared catalog where available for the interaction class. Develop personal prompts in PERSONAL tier first; submit for testing when satisfied with performance; promote to TEAM tier with supervisor approval. If a prompt consistently triggers retry due to standards violations, check the failure_mode_hints and revise the prompt content before re-submitting for testing.

---

## 14. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — incorporates mutation hint pattern from production engineering research |

## Appendix D: Red Team Findings
*Pending — Phase 1 gate review.*
