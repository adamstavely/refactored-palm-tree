# WSF — Weak Signal Fusion Service
### LACHESIS · *"One of the three Fates — she who measures the thread, determines its length, and sequences its unfolding; the one who sees which small threads compose the larger tapestry"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `WSF` |
| **Epithet** | `LACHESIS` |
| **Full name** | Weak Signal Fusion Service |
| **Namespace** | `themis-warning` |
| **Layer** | Intelligence Layer — Warning |
| **Build phase** | Year 3 |
| **Build priority** | 12 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — fuses individually insignificant signals into analytically significant patterns |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**WSF/LACHESIS answers: Are there dispersed, individually weak signals across multiple sources and collection types that, taken together, constitute an analytically significant pattern that no individual signal would surface?**

### 1.2 Why This Service Exists

Intelligence failures are frequently failures to connect small pieces of evidence that each, individually, fell below the analytical threshold. The pieces existed in the corpus; the pattern they formed when assembled did not get assembled. Every analyst who touched one piece thought it insufficient to act on. No analyst saw all the pieces together.

LACHESIS is the service that assembles the pieces. It aggregates weak signals — individually below the analytical threshold, individually ambiguous, individually dismissible — and detects when their combination forms a pattern that has historical precedent as a precursor to significant events. The fusion function is the key design element: not just counting signals but identifying whether their combination, in a specific temporal sequence, matches known precursor patterns.

### 1.3 Design Principles

- **Sequence matters as much as count.** Four weak signals appearing in random order are less significant than the same four signals appearing in a historically established temporal sequence. LACHESIS tracks sequence, not just presence.
- **Weak signal sources are heterogeneous.** Weak signals come from CASSANDRA anomalies, SCRIBE document updates, ARGUS-LACUNA coverage changes, TRS leading indicator status changes, and directly from corpus additions. LACHESIS aggregates across all these types.
- **Pattern matching is against historical precedent.** LACHESIS does not invent new patterns — it matches detected signal combinations against historically validated precursor patterns. New patterns require human analytical validation before being added to the pattern library.
- **Fusion results feed SENTINEL.** LACHESIS is not the warning endpoint — it is the fusion layer that produces structured signal fusions for SENTINEL to synthesise into strategic warnings.

---

## 2. Core Responsibilities

### 2.1 Primary Function

WSF/LACHESIS aggregates weak signals from across the intelligence layer, applies temporal sequence analysis to identify whether signal combinations match historically validated precursor patterns, produces SignalFusion records with aggregate strength and historical precedent attribution, and publishes fusion events to SENTINEL for strategic warning synthesis.

### 2.2 Secondary Functions

- Weak signal ingestion: receiving signals from CASSANDRA, SCRIBE, ARGUS-LACUNA, TRS, and corpus additions
- Pattern library management: maintaining the library of historically validated precursor patterns (analyst-validated, not AI-generated)
- Temporal window management: tracking which signals are still within the temporal window relevant to an emerging pattern
- Aggregate strength computation: computing the overall signal strength from individual weak signal contributions and sequence completion
- Signal decay: reducing the weight of signals as they age beyond their historically established temporal window

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
WeakSignal:
  signal_id:               uuid
  source_service:          CASSANDRA | SCRIBE | ARGUS_LACUNA | TRS | CORPUS
  source_ref_id:           uuid | null
  domain:                  str
  entity_id:               uuid | null
  signal_type:             str
  strength:                float            # individually small: < 0.3 to qualify
  content_hash:            str
  detected_at:             datetime
  expires_at:              datetime         # temporal window expiry

PrecursorPattern:
  pattern_id:              uuid
  name:                    str
  domain:                  str
  signal_sequence:         [SignalStep]     # ordered sequence defining the pattern
  historical_evidence:     int              # validated historical cases
  typical_lead_days:       int              # how many days before significant event
  analyst_validated:       bool             # human validation required
  validated_by:            str | null
  validation_date:         datetime | null

SignalStep:
  step_id:                 uuid
  pattern_id:              uuid
  sequence_position:       int
  signal_type:             str
  source_service:          str
  max_days_after_prior:    int              # temporal constraint on sequence

