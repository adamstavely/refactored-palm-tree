# SKS — Skill Registry Service
### DAEDALUS · *"Greek craftsman who built the labyrinth and crafted the wings of Icarus — the architect of functional frameworks that give capability its form"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `SKS` |
| **Epithet** | `DAEDALUS` |
| **Full name** | Skill Registry Service |
| **Namespace** | `themis-interaction` |
| **Layer** | Interaction Layer |
| **Build phase** | Phase 2–3 (Weeks 5–28) |
| **Build priority** | 20 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — governs reusable analytical methodology expressed as versioned skills |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**SKS/DAEDALUS answers: Is this analytical skill versioned, tested for adversarial robustness, rated by real-world outcome quality, and appropriate for this session's context — and when this analyst leaves, does their methodological expertise survive?**

### 1.2 Why This Service Exists

A prompt is what you say to the AI. A skill is how an expert analyst approaches a problem — the full methodology: what sources to query first, how to structure the analytical chain, what uncertainty to express and when, how to synthesise partial evidence across collection types, what follow-on questions to generate.

Senior analysts develop effective analytical approaches over years. Without SKS, that expertise lives in individual heads and informal notes. When the analyst leaves, the methodology leaves with them. With SKS, the methodology is encoded as a tested, versioned, rated skill that junior analysts can use immediately — with the same quality guarantees as the original expert's approach.

Skills also encode what prompts alone cannot: tool configurations (which MCP servers to call and in what order), methodology definitions (step-by-step analytical workflow), input/output schemas (what the skill expects and what it produces). A skill is a composable unit of analytical capability, not just a communication artifact.

The outcome quality rating — driven by OFS/NEMESIS data — is the design element that makes SKS a self-improving system rather than a static library. Skills that consistently produce assessments confirmed by real-world outcomes rise in rating. Skills associated with disconfirmed assessments are flagged for review. The shared library gets better as the platform accumulates outcome data.

### 1.3 Why This Service Is Twentieth

SKS requires PRS to be operational (skills reference PRS prompt versions), MGS/TERMINUS for tool configuration validation, and OFS/NEMESIS (Year 2) for outcome quality ratings. Phase 2-3 is appropriate because the platform needs to be generating sessions and assessments before skill quality can be measured or the shared library becomes meaningful.

### 1.4 Design Principles

- **Skills are more than prompts.** A skill is the combination of a prompt, tool configurations, a methodology definition, and input/output schemas. Reducing a skill to a prompt is a design error.
- **The graduated sharing model creates quality pressure.** PERSONAL → TEAM → ORGANIZATION → PLATFORM is not bureaucracy — it is a quality escalation path. Each tier requires additional review and carries additional validation. Analysts who want to reach PLATFORM tier have a genuine incentive to produce high-quality skills.
- **Outcome rating is the most credible quality signal.** Expert opinion that a skill is good is less credible than data showing that assessments produced using this skill were confirmed by real-world outcomes. OFS/NEMESIS outcome data is the signal that cannot be gamed.
- **STOA is the primary consumer.** Skills are not just for human analysts selecting a methodology manually — STOA research orchestration draws on the skill registry to compose analytical workflows. STOA-invoked skills need the same quality guarantees as analyst-selected skills.
- **Deprecation is active, not passive.** Skills that underperform in outcome ratings or that have been superseded by better approaches are actively deprecated and migrated, not left to slowly fall out of use.

### 1.5 Explicit Out of Scope

- **Skill execution.** SKS stores and governs skills. STOA, ATHENA, and agents execute them.
- **Prompt content.** PRS/PROMETHEUS governs the prompt component of a skill. SKS references prompt versions by version_id but does not store prompt content.
- **MCP server management.** MGS/TERMINUS governs MCP server registration and access. SKS references MCP tools by server_name and operation; it does not manage MCP server security.

---

## 2. Core Responsibilities

### 2.1 Primary Function

SKS/DAEDALUS maintains a versioned registry of analytical skills — from personal methodology drafts through organisation-wide approved frameworks — with graduated sharing governance, red team adversarial robustness evaluation for shared tier promotion, outcome quality rating from OFS/NEMESIS data, and MOIRAI-attested version hashes logged on every skill invocation. Skills in the registry are the primary input to STOA research orchestration and are available for manual selection by analysts in ATHENA.

### 2.2 Secondary Functions

- Skill invocation recording: logging every skill invocation to MOIRAI with the version hash, who invoked it, and the session context
- Outcome quality rating: computing and updating per-skill quality scores from OFS/NEMESIS outcome data
- Red team evaluation scheduling: routing skills to the Research & Red Team for adversarial robustness assessment before ORGANIZATION/PLATFORM promotion
- STOA integration: providing skill catalog and invocation interfaces for research orchestration
- Skill composition validation: checking that the prompt version, MCP tool configurations, and methodology definition referenced in a skill are compatible and current
- Deprecation management with successor mapping: ensuring deprecated skills have documented successors and migration paths

