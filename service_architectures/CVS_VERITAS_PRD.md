# CVS — Corroboration & Verification Service
### VERITAS · *"Latin for 'truth' — the Roman goddess of truth, daughter of Saturn"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `CVS` |
| **Epithet** | `VERITAS` |
| **Full name** | Corroboration & Verification Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 3–4 (Weeks 9–28) |
| **Build priority** | 6 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — verifies that cited sources exist and say what the AI claims |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**CVS/VERITAS answers: Does this source exist, and does it actually say what the AI claims it says?**

### 1.2 Why This Service Exists

LLMs generate plausible-sounding source citations that do not exist. This is not a model quality deficiency — it is a structural property of how language models generate text. The generation process produces the source citation that would most plausibly follow the claim in the training distribution, not the source that the claim actually came from. The citation looks correct; it is fabricated.

Without CVS, the analyst has no systematic mechanism to catch fabricated sources before acting on them. A provenance badge that says GRND is meaningful only if the cited source has been verified to exist and to support the cited claim. Without verification at generation time, a GRND badge is a statement about what the model retrieved, not about whether the retrieval accurately reflects an accessible source.

CVS is the service that makes the GRND/PARAM/SYNTH source type distinction analytically meaningful rather than cosmetic.

### 1.3 Why This Service Is Sixth

CVS requires MOIRAI to be operational (to record verification events in the provenance chain) and PCES to be operational (for session-scoped access to the intelligence corpus). It is Phase 3-4 because it needs the corpus indexing and retrieval infrastructure to have been established — CVS verifies citations against the accessible corpus, which requires the corpus to exist and be queryable.

### 1.4 Design Principles

- **Verification is against accessible sources, not truth.** CVS checks whether a source is in the accessible intelligence corpus and whether the AI's characterisation is accurate relative to that source. It does not determine whether the source's underlying claim is true.
- **Citation absence is reported, not assumed.** A source that CVS cannot find in the accessible corpus is flagged as `SOURCE_NOT_FOUND`, not automatically classified as fabricated. The source may exist at a higher classification level than the session's clearance. This distinction matters for analyst action.
- **Corroboration scoring is advisory, not gating.** A low corroboration score means few accessible sources agree with the claim — it does not block the session. The analyst must act on the signal.
- **Verification does not replace analyst judgment.** A source that exists and accurately supports the AI's characterisation may still reflect stale intelligence (TVS/KAIROS handles this), an adversarially placed source (IAS/SCUDO handles this), or a source the analyst should weigh differently for domain reasons. CVS verifies existence and accuracy; it does not evaluate significance.

### 1.5 Explicit Out of Scope

- **Verifying PARAM claims against external sources.** PARAM claims have no specific retrieved source. Verification of PARAM claims requires independent collection — CVS provides the `COULD_NOT_VERIFY_PARAM` flag to surface this, but the verification path itself is the analyst's responsibility.
- **Assessing source quality or reliability.** CVS verifies that a source exists and says what the AI claims. Whether that source is a high-quality, reliable collector is a separate analytical judgment.
- **Real-time collection queries.** CVS queries the accessible corpus; it does not task live collection.

---

## 2. Core Responsibilities

### 2.1 Primary Function

CVS/VERITAS checks every AI-cited source reference in an ATHENA session against the accessible intelligence corpus — confirming the source exists, confirming the AI's characterisation of the source is accurate, and computing a corroboration score reflecting how many independent sources in the accessible corpus support the claimed content. It surfaces these results as verification badges in ATHENA and publishes verification events to MOIRAI for the provenance record.

### 2.2 Secondary Functions

- Corroboration scoring: counting independent sources that support a claim, weighted by source type independence
- Cross-reference detection: identifying when multiple AI citations trace back to the same underlying source (reducing apparent corroboration)
- Fabrication pattern monitoring: tracking fabrication rates by model version and interaction class for Research & Red Team analysis
- FGTS integration: publishing high-confidence SOURCE_NOT_FOUND findings as weighted corrections to the calibration corpus

