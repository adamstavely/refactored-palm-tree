# CPS — Capability Profiling Service
### APORIA · *"Greek for 'impasse' or 'being at a loss' — the philosophical state of recognising the limits of one's knowledge; what is known about where knowing stops"*
*THEMIS Platform · Service PRD · v1.0 · **PROPOSED — Pending ARB Approval***

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `CPS` |
| **Epithet** | `APORIA` |
| **Full name** | Capability Profiling Service |
| **Namespace** | `themis-quality` (proposed) |
| **Layer** | Quality Layer — Competence Axis (proposed) |
| **Build phase** | Proposed — Year 2+ pending ARB approval |
| **Build priority** | Proposed — would be 24th service if approved |
| **Owner team** | THEMIS Platform Team |
| **Status** | **PROPOSED** |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | **Competence (proposed 4th axis)** — answers whether the AI is reliably capable of the task |

---

> **Proposal status note:** CPS/APORIA is a proposed extension to the THEMIS platform representing the Competence Axis — a fourth accountability axis beyond Origin, Currency, and Trust. It is not part of the approved 23-service platform or 15-service intelligence layer. This PRD is the formal proposal document for ARB review. Approval requires: (1) ARB evaluation methodology endorsement, (2) IOB approval of hard confidence ceiling policy, (3) Research & Red Team capability benchmark programme commitment.

---

## 1. The Competence Axis Proposal

### 1.1 The Gap in the Current Design

The three approved accountability axes address distinct failure modes:

- **Origin**: Did the AI accurately represent its sources?
- **Currency**: Is the intelligence still valid?
- **Trust**: Is the analyst's reliance on AI calibrated to its accuracy?

None of these answers a fourth question: **is this AI model actually capable of performing this specific task reliably?**

TCS/MIMIR tracks calibration for an analyst's reliance on AI in a domain. A well-calibrated analyst who has learned to discount AI outputs in domains where the AI underperforms is valuable — but the calibration is about the relationship between analyst behaviour and AI accuracy, not about the AI's intrinsic capability for specific task types. An analyst working in a new domain where no calibration data exists, using an AI on a claim type that systematic evaluation has shown the model cannot reliably perform, has no signal that they are operating outside the model's reliable capability.

CPS/APORIA provides that signal.

### 1.2 The Question This Service Answers

**CPS/APORIA answers: For this specific claim type in this specific domain with this model version — is the AI operating in a zone of reliable capability, variable capability, or known incapability?**

### 1.3 Why Empirical Derivation Is Required

The answer cannot be the model's self-reported confidence. LLMs produce confident-sounding outputs on claim types where their underlying accuracy is systematically poor. Self-reported confidence is not a reliable indicator of capability. Capability zones must be derived empirically — from structured evaluation benchmarks that test the model on known-answer claim types in specific domains, and from operational outcome data accumulated by OFS/NEMESIS.

This is the most resource-intensive element of the proposal: the evaluation programme. Building capability profiles requires the Research & Red Team to design and run structured capability benchmarks for each domain × claim type combination before CPS can populate its zone assessments. This is not a one-time effort — it must be re-run on each new model version.

### 1.4 Capability Zones

Three zones, empirically derived:

**Green** — The model demonstrates reliable accuracy on this claim type in this domain across the evaluation benchmark. Confidence signals from TCS/MIMIR are appropriate. No additional ceiling applied.

**Amber** — The model demonstrates meaningful accuracy variance across the evaluation benchmark: works well on some cases in this claim type/domain, fails on others. The distinguishing conditions are not fully characterised. Analysts should apply additional scrutiny. UCS/TYCHE receives an elevated model_dominance signal.

**Red** — The model demonstrates systematic error or inability in this claim type/domain combination across the evaluation benchmark. A hard confidence ceiling is applied by UCS/TYCHE regardless of TCS/MIMIR calibration. Analyst substitution or constraint of dissemination is the recommended response.

---

## 2. Core Responsibilities (Proposed)

### 2.1 Primary Function

CPS/APORIA maintains empirically-derived capability zone profiles for each combination of (model_version, domain, claim_type) — building these profiles from structured Research & Red Team evaluation benchmarks and operational OFS/NEMESIS outcome data — and publishes capability zone assignments to UCS/TYCHE for incorporation into uncertainty profiles, to TCS/MIMIR for capability-stratified calibration, and to ATHENA for hard confidence ceiling enforcement in Red zones.

