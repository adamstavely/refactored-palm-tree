# UCS — Uncertainty Characterization Service
### TYCHE · *"Greek goddess of fortune and chance — specifically the aleatory: what is uncertain because it has not yet happened, not because we have not yet looked"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `UCS` |
| **Epithet** | `TYCHE` |
| **Full name** | Uncertainty Characterization Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Year 2 · Q1 (Addendum F) |
| **Build priority** | 12 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust — characterises the nature of uncertainty rather than quantifying it as a single score |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**UCS/TYCHE answers: What kind of uncertainty is this — and what would reduce it?**

### 1.2 Why This Service Exists

The single calibrated confidence score (high/medium/low) that TCS/MIMIR produces is useful but incomplete. It tells the analyst how reliable their AI reliance has been in this domain for this claim type. It does not tell them why the confidence is at this level, or what kind of uncertainty they are facing, or what they could do to address it.

Three fundamentally different kinds of uncertainty look identical to a single confidence score:

**Aleatory uncertainty**: the adversary has not decided. The outcome is genuinely probabilistic. No collection resolves it. No better AI model resolves it. The correct response is a probability distribution, not an assertion.

**Epistemic uncertainty**: we lack collection or analytical coverage. This is reducible. Better collection, better retrieval, better analytical coverage reduces it. The correct response is a collection requirement or a qualified judgment.

**Model uncertainty**: the AI's capability limitations. The Competence axis addresses this. Better capability or human substitution reduces it. The correct response is constraint or substitution.

A decision-maker who sees "medium confidence" needs to know which of these three they are facing. Medium confidence because the adversary hasn't decided yet requires a different response than medium confidence because we have a collection gap, which requires a different response than medium confidence because this claim type is outside the model's reliable capability. The three cases produce the same score and demand different actions.

### 1.3 Redesign Note: Quantification to Characterization

The original design specified UQS/TYCHE as an Uncertainty **Quantification** Service, producing numerical weights for each uncertainty type (aleatory: 0.35, epistemic: 0.42, model: 0.23). The red team assessment identified this as not computationally achievable: the inputs available — collection coverage from ARGUS-LACUNA, capability zone from CPS/APORIA — are proxies for the uncertainty types, not the decomposition itself. Composing proxies into precise decimal weights creates the appearance of measurement that does not exist.

UCS/TYCHE characterises uncertainty using ordinal signals (high/medium/low per component) derived from the claim-type taxonomy and observable signals. The analyst's judgment — through three structured questions — is the resolution mechanism, not an algorithm. This is both more honest and more useful.

### 1.4 Design Principles

- **Ordinal characterisation, not numerical decomposition.** The service produces `aleatory_dominance: high|medium|low`, not `aleatory_weight: 0.35`. Ordinal signals are computable from available inputs. Precise numerical weights are not.
- **Claim type is the primary routing signal.** The strongest predictor of uncertainty type is what kind of claim is being made. Future adversary intent claims are inherently aleatory. Collection-dependent factual claims are primarily epistemic. The claim-type taxonomy is the primary routing mechanism, not runtime computation.
- **Analyst judgment is the resolution mechanism.** TYCHE pre-populates three structured questions from observable signals. The analyst resolves them. The analyst's completed responses — not the algorithmic pre-population — are the output. This reflects the epistemically correct distribution of responsibility: the analyst has domain knowledge the service cannot access.
- **Reducibility is the actionable output.** The most operationally useful thing UCS produces is the `reducibility` object — what action would reduce each uncertainty component. This converts a characterisation into a decision support tool.
- **Prerequisite for five services.** UCS/TYCHE must be deployed before PCS/IRIS (which needs the decomposed profile for the consumer package three-sentence translation), CLS/PROTEUS (which adds a partner_weight dimension), and TRS/CHRONOS (which needs temporal uncertainty as a distinct component).

### 1.5 Explicit Out of Scope