### 2.3 What This Service Does Not Decide

CVS verifies source existence and citation accuracy. Whether a verified source should be weighted differently given its collection provenance, recency, or reliability profile is an analytical judgment belonging to the analyst. Whether a SOURCE_NOT_FOUND result warrants blocking the session is a policy decision — CVS flags; it does not block.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
VerificationResult:
  result_id:             uuid
  session_id:            uuid
  turn_id:               uuid
  claim_id:              uuid
  source_reference:      str              # the citation text the AI produced
  source_type:           GRND | PARAM | SYNTH | TRANSCRIPT | IMAGE | VIDEO | AUDIO | OCR | MEMORY
  verdict:               VerificationVerdict
  source_found:          bool
  source_id:             uuid | null      # FK → corpus source, if found
  citation_accurate:     bool | null      # null if source not found
  accuracy_confidence:   float | null     # 0.0–1.0
  corroboration_score:   float            # 0.0–1.0; independent sources supporting claim
  corroborating_sources: [uuid]          # corpus source IDs that corroborate
  cross_references:      [uuid]          # sources that appear independent but share lineage
  timestamp:             datetime

VerificationVerdict:
  VERIFIED_ACCURATE         # source found; citation is accurate
  VERIFIED_INACCURATE       # source found; AI characterised it inaccurately
  SOURCE_NOT_FOUND          # source reference not in accessible corpus
  SOURCE_NOT_FOUND_CLEARANCE # source reference may exist above session clearance
  COULD_NOT_VERIFY_PARAM    # PARAM claim; no specific source to verify
  PARTIAL_ACCURACY          # source found; some aspects accurate, some not
  VERIFICATION_FAILED       # technical failure; verification not completed

CorroborationRecord:
  record_id:             uuid
  claim_id:              uuid
  session_id:            uuid
  independent_source_count:  int
  weighted_corroboration:    float        # weighted by source type independence
  single_origin_risk:        bool         # true if multiple sources share a lineage
  corroboration_level:   HIGH | MEDIUM | LOW | UNCORROBORATED
  assessed_at:           datetime

FabricationFlag:
  flag_id:               uuid
  session_id:            uuid
  turn_id:               uuid
  claim_id:              uuid
  model_version:         str
  interaction_class:     str
  reference_text:        str              # the fabricated citation text
  detected_at:           datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | VerificationResult, CorroborationRecord, FabricationFlag | Session + 7 years |
| Corpus query layer | Elasticsearch | Queried for source existence and citation accuracy checking | Mirrors intelligence corpus |
| Event store | MOIRAI | Signed verification events | Indefinite |
| Fabrication pattern cache | Redis | Recent fabrication patterns for anomaly detection | 30 days |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| VerificationResult | Inherits session classification | Citation content is session-compartmented |
| CorroborationRecord | Inherits session classification | Session-compartmented |
| FabricationFlag | Controlled Unclassified | Fabrication patterns accessible to Research & Red Team |

### 3.4 Retention and Purge Policy

VerificationResult and CorroborationRecord retained for session lifetime plus seven years. FabricationFlag records retained indefinitely for model quality monitoring. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /verify/claim
  Auth:     inference gateway service account | ATHENA service account
  Request:  {
    session_id:        uuid,
    turn_id:           uuid,
    claim_id:          uuid,
    source_references: [str],     # all citations the AI produced for this claim
    claim_text:        str,
    source_type:       str
  }
  Response: {
    results:           [VerificationResult],
    corroboration:     CorroborationRecord,
    overall_verdict:   str,
    fabrication_detected: bool
  }
  SLA: p99 < 1000ms

POST /verify/source
  Auth:     inference gateway service account
  Request:  {
    source_reference:  str,
    claim_text:        str,
    session_id:        uuid
  }
  Response: VerificationResult
  SLA: p99 < 500ms

GET /verification/{claim_id}
  Auth:     session token (PCES-validated)
  Response: { result: VerificationResult, corroboration: CorroborationRecord }

