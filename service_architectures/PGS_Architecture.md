# PGS — Policy & Guardrails Service
### NOMOS · *"Greek for 'law as convention'"*
*Part of the THEMIS Platform · Safety Gate Layer · Build Priority: 2*

---

## Design Philosophy

The PCES governs **data** — what content can enter an AI context. The Policy and Guardrails Service (PGS) governs **interactions** — what kinds of queries and use patterns are permitted regardless of the data involved. A prompt can be impermissible even when all the data it references is appropriately scoped.

> **Interaction Governance Is Policy, Not Code.** The PGS expresses the firm's AI usage policy as executable rules. When bar association guidance changes, when a new jurisdiction's rules apply, when a practice group establishes new norms — those changes are configuration updates to the policy rule engine, not software releases.

---

## Architecture Overview

```
Incoming Prompt (from analyst)
    │
    ▼
┌────────────────────────────────────────────────────────┐
│            Policy & Guardrails Service                 │
│                                                        │
│  PRE-PROMPT EVALUATION                                 │
│  ┌──────────────────┐  ┌─────────────┐  ┌──────────┐  │
│  │ Interaction      │  │ PII / Sens. │  │ Policy   │  │
│  │ Classifier       │  │ Detector    │  │ Rule     │  │
│  │                  │  │             │  │ Engine   │  │
│  └──────┬───────────┘  └──────┬──────┘  └────┬─────┘  │
│         └────────────────────┬┘               │       │
│                              ▼                │       │
│                   ┌──────────────────┐        │       │
│                   │ Enforcement Engine│◄───────┘       │
│                   │ PASS/WARN/BLOCK   │                │
│                   └────────┬─────────┘                │
│                            │                           │
│  POST-GENERATION EVALUATION│                           │
│                   ┌────────▼──────────┐               │
│                   │ Output Screener    │               │
│                   │ (before delivery)  │               │
│                   └────────────────────┘               │
│                                                        │
│                   ┌────────────────────┐               │
│                   │ Policy Audit Log    │               │
│                   └────────────────────┘               │
└────────────────────────────────────────────────────────┘
```

---

## Policy Rule Engine

```yaml
PolicyRule:
  rule_id:         uuid
  name:            str
  version:         semver
  priority:        int          # lower = evaluated first
  scope:           global | practice_group | matter_type | jurisdiction
  trigger:         interaction_type | pii_detected | keyword_match | pattern
  condition:       CEL expression  # Common Expression Language — auditable
  action:          BLOCK | WARN | TRANSFORM | REQUIRE_APPROVAL
  action_params:   { message, redirect, approver_role, transform_fn }
  effective_from:  ISO8601
  effective_until: ISO8601 | null
  authored_by:     user_id
  approved_by:     user_id
  rationale:       str          # required
```

### Rule Categories

| Category | Description |
|---|---|
| Interaction Type | Blocks/warns on specific use patterns: third-party legal advice, UPL risk, system prompt extraction, jailbreak patterns |
| Data Sensitivity | Governs PII, PHI, financial data appearing in prompts regardless of privilege status |
| Jurisdictional | Practice-area and jurisdiction-specific rules; vary for litigation vs. transactional, US vs. international |
| Output Quality | Post-generation: flags fabricated citations, unqualified legal advice, low-validity source reliance |
| Usage Pattern | Detects anomalous patterns: volume spikes, repeated reformulation of blocked prompts, off-hours activity |

---

## Interaction Classification

| Class | Description | Enforcement |
|---|---|---|
| `research` | Information gathering, case background, legal research | Lowest restriction |
| `document_drafting` | Generating work product: briefs, contracts, memos | Medium; output screener active |
| `evidence_analysis` | Analyzing exhibits, depositions, financial records | PCES enforcement active; TVS scores surfaced |
| `legal_advice` | Generating advice or recommendations | Highest scrutiny; recipient context required |
| `third_party_facing` | Content destined for unrepresented party | Requires attorney review gate |
| `system_probe` | Interrogating model configuration or attempting policy bypass | BLOCK |
| `administrative` | Scheduling, file management, non-legal tasks | Minimal restriction |

**Confidence thresholds:** High-confidence classifications trigger enforcement directly. Low-confidence classifications of high-risk types (legal_advice, system_probe) trigger enforcement conservatively — when in doubt about a dangerous category, treat it as that category.

---

## PII & Sensitive Data Detection

| PII Type | Examples | Default Action |
|---|---|---|
| Standard PII | SSN, DOB, financial account numbers, government IDs | WARN + log; require confirmation |
| PHI | Medical record numbers, diagnoses, treatment data | BLOCK unless health law matter; require attorney approval |
| Third-Party PII | PII of individuals not party to the active matter | WARN; flag for data governance review |
| Case-Sensitive | Witness identity in sealed matters, protected sources | BLOCK; alert matter supervisor |
| Opposing Counsel | Communications from or about opposing counsel | FLAG; CoI check triggered |

---

## Runtime Evaluation Pipeline

**Pre-prompt:**
```
1. PII & sensitive data detection  (parallel)
2. Interaction classification       (parallel)
3. Load applicable policy rules     (scoped to: user role, matter type, jurisdiction)
4. Evaluate rules in priority order
   → First BLOCK rule matched: reject; return PolicyViolationEvent
   → WARN rules accumulated: proceed with warnings attached to turn record
   → REQUIRE_APPROVAL: queue for attorney review; block until approved
5. Log PolicyEvaluationEvent (regardless of outcome)
6. PASS → forward to PCES → forward to retrieval layer
```