### 2.3 What This Service Does Not Decide

SKS governs skill versions, quality ratings, and sharing tiers. Whether a specific skill is analytically appropriate for a specific requirement, whether a skill with a low outcome rating may still be used in a specific context, and whether the outcome data for a skill reflects the skill quality or the difficulty of the underlying analytical problem are human analytical decisions.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
Skill:
  skill_id:                uuid
  skill_code:              str              # stable identifier across versions
  name:                    str
  description:             str
  version:                 str
  version_hash:            str              # SHA-256 of skill definition

  # Composition
  prompt_version_id:       uuid | null      # FK → PRS/PROMETHEUS PromptVersion
  tool_config:             [McpToolConfig]
  methodology_definition:  str              # step-by-step analytical workflow in structured text
  input_schema:            {}               # what the skill expects as input
  output_schema:           {}               # what the skill produces

  # Governance
  sharing_tier:            PERSONAL | TEAM | ORGANIZATION | PLATFORM
  approved_by:             str | null
  red_team_score:          float | null     # adversarial robustness from Research & Red Team
  outcome_quality_rating:  float | null     # 0.0–1.0 from OFS/NEMESIS outcome data
  outcome_sample_size:     int              # how many outcomes inform the rating
  standards_compliant:     bool

  # Context
  interaction_class:       str | null
  domain:                  str | null
  primary_claim_types:     [str]

  # Metadata
  created_by:              str              # analyst ID hash
  created_at:              datetime
  successor_skill_id:      uuid | null      # if deprecated, what replaces this
  status:                  DRAFT | REVIEW | APPROVED | DEPRECATED

McpToolConfig:
  server_name:             str
  operations_used:         [str]
  configuration:           {}               # server-specific configuration
  validated:               bool             # has MGS validated this config is accessible?

SkillInvocationRecord:
  record_id:               uuid
  session_id:              uuid
  turn_id:                 uuid
  skill_id:                uuid
  version_hash:            str
  invoked_by:              ANALYST | STOA | AGENT
  invocation_context:      str              # analytical context for this invocation
  invoked_at:              datetime
  outcome_linkage:         uuid | null      # FK → OFS/NEMESIS OutcomeRecord if available

SkillOutcomeRating:
  rating_id:               uuid
  skill_id:                uuid
  confirmed_count:         int
  disconfirmed_count:      int
  ambiguous_count:         int
  invocation_count:        int              # total invocations with any outcome data
  quality_score:           float            # confirmed / (confirmed + disconfirmed)
  last_updated:            datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | Skill, SkillInvocationRecord, SkillOutcomeRating | Indefinite |
| Catalog cache | Redis | APPROVED skills by interaction_class and domain | 1h TTL + invalidation |
| Event store | MOIRAI | Signed invocation and rating events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| Skill (definition) | Controlled Unclassified | Tier-restricted for PERSONAL/TEAM; platform-wide for ORGANIZATION/PLATFORM |
| SkillInvocationRecord | Inherits session classification | Session-compartmented |
| SkillOutcomeRating | Controlled Unclassified | Research & Red Team and IOB access |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /skills
  Auth:     analyst session token
  Request:  {
    skill_code:            str,
    name:                  str,
    description:           str,
    prompt_version_id:     uuid | null,
    tool_config:           [McpToolConfig],
    methodology_definition:str,
    input_schema:          {},
    output_schema:         {},
    interaction_class:     str | null,
    domain:                str | null
  }
  Response: { skill_id: uuid, version_hash: str }

POST /skills/{skill_id}/promote
  Auth:     supervisor token (TEAM) | analytic standards authority + red_team_score (ORGANIZATION/PLATFORM)
  Request:  { target_tier: str, approval_ref: str | null }
  Response: { skill_id: uuid, new_tier: str }

POST /skills/{skill_id}/invoke
  Auth:     ATHENA service account | STOA service account | analyst session token
  Request:  {
    session_id:            uuid,
    turn_id:               uuid,
    invoked_by:            str,
    invocation_context:    str
  }
  Response: { record_id: uuid, version_hash: str }
  SLA: p99 < 200ms

GET /skills/catalog
  Auth:     analyst session token | STOA service account
  Params:   interaction_class: str, domain: str, tier_min: str
  Response: [{ skill_id, skill_code, name, sharing_tier, outcome_quality_rating, version_hash }]
  SLA: p99 < 100ms (from Redis cache)