- **Computing analytical probability estimates.** UCS characterises what kind of uncertainty applies; computing probability distributions over adversary actions is an analytical function, not a platform function.
- **Providing ground truth for the uncertainty characterisation.** UCS produces a structured framework for analyst reasoning about uncertainty. Whether the characterisation is correct in a specific case is an analytical judgment.

---

## 2. Core Responsibilities

### 2.1 Primary Function

UCS/TYCHE produces a structured uncertainty profile for analytical claims — characterising whether the dominant uncertainty is aleatory (not reducible), epistemic (reducible by collection), or model (reducible by capability) — using the claim-type taxonomy as the primary routing signal and observable inputs from ARGUS-LACUNA and CPS/APORIA as supporting signals. It surfaces three structured analyst-resolution questions in ATHENA and records the analyst's completed characterisation to MOIRAI. The completed profile feeds PCS/IRIS for consumer package generation, CLS/PROTEUS for partner weight integration, and TRS/CHRONOS for temporal uncertainty profiling.

### 2.2 Secondary Functions

- Claim-type taxonomy management: version-controlled claim-type taxonomy with IOB approval for changes
- Pre-population of analyst-resolution questions from observable signals
- Reducibility vector generation: specific recommended actions to reduce each uncertainty component
- Temporal uncertainty component for TRS/CHRONOS: `temporal_horizon_dominance` as an additional profile field for forecast products
- Partner weight component for CLS/PROTEUS: `partner_dominance: high|medium|low` when partner-sourced intelligence is the primary input

### 2.3 What This Service Does Not Decide

UCS characterises the type of uncertainty present. What probability to assign to adversary actions, whether a collection requirement should be generated, and whether an assessment should proceed despite uncertainty are analytical and management decisions. The characterisation is a structured input to those decisions; it is not the decision.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
UncertaintyProfile:
  profile_id:                uuid
  session_id:                uuid
  turn_id:                   uuid
  claim_id:                  uuid
  
  # Uncertainty characterization
  primary_uncertainty_type:  aleatory | epistemic | model | partner | mixed
  aleatory_dominance:        high | medium | low
  epistemic_dominance:       high | medium | low
  model_dominance:           high | medium | low
  partner_dominance:         high | medium | low | null   # present when CLS/PROTEUS contributes

  # Reducibility
  reducibility:              Reducibility

  # Analyst resolution (populated by analyst in ATHENA)
  analyst_resolution:        AnalystResolution

  # Supporting signals
  confidence_ceiling:        float | null      # from CPS/APORIA red zone; null if no ceiling
  claim_type_basis:          str               # the claim type taxonomy entry that drove routing
  epistemic_signal:          EpistemicSignal | null
  model_signal:              ModelSignal | null

  override_logged:           bool              # true when analyst overrode taxonomy routing
  taxonomy_version:          str
  created_at:                datetime

Reducibility:
  aleatory:  not_reducible                     # always — inherent indeterminacy
  epistemic: CollectionRecommendation | none_identified
  model:     human_substitution | capability_development | constrain_dissemination | none_identified
  partner:   seek_corroboration | partner_development | none_identified | null

AnalystResolution:
  collection_would_change:   boolean | null    # would better collection change this assessment?
  adversary_has_decided:     boolean | null    # has the adversary resolved this question?
  within_ai_capability:      boolean | null    # is this within AI reliable capability?
  resolution_rationale:      str              # required when overriding taxonomy routing

EpistemicSignal:
  argus_lacuna_coverage:     float             # collection coverage score from ARGUS-LACUNA
  coverage_available:        bool              # whether ARGUS-LACUNA was queryable

ModelSignal:
  capability_zone:           green | amber | red | null   # from CPS/APORIA
  capability_available:      bool              # whether CPS/APORIA was queryable