### 2.2 Secondary Functions

- Evaluation programme management: scheduling and tracking Research & Red Team capability benchmarks per model version and domain
- Capability zone update from outcome data: when OFS/NEMESIS accumulates sufficient outcome data for a domain × claim type combination, updating the zone assessment
- Model version transition: re-evaluating all capability zones when MDS/KRONOS registers a new model version
- Capability surface reporting: IOB-facing summary of capability zones across the platform's analytical domains

---

## 3. Data Architecture (Proposed)

### 3.1 Primary Data Models

```yaml
CapabilityProfile:
  profile_id:              uuid
  model_version:           str
  domain:                  str
  claim_type:              str
  zone:                    green | amber | red | unassessed
  confidence_ceiling:      float | null    # hard ceiling for UCS/TYCHE; null if green
  evaluation_basis:        BENCHMARK | OUTCOME_DATA | BOTH | NONE
  benchmark_accuracy:      float | null    # from Research & Red Team evaluation
  outcome_accuracy:        float | null    # from OFS/NEMESIS operational data
  sample_size:             int
  last_evaluated:          datetime
  evaluating_team:         str             # Research & Red Team reference
  iob_endorsed:            bool            # hard ceiling policy requires IOB endorsement

CapabilityEvaluation:
  evaluation_id:           uuid
  profile_id:              uuid
  evaluation_type:         BENCHMARK | OPERATIONAL_REVIEW
  evaluated_at:            datetime
  evaluated_by:            str
  accuracy_score:          float
  case_count:              int
  failure_pattern:         str | null      # systematic failure description if amber/red
  notes:                   str | null
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Profile store | PostgreSQL | CapabilityProfile, CapabilityEvaluation | Indefinite |
| Profile cache | Redis | Active profiles (hot path for UCS/TYCHE and TCS) | 4h TTL |
| Event store | MOIRAI | Signed zone assignment events | Indefinite |

---

## 4. API Contract (Proposed)

### 4.1 Endpoints

```
GET /profiles/{model_version}/{domain}/{claim_type}
  Auth:     UCS service account | TCS service account | analyst session token
  Response: {
    profile_id:            uuid,
    zone:                  green | amber | red | unassessed,
    confidence_ceiling:    float | null,
    evaluation_basis:      str,
    sample_size:           int,
    last_evaluated:        datetime
  }
  SLA: p99 < 100ms (cached)

GET /profiles/summary
  Auth:     IOB token | platform operator
  Params:   model_version: str
  Response: {
    total_profiles:        int,
    green_count:           int,
    amber_count:           int,
    red_count:             int,
    unassessed_count:      int,
    by_domain:             [{ domain, green, amber, red, unassessed }]
  }

POST /evaluations
  Auth:     Research & Red Team token
  Request:  {
    model_version:         str,
    domain:                str,
    claim_type:            str,
    accuracy_score:        float,
    case_count:            int,
    failure_pattern:       str | null
  }
  Response: {
    profile_id:            uuid,
    zone_assigned:         str,
    prior_zone:            str | null,
    ceiling_applied:       float | null
  }

