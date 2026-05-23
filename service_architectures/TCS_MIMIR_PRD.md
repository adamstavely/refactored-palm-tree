# TCS — Trust Calibration Service
### MIMIR · *"Norse for the well of wisdom — the source of knowledge and memory at the root of Yggdrasil"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `TCS` |
| **Epithet** | `MIMIR` |
| **Full name** | Trust Calibration Service |
| **Namespace** | `themis-core` |
| **Layer** | Core Infrastructure |
| **Build phase** | Phase 3–4 (Weeks 9–28) |
| **Build priority** | 4 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust — measures and surfaces analyst-AI reliance calibration |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**TCS/MIMIR answers: Is this analyst's reliance on AI appropriately calibrated for this claim type in this domain with this model version?**

### 1.2 Why This Service Exists

The Trust axis of THEMIS addresses a failure mode that is invisible without measurement: an analyst who systematically over-relies on AI in a domain where the model's actual accuracy is lower than the analyst believes. Over-reliance is not detectable from any single interaction. It emerges as a pattern across many interactions, and the pattern is only visible if someone is tracking it.

Without TCS, the confidence signals ATHENA surfaces are the model's self-reported confidence — a signal that does not reliably correlate with accuracy (see Addendum D). With TCS, confidence signals are grounded in empirical measurement of this analyst's accuracy when they rely on AI in this specific domain. Those are fundamentally different epistemic objects. The first is a model property. The second is a relationship property — and it is the relationship that determines whether the analyst's work product is sound.

TCS is also the service that makes the calibration system attack-resistant. The gaming probability score detects patterns of performative verification that would corrupt the ground truth corpus if accepted unchallenged. Without gaming detection, a motivated analyst could systematically bias their own calibration record.

### 1.3 Why This Service Is Fourth

TCS requires a functioning event chain (MOIRAI) to receive calibration signals from FGTS/ALETHEIA, requires verified session data (PCES) to associate calibration with the correct analyst, and requires interaction classification (PGS) to route calibration updates to the correct domain cell. TCS cannot produce meaningful calibration without these foundations. It is Phase 3-4 because it needs the quality layer to have been accumulating correction data before its posteriors are worth using.

### 1.4 Design Principles

- **Calibration is a relationship, not a model property.** The calibration posterior for analyst A on claim type B in domain C with model version D is specific to that combination. Domain expertise does not transfer across domains. Model accuracy in one domain does not transfer across domains. No prior from any adjacent combination is substituted without evidence.
- **Cold start is explicit, not hidden.** Analysts in PRIOR ONLY state see this clearly in ATHENA. The confidence signals they receive are less reliable than those in CALIBRATED state, and the interface communicates this.
- **The calibration model is the most sensitive platform artifact.** If FGTS corrupts the ground truth corpus, TCS produces bad calibration signals, which produce bad confidence signals, which mislead analysts. The five-factor weighting in FGTS exists to protect TCS from this attack.
- **TCS signals but does not decide.** TCS surfaces confidence signals and calibration states. It does not block actions, restrict access, or produce analytical conclusions. Every TCS output is advisory.
- **Capability zone stratification is hard, not soft.** For claims in a CPS/APORIA red zone, TCS applies a hard confidence ceiling regardless of calibration score. This is not a guideline — it is an architectural constraint.

### 1.5 Explicit Out of Scope

- **Evaluating analytical quality or correctness.** TCS measures reliance calibration. Whether the analyst's conclusion was analytically sound is separate from whether their reliance on AI was calibrated.
- **Producing ground truth.** TCS consumes ground truth from FGTS/ALETHEIA. It does not determine what constitutes a verified correction.
- **Disciplinary decisions.** TCS surfaces calibration data and gaming probability to supervisors. Whether that data warrants any management action is a human decision.

---

## 2. Core Responsibilities

### 2.1 Primary Function

TCS/MIMIR maintains per-analyst, per-domain, per-claim-type, per-model-version Bayesian calibration posteriors — tracking the relationship between each analyst's reliance on AI and the AI's actual accuracy in that specific combination. It surfaces calibrated confidence signals to ATHENA, tracks the analyst's calibration state (PRIOR ONLY → CALIBRATING → CALIBRATED), computes a gaming probability score for each session to protect the ground truth corpus from performative verification, and provides a supervisory visibility surface that makes calibration variation across the analyst pool visible without exposing individual analyst identity to non-supervisors.

### 2.2 Secondary Functions