ClaimTypeTaxonomyEntry:
  entry_id:                  uuid
  claim_type:                str
  dominant_uncertainty:      aleatory | epistemic | model | mixed
  aleatory_default:          high | medium | low
  epistemic_default:         high | medium | low
  model_default:             high | medium | low
  reasoning:                 str               # why this claim type has this default profile
  examples:                  [str]
  taxonomy_version:          str

TaxonomyVersion:
  version_id:                uuid
  version_string:            str
  entries:                   [ClaimTypeTaxonomyEntry]
  effective_from:            datetime
  iob_approval_ref:          str
  change_summary:            str
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | UncertaintyProfile, TaxonomyVersion, ClaimTypeTaxonomyEntry | Session + 7 years |
| Taxonomy cache | Redis | Active taxonomy version (hot path) | Taxonomy TTL + invalidation |
| Event store | MOIRAI | Signed profile events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| UncertaintyProfile | Inherits session classification | Session-compartmented |
| AnalystResolution | Inherits session classification | Session-compartmented |
| TaxonomyVersion | Controlled Unclassified | Platform-wide; IOB approval records controlled |

### 3.4 Retention and Purge Policy

UncertaintyProfile records retained for session lifetime plus seven years. TaxonomyVersion records retained indefinitely — historical taxonomy versions must be queryable to interpret historical profile outputs. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /characterize
  Auth:     inference gateway service account | ATHENA service account
  Request:  {
    session_id:          uuid,
    turn_id:             uuid,
    claim_id:            uuid,
    claim_text:          str,
    claim_type:          str,              # from PGS/NOMOS interaction classification
    source_type:         str,
    partner_sourced:     bool
  }
  Response: {
    profile_id:          uuid,
    primary_type:        str,
    dominance:           { aleatory: str, epistemic: str, model: str, partner: str | null },
    reducibility:        Reducibility,
    pre_populated:       AnalystResolution,    # questions pre-populated; analyst confirms
    confidence_ceiling:  float | null,
    taxonomy_basis:      str
  }
  SLA: p99 < 300ms

POST /profile/{profile_id}/resolve
  Auth:     ATHENA service account (submitting analyst resolution)
  Request:  {
    analyst_resolution:  AnalystResolution,
    override_rationale:  str | null
  }
  Response: {
    profile_id:          uuid,
    resolution_recorded: bool,
    final_primary_type:  str
  }

GET /profile/{profile_id}
  Auth:     session token | supervisor token | IOB token
  Response: UncertaintyProfile with full AnalystResolution

GET /taxonomy/current
  Auth:     any service account
  Response: { version_string, entry_count, effective_from }
  SLA: p99 < 50ms (cached)

GET /taxonomy/{version_id}
  Auth:     supervisor token | IOB token
  Response: TaxonomyVersion with full entries

GET /session/{session_id}/uncertainty-summary
  Auth:     session token | supervisor token
  Response: {
    session_id:          uuid,
    profiles:            int,
    primary_type_distribution: { aleatory: int, epistemic: int, model: int, mixed: int },
    unresolved:          int,
    override_count:      int
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis },
    active_taxonomy_version: str,
    moirai_sync:         boolean,
    last_event_hash:     str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          UCS_PROFILE_CREATED
