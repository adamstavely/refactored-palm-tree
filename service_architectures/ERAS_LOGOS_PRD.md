# ERAS — Explainability & Reasoning Audit Service
### LOGOS · *"Greek for 'reason' / 'word'"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `ERAS` |
| **Epithet** | `LOGOS` |
| **Full name** | Explainability & Reasoning Audit Service |
| **Namespace** | `themis-quality` |
| **Layer** | Knowledge Layer |
| **Build phase** | Phase 7–8 (Weeks 61–66) |
| **Build priority** | 9 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — surfaces the reasoning chain behind AI claims |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**ERAS/LOGOS answers: Why did the AI say what it said?**

The Provenance Service answers *where did this content come from?* TVS/KAIROS answers *does this evidence still hold?* ERAS answers a third, distinct question about the reasoning process itself — not the sources used, but how the AI moved from those sources to its output.

### 1.2 Why This Service Exists

In intelligence analysis, an analyst has a duty of analytical tradecraft — they must understand the basis for any assessment or recommendation they adopt. AI that produces an assessment without an articulable reasoning chain creates analytical accountability risk: the analyst cannot defend their work product, oversight bodies cannot evaluate the analytical process, and the platform cannot distinguish a well-reasoned assessment that happens to be wrong from a poorly-reasoned assessment that happens to be right.

Without ERAS, the provenance record shows what sources were retrieved. It does not show how the AI used those sources to reach a conclusion, which claims were supported by retrieved evidence, and which were parametric assertions with no retrieval backing. The Origin axis is incomplete without reasoning capture.

### 1.3 Why This Service Is Positioned Last

ERAS is the highest-build-priority platform service because it requires everything else to be trustworthy before its output is meaningful.

An explanation of reasoning derived from improperly retrieved content, evaluated without compartment enforcement, with miscalibrated analyst trust is an explanation of a compromised output. Build the conditions for trustworthy AI first — PCES, MOIRAI, TCS, FGTS, IAS — then audit the reasoning. An ERAS record attached to a session where the retrieval was adversarially manipulated documents a corrupted reasoning process, not a sound one.

### 1.4 Design Principles

- **Explanation records are evidence, not debugging artifacts.** The ReasoningCapture is an oversight-submissible record, not a developer log. Its content, completeness, and integrity must meet the same standards as MOIRAI provenance events.
- **ERAS generates data; humans decide what it means.** ERAS does not label unsupported claims as hallucinations definitively. The model may draw on valid parametric knowledge. ERAS flags; the analyst verifies.
- **Completeness scoring reflects process, not conclusion.** A low completeness score indicates analytical process quality, not whether the conclusion is correct.
- **Policy neutrality on disclosure.** ERAS generates structured disclosure data in configurable formats. What is disclosed, to whom, and when is a policy decision owned by the appropriate analytic authority — not the platform.

### 1.5 Explicit Out of Scope

- **Determination of whether claims are factually correct.** ERAS flags unsupported claims; it does not independently verify them.
- **Attribution of intent to the AI.** ERAS captures what reasoning was expressed; it does not infer whether the model was "trying" to mislead.
- **Replacement for analyst review.** ERAS surfaces reasoning structure; an analyst remains responsible for evaluating whether that reasoning is sound.

---

## 2. Core Responsibilities

### 2.1 Primary Function

ERAS captures the reasoning chain behind every AI response — the chain-of-thought, the claims made, the evidence cited for each claim, and the claims that have no retrieval backing — and stores these as structured, queryable reasoning records linked to the MOIRAI provenance chain. It enables analysts to ask "why did the AI say that?" and receive a structured breakdown; oversight bodies to query aggregate unsupported claim rates by analyst, interaction class, and domain; and the platform to use reasoning quality signals as a calibration input to TCS/MIMIR.

### 2.2 Secondary Functions