GET /session/{session_id}/fabrication-report
  Auth:     supervisor token | IOB token
  Response: {
    session_id:          uuid,
    fabrication_count:   int,
    total_citations:     int,
    fabrication_rate:    float,
    flags:               [FabricationFlag]
  }

GET /audit/fabrication-rates?model_version={version}&from={date}&to={date}
  Auth:     Research & Red Team token | IOB token
  Response: {
    model_version:       str,
    period:              { from, to },
    total_citations:     int,
    fabrication_count:   int,
    fabrication_rate:    float,
    by_interaction_class:[{ class, rate }],
    by_source_type:      [{ type, rate }]
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces, elasticsearch_corpus },
    corpus_index_lag_ms:  int,
    last_event_hash:      str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          CVS_VERIFICATION_COMPLETE
service_id:         "CVS"
session_id:         uuid
turn_id:            uuid
classification:     str
event_payload:
  claim_id:               uuid
  verdict:                str
  source_found:           bool
  citation_accurate:      bool | null
  corroboration_level:    str
  fabrication_detected:   bool
  source_type:            str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          CVS_FABRICATION_DETECTED
event_payload:
  claim_id:               uuid
  model_version:          str
  interaction_class:      str
  reference_text:         str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `CVS_VERIFICATION_COMPLETE` | Every claim verification | MOIRAI, ATHENA (badge update), FGTS (if SOURCE_NOT_FOUND) |
| `CVS_FABRICATION_DETECTED` | SOURCE_NOT_FOUND with high fabrication confidence | MOIRAI, Research & Red Team alert, IAS/SCUDO (fabrication pattern feed) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| TVS/KAIROS | `TVS_SOURCE_INVALIDATED` | Marks previously verified sources as stale in VerificationResult; re-evaluation queued |
| IAS/SCUDO | `IAS_ADVERSARIAL_SOURCE_DETECTED` | Downgrades VerificationResult accuracy_confidence for that source |
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | Resets fabrication rate baseline for new model version |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed event emission | Async event | Events buffered; verification still proceeds |
| PCES/AEGIS | Classification Enforcement | Session token validation; corpus access scope | Sync | Verification proceeds for already-scoped session |
| Corpus index | N/A (Elasticsearch) | Source existence and citation accuracy lookup | Sync | Verification returns `VERIFICATION_FAILED`; session proceeds with no CVS badge |

### 5.2 Feeds Into

| Service | Epithet | What CVS provides | How |
|---|---|---|---|
| ATHENA | Interface | Source verification badges; corroboration indicators | API |
| FGTS/ALETHEIA | Ground Truth | SOURCE_NOT_FOUND results as weighted corrections | `CVS_FABRICATION_DETECTED` event |
| IAS/SCUDO | Adversarial Screening | Fabrication pattern data for adversarial source injection detection | Fabrication flag feed |
| IOB Reporting | Oversight | Fabrication rates by model version; per-session verification reports | Audit endpoints |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Corpus queries are scoped to the session's granted compartments. A source at a higher classification than the session's clearance returns `SOURCE_NOT_FOUND_CLEARANCE` — not a fabrication flag, but a signal that verification cannot be completed at this clearance level.
- **Compartment inheritance:** VerificationResult inherits session classification. Corroboration sources returned in the result are within the session's compartment scope.
- **Failure behavior:** PCES unavailable → corpus queries proceed with the session scope established at initialization; endpoint validation falls back to cached session context.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Single source verification | 100ms | 300ms | 500ms |
| Claim verification (multiple sources) | 200ms | 600ms | 1000ms |
| Corroboration scoring | 300ms | 700ms | 1200ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Verification requests/second | 100 |
| Corpus lookups/second | 500 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — CVS unavailability degrades GRND badge reliability |
| MOIRAI event durability | 99.999% |
| RTO | 10 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| Elasticsearch corpus | Verification returns `VERIFICATION_FAILED` on all checks; alert fires | Source badges show `UNVERIFIED` in ATHENA; analyst must verify manually |
| MOIRAI | Events buffered; verification still proceeds | No analyst-facing impact; provenance gap logged |
| IAS/SCUDO adversarial feed | Fabrication confidence scores computed without adversarial pattern data | Fabrication detection less sensitive; alert fires |