service_id:         "UCS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  profile_id:             uuid
  claim_id:               uuid
  primary_type:           str
  aleatory_dominance:     str
  epistemic_dominance:    str
  model_dominance:        str
  confidence_ceiling:     float | null
  taxonomy_version:       str
  analyst_resolved:       false            # always false at creation; updated on resolve
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          UCS_PROFILE_RESOLVED
event_payload:
  profile_id:             uuid
  final_primary_type:     str
  override_logged:        bool
  collection_needed:      bool             # true if analyst resolution indicates epistemic gap
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `UCS_PROFILE_CREATED` | Every claim characterisation | MOIRAI, ATHENA (surface three questions) |
| `UCS_PROFILE_RESOLVED` | Analyst completes resolution questions | MOIRAI, PCS/IRIS (consumer package), TIS/DIKE (if collection needed) |
| `UCS_TAXONOMY_UPDATED` | New taxonomy version activated | MOIRAI, all services (cache invalidation) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| CPS/APORIA | `CPS_CAPABILITY_ZONE_ASSIGNED` | Updates model_dominance signal; applies confidence ceiling if red zone |
| ARGUS-LACUNA | `CGS_GAP_IDENTIFIED` | Updates epistemic_dominance signal for affected domain/claim type |
| PGS/NOMOS | `PGS_INPUT_SCREENED` | Interaction class used for claim-type taxonomy routing |
| CLS/PROTEUS | `CLS_PARTNER_SOURCED` | Adds partner_dominance dimension to profile |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed profile events | Async event | Events buffered; profiling still proceeds |
| PCES/AEGIS | Classification Enforcement | Session token validation | Sync | Proceeds with cached session context |
| CPS/APORIA | Capability Profiling | Capability zone for model_dominance signal and ceiling | Async event | Model signal defaults to amber (neutral); no ceiling applied |
| ARGUS-LACUNA | Collection Gap Service | Collection coverage for epistemic_dominance signal | Async query | Epistemic signal defaults to medium (neutral) |

### 5.2 Feeds Into

| Service | Epithet | What UCS provides | How |
|---|---|---|---|
| ATHENA | Interface | Three structured questions; dominance indicators; confidence ceiling | API |
| PCS/IRIS | Policymaker Communication | Full UncertaintyProfile for consumer package three-sentence translation | API query |
| CLS/PROTEUS | Coalition Liaison | partner_dominance dimension in profile | Partner-sourced signal integration |
| TRS/CHRONOS | Temporal Reasoning | Temporal uncertainty profile extension | API |
| TIS/DIKE | Tasking Integration | Collection needed signal from resolved profiles | `UCS_PROFILE_RESOLVED` event |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Session token validated on analyst-facing read and resolution endpoints.
- **Compartment inheritance:** UncertaintyProfile inherits session classification.
- **Failure behavior:** Proceeds with cached session context; profile creation unaffected.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Profile creation | 50ms | 150ms | 300ms |
| Taxonomy lookup (cached) | 2ms | 5ms | 50ms |
| Analyst resolution recording | 30ms | 100ms | 200ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Profile creations/second | 100 |
| Resolution submissions/second | 50 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — UCS unavailability means uncertainty profiles not generated; PCS/IRIS degrades |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| CPS/APORIA | Model dominance defaults to amber (medium). No capability ceiling applied. Alert. | Uncertainty characterisation less precise |
| ARGUS-LACUNA | Epistemic dominance defaults to medium. Alert. | Uncertainty characterisation less precise |
| Redis (taxonomy cache) | Taxonomy loaded from PostgreSQL on each request (higher latency). Alert. | Profile creation latency increases |

---

## 7. Security Model

### 7.1 Authentication