- Completeness scoring per session turn (ratio of supported to total claims, confidence coverage, alternative consideration)
- Confidence signal extraction and normalization from linguistic uncertainty markers
- Reasoning record export for IOB oversight reporting and tradecraft compliance disclosure
- Aggregate audit queries: unsupported claim rates by analyst, by interaction class, by domain

### 2.3 What This Service Does Not Decide

ERAS generates the structured reasoning record. What portions of that record are disclosed, in which format, to which oversight authority, and on what schedule are policy decisions owned by the analytic standards authority and general counsel. The platform is designed to be policy-neutral on disclosure obligations while ensuring the data exists to satisfy whatever policy applies.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ReasoningCapture:
  capture_id:           uuid
  turn_id:              uuid          # FK → MOIRAI turn record
  session_id:           uuid          # FK → MOIRAI session record
  interaction_class:    str           # RESEARCH | EVIDENCE_ANALYSIS | SYNTHESIS | etc.
  domain:               str           # analytical domain for calibration routing
  chain_of_thought:     str           # raw chain-of-thought text from model
  claims:               [Claim]       # all claims identified in the response
  confidence_signals:   [Signal]      # extracted linguistic uncertainty markers
  alternatives:         [str]         # alternative framings model considered
  evidence_citations:   [chunk_id]    # source chunks model explicitly cited
  unsupported_claims:   [Claim]       # claims with no retrieval backing in context
  completeness:         CompletenessScore
  classification:       str           # inherited from session
  created_at:           datetime

Claim:
  claim_id:             uuid
  capture_id:           uuid          # FK → ReasoningCapture
  text:                 str           # verbatim claim text
  claim_type:           factual | analytical | strategic | procedural
  supporting_chunks:    [chunk_id]    # FK → retrieval chunks
  confidence:           high | medium | low | uncertain
  validity_at_capture:  float         # TVS score of supporting chunks at capture time
  is_unsupported:       bool          # true if supporting_chunks is empty
  flagged_for_review:   bool

Signal:
  signal_id:            uuid
  capture_id:           uuid
  text:                 str           # extracted linguistic phrase
  level:                definitive | strong | qualified | uncertain
  claim_id:             uuid | null   # FK → associated claim, if attributable

CompletenessScore:
  capture_id:           uuid
  supported_claim_ratio: float        # claims with evidence / total claims
  confidence_coverage:  float         # claims with explicit confidence signal / total
  alternatives_present: bool          # were alternative framings surfaced?
  evidence_validity_mean: float       # mean TVS score of cited evidence at capture
  composite:            float         # weighted combination (weights configurable)
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | ReasoningCapture, Claim, Signal, CompletenessScore records | Session + 7 years |
| Event store | MOIRAI (signed events) | Immutable reasoning event records | Indefinite |
| Index | Elasticsearch | Full-text search across chain_of_thought and claim text | Mirrors primary |
| Adversarial index | Elasticsearch (separate index) | Reasoning records from adversarial sessions | Mirrors primary + special access |

The adversarial index is maintained as a distinct category — highest-value content for understanding model failure modes, accessible only to the Research and Red Team and IOB.

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| ReasoningCapture | Inherited from session | PCES enforced at write and query time |
| Claim text | Inherited from session | Cannot be queried across compartment boundaries |
| Completeness scores | Inherited from session | Aggregate queries available only within compartment |
| Client disclosure reports | Configurable (typically lower than session) | Sanitised by disclosure module before delivery |

### 3.4 Retention and Purge Policy

ReasoningCapture records and associated Claim and Signal records are retained for the life of the session plus seven years, consistent with analytical record retention requirements. CompletenessScore records are retained indefinitely for calibration model training purposes. MOIRAI-signed ERAS events cannot be purged without breaking the hash chain; purge authority applies only to the PostgreSQL operational store and requires IOB sign-off.

---

## 4. API Contract

### 4.1 Endpoints