---

## 7. Security Model

### 7.1 Authentication

Inference gateway and ATHENA submit verification requests via service accounts. Session token required for analyst-facing read endpoints. Supervisor and IOB tokens for audit endpoints.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Inference gateway | `POST /verify/*` | Service account |
| Analyst (own session) | `GET /verification/{claim_id}` for own sessions | Session token |
| Supervisor | Fabrication reports for their team | Supervisor token |
| Research & Red Team | Fabrication rate audit | Research team token |
| IOB | All endpoints | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/cvs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/cvs/db-credentials` | 30 days |
| Elasticsearch credentials | `themis/cvs/es-credentials` | 30 days |

### 7.4 Adversarial Threat Surface

**Adversarial source injection**: an attacker who can write to the intelligence corpus can create sources that appear legitimate and verify correctly against CVS's checks. IAS/SCUDO's corpus poisoning detection is the upstream defense; CVS is downstream and will verify a successfully injected source as legitimate. This is a known limitation — CVS verifies accuracy against accessible sources, not the truthfulness of those sources.

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| False VERIFIED_ACCURATE (inaccurate characterisation not caught) | Medium | P1 — analyst relies on incorrectly verified citation | Research & Red Team accuracy benchmarking | Accuracy scoring threshold conservative; analyst training on what VERIFIED_ACCURATE means |
| SOURCE_NOT_FOUND_CLEARANCE misclassified as fabrication | Low | P2 — analyst misled about source nature | Clearance-level signal in corpus query | Explicit CLEARANCE variant in verdict; analyst training |
| Corpus index staleness (source invalidated but index not updated) | Low | P2 — invalidated source shows as VERIFIED | TVS/KAIROS invalidation event consumption | Alert on TVS event processing lag |
| Corroboration inflation (single-origin sources counted as independent) | Medium | P2 — misleading corroboration score | Cross-reference detection | Cross-reference detection mandatory in corroboration scoring |

### 8.1 Known Design Risks

- **PARAM claim verification path is structurally unavailable.** CVS can flag a PARAM claim with `COULD_NOT_VERIFY_PARAM` but cannot verify it — there is no specific source to check against. The only verification path is analyst-performed independent collection or expert consultation. This is a design constraint, not a fixable engineering problem. Training must explicitly address the PARAM verification path.
- **Citation accuracy scoring is an NLP problem without guaranteed accuracy.** Determining whether the AI's characterisation of a source accurately reflects the source requires semantic comparison of the AI's claim against the source content. Current NLP approaches have meaningful false positive and false negative rates on this task. The accuracy_confidence field expresses this uncertainty — a low confidence score should route to the human review queue, not produce a confident VERIFIED_ACCURATE verdict.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `cvs.verification.latency_p99` | Histogram | `> 1000ms for 5m` | P1 |
| `cvs.fabrication_rate` | Gauge | `> 5% over 1h` | P1 |
| `cvs.corpus.unavailable_rate` | Counter | `> 1% over 5m` | P0 |
| `cvs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `cvs.corpus_index.lag_ms` | Gauge | `> 60000` (1 minute) | P2 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces, elasticsearch_corpus },
  corpus_index_lag_ms:   int,
  fabrication_rate_24h:  float,
  last_event_hash:       str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":         "ISO-8601",
  "service":           "CVS/VERITAS",
  "event":             "VERIFICATION_COMPLETE | FABRICATION_DETECTED | CORPUS_UNAVAILABLE",
  "claim_id":          "uuid",
  "session_id":        "uuid",
  "verdict":           "string",
  "source_found":      true,
  "fabrication":       false,
  "duration_ms":       0
}
```

---

## 10. Cryptographic Attestation

### 10.1 Event Signing

- **Vault key path:** `themis/cvs/signing-key`
- **Algorithm:** HMAC-SHA256
- **Chain participation:** Yes — full participant

### 10.2 What This Service Attests

The MOIRAI record for CVS proves that at a specific time, a specific AI citation was checked against the accessible corpus, with a specific verdict, and the verification decision has not been altered since recording. Fabrication detection events are permanently recorded, creating an auditable record of model citation quality over time.

### 10.3 What This Service Cannot Prove

CVS cannot prove that a VERIFIED_ACCURATE citation accurately reflects the truth. It proves the source exists and the AI's characterisation was accurate relative to the source content. A source that is itself inaccurate produces a VERIFIED_ACCURATE result for a false claim.

---

## 11. Implementation Roadmap

### Phase 1 — Source Existence Verification (Weeks 9–16)

- VerificationResult schema and `POST /verify/source` endpoint
- Source existence lookup against corpus Elasticsearch index
- `SOURCE_NOT_FOUND` and `SOURCE_NOT_FOUND_CLEARANCE` verdict logic
- MOIRAI event emission: `CVS_VERIFICATION_COMPLETE`
- Basic ATHENA badge integration (exists / not found)

**Phase gate criterion:** Every GRND source citation produces a VerificationResult with SOURCE_NOT_FOUND or VERIFIED (partial — citation accuracy not yet checked). MOIRAI events chained correctly.

### Phase 2 — Citation Accuracy and Corroboration (Weeks 17–28)

- Semantic citation accuracy scoring (AI characterisation vs. source content)
- CorroborationRecord computation with cross-reference detection
- `CVS_FABRICATION_DETECTED` event emission
- FGTS integration for fabrication corrections
- Fabrication rate monitoring and audit endpoints
- VERIFIED_ACCURATE / VERIFIED_INACCURATE / PARTIAL_ACCURACY verdict differentiation

**Phase gate criterion:** Citation accuracy scoring produces meaningful differentiation on test set (accuracy ≥ 70% on labelled verification cases). Corroboration scoring demonstrates cross-reference detection. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate CVS deployment.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

Each AI citation in ATHENA shows a CVS badge: green checkmark (VERIFIED_ACCURATE), amber warning (PARTIAL_ACCURACY or COULD_NOT_VERIFY_PARAM), red X (VERIFIED_INACCURATE or SOURCE_NOT_FOUND), grey dash (VERIFICATION_FAILED). Expanding the badge shows the verdict details and the corroboration level.

### 13.2 What the Analyst Should Do

VERIFIED_ACCURATE: the citation is in the corpus and the AI's characterisation is accurate. Still check TVS validity — a verified citation may reference stale intelligence. VERIFIED_INACCURATE: do not use this citation. The AI mischaracterised the source. Examine the source directly to understand what it actually says. SOURCE_NOT_FOUND: do not rely on this citation until independently verified. The source may exist at a higher classification level — check with your security officer if this is operationally important. COULD_NOT_VERIFY_PARAM: this is the expected result for PARAM claims. The verification path is independent collection.

### 13.3 What the Signal Does Not Mean

VERIFIED_ACCURATE does not mean the underlying intelligence claim is true — it means the source exists and the AI's characterisation of it is accurate. A corroboration score of HIGH does not mean the claim is well-supported — check whether corroborating sources share a lineage (single-origin risk indicator).

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Citation accuracy scoring model.** The accuracy scoring model — comparing AI characterisations to source content — will have meaningful error rates that vary by claim type. A benchmark is required before Phase 2 deployment. Resolution path: Research & Red Team benchmarking on 200 labelled citation-accuracy pairs before Phase 2 gate.

### 14.2 Operational Assumptions

- **Assumption 1: The intelligence corpus is queryable at the Elasticsearch layer.** CVS depends on a full-text searchable corpus index. If corpus ingestion is delayed or the index is incomplete, CVS will produce high SOURCE_NOT_FOUND rates that do not reflect fabrication but reflect index completeness.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 3 gate review.*