- Reliance-Accuracy Index (RAI) computation: continuous tracking of whether analyst reliance rate matches analyst accuracy rate for each domain-claim-type cell
- Capability-stratified calibration: separate posterior maintenance for green/amber/red capability zones (from CPS/APORIA) within each domain
- Supervised onboarding calibration: structured calibration sessions before operational use to provide cold-start data
- Model version change handling: flagging calibration posteriors as potentially stale when MDS/KRONOS detects a model version change
- Supervisory dashboard data: aggregate calibration metrics across the analyst pool, visible to supervisors with appropriate scope

### 2.3 What This Service Does Not Decide

TCS surfaces calibration states and confidence signals. Whether an analyst's calibration state warrants restricted access, additional supervision, or any other action is a human management decision. TCS does not block sessions, restrict access, or generate disciplinary records. The gaming probability score is a calibration quality indicator — it is not a misconduct determination.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
CalibrationPosterior:
  posterior_id:          uuid
  analyst_id:            str             # hashed for non-IOB contexts
  domain:                str             # e.g., TECHNICAL_PROGRAMS, GEOPOLITICAL, HUMINT
  claim_type:            str             # e.g., CAPABILITY_ASSESSMENT, INTENT_ANALYSIS
  model_version:         str             # from MDS/KRONOS
  capability_zone:       green | amber | red | null  # from CPS/APORIA
  state:                 PRIOR_ONLY | CALIBRATING | CALIBRATED
  alpha:                 float           # Beta distribution: successes
  beta:                  float           # Beta distribution: failures
  n_observations:        int             # total weighted observations
  last_updated:          datetime
  stale_flagged:         bool            # true if model version changed since last update
  stale_flagged_at:      datetime | null

RelianceAccuracyIndex:
  rai_id:                uuid
  posterior_id:           uuid
  rai_score:             float           # 0-1, higher = better calibrated reliance
  reliance_rate:         float           # fraction of turns where analyst accepted AI output
  accuracy_rate:         float           # fraction of accepted outputs later verified correct
  observation_window:    int             # turns in this computation
  computed_at:           datetime

GamingProbabilityScore:
  score_id:              uuid
  analyst_id:            str
  session_id:            uuid
  gaming_probability:    float           # 0-1; higher = more likely performative
  signal_breakdown:
    intention_gate_semantic_similarity:  float   # similarity across prior entries
    source_dwell_time_score:             float   # time spent on source before verifying
    batch_dismissal_rate:                float   # rate of rapid queue dismissal
    commitment_drift_score:              float   # divergence between stated and actual approach
    verification_completion_rate:        float   # fraction of verification actions completed
  computed_at:           datetime

CalibrationStateTransition:
  transition_id:         uuid
  posterior_id:          uuid
  from_state:            str
  to_state:              str
  trigger:               str             # N_OBSERVATIONS | MODEL_CHANGE | ADMIN_RESET
  n_observations_at_transition: int
  transitioned_at:       datetime

CalibrationBenchmark:
  benchmark_id:          uuid
  domain:                str
  claim_type:            str
  model_version:         str
  source:                str             # benchmark source / evaluation set name
  accuracy_mean:         float           # benchmark accuracy for these parameters
  accuracy_variance:     float
  n_benchmark_cases:     int
  valid_until:           datetime | null
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CalibrationPosterior, RAI, GamingProbabilityScore, Transitions | Indefinite |
| Hot cache | Redis | Active session posterior reads (frequently queried during active sessions) | Session TTL |
| Event store | MOIRAI | Signed calibration events | Indefinite |
| Benchmark store | PostgreSQL | CalibrationBenchmark records for cold start | Updated per model version |

The calibration posteriors are the most sensitive data in the platform. Access to individual analyst posteriors is restricted to the analyst themselves, their supervisor, and IOB. Aggregate calibration metrics (pooled, de-identified) are available more broadly.

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| CalibrationPosterior | Controlled Unclassified | Individual analyst data — restricted access |
| GamingProbabilityScore | Controlled Unclassified | Supervisor and IOB access only |
| RAI | Controlled Unclassified | Analyst (own only), supervisor (team), IOB (all) |
| Aggregate metrics | Unclassified | Available to platform team for monitoring |
| CalibrationBenchmark | Controlled Unclassified | Platform team and IOB |

### 3.4 Retention and Purge Policy