```
# Analyst query — why did the AI reason this way?
GET /explain/{turn_id}
  Auth:     session token (PCES-validated)
  Response: {
    summary:                str,        # plain-language reasoning summary
    chain_of_thought:       str,        # full chain-of-thought text
    key_claims: [
      {
        text:               str,
        claim_type:         str,
        confidence:         str,
        supporting_evidence:[{ chunk_id, source_ref, validity_score }],
        is_unsupported:     bool
      }
    ],
    unsupported_claims:     [{ text, claim_type }],
    alternatives_considered:[str],
    confidence_summary:     { high: int, medium: int, low: int, uncertain: int },
    completeness:           { composite: float, supported_ratio: float }
  }

# Compliance query — aggregate unsupported claim rates
GET /audit/{session_id}/claim-quality?class={interaction_class}&domain={domain}
  Auth:     supervisor token or IOB token
  Response: {
    session_id:             uuid,
    filters:                { class: str, domain: str },
    unsupported_claim_rate: float,
    turn_count:             int,
    completeness_mean:      float,
    by_analyst:             [{ analyst_id, unsupported_rate, turn_count }]
  }

# Oversight query — full reasoning record for a session
GET /session/{session_id}/reasoning-record
  Auth:     IOB token
  Response: {
    session_id:             uuid,
    turn_count:             int,
    captures:               [ReasoningCapture],
    aggregate_completeness: float,
    unsupported_claim_rate: float,
    provenance_certificate: str     # MOIRAI chain reference
  }

# Disclosure report — for oversight submission
GET /disclosure/{session_id}?format={jurisdiction_code}
  Auth:     disclosure authority token
  Response: {
    report:  str,           # plain-language AI usage summary
    format:  str,           # jurisdiction_code used
    redacted:bool           # true if prompt content / analyst identities redacted
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:           ERAS_CAPTURE_CREATED
service_id:          "ERAS"
turn_id:             uuid
session_id:          uuid
classification:      str
event_payload:
  capture_id:              uuid
  unsupported_claim_count: int
  completeness_composite:  float
  claim_count:             int
  had_alternatives:        bool
prev_event_hash:     str
signature:           str
timestamp:           datetime

EventType:           ERAS_UNSUPPORTED_CLAIM_FLAGGED
event_payload:
  capture_id:        uuid
  claim_id:          uuid
  claim_text:        str           # verbatim claim flagged
  claim_type:        str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `ERAS_CAPTURE_CREATED` | Every session turn with AI response | TCS/MIMIR, PGS/NOMOS, IOB reporting |
| `ERAS_UNSUPPORTED_CLAIM_FLAGGED` | When is_unsupported is true on any Claim | ATHENA (surface in verification queue), TCS/MIMIR |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| MOIRAI | `SESSION_TURN_CREATED` | Triggers reasoning capture for the new turn |
| TVS/KAIROS | `SOURCE_VALIDITY_UPDATED` | Updates validity_at_capture for affected claims |
| MDS/KRONOS | `MODEL_VERSION_CHANGED` | Flags all active ReasoningCaptures for re-baseline annotation |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Receives turn_id for linkage; emits signed events | Async event | Capture buffered locally; replayed when MOIRAI recovers |
| TVS/KAIROS | Temporal Validity | Evidence validity scores at capture time frozen in record | Sync query | Capture proceeds; validity_at_capture marked as unavailable |
| TCS/MIMIR | Trust Calibration | Unsupported claim rates as calibration signal input | Async event | Calibration update deferred; no analyst-facing impact |
| PGS/NOMOS | Analytic Standards | Completeness scores feed output screening threshold | Async event | Screening threshold reverts to default; alert fired |
| PCES/AEGIS | Classification Enforcement | All query endpoints gated by PCES compartment check | Sync | Request blocked; PCES_UNAVAILABLE returned |

### 5.2 Feeds Into

| Service | Epithet | What ERAS provides | How |
|---|---|---|---|
| TCS/MIMIR | Trust Calibration | Unsupported claim rates as additional calibration signal | MOIRAI event → TCS consumer |
| PGS/NOMOS | Analytic Standards | CompletenessScore feeds output screening threshold; low completeness tightens screening | MOIRAI event |
| ATHENA | Interface | Unsupported claim flag surfaces in verification queue | API call at session render |
| IOB Reporting | Oversight | Aggregate claim quality data; disclosure reports | Audit query endpoint |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Every `/explain`, `/audit`, and `/session` query endpoint validates the requesting session token against PCES before returning any data.
- **Compartment inheritance:** ReasoningCapture records inherit the classification of the session they belong to. Cross-compartment queries are rejected.
- **Failure behavior:** If PCES is unreachable, all data-returning endpoints return `503 PCES_UNAVAILABLE`. Capture creation (write path) continues — data is written with the classification metadata from the session context already validated at session start.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| `/explain/{turn_id}` | 200ms | 500ms | 1000ms |
| Reasoning capture (per turn, async) | 500ms | 1500ms | 3000ms |
| `/audit` compliance query | 500ms | 2000ms | 5000ms |
| MOIRAI event emit | 100ms | 300ms | 500ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Peak capture requests/second | 50 (matches platform peak session throughput) |
| Sustained capture requests/second | 20 |
| `/explain` queries/second | 10 |
| Audit queries/second | 2 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% (non-critical path — capture failures buffer, not block) |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Captures buffered in local queue; replayed on recovery. `/explain` still available from PostgreSQL. | None during outage; provenance chain gap logged |
| TVS/KAIROS | Capture proceeds; `validity_at_capture` fields marked `null`. | Completeness scores degrade (validity component missing) |
| TCS/MIMIR | Calibration signals queued; replayed when available. | No analyst-facing impact |
| PGS/NOMOS | Output screening threshold reverts to default (most restrictive). Alert fired. | Analyst may experience tighter output screening than calibrated |
| PCES/AEGIS | All query endpoints return `503`. Write path continues. | Analysts cannot query reasoning explanations until PCES recovers |

---

## 7. Security Model

### 7.1 Authentication

All endpoints require session tokens validated by PCES/AEGIS. Audit and disclosure endpoints require supervisor or IOB tokens. No unauthenticated endpoints.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst (ATHENA session) | `/explain/{turn_id}` for own turns only | Session token from PCES |
| Supervisor | `/audit` queries within their team's compartment | Role-based supervisor token |
| IOB / Oversight | `/session/{id}/reasoning-record`, `/disclosure` | IOB token, read-only |
| Agent session (SCBS-bounded) | No direct ERAS access | Agents do not query reasoning records |
| Research & Red Team | Full read including adversarial index | Research team token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/eras/signing-key` | 90 days |
| PostgreSQL credentials | `themis/eras/db-credentials` | 30 days |
| Elasticsearch credentials | `themis/eras/es-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

The reasoning capture prompt injection is the primary threat: if adversarial content in the retrieved corpus manipulates the model's chain-of-thought, the ReasoningCapture records a manipulated reasoning process. IAS/SCUDO screens retrieved content before it reaches the context window; ERAS records what the model actually produced, whether or not the pre-screen caught everything. The adversarial index exists precisely to make these cases available for Research and Red Team analysis.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Chain-of-thought extraction fails (model doesn't produce structured CoT) | High | P2 — degraded explanation quality | `capture.chain_of_thought` null rate > 5% | Structured CoT prompting in system templates; fallback extraction heuristics |
| Unsupported claim detection rate too high (false positives) | Medium | P2 — analyst alert fatigue | Analyst dismissal rate > 80% on flagged claims | Confidence threshold tuning; domain-specific calibration |
| MOIRAI event queue backlog | Low | P1 — provenance gap grows | Queue depth > 1000 events | Auto-scaling consumer; alert at 500 |
| Elasticsearch index lag | Low | P2 — audit queries return stale data | Index lag > 60s | Staleness indicator in audit response |

### 8.1 Known Design Risks

- **CoT extraction reliability varies by model.** Chain-of-thought prompting produces structured reasoning from most current models but is not guaranteed. A model update that changes CoT formatting breaks the extraction pipeline. Mitigation: extraction is model-version-aware; MDS/KRONOS triggers re-calibration on model change.
- **Unsupported claim detection has no ground truth.** The model may generate correct claims from parametric knowledge that appear as "unsupported" because no chunk was retrieved. There is no current method to distinguish true hallucinations from valid parametric claims. ERAS flags; it does not determine. This limitation must be stated explicitly in analyst training (Module 1).
- **Completeness scoring weights are configurable but not yet empirically validated.** The composite score weights are reasonable defaults; they have not been validated against analytical accuracy outcomes. OFS/NEMESIS outcome data is required before weight calibration is possible (Year 2).

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `eras.capture.latency_p99` | Histogram | `> 3000ms for 5m` | P1 |
| `eras.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `eras.unsupported_claim.rate` | Gauge | `> 40% sustained over 1h` | P2 |
| `eras.completeness.mean` | Gauge | `< 0.5 sustained over 1h` | P2 |
| `eras.cot_extraction.null_rate` | Counter | `> 5% over 30m` | P1 |
| `eras.pces.rejection_rate` | Counter | Spike > 10x baseline | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status:           "healthy" | "degraded" | "unavailable",
  dependencies: {
    moirai:         "healthy" | "unavailable",
    tvs:            "healthy" | "unavailable",
    pces:           "healthy" | "unavailable",
    elasticsearch:  "healthy" | "degraded" | "unavailable"
  },
  moirai_sync:      boolean,
  last_event_hash:  string,
  queue_depth:      int,          # pending MOIRAI events
  index_lag_ms:     int           # Elasticsearch indexing lag
}
```

### 9.3 Log Schema

```json
{
  "timestamp":      "ISO-8601",
  "service":        "ERAS/LOGOS",
  "level":          "INFO | WARN | ERROR",
  "event":          "CAPTURE_CREATED | CLAIM_FLAGGED | QUERY_SERVED | ...",
  "session_id":     "uuid | null",
  "turn_id":        "uuid | null",
  "capture_id":     "uuid | null",
  "classification": "string",
  "duration_ms":    0,
  "fields": {
    "unsupported_count": 0,
    "claim_count":       0,
    "completeness":      0.0
  }
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/eras/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant
- **Prev_event_hash source:** Prior ERAS event in the same session's event stream

### 10.2 What This Service Attests

The MOIRAI record for ERAS proves that at a specific time in a specific session turn, the AI produced a reasoning chain containing N claims, of which M were unsupported by retrieved evidence, and that the composite completeness score was X. The record has not been altered since it was written. An oversight body can query the ERAS event chain to reconstruct the full analytical reasoning history for any session.

### 10.3 What This Service Cannot Prove

The record does not prove that the chain-of-thought accurately reflects the model's internal computation — chain-of-thought is produced text, not a window into model weights. It does not prove that unsupported claims are factually wrong. It does not prove that supported claims are factually correct, only that the model cited evidence for them.

---

## 11. Implementation Roadmap

### Phase 1 — Capture Foundation (Weeks 61–62)

- Structured reasoning prompts in system prompt templates (coordinate with PRS/PROMETHEUS)
- ReasoningCapture schema; raw chain-of-thought stored per turn
- Basic claim extraction: factual and analytical assertion identification
- Turn_id linkage to MOIRAI provenance record
- `ERAS_CAPTURE_CREATED` event emission to MOIRAI

**Phase gate criterion:** Every session turn produces a ReasoningCapture record with non-null chain_of_thought and at least basic claim identification. MOIRAI event emission verified by hash chain audit.

### Phase 2 — Indexing and Quality Model (Weeks 63–64)

- Claim-to-evidence mapping: supporting_chunks linked to each claim
- Unsupported claim detection and `ERAS_UNSUPPORTED_CLAIM_FLAGGED` events
- Confidence signal extraction: linguistic uncertainty normalization
- CompletenessScore computation per turn
- Unsupported claim surface in ATHENA verification queue
- TCS/MIMIR integration: unsupported claim rates as calibration signal

**Phase gate criterion:** Unsupported claim rate measurable per analyst per domain. TCS/MIMIR receiving calibration signals from ERAS. CompletenessScore available on all captures.

### Phase 3 — Oversight and Audit Surface (Weeks 65–66)

- `/explain/{turn_id}` analyst query endpoint
- `/audit` compliance query endpoint with compartment enforcement
- `/session/{id}/reasoning-record` IOB query endpoint
- `/disclosure` report generator with jurisdiction-configurable formats
- Adversarial reasoning records indexed as distinct category

**Phase gate criterion:** IOB can query full reasoning record for any session by session_id. Disclosure report generator produces jurisdiction-compliant output. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| None | No GC items gate ERAS deployment specifically. | — |

*Note: ERAS disclosure report output formats are jurisdiction-configurable without engineering changes. Policy decisions about what to disclose, to whom, and when are owned by the analytic standards authority. ERAS is designed to be policy-neutral.*

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In the ATHENA verification queue, claims flagged as unsupported appear with an ERAS indicator: a chain-link icon with a broken link. Expanding the claim shows the full text, the claim type, and the message: "No retrieved evidence supports this claim. The AI may have drawn on parametric knowledge. Verify independently before use."

The `/explain` endpoint is exposed in ATHENA as an expandable "reasoning breakdown" panel on any AI response, showing the key claims, their confidence levels, their supporting evidence chunks with validity scores, and the alternatives the AI considered.

### 13.2 What the Analyst Should Do

When an unsupported claim flag appears: do not accept the claim at face value. The correct verification path is independent collection, subject matter expert consultation, or explicit acknowledgment that the claim could not be verified. "Verified independently" is the correct action — not "confirmed against source" (there is no specific source to check). The distinction matters for FGTS weighting.

When reviewing the reasoning breakdown: check whether the alternatives the AI considered included the analytical counter-position. If no alternative framing was surfaced, trigger the counter-position intervention manually before proceeding.

### 13.3 What the Signal Does Not Mean

An unsupported claim flag does not mean the claim is wrong. The AI may be drawing on accurate parametric knowledge — a capability or program characteristic that exists in training data but was not retrieved in this session. The flag means the analyst must verify through an independent path, not that the claim is false.

A low completeness score does not mean the analytical conclusion is incorrect. It means the reasoning process was incomplete by the platform's standards. An analytically sound conclusion can follow from reasoning that the completeness model rates poorly.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Chain-of-thought consistency across model versions.** Current extraction assumes the model produces CoT in a parseable format. Different model versions produce different CoT structures. Resolution path: extraction module is model-version-aware, with format configurations maintained per version in a service config store. MDS/KRONOS triggers re-calibration on version change.

- **Q2: Claim extraction accuracy on complex multi-sentence reasoning.** The claim extraction pipeline has been designed but not benchmarked on intelligence-domain analytical text. Resolution path: Research and Red Team benchmarking against a sample of closed requirements with known outcomes (Phase 2 gate requirement).

### 14.2 Research Dependencies

- **No open research dependencies.** ERAS does not depend on unsolved ML problems. Chain-of-thought prompting, claim extraction, and completeness scoring are implementable with current technology. The known limitation (unable to distinguish valid parametric knowledge from hallucination) is a design constraint, not a research dependency.

### 14.3 Operational Assumptions

- **Assumption 1: Models will produce structured chain-of-thought when prompted.** Risk if wrong: CoT null rate rises, reasoning capture degrades to shallow heuristics. Mitigation: fallback extraction pipeline for unstructured reasoning text.
- **Assumption 2: Analysts will read the reasoning breakdown before accepting AI conclusions.** Risk if wrong: ERAS data exists but doesn't change behaviour. Mitigation: ATHENA intervention design (Module 1 training, verification queue gate) creates structural friction that requires engagement with ERAS outputs.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD — adapted from ERAS_Architecture.md source document |

---

## Appendix A: Confidence Signal Taxonomy

| Level | Example language | Analyst guidance |
|---|---|---|
| **Definitive** | "The intelligence clearly indicates..." / "unambiguous" | High — strong claim; verify evidence currency via TVS |
| **Strong** | "Collection has consistently shown..." / "well-established" | Medium-high — verify citation currency |
| **Qualified** | "It appears that..." / "arguably" / "likely" | Medium — flag for independent review |
| **Uncertain** | "This is a developing situation..." / "contested" | Low — do not rely without independent collection |

---

## Appendix B: Query Patterns

```
# Analyst: why did the AI assess adversary capability this way?
GET /explain/3fa85f64-5717-4562-b3fc-2c963f66afa6
→ {
    summary: "The assessment rests on 4 primary claims, 2 of which are unsupported by
              retrieved evidence. The model considered 2 alternative framings.",
    key_claims: [
      { text: "The program has reached Stage 3 development...",
        confidence: "strong",
        supporting_evidence: [
          { chunk_id: "c-001", source_ref: "SIGINT-2025-0441",
            validity_score: 0.91 }
        ],
        is_unsupported: false },
      { text: "Testing is expected within 18 months...",
        confidence: "qualified",
        supporting_evidence: [],
        is_unsupported: true }
    ],
    unsupported_claims: [
      { text: "Testing is expected within 18 months...", claim_type: "analytical" }
    ],
    alternatives_considered: [
      "The program may be in an earlier stage with deceptive indicators of advancement",
      "The collection may reflect deliberate display rather than operational capability"
    ],
    confidence_summary: { high: 0, medium: 2, low: 1, uncertain: 1 }
  }

# Supervisor: unsupported claim rates for this analyst this quarter
GET /audit/session-456/claim-quality?class=capability_assessment&domain=technical_programs
→ {
    unsupported_claim_rate: 0.23,
    turn_count: 47,
    completeness_mean: 0.71,
    by_analyst: [
      { analyst_id: "A-001", unsupported_rate: 0.19, turn_count: 31 },
      { analyst_id: "A-002", unsupported_rate: 0.31, turn_count: 16 }
    ]
  }
```

---

## Appendix C: Calibration and Outcome Data

### What ERAS contributes to calibration

ERAS provides TCS/MIMIR with per-analyst, per-domain unsupported claim rates as an additional calibration signal. Sessions where the analyst accepted claims with high unsupported rates without independent verification are flagged as lower-quality verification episodes, reducing their FGTS calibration weight. The signal weight in TCS is configurable; initial deployment uses a 15% weight relative to explicit analyst verification corrections.

### How outcome data from OFS/NEMESIS affects ERAS

When OFS/NEMESIS closes a requirement with a CONFIRMED or DISCONFIRMED outcome, ERAS correlates the outcome against the completeness scores and unsupported claim rates of the contributing sessions. This feeds two improvements: completeness scoring weight calibration (which components of the composite score predicted analytical accuracy) and claim type accuracy rates by domain (which claim types are systematically unsupported and later confirmed versus unsupported and later disconfirmed). Year 2 data floor applies — sufficient outcome data required before weight calibration is meaningful.

---

## Appendix D: Red Team Findings

*Pending red team evaluation — scheduled for Phase 7 gate review.*

*When complete, this section will contain adversarial CoT manipulation scenarios, unsupported claim detection bypass attempts, classification boundary violation attempts via audit queries, and the Research and Red Team's P0/P1/P2/P3 classification of findings.*

---

*THEMIS Platform · ERAS/LOGOS Service PRD v1.0*  
*Companion documents: THEMIS Platform Design v1.0 · Addenda A–F · THEMIS-Service-PRD-Template.md*