GET /health
  Response: {
    status, dependencies: { moirai, ogs, redis },
    profiles_total:        int,
    red_zone_count:        int,
    unassessed_count:      int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          CPS_ZONE_ASSIGNED
service_id:         "CPS"
session_id:         null
classification:     CONTROLLED_UNCLASSIFIED
event_payload:
  profile_id:             uuid
  model_version:          str
  domain:                 str
  claim_type:             str
  zone:                   str
  confidence_ceiling:     float | null
  evaluation_basis:       str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          CPS_RED_ZONE_DETECTED
event_payload:
  domain:                 str
  claim_type:             str
  model_version:          str
  confidence_ceiling:     float
  failure_pattern:        str
```

---

## 5. Integration Map (Proposed)

### 5.1 Feeds Into (Proposed)

| Service | Epithet | What CPS provides | How |
|---|---|---|---|
| UCS/TYCHE | Uncertainty | model_signal (capability zone) and confidence_ceiling | API |
| TCS/MIMIR | Trust Calibration | Capability zone for stratified calibration | `CPS_ZONE_ASSIGNED` event |
| ATHENA | Interface | Red zone ceiling indicator in session | API |
| IAS/SCUDO | Adversarial Screening | Red zone claim types as higher-sensitivity screening targets | API |
| IOB Reporting | Oversight | Capability surface summary | API |

---

## 6. Non-Functional Requirements (Proposed)

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Profile lookup (cached) | 5ms | 20ms | 100ms |
| Zone assignment (evaluation input) | 200ms | 500ms | 1000ms |

---

## 7. Known Design Risks (Proposed)

### 7.1 The Evaluation Programme Is the Critical Path

CPS/APORIA's value is entirely dependent on the quality and coverage of the Research & Red Team evaluation programme. An unassessed profile is operationally equivalent to no service. The Research & Red Team must commit to:
- Evaluation benchmark development for each domain × claim type combination in the platform's analytical scope
- Re-evaluation on each MDS/KRONOS model version change
- Operational review programme using OFS/NEMESIS outcome data

This is a sustained research investment, not a one-time project. The ARB approval decision must include a commitment to this programme.

### 7.2 Amber Zone Is Analytically Ambiguous

The Green/Red distinction is clean. The Amber zone is not. A model that performs well on 70% of cases in a domain × claim type combination and fails on 30% is in Amber — but the analyst working on a specific case has no way to know whether they are in the 70% or the 30%. The design response (elevating model_dominance in UCS/TYCHE, surfacing the Amber indicator in ATHENA) is correct but incomplete. Research & Red Team work to characterise what distinguishes the passing 70% from the failing 30% — which specific features of a claim predict failure within an Amber zone — is the highest-value research investment for CPS after initial deployment.

### 7.3 Hard Red Zone Ceilings Require IOB Endorsement

Applying a hard confidence ceiling to any claim type in any domain regardless of TCS/MIMIR calibration is a strong design assertion — it says the platform's structural assessment of capability overrides the analyst's empirical calibration record. This requires IOB endorsement of the policy before implementation. The ceiling is not an architectural default; it is a governed policy decision.

---

## 8. Proposal Requirements for ARB Approval

The following must be satisfied before ARB considers CPS/APORIA for approval:

1. **Research & Red Team evaluation programme design.** A specific programme document specifying which domains and claim types will be evaluated, the benchmark methodology, the accuracy thresholds for Green/Amber/Red zone assignment, and the re-evaluation schedule.

2. **IOB endorsement of hard ceiling policy.** The policy defining when a Red zone is assigned and what the confidence ceiling value is, signed off by the IOB.

3. **Pilot evaluation results.** At least one complete domain × claim type evaluation (benchmark + zone assignment) demonstrating the methodology works before the service is built.

4. **Model version transition plan.** A specific plan for how capability zones will be re-evaluated when MDS/KRONOS registers a new model version, including the acceptable window for operating on stale zone assessments.

---

## 9. Implementation Roadmap (Proposed — Contingent on ARB Approval)

### Phase 1 — Core Zone Registry and Evaluation Pipeline

- CapabilityProfile and CapabilityEvaluation schemas
- Research & Red Team evaluation submission endpoint
- Zone assignment logic (Green/Amber/Red thresholds)
- UCS/TYCHE model_signal integration
- TCS/MIMIR capability zone stratification
- Red zone ceiling enforcement via UCS/TYCHE

### Phase 2 — OFS/NEMESIS Outcome Integration and Comprehensive Coverage

- Operational outcome data integration to update zone assessments
- Evaluation coverage expansion across all platform domains
- MDS/KRONOS model version transition workflow
- IOB capability surface reporting

---

## 10. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| ARB approval | Full capability profiling service approval | Phase 1 begins |
| IOB endorsement | Hard confidence ceiling policy — which capability zone assignments trigger ceilings and what ceiling values apply | Red zone ceiling implementation |
| Research & Red Team programme commitment | Evaluation benchmark development and maintenance | Profiles have value only if evaluations are conducted |

---

## 11. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial proposal PRD — Addendum D Competence Axis specification |

## Appendix D: Red Team Findings

*No red team assessment has been conducted. This is a prerequisite for ARB approval. The Research & Red Team must assess the evaluation methodology and zone assignment thresholds before the ARB review.*