CalibrationPosterior records are retained indefinitely — the calibration history is valuable for longitudinal analysis and should not be purged without IOB authority. GamingProbabilityScore records are retained for session lifetime plus two years. Benchmark records are replaced when a new model version is evaluated.

---

## 4. API Contract

### 4.1 Endpoints

```
GET /calibration/{analyst_id}/{domain}/{claim_type}
  Auth:     session token (own) | supervisor token (team) | IOB token (all)
  Params:   model_version: str | null (defaults to current active)
            capability_zone: str | null
  Response: {
    posterior:          CalibrationPosterior,
    rai:                RelianceAccuracyIndex,
    confidence_signal:  { level: high|medium|low, ceiling_applied: bool },
    state:              PRIOR_ONLY | CALIBRATING | CALIBRATED
  }
  SLA: p99 < 200ms (p50 < 20ms from Redis cache)

GET /confidence/{session_id}/{turn_id}/{claim_id}
  Auth:     session token | ATHENA service account
  Response: {
    confidence_level:   high | medium | low,
    calibration_state:  PRIOR_ONLY | CALIBRATING | CALIBRATED,
    ceiling_applied:    bool,           # true if capability zone cap applied
    ceiling_reason:     str | null,
    uncertainty_profile: UncertaintyProfile | null   # from UCS/TYCHE if available
  }
  SLA: p99 < 100ms

POST /gaming/score
  Auth:     FGTS service account
  Request:  {
    session_id:         uuid,
    analyst_id:         str,
    behavioral_signals: GamingSignals
  }
  Response: GamingProbabilityScore
  SLA: p99 < 500ms

GET /supervisor/dashboard
  Auth:     supervisor token
  Response: {
    team_analysts: [
      {
        analyst_id:   "redacted",         # not exposed to supervisor by default
        domains: [
          {
            domain:           str,
            claim_type:       str,
            state:            str,
            rai_score:        float,
            gaming_probability_mean: float
          }
        ]
      }
    ]
  }

POST /calibration/update
  Auth:     FGTS service account
  Request:  {
    analyst_id:         str,
    domain:             str,
    claim_type:         str,
    model_version:      str,
    correction_weight:  float,          # from FGTS 5-factor weighting
    outcome:            CORRECT | INCORRECT
  }
  Response: {
    posterior_id:       uuid,
    alpha_new:          float,
    beta_new:           float,
    state_changed:      bool,
    new_state:          str | null
  }
  SLA: p99 < 300ms

GET /health
  Response: {
    status, dependencies: { moirai, pces, fgts, redis },
    active_posteriors:  int,
    stale_flagged_count:int,
    cache_hit_rate:     float,
    last_event_hash:    str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          TCS_CALIBRATION_UPDATED
service_id:         "TCS"
session_id:         uuid | null
classification:     CONTROLLED_UNCLASSIFIED
event_payload:
  posterior_id:           uuid
  analyst_id_hash:        str         # SHA-256 — not plain text
  domain:                 str
  claim_type:             str
  model_version:          str
  correction_weight:      float
  state_before:           str
  state_after:            str
  alpha_before:           float
  alpha_after:            float
  beta_before:            float
  beta_after:             float
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          TCS_STALE_FLAGGED
event_payload:
  affected_posterior_count: int
  model_version_old:        str
  model_version_new:        str
  flagged_at:               datetime

EventType:          TCS_GAMING_DETECTED
event_payload:
  session_id:               uuid
  analyst_id_hash:          str
  gaming_probability:       float
  primary_signal:           str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `TCS_CALIBRATION_UPDATED` | Every posterior update from FGTS | MOIRAI, ATHENA (confidence signal refresh) |
| `TCS_STATE_TRANSITIONED` | Calibration state change | MOIRAI, ATHENA (state indicator update) |
| `TCS_STALE_FLAGGED` | MDS/KRONOS model version change | MOIRAI, ATHENA (stale calibration warning) |
| `TCS_GAMING_DETECTED` | Gaming probability > configured threshold | MOIRAI, supervisor notification |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| FGTS/ALETHEIA | `FGTS_CORRECTION_WEIGHTED` | Triggers `POST /calibration/update` with the weighted correction |
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | Flags all affected CalibrationPosteriors as stale |
| CPS/APORIA | `CPS_CAPABILITY_ZONE_ASSIGNED` | Updates capability_zone on affected posteriors; applies hard ceiling if red zone |
| PGS/NOMOS | `PGS_INPUT_SCREENED` | Routes calibration update to correct domain-claim_type cell based on interaction_class |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| FGTS/ALETHEIA | Feedback & Ground Truth | Weighted correction data for posterior updates | Async event | Posterior updates queue; calibration freezes until FGTS recovers |
| MOIRAI | Provenance | Signed calibration events | Async event | Events buffered; calibration continues |
| MDS/KRONOS | Model Drift Detection | Model version change notifications for stale flagging | Async event | Stale flagging deferred; posteriors may be used beyond model change |
| CPS/APORIA | Capability Profiling | Capability zone assignments for stratified calibration | Async event | Capability stratification falls back to un-zoned posteriors |
| PGS/NOMOS | Analytic Standards | Interaction class for routing | Async event | Routing uses default interaction class |

### 5.2 Feeds Into

| Service | Epithet | What TCS provides | How |
|---|---|---|---|
| ATHENA | Interface | Calibrated confidence signals; calibration state indicators; gaming probability (supervisor view) | API |
| UCS/TYCHE | Uncertainty Characterization | Calibration-based model uncertainty component | API query |
| PCS/IRIS | Policymaker Communication | Calibration accuracy context for consumer package | API query |
| IOB Reporting | Oversight | Calibration state distribution; gaming detection summary | Supervisory dashboard |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** All analyst-specific calibration endpoints validate session token. Individual posterior data is scoped to the analyst themselves (or supervisor/IOB with appropriate token).
- **Compartment inheritance:** Calibration data is not compartmented at the claim level — it is controlled at the analyst identity level. Individual analyst data requires the analyst's session token or a supervisor/IOB token with explicit authority.
- **Failure behavior:** PCES unavailable → individual analyst endpoints unavailable; aggregate metrics endpoints available via service account with degraded access.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| `/confidence/{session}/{turn}/{claim}` (cached) | 5ms | 15ms | 100ms |
| Posterior read (`/calibration/...`) | 20ms | 50ms | 200ms |
| Posterior update (`/calibration/update`) | 50ms | 150ms | 300ms |
| Gaming score computation | 100ms | 300ms | 500ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Confidence signal reads/second | 500 (one per claim rendered in ATHENA) |
| Posterior updates/second | 50 (one per verified correction) |
| Gaming score computations/session | 1 (computed at session close) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — TCS degradation means calibration signals fall back to priors |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| FGTS/ALETHEIA | Posterior updates queue; calibration freezes. Confidence signals served from last-known posteriors. | Confidence signals become stale; PRIOR ONLY state extended for new analysts |
| Redis cache | Posterior reads from PostgreSQL (higher latency). Alert. | Confidence signal latency increases from 5ms to 200ms |
| MDS/KRONOS | Stale flagging deferred. Posteriors used beyond model change without warning. Alert. | Confidence signals may reflect prior model behaviour |
| CPS/APORIA | Capability zone stratification falls back to un-zoned posteriors. Hard ceilings not applied. Alert. | Confidence signals for amber/red zone claims may be inflated |

---

## 7. Security Model

### 7.1 Authentication

ATHENA and inference gateway access confidence signals via service account. Individual analyst calibration requires session token. Supervisor dashboard requires supervisor token. IOB endpoints require IOB token.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst (own session) | Own posterior, confidence signals | Session token |
| ATHENA / inference gateway | Confidence signals for active session | Service account |
| FGTS/ALETHEIA | `POST /calibration/update`; `POST /gaming/score` | Service account |
| Supervisor | Supervisory dashboard (team scope, analyst identities redacted) | Supervisor token |
| IOB | All endpoints including individual analyst data | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/tcs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/tcs/db-credentials` | 30 days |
| Redis credentials | `themis/tcs/redis-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

**Calibration corpus poisoning** is the primary threat: a coordinated group of analysts with good gaming scores submitting consistent but false corrections to systematically corrupt calibration in a specific domain. The five-factor weighting in FGTS provides the first line of defense. TCS provides the second: gaming probability scoring detects patterns within individual sessions that suggest performative verification even when the correction content appears legitimate. The IOB monthly report provides the third: aggregate calibration anomalies across the analyst pool are surfaced for human review.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Cell sparsity — posterior unreliable (N < 10) | High (many cells will start sparse) | P2 — misleading confidence signals for new analyst-domain combinations | State machine (PRIOR ONLY flag until N threshold) | Explicit PRIOR ONLY state in ATHENA; benchmark priors initialized conservatively |
| Gaming detection false positive (legitimate analyst flagged) | Medium | P2 — unfair supervisor alert | Analyst complaint mechanism | Gaming probability threshold conservative; supervisor review required before any action |
| Posterior corruption via weighted input manipulation | Low | P1 — calibration signals degraded for affected domain | IOB aggregate monitoring | FGTS weighting as defense; IOB monthly report as detection |
| Model version change without stale flagging | Low | P1 — stale confidence signals served | MDS/KRONOS event consumption monitoring | Alert on event consumption gap > 60 seconds |

### 8.1 Known Design Risks

- **Cell sparsity is the fundamental limitation.** The calibration model stratifies by (analyst, domain, claim type, model version, capability zone) — potentially thousands of cells. Most will have N < 10 for most of the platform's operational life, especially for junior analysts in new domains. The Bayesian prior helps but does not eliminate the sparsity problem. Confidence intervals must be wide in sparse cells, and ATHENA must surface this uncertainty. The PRIOR ONLY and CALIBRATING states exist precisely to communicate this to analysts, but the communication design must be explicit.
- **Gaming probability score weights are not empirically validated.** The five-signal composite is theoretically motivated but has not been validated against a labelled dataset of genuine vs. performative verification behaviour. The weights are initial best estimates. Calibration of the gaming model itself requires Research and Red Team evaluation. This is noted as an open research question (see Section 14).
- **Benchmark priors are inherently generic.** The cold-start benchmark is drawn from general model evaluation data. For IC-specific analytical tasks, model accuracy in controlled benchmarks may not predict operational accuracy. The benchmark priors should be treated as weak priors that update quickly — not as reliable estimates.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `tcs.confidence.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `tcs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `tcs.posterior.stale_ratio` | Gauge | `> 20% of active posteriors stale` | P1 |
| `tcs.gaming.detection_rate` | Gauge | Spike > 5x baseline | P2 |
| `tcs.posterior.prior_only_ratio` | Gauge | For monitoring; no alert threshold | Informational |
| `tcs.cache.hit_rate` | Gauge | `< 90% on confidence reads` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status:                 "healthy" | "degraded" | "unavailable",
  dependencies: {
    moirai:               "healthy" | "unavailable",
    fgts:                 "healthy" | "unavailable",
    redis:                "healthy" | "unavailable",
    mds_kronos:           "healthy" | "unavailable"
  },
  active_posteriors:      int,
  stale_flagged_count:    int,
  cache_hit_rate:         float,
  pending_updates:        int,       # queued posterior updates
  last_event_hash:        str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":           "ISO-8601",
  "service":             "TCS/MIMIR",
  "level":               "INFO | WARN | ERROR",
  "event":               "POSTERIOR_UPDATED | STATE_TRANSITIONED | GAMING_DETECTED | STALE_FLAGGED",
  "posterior_id":        "uuid | null",
  "session_id":          "uuid | null",
  "domain":              "string",
  "claim_type":          "string",
  "model_version":       "string",
  "duration_ms":         0,
  "fields": {
    "state":             "PRIOR_ONLY | CALIBRATING | CALIBRATED",
    "n_observations":    0,
    "gaming_probability":0.0
  }
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/tcs/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant
- **Prev_event_hash source:** Prior TCS event in the platform-wide calibration event stream (not session-scoped — calibration events span sessions)

### 10.2 What This Service Attests

The MOIRAI record for TCS proves that specific calibration updates were applied to specific posteriors at specific times, and that those posterior states have not been altered since they were recorded. An oversight body can reconstruct the full calibration history for any analyst-domain combination by querying TCS events.

### 10.3 What This Service Cannot Prove

The calibration record does not prove the posterior accurately reflects the analyst's true accuracy. If FGTS was corrupted before its inputs reached TCS, the posterior reflects a corrupted training signal. TCS attests the update was applied as specified; it does not attest the specification was accurate.

---

## 11. Implementation Roadmap

### Phase 1 — Posterior Engine and Confidence Signals (Weeks 9–16)

- CalibrationPosterior schema with Beta distribution
- Benchmark prior initialization from evaluation data
- Calibration state machine: PRIOR ONLY → CALIBRATING → CALIBRATED
- `GET /confidence` endpoint (served from priors initially)
- `POST /calibration/update` endpoint for FGTS inputs
- MOIRAI event emission: TCS_CALIBRATION_UPDATED, TCS_STATE_TRANSITIONED
- Redis hot cache for active session posteriors

**Phase gate criterion:** Confidence signals served to ATHENA for all active sessions. State machine transitions validated in test. MOIRAI events produced and chained correctly.

### Phase 2 — Gaming Detection, Supervisory Dashboard, and Model Version Handling (Weeks 17–28)

- Gaming probability score computation (five-signal composite)
- `POST /gaming/score` endpoint for FGTS
- Supervisory dashboard endpoint with de-identified team view
- MDS/KRONOS integration for stale flagging on model version change
- CPS/APORIA integration for capability-zone-stratified calibration
- Hard confidence ceiling enforcement for red-zone claims
- IOB audit endpoints

**Phase gate criterion:** Gaming probability score computed per session. Stale flagging fires correctly on test model version change. Red-zone hard ceiling applied and verified. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| None | No GC items gate TCS deployment. | — |

*Note: The supervisory dashboard design — specifically what calibration data supervisors can see and in what form — may require policy guidance from the analytic standards authority before full deployment. The current design de-identifies analyst data in the supervisor view; any proposal to expose individual analyst identities requires explicit policy approval.*

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In ATHENA, the calibration state indicator appears in the session header: a green dot (CALIBRATED), amber dot (CALIBRATING), or grey dot (PRIOR ONLY). Hovering shows the state definition and what it means for the reliability of confidence signals in this session. Confidence signals (high/medium/low) appear on each AI claim. When a hard ceiling is applied (red-zone claim), the confidence signal shows "CAPABILITY CEILING APPLIED" rather than a standard confidence level.

### 13.2 What the Analyst Should Do

In PRIOR ONLY or CALIBRATING state: treat confidence signals as initial estimates, not empirical measurements. Apply more independent verification than the signal alone would suggest. The calibration system needs data to become accurate — genuine verification behaviour contributes to this.

When a capability ceiling is applied: verify the claim through an independent path. The ceiling indicates the system has assessed this claim type as outside the model's reliable capability in this domain. The appropriate response is independent collection or explicit acknowledgment that the claim cannot be verified.

### 13.3 What the Signal Does Not Mean

CALIBRATED state does not mean the analyst's judgments are correct — it means their reliance on AI has been empirically calibrated to the model's accuracy in this domain. An analyst can be well-calibrated and wrong. A PRIOR ONLY indicator does not mean the AI is inaccurate — it means the system lacks sufficient data to characterize this analyst's accuracy in this combination.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Minimum N for reliable calibration.** What is the minimum number of weighted corrections before the CALIBRATING → CALIBRATED transition is meaningful? The current threshold (configurable, default 30 weighted observations) is a design choice, not an empirically derived threshold. Resolution path: Research and Red Team pre-registered study comparing calibration accuracy at N=10, 30, 50, 100.
- **Q2: Cross-domain calibration transfer.** Can calibration in a well-covered domain inform priors in an adjacent domain? If an analyst has high accuracy on technical intelligence claims in one domain, should their prior for technical claims in an adjacent domain be stronger than a generic benchmark? Currently: no transfer. This is conservative but potentially slow to reach CALIBRATED in new domains. Resolution path: empirical study of calibration transfer patterns once 6 months of operational data exists.

### 14.2 Research Dependencies

- **RQ-1 (Addendum D): Reliable epistemic self-assessment in LLMs.** TCS replaces model self-reported confidence with empirically calibrated signals. This is correct. But TCS's confidence signals are only as good as the verification data they're based on. If analysts cannot reliably verify claims (especially PARAM claims), the calibration data is itself unreliable. This is a known limitation, not a solvable problem with current techniques — acknowledged as a design constraint.

### 14.3 Operational Assumptions

- **Assumption 1: Analysts engage genuinely with the verification interface.** TCS calibration depends on verification data from FGTS. If analysts performatively verify (triggering the gaming detection), calibration is compromised. The gaming detection is the mitigation — but it requires the gaming signal threshold to be set correctly.
- **Assumption 2: The analyst pool is large enough for meaningful aggregate statistics.** The supervisory dashboard aggregate metrics and IOB reporting are meaningful only if the pool is sufficiently large. For a team of 5 analysts, individual-level patterns dominate the aggregate. Minimum meaningful pool size for aggregate monitoring: approximately 15 analysts in the same domain-claim-type combination.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

---

## Appendix D: Red Team Findings

*Pending red team evaluation — scheduled for Phase 3 gate review.*
