# MDS — Model Drift & Version Service
### KRONOS · *"Greek personification of time — he who orders the sequence of events and holds all things accountable to their moment"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MDS` |
| **Epithet** | `KRONOS` |
| **Full name** | Model Drift & Version Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 3–4 (Weeks 9–28) |
| **Build priority** | 7 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — tracks model versions and flags calibration as stale when the model changes |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MDS/KRONOS answers: Has the AI model changed since this calibration data was established — and if so, what must be re-evaluated?**

### 1.2 Why This Service Exists

Calibration in TCS/MIMIR is per-model-version. An analyst whose accuracy in a domain has been measured against model version 3.1 has not been measured against model version 3.2. Those two models may behave identically in that domain, or they may differ in ways that change the accuracy baseline. Without model version tracking, the platform treats calibration posteriors as stable across model updates — and they are not. A model that improves at SIGINT analysis will make analysts appear better calibrated than they are (because the model is now more accurate, but the posterior still reflects accuracy against the prior model). A model that regresses will make analysts appear worse calibrated.

Prompts face the same problem. A prompt in PRS/PROMETHEUS was tested against a specific model version. The same prompt applied to a new model version may produce systematically different outputs — different citation patterns, different uncertainty expression, different source type behaviour. Without version change detection, the prompt library operates on stale behavioural assumptions.

MDS/KRONOS is the service that keeps the rest of the platform honest about model time.

### 1.3 Why This Service Is Seventh

MDS depends on MOIRAI to record version change events and on TCS/MIMIR to receive stale-flagging notifications. Both must be operational before MDS can perform its primary function. MDS is Phase 3-4 because model version changes begin to matter as soon as calibration data starts accumulating — which starts in Phase 3-4 when TCS is deployed.

### 1.4 Design Principles

- **Version changes are events, not administrative updates.** A model version change produces a signed MOIRAI event. The full history of model versions deployed against the platform is part of the provenance chain.
- **Stale-flagging is conservative.** When a model version changes, all calibration posteriors for that model are flagged as potentially stale — not just the ones most likely to be affected. It is better to flag conservatively and allow posteriors to re-certify than to miss a behavioural change.
- **Drift characterisation informs but does not gate.** MDS characterises the behavioural difference between model versions. Platform operators decide what to do with that characterisation — whether to require re-calibration, hold model deployment, or proceed with stale warnings. MDS does not make those decisions.
- **Prompt regression detection extends model version tracking.** A model version change that changes prompt behaviour is as significant as the version change itself. PRS/PROMETHEUS is notified whenever MDS registers a version change.

### 1.5 Explicit Out of Scope

- **Model procurement decisions.** MDS tracks model versions; it does not decide which models should be deployed.
- **Approval authority for model version changes.** MDS records and characterises version changes; the ARB approves them. MDS provides the technical data for the ARB decision.
- **Training or fine-tuning of models.** MDS is a monitoring service, not a training infrastructure.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MDS/KRONOS maintains a version registry of every AI model version deployed against the THEMIS platform, characterises the behavioural differences between consecutive versions through a standardised evaluation suite, flags TCS/MIMIR calibration posteriors as potentially stale when a new model version is registered, and notifies PRS/PROMETHEUS and SKS/DAEDALUS that prompts and skills require regression testing against the new version.

### 2.2 Secondary Functions

- Behavioural profile maintenance: structured characterisation of each model version's output distribution across interaction classes and claim types
- MCP server version tracking: extending version management to MCP servers registered in MGS/TERMINUS (behaviorally equivalent to model version changes when MCP server behaviour changes)
- Drift severity classification: assessing whether a version transition is minor (similar behaviour) or major (significant behaviour change requiring priority re-calibration)
- Deployment hold recommendations: when drift severity is classified as major, generating a hold recommendation for the ARB before the new version is used in analytical sessions

### 2.3 What This Service Does Not Decide

MDS characterises version changes and drift. Whether a specific model version should be approved for analytical use, whether an existing calibration is sufficient despite stale flagging, and whether to roll back to a prior model version are decisions owned by the ARB and platform operators. MDS provides the technical input; humans decide.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ModelVersion:
  version_id:            uuid
  model_name:            str              # e.g., "claude-sonnet-4"
  version_string:        str              # e.g., "claude-sonnet-4-20250514"
  deployment_date:       datetime
  registered_by:         str              # who registered this version
  arb_approval_ref:      str | null       # ARB decision reference; null if not yet approved
  status:                PENDING | APPROVED | ACTIVE | DEPRECATED
  behavioural_profile:   BehaviouralProfile
  deployment_notes:      str | null