Inference gateway and ATHENA submit profile creation via service accounts. Analyst resolution uses ATHENA service account (the analyst's identity is captured in the session context). Taxonomy management requires Research & Red Team token with IOB approval reference for version publication.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Inference gateway / ATHENA | Profile creation, resolution | Service account |
| Analyst (own session) | Session uncertainty summary | Session token |
| Supervisor | Team session summaries | Supervisor token |
| Research & Red Team | Taxonomy management | Research team token |
| IOB | Full profile access; taxonomy approval | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/ucs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/ucs/db-credentials` | 30 days |
| Redis credentials | `themis/ucs/redis-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Taxonomy misrouting (wrong primary type assigned by claim type) | Medium | P2 — analyst receives incorrect uncertainty characterisation | Analyst override rate monitoring | Conservative taxonomy defaults; Research & Red Team validation |
| Analyst resolution non-engagement (questions dismissed without genuine engagement) | High | P2 — profile records pre-populated values not analyst judgment | Rapid resolution rate monitoring | Training emphasis; resolution completeness monitoring |
| Circular dependency delay (UCS needs CPS/APORIA, which needs UCS context) | Low | P2 — model signal unavailable for initial profile | Timeout handling with default | UCS proceeds with model=medium if CPS signal not available within 100ms |

### 8.1 Known Design Risks

- **The taxonomy is the load-bearing component and its accuracy is unvalidated.** The claim-type taxonomy pre-populates uncertainty dominance based on the type of claim being made. The taxonomy entries are expert judgments, not empirically validated. Future adversary intent claims defaulting to aleatory=high may be wrong for specific subclaim types. Resolution path: OFS/NEMESIS outcome data will eventually show which claim types had mis-characterised uncertainty; taxonomy calibration is a Year 3 task.
- **Analyst resolution engagement is not guaranteed.** The service records what the analyst answers, not whether they answered genuinely. A rapid click-through of the three questions produces a MOIRAI-attested resolution that looks identical to a carefully considered resolution. This is a fundamental limitation of self-report mechanisms. The gaming probability signal from TCS provides a partial complement, but cannot fully detect non-engagement with uncertainty characterisation specifically.
- **UCS is a prerequisite for five services — deployment must be first among Year 2 Q1 services.** PCS/IRIS, CLS/PROTEUS, TRS/CHRONOS and others depend on UCS profiles. Any delay in UCS deployment cascades to these services. UCS must be treated as the Year 2 Phase 1 priority.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `ucs.profile.latency_p99` | Histogram | `> 300ms for 5m` | P1 |
| `ucs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `ucs.resolution.override_rate` | Gauge | `> 30%` (high analyst disagreement with taxonomy) | P2 |
| `ucs.resolution.rapid_rate` | Gauge | `> 40%` resolved in < 5 seconds | P2 |
| `ucs.taxonomy.version_age_days` | Gauge | `> 365` (taxonomy not updated in over a year) | P2 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces, redis },
  active_taxonomy_version: str,
  taxonomy_age_days:       int,
  profiles_today:          int,
  override_rate_24h:       float,
  last_event_hash:         str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":           "ISO-8601",
  "service":             "UCS/TYCHE",
  "event":               "PROFILE_CREATED | PROFILE_RESOLVED | TAXONOMY_UPDATED",
  "profile_id":          "uuid",
  "session_id":          "uuid",
  "primary_type":        "aleatory | epistemic | model | mixed",
  "override_logged":     false,
  "taxonomy_version":    "string",
  "duration_ms":         0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/ucs/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant

### 10.2 What This Service Attests

The MOIRAI record for UCS proves that for every claim in every analytical session, a structured uncertainty characterisation was produced, the analyst's resolution was recorded, and any analyst override of the taxonomy routing was explicitly logged. The consumer package (PCS/IRIS) can reference the MOIRAI-attested profile as the basis for its uncertainty language.

### 10.3 What This Service Cannot Prove

The profile proves the characterisation was produced and the resolution was recorded. It does not prove the analyst engaged genuinely with the resolution questions. It does not prove the taxonomy routing was correct for this specific claim.

---

## 11. Implementation Roadmap

### Phase 1 — Core Characterization and Taxonomy (Year 2, Weeks 1–8)

- ClaimTypeTaxonomyEntry schema and initial taxonomy v1.0 (Research & Red Team + analytic standards authority to specify entries)
- UncertaintyProfile schema and `POST /characterize` endpoint
- Taxonomy-routing logic: claim_type → dominance defaults
- `POST /profile/{id}/resolve` analyst resolution recording
- MOIRAI events: `UCS_PROFILE_CREATED`, `UCS_PROFILE_RESOLVED`
- Redis taxonomy cache with event-driven invalidation
- ATHENA integration: three structured questions surface in ATHENA interface

**Phase gate criterion:** Every claim in every ATHENA session produces a UncertaintyProfile. Three questions surface in ATHENA with taxonomy-based pre-population. Analyst resolution recorded in MOIRAI. Taxonomy v1.0 includes all primary claim types used in IC analytical sessions.

### Phase 2 — Observable Signal Integration and Downstream Feeds (Year 2, Weeks 9–16)

- ARGUS-LACUNA epistemic signal integration
- CPS/APORIA model signal and confidence ceiling integration
- CLS/PROTEUS partner_dominance dimension
- TRS/CHRONOS temporal_horizon_dominance extension
- PCS/IRIS profile consumption for consumer package generation
- TIS/DIKE collection_needed signal routing
- IOB taxonomy management and version governance workflow

**Phase gate criterion:** CPS/APORIA red zone triggers confidence ceiling in profiles. ARGUS-LACUNA coverage score adjusts epistemic dominance. PCS/IRIS consumer package generates correct three-sentence uncertainty translation from profile. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| None specific | Taxonomy changes require IOB approval, per the same governance process as PGS/NOMOS policy rules. No GC item required — this is a platform standards decision. | IOB taxonomy approval before each TaxonomyVersion publication |

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

For each AI claim in ATHENA, three structured questions appear in the claim detail panel (not modal-blocking, but persistent until resolved):

1. **Would better collection change this assessment?** Pre-populated from ARGUS-LACUNA coverage signal. If yes → epistemic; collection gap is the actionable response.

2. **Has the adversary resolved this question yet?** Pre-populated from claim type taxonomy. If uncertain or no → aleatory; express as probability distribution, not assertion.

3. **Is this claim type within the AI's reliable capability?** Pre-populated from CPS/APORIA zone. If uncertain or no → model; constraint or human substitution is the response.

The analyst confirms, overrides with rationale, or marks as uncertain. The uncertainty indicator in the claim shows the dominant type (purple = aleatory, amber = epistemic, blue = model) and the reducibility recommendation.

### 13.2 What the Analyst Should Do

Engage genuinely with the three questions. The pre-population is a starting point based on claim type and observable signals — the analyst's domain knowledge may lead them to different answers, and that override is valuable data. An override with a clearly reasoned rationale is better than a confirmed pre-population that doesn't reflect the analyst's actual judgment. Epistemic flags generate a collection recommendation — act on it if the claim is operationally important.

### 13.3 What the Signal Does Not Mean

Purple (aleatory) does not mean the assessment is impossible — it means the uncertainty is inherent to the subject matter, not a collection gap. The correct response is a probability distribution, not abandoning the assessment. Amber (epistemic) does not mean the AI is unreliable — it means better collection could improve the assessment. Blue (model) does not mean the assessment is wrong — it means the AI's capability limits apply here and additional scrutiny is warranted.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Taxonomy validation against outcome data.** The initial taxonomy entries are expert judgments. OFS/NEMESIS outcome data will eventually show which claim types had systematically mis-characterised uncertainty (claims characterised as epistemic that were later confirmed despite collection gaps, for example). A taxonomy calibration study should be designed and pre-registered before Year 2 Q1 deployment. Resolution path: Research & Red Team to specify outcome-based taxonomy validation criteria.

### 14.2 Operational Assumptions

- **Assumption 1: Taxonomy v1.0 covers the primary claim types encountered in operational sessions.** If analysts regularly make claim types not covered by the taxonomy, the system falls back to a generic `mixed` characterisation. The Research & Red Team must review a sample of historical analytical sessions to identify claim types before taxonomy v1.0 is published.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — redesigned from UQS (Uncertainty Quantification) to UCS (Uncertainty Characterization) per red team assessment. See Addendum F for redesign rationale. |

## Appendix D: Red Team Findings
*Pending — Year 2 Q1 gate review. Original design (Uncertainty Quantification Service) was red-teamed and redesigned. This PRD reflects the post-red-team architecture.*