GET /skills/{skill_id}
  Auth:     session token (tier-appropriate)
  Response: Skill with full definition

GET /skills/{skill_id}/outcome-rating
  Auth:     any service account | supervisor token
  Response: SkillOutcomeRating

GET /health
  Response: {
    status, dependencies: { moirai, pcs, redis },
    approved_skills:       int,
    pending_red_team:      int,
    outcome_rated_skills:  int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          SKS_SKILL_INVOKED
service_id:         "SKS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  skill_id:               uuid
  version_hash:           str
  sharing_tier:           str
  invoked_by:             str
  interaction_class:      str | null
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          SKS_OUTCOME_RATING_UPDATED
event_payload:
  skill_id:               uuid
  quality_score:          float
  confirmed_count:        int
  disconfirmed_count:     int
  sample_size:            int
```

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| OFS/NEMESIS | `OFS_OUTCOME_CLASSIFIED` | Links outcome to SkillInvocationRecord; updates SkillOutcomeRating |
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | Flags skills referencing affected prompt versions for re-evaluation |
| PRS/PROMETHEUS | `PRS_PROMPT_DEPRECATED` | Flags skills referencing deprecated prompt version for review |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| PRS/PROMETHEUS | Prompt Repository | Prompt version validation in skill composition | Sync (on create/promote) | Skill creation fails if prompt version invalid |
| MGS/TERMINUS | MCP Gateway | Tool configuration validation for skills | Sync (on create) | Tool configs marked as unvalidated |
| MOIRAI | Provenance | Invocation events | Async event | Events buffered; invocation still recorded in PostgreSQL |
| OFS/NEMESIS | Outcome Feedback | Outcome quality rating updates | Async event | Ratings not updated until OFS recovers |

### 5.2 Feeds Into

| Service | Epithet | What SKS provides | How |
|---|---|---|---|
| STOA | Research Orchestration | Skill catalog for workflow composition | API |
| ATHENA | Interface | Skill catalog for manual analyst selection | API |
| IOB Reporting | Oversight | Skill invocation history; outcome quality trends | Audit endpoint |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Catalog lookup (cached) | 5ms | 20ms | 100ms |
| Skill invocation record | 30ms | 80ms | 200ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — SKS unavailability degrades STOA orchestration and manual skill selection |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst (own) | PERSONAL; browse TEAM+ catalog | Session token |
| STOA | Catalog read; invocation record | Service account |
| Supervisor | TEAM promotion approval | Supervisor token |
| Research & Red Team | Adversarial evaluation; full catalog | Research team token |
| Analytic standards authority | ORGANIZATION/PLATFORM promotion | Authority token |
| IOB | Full audit | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Low outcome sample size (skills promoted before sufficient outcome data) | High initially | P2 — outcome_quality_rating unreliable | Sample size threshold display | Display sample_size alongside rating; minimum sample size requirement for platform tier |
| Skill composition drift (prompt version deprecated while skill is active) | Medium | P2 — skill references deprecated methodology | PRS deprecation event handling | Automatic flag when referenced prompt is deprecated; require skill re-validation |

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `sks.catalog.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `sks.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `sks.outcome_rating.low_skill_count` | Gauge | Skills with quality_score < 0.3 in PLATFORM tier | P2 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/sks/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every skill invocation — including the version hash of the skill and who invoked it — is permanently recorded. The analytical methodology used in any session is fully auditable from the MOIRAI chain.

---

## 11. Implementation Roadmap

### Phase 1 — Registry and Invocation Record (Weeks 5–16)

- Skill schema with version_hash and composition validation
- Skill creation, promotion, and invocation endpoints
- Redis catalog cache
- MOIRAI invocation events
- STOA and ATHENA integration

**Phase gate criterion:** Every skill invocation produces a MOIRAI-attested record. Catalog serves correctly filtered results. STOA can query and invoke skills.

### Phase 2 — Outcome Rating, Red Team Pipeline, and Deprecation (Weeks 17–28)

- OFS/NEMESIS outcome linkage and SkillOutcomeRating computation
- Red team adversarial robustness evaluation routing for ORGANIZATION/PLATFORM promotion
- Deprecation workflow with successor mapping
- PRS deprecation event handling (flags skills referencing deprecated prompts)

**Phase gate criterion:** Outcome ratings computed and updated for test skills with OFS outcome data. Red team evaluation gates ORGANIZATION tier promotion. Deprecation produces MOIRAI event with successor. ARB sign-off.

---

## 12. Policy Dependencies

No GC items gate SKS. Shared promotion governance (ORGANIZATION/PLATFORM) is an analytic standards authority responsibility.

---

## 13. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 2 gate review.*