BehaviouralProfile:
  profile_id:            uuid
  version_id:            uuid
  evaluated_at:          datetime
  evaluation_suite_version: str
  source_citation_rate:  float            # fraction of turns with at least one GRND citation
  param_rate:            float            # fraction of claims flagged as PARAM
  uncertainty_expression_rate: float      # fraction of outputs with explicit uncertainty hedging
  by_interaction_class:  [{ class: str, accuracy_mean: float, n_cases: int }]
  by_claim_type:         [{ type: str, accuracy_mean: float, n_cases: int }]

DriftAssessment:
  assessment_id:         uuid
  version_from_id:       uuid
  version_to_id:         uuid
  assessed_at:           datetime
  overall_drift_score:   float            # 0.0 (identical) – 1.0 (completely different)
  severity:              MINOR | MODERATE | MAJOR
  by_interaction_class:  [{ class, drift_score, direction }]
  by_claim_type:         [{ type, drift_score, direction }]
  prompt_regression_risk: bool            # true if drift likely affects prompt behaviour
  hold_recommended:      bool
  hold_rationale:        str | null

McpServerVersion:
  mcp_version_id:        uuid
  server_name:           str
  version_string:        str
  registered_at:         datetime
  prior_version_id:      uuid | null
  behaviour_change_detected: bool
  registry_entry_id:     uuid             # FK → MGS/TERMINUS MCP registry
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | ModelVersion, BehaviouralProfile, DriftAssessment, McpServerVersion | Indefinite |
| Event store | MOIRAI | Signed version and drift events | Indefinite |
| Evaluation cache | Redis | Recent evaluation suite results for comparison | 30 days |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ModelVersion | Controlled Unclassified | Platform-wide; accessible to all platform services |
| BehaviouralProfile | Controlled Unclassified | Accessible to Research & Red Team and ARB |
| DriftAssessment | Controlled Unclassified | Accessible to ARB, platform operators, IOB |

### 3.4 Retention and Purge Policy

ModelVersion and BehaviouralProfile records are retained indefinitely — the model history is required to interpret historical calibration records. DriftAssessment records are retained indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /versions/register
  Auth:     platform operator token
  Request:  {
    model_name:        str,
    version_string:    str,
    deployment_notes:  str | null
  }
  Response: {
    version_id:        uuid,
    evaluation_status: PENDING,
    estimated_evaluation_time: str
  }

POST /versions/{version_id}/evaluate
  Auth:     platform operator token
  Request:  { evaluation_suite_version: str }
  Response: { profile_id: uuid, evaluation_started_at: datetime }

GET /versions/active
  Auth:     any service account
  Response: ModelVersion (active version only)
  SLA: p99 < 50ms (cached)

GET /versions/{version_id}
  Auth:     any service account
  Response: ModelVersion with BehaviouralProfile

POST /versions/{version_id}/promote
  Auth:     ARB approval token (requires arb_approval_ref)
  Request:  { arb_approval_ref: str }
  Response: {
    version_id:        uuid,
    promoted_at:       datetime,
    drift_from_prior:  DriftAssessment,
    stale_posterior_count: int,
    prompts_flagged:   int
  }

GET /drift/{version_from}/{version_to}
  Auth:     ARB token | platform operator | Research & Red Team
  Response: DriftAssessment

GET /stale/posteriors
  Auth:     TCS service account | platform operator
  Response: {
    stale_count:       int,
    by_domain:         [{ domain, count }],
    version_flagged_for: str
  }

POST /mcp-servers/version
  Auth:     MGS/TERMINUS service account
  Request:  { server_name: str, version_string: str, registry_entry_id: uuid }
  Response: { mcp_version_id: uuid, behaviour_change_detected: bool }