SignalFusion:
  fusion_id:               uuid
  pattern_id:              uuid
  matching_signals:        [{ signal_id: uuid, step_id: uuid }]
  sequence_complete:       bool             # all steps matched
  sequence_completion:     float            # fraction of steps matched
  aggregate_strength:      float
  temporal_coherence:      float            # how well signals match expected timing
  historical_precedent:    str
  domain:                  str
  entity_id:               uuid | null
  first_signal_at:         datetime
  last_signal_at:          datetime
  status:                  EMERGING | ESTABLISHED | PEAKED | DECLINING
  sentinel_notified:       bool
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Signal store | PostgreSQL | WeakSignal (rolling 90-day window) | 90 days |
| Pattern library | PostgreSQL | PrecursorPattern, SignalStep | Indefinite |
| Fusion store | PostgreSQL | SignalFusion | Indefinite |
| Active pattern cache | Redis | Emerging fusions (fast signal matching) | Pattern TTL |
| Event store | MOIRAI | Signed fusion events | Indefinite |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /signals/ingest
  Auth:     CASSANDRA | SCRIBE | TRS | ARGUS-LACUNA service accounts
  Request:  {
    source_service:        str,
    source_ref_id:         uuid | null,
    domain:                str,
    entity_id:             uuid | null,
    signal_type:           str,
    strength:              float,
    content_hash:          str
  }
  Response: {
    signal_id:             uuid,
    fusion_matches:        int,            # how many active fusions this contributes to
    new_fusions_created:   int
  }
  SLA: p99 < 500ms

GET /fusions/active
  Auth:     analyst session token | supervisor token | SENTINEL service account
  Params:   domain: str, status: str, min_strength: float
  Response: [SignalFusion]

GET /fusions/{fusion_id}
  Auth:     analyst session token | supervisor token
  Response: SignalFusion with full signal detail

GET /patterns
  Auth:     analyst session token | Research & Red Team
  Response: [PrecursorPattern]

POST /patterns
  Auth:     Research & Red Team token
  Request:  PrecursorPattern (analyst_validated must be true with validation_by and date)
  Response: { pattern_id: uuid }

GET /health
  Response: {
    status, dependencies: { moirai, pces, redis },
    active_signals:        int,
    active_fusions:        int,
    pattern_count:         int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          WSF_FUSION_ESTABLISHED
service_id:         "WSF"
session_id:         null
classification:     str
event_payload:
  fusion_id:              uuid
  pattern_id:             uuid
  domain:                 str
  aggregate_strength:     float
  sequence_complete:      bool
  signal_count:           int
prev_event_hash:    str
signature:          str
timestamp:          datetime
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `WSF_FUSION_ESTABLISHED` | New fusion detected | MOIRAI, SENTINEL, PYTHIA |
| `WSF_FUSION_STRENGTHENED` | Fusion aggregate_strength crosses threshold | MOIRAI, SENTINEL |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| CASSANDRA | Anomaly Detection | Anomaly signals as strong weak inputs | Async event | Fewer signals; fusion quality degrades |
| MOIRAI | Provenance | Signed fusion events | Async event | Events buffered |

### 5.2 Feeds Into

| Service | Epithet | What LACHESIS provides | How |
|---|---|---|---|
| SENTINEL | Strategic Warning | Signal fusions for warning synthesis | Event + API |
| PYTHIA | Anticipatory | Emerging fusion alerts for active sessions | Event |
| IOB Reporting | Oversight | Active fusion summary | API |

---

## 6. Known Design Risks

- **The precursor pattern library is the most analytically sensitive component.** Patterns encode what combinations of signals historically preceded significant events. This library is classified and analytically authoritative. The Research & Red Team must develop and validate patterns from historical IC cases before Phase 1 deployment — without a validated pattern library, LACHESIS produces no fusions. Developing the pattern library is a multi-month analytical effort.
- **Temporal coherence scoring is complex.** Whether signals are appearing in the expected temporal sequence requires knowing what the expected sequence is — which requires the precursor pattern to specify timing constraints. Patterns without good timing data produce permissive temporal coherence scores. Resolution path: the pattern library specification must include best-available timing data from historical cases; uncertain timing expressed as wide windows rather than precise thresholds.

---

## 7. Implementation Roadmap

### Phase 1 — Signal Ingestion and Pattern Matching (Year 3, Weeks 1–12)

- WeakSignal, PrecursorPattern, SignalFusion schemas
- Signal ingestion endpoint for all source services
- Pattern matching engine with sequence tracking
- Initial precursor pattern library (Research & Red Team to deliver before Phase 1)
- MOIRAI events; SENTINEL integration

**Phase gate criterion:** Signal ingestion from CASSANDRA, SCRIBE, TRS operational. Pattern matching produces fusions on test signal sets that match known precursor patterns. Pattern library contains at least 10 validated patterns before deployment.

### Phase 2 — Temporal Coherence, Signal Decay, and Pattern Expansion (Year 3, Weeks 13–24)

- Full temporal coherence scoring
- Signal decay beyond pattern temporal windows
- Research & Red Team pattern expansion pipeline
- PYTHIA fusion alerts

**Phase gate criterion:** Temporal coherence scoring differentiates ordered from unordered signal sets. Signal decay functions correctly. ARB sign-off.

---

## 8. Policy Dependencies

No GC items gate LACHESIS. The precursor pattern library is a classified analytical resource maintained by the Research & Red Team under the analytic standards authority.

---

## 9. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 3 gate review. Research & Red Team must deliver validated precursor pattern library before Phase 1.*