**Post-generation Output Screener:**
```yaml
OutputScreenerResult:
  pass:               bool
  flags: [
    { type: FABRICATED_CITATION,  span: [start,end], citation: str },
    { type: UPL_RISK,             span: [start,end], jurisdiction: str },
    { type: UNQUALIFIED_ADVICE,   span: [start,end], confidence: float },
  ]
  action:   DELIVER | DELIVER_WITH_WARNING | HOLD_FOR_REVIEW | BLOCK
```

---

## HITL Workflow Orchestration

### Two Distinct Mechanisms

| Type | Rule Action | Description | Examples |
|---|---|---|---|
| **Supervised Gate** | REQUIRE_APPROVAL | AI generation held pending human review. Reviewer can approve, modify, or reject. | Ambiguous interaction class, privilege flag, budget soft ceiling |
| **Hard Stop** | BLOCK — no override | System prohibited from proceeding. Requires formal out-of-band process involving ethics counsel. | Active adverse-party CoI on current client, sealed matter violation, budget hard ceiling |

### Hold Queue Schema

```yaml
HoldRecord:
  hold_id:             uuid
  turn_id:             uuid           # FK → Provenance Service turn record
  session_id:          uuid
  analyst_id:          uuid
  trigger_rule_id:     uuid
  trigger_category:    privilege | policy | pii | budget | coi_flag
  interaction_class:   str
  matter_id:           uuid
  prompt_assembly_id:  uuid           # FK → PromptAssembly (what would be sent)
  created_at:          ISO8601
  sla_deadline:        ISO8601
  assigned_to:         user_id | null
  status:              open | approved | modified | rejected | escalated
  resolution:          HoldResolution | null
```

### Reviewer Assignment Matrix

| Trigger | Interaction Class | Assigned Reviewer |
|---|---|---|
| privilege | any | Supervising attorney on matter |
| coi_flag | any | Ethics counsel — cannot be delegated |
| policy | legal_advice | Senior partner + practice group lead |
| policy | third_party_facing | Supervising attorney on matter |
| pii | any | Matter supervisor |
| budget | any | Practice group finance lead |

### SLA & Escalation

| Trigger | SLA Target | Escalation To | On Breach |
|---|---|---|---|
| privilege | 2 business hours | Ethics counsel | Hold auto-rejected; analyst notified |
| coi_flag | 30 minutes | General Counsel | Hard stop activated |
| policy | 4 business hours | Practice group lead | Hold escalated |
| pii | 4 business hours | Matter supervisor | Escalated to data governance lead |
| budget | 1 business day | CFO or delegate | Budget hard ceiling activated |

### HoldResolution Schema

```yaml
HoldResolution:
  resolution_id:       uuid
  hold_id:             uuid
  turn_id:             uuid
  reviewer_id:         uuid
  resolved_at:         ISO8601
  decision:            approved | modified | rejected
  rationale:           str            # required; minimum 20 characters
  modified_prompt_id:  uuid | null    # if decision == modified
  provenance_event_id: uuid           # written to Provenance Service; immutable
```

---

## Policy Authoring & Governance

| Rule Scope | Approval Requirement | Applicability |
|---|---|---|
| Global rules | AI Governance Committee + General Counsel | Affects all users and matters |
| Practice group rules | Practice group leader + one peer review | Scoped to practice group |
| Matter-type rules | Senior partner + practice group leader | Scoped to matter type |
| Emergency rules | General Counsel unilateral; 48h review required | Immediate effect; auto-expires 30 days |

**Testing framework:** Every rule change must pass a test suite before production — regression set of known-good interactions that must still pass, known-violation set that must still trigger, and human review set of ambiguous cases the rule author must manually adjudicate.

**HADES co-authorship:** Every PGS rule must have a corresponding HADES probe scenario authored at the same time and committed in the same pull request.

---

## Implementation Roadmap

### Phase 1 — Core Pipeline (Weeks 1–6)
- Interaction classifier v1: research, document_drafting, evidence_analysis, system_probe
- Basic policy rule engine: CEL expression evaluation, priority ordering
- PII detection: standard PII and PHI categories
- PolicyEvaluationEvent wired to Provenance Service turn records
- Pre-prompt BLOCK and WARN enforcement with analyst-facing explanations

### Phase 2 — Output Screening & Governance (Weeks 7–14)
- Output Screener: fabricated citation detection, UPL risk flagging
- Policy authoring UI: rule builder, approval workflow, version history
- REQUIRE_APPROVAL → HITL Hold Queue: hold records, SLA tracking, approval recording
- Testing framework: regression suite, promotion gating
- Jurisdictional rule scoping: matter-type and jurisdiction-aware rule loading

### Phase 3 — Advanced Classification & Patterns (Weeks 15–20)
- `legal_advice` and `third_party_facing` classifiers
- Usage pattern anomaly detection: volume spikes, repeated bypass attempts
- TCS integration: interaction class fed to calibration scoring
- Emergency rule mechanism with auto-expiry

---

**Depends on:** PCES / AEGIS
**Feeds into:** PROV / MOIRAI (PolicyEvaluationEvents), TCS / MIMIR (interaction class), FGS / PLUTUS (budget integration)