GET /health
  Response: {
    status, dependencies: { moirai },
    active_version:    str,
    pending_evaluation:bool,
    last_event_hash:   str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          MDS_MODEL_VERSION_REGISTERED
service_id:         "MDS"
session_id:         null
classification:     CONTROLLED_UNCLASSIFIED
event_payload:
  version_id:             uuid
  version_string:         str
  registered_by:          str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          MDS_MODEL_VERSION_CHANGED
event_payload:
  version_from:           str
  version_to:             str
  drift_severity:         MINOR | MODERATE | MAJOR
  stale_posterior_count:  int
  prompts_flagged:        int
  hold_recommended:       bool

EventType:          MDS_MCP_VERSION_CHANGED
event_payload:
  server_name:            str
  version_from:           str
  version_to:             str
  behaviour_change:       bool
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `MDS_MODEL_VERSION_CHANGED` | New model version promoted to ACTIVE | TCS/MIMIR (stale flagging), PRS/PROMETHEUS (prompt re-test), SKS/DAEDALUS (skill re-test), FGTS (baseline reset) |
| `MDS_MCP_VERSION_CHANGED` | New MCP server version registered with behaviour change | MGS/TERMINUS, PRS/PROMETHEUS |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| ARB (admin) | Model version approval | Updates ModelVersion status to APPROVED; enables promotion |
| MGS/TERMINUS | MCP server version registration | Creates McpServerVersion record; assesses behaviour change |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Version change events in the provenance chain | Async event | Events buffered; version registration still proceeds |

### 5.2 Feeds Into

| Service | Epithet | What MDS provides | How |
|---|---|---|---|
| TCS/MIMIR | Trust Calibration | Stale flagging on model version change | `MDS_MODEL_VERSION_CHANGED` event |
| PRS/PROMETHEUS | Prompt Repository | Prompt regression testing trigger on version change | `MDS_MODEL_VERSION_CHANGED` event |
| SKS/DAEDALUS | Skill Registry | Skill regression testing trigger on version change | `MDS_MODEL_VERSION_CHANGED` event |
| FGTS/ALETHEIA | Ground Truth | Correction baseline reset for new model version | `MDS_MODEL_VERSION_CHANGED` event |
| MGS/TERMINUS | MCP Gateway | MCP server version change notification | `MDS_MCP_VERSION_CHANGED` event |
| ATHENA | Interface | Stale calibration warnings when active version changes mid-session | API |
| ARB | Governance | Drift assessment data for model version approval decisions | API |

### 5.3 PCES/AEGIS Integration

MDS manages platform-level model version data, not session-level data. PCES enforcement does not apply to version registry queries (these are Controlled Unclassified platform metadata). Analyst-facing exposure of model version information in ATHENA is session-scoped but the version data itself is not compartmented.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Active version lookup (cached) | 2ms | 5ms | 50ms |
| Version registration | 100ms | 300ms | 500ms |
| Drift assessment | 5s | 30s | 120s |
| Evaluation suite run | 10 min | 30 min | 60 min |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — MDS unavailability means version changes cannot be registered; existing sessions unaffected |
| MOIRAI event durability | 99.999% |
| RTO | 30 minutes |
| RPO | 5 minutes |

### 6.3 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Events buffered; version registry still operational | No analyst-facing impact; provenance gap logged |

---

## 7. Security Model

### 7.1 Authentication

Version registration and promotion require platform operator tokens with ARB approval references for promotion. Service-to-service version queries use service accounts. ARB approval workflow uses ARB-specific tokens with audit logging.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Platform operator | Version registration, evaluation trigger | Operator token |
| ARB | Version promotion (requires arb_approval_ref) | ARB approval token |
| Any THEMIS service | `GET /versions/active` | Service account |
| Research & Red Team | Drift assessment read; behavioural profiles | Research team token |
| MGS/TERMINUS | MCP server version registration | Service account |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/mds/signing-key` | 90 days |
| PostgreSQL credentials | `themis/mds/db-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

**Unauthorized model version promotion**: bypassing ARB approval to deploy an unevaluated model version. Mitigation: promotion endpoint requires both operator token and a valid arb_approval_ref that references an IOB-logged decision. No promotion is possible without a logged ARB decision. **Evaluation suite manipulation**: producing a falsely positive drift assessment to block a model update. Mitigation: evaluation suite is versioned, run in an isolated environment, and results are signed by MDS before submission to ARB.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Model update deployed without MDS registration | Low | P1 — calibration used without stale flagging | Model version monitoring in inference gateway | Inference gateway reports active model version to MDS on each request; discrepancy alert |
| Drift assessment classification error (MINOR instead of MAJOR) | Low | P1 — model with significant behaviour change deployed without hold | Research & Red Team review of each assessment | All MAJOR assessments reviewed by Research & Red Team before ARB decision |
| MCP server version change undetected | Medium | P2 — prompts operate on stale MCP behaviour assumptions | MGS/TERMINUS version reporting | MGS/TERMINUS required to report version on each server registration update |

### 8.1 Known Design Risks

- **The evaluation suite cannot characterise all model behaviours.** The behavioural profile captures a defined set of measurable properties. Model behaviours that the evaluation suite doesn't test for will not appear in the drift assessment. Mitigation: the Research & Red Team maintains the evaluation suite and is responsible for expanding it as new behavioural concerns are identified.
- **Model version strings may not reliably identify behavioural changes.** Providers may silently update model behaviour under the same version string (this has occurred with major commercial providers). MDS cannot detect this without continuous behavioural monitoring. Mitigation: inference gateway continuously checks active model behaviour against the registered behavioural profile; significant divergence generates an alert even without a version string change.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `mds.version.mismatch_detected` | Counter | `> 0` | P0 |
| `mds.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `mds.drift.major_count` | Counter | Any major drift assessment | P1 |
| `mds.mcp.unregistered_version_rate` | Counter | `> 0 over 1h` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai },
  active_version:        str,
  pending_evaluations:   int,
  last_event_hash:       str,
  version_registry_count:int
}
```

### 9.3 Log Schema

```json
{
  "timestamp":         "ISO-8601",
  "service":           "MDS/KRONOS",
  "event":             "VERSION_REGISTERED | VERSION_PROMOTED | DRIFT_ASSESSED | MCP_VERSION_CHANGED",
  "version_id":        "uuid",
  "version_string":    "string",
  "drift_severity":    "MINOR | MODERATE | MAJOR | null",
  "duration_ms":       0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/mds/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant

### 10.2 What This Service Attests

The MOIRAI record for MDS proves the complete model version history deployed against the platform — every version, when it was registered, when it was promoted, and the drift assessment from its predecessor. An oversight body can reconstruct exactly which model version was active at any point in the platform's history, which is required to interpret historical calibration data.

### 10.3 What This Service Cannot Prove

MDS records the version string and behavioural profile at registration time. It cannot prove the model provider did not modify the model's behaviour without changing the version string.

---

## 11. Implementation Roadmap

### Phase 1 — Version Registry (Weeks 9–16)

- ModelVersion schema and registration endpoint
- Active version lookup endpoint with Redis cache
- `MDS_MODEL_VERSION_REGISTERED` and `MDS_MODEL_VERSION_CHANGED` MOIRAI events
- TCS/MIMIR stale-flagging integration
- PRS/PROMETHEUS and SKS/DAEDALUS re-test notification

**Phase gate criterion:** Model version change produces MOIRAI event and TCS stale flagging within 60 seconds. Active version lookup p99 < 50ms.

### Phase 2 — Drift Assessment and MCP Version Tracking (Weeks 17–28)

- BehaviouralProfile evaluation suite (baseline interaction class × claim type matrix)
- DriftAssessment computation and severity classification
- ARB approval workflow integration
- Deployment hold recommendation generation
- MCP server version tracking via MGS/TERMINUS
- Continuous behavioural monitoring for version string mismatch detection

**Phase gate criterion:** Drift assessment produces severity classification on test version pairs. ARB approval workflow logs to MOIRAI. MCP server version change detected and published. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate MDS deployment. Model procurement and approval governance is an ARB charter matter, not a GC policy matter.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

When a model version change occurs mid-session or since the analyst's last session, ATHENA shows an amber banner: "The AI model has been updated since your calibration was established. Confidence signals may not reflect the updated model's behaviour. Calibration is being recertified." The calibration state indicator reverts to CALIBRATING or PRIOR ONLY until recertification.

### 13.2 What the Analyst Should Do

Apply additional independent verification while calibration is in PRIOR ONLY or CALIBRATING state following a model version change. This is not a sign of platform failure — it is the platform being honest about the limits of its calibration data across model transitions.

### 13.3 What the Signal Does Not Mean

A model version change does not mean the AI is less accurate. The new model may be more accurate. It means the existing calibration data was measured against a prior model and has not yet been validated against the current one.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Evaluation suite comprehensiveness.** The evaluation suite measures source citation rate, PARAM rate, and uncertainty expression rate across interaction classes. Additional behavioural dimensions that matter for analytical accuracy are not yet in the suite. Resolution path: Research & Red Team to specify additional evaluation dimensions for Phase 2 evaluation suite.

### 14.2 Operational Assumptions

- **Assumption 1: Model providers announce version changes in advance.** If a provider silently updates a model, MDS cannot pre-register the version. The continuous behavioural monitoring component is the mitigation, but it introduces a detection lag. Resolution path: contractual requirement in model provider agreements for advance version notification.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 3 gate review.*
