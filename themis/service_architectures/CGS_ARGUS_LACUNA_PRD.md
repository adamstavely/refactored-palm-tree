# CGS — Collection Gap Service
### ARGUS-LACUNA · *"Argus (the all-seeing) meeting Lacuna (Latin for gap, void, or missing piece) — the watchkeeper who maps not what is seen but the precise shape of what is not"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `CGS` |
| **Epithet** | `ARGUS-LACUNA` |
| **Full name** | Collection Gap Service |
| **Namespace** | `themis-warning` |
| **Layer** | Intelligence Layer — Warning |
| **Build phase** | Year 2 · Q4 |
| **Build priority** | 9 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — maps the precise shape of what we do not know |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**CGS/ARGUS-LACUNA answers: For this analytical requirement, domain, or entity — what collection coverage do we lack, how significant is that gap, and can we characterise it precisely enough to task collection against it?**

### 1.2 Why This Service Exists

An analyst who does not know what they do not know cannot caveat their assessment appropriately. The standard analytical confidence signal tells the analyst how reliable prior assessments of this type have been. It does not tell them whether the current assessment is missing a category of evidence that would change the conclusion entirely.

ARGUS-LACUNA maps the gap. Not vaguely ("we have limited collection on this programme") but precisely ("we lack HUMINT access to the facility associated with this entity, and our SIGINT coverage of technical communications for this programme has been absent since this date"). A precisely characterised gap can be submitted as a collection requirement. A vaguely characterised gap cannot.

The relationship to KCS/ARGUS is the positive-to-negative inversion: KCS/ARGUS maintains the coverage map (what we have). ARGUS-LACUNA works from the inverse — given what we need for this analytical question and what KCS/ARGUS shows we have, what is the gap? The two services together form the complete coverage picture.

### 1.3 Design Principles

- **Gap characterisation is what distinguishes ARGUS-LACUNA from other gap detectors.** RQS/HERMES detects retrieval gaps (what was not found in this session). TVS/KAIROS detects currency gaps (sources that are stale). ARGUS-LACUNA characterises collection gaps (what collection has never been produced or is structurally absent). These are different problems requiring different solutions.
- **GC-2 governs disclosure.** What analysts are told about collection gaps — and whether that disclosure reveals collection methods or capabilities — is a policy decision, not a technical one. ARGUS-LACUNA generates the gap assessment; GC-2 governs what is surfaced to analysts.
- **The gap indicator feeds UCS/TYCHE.** The most direct analytical use of ARGUS-LACUNA output is as the epistemic_signal in UCS/TYCHE: low collection coverage in this domain elevates epistemic_dominance in the uncertainty profile, correctly characterising why confidence is limited.
- **Gaps are graded, not binary.** A domain with no collection is different from one with outdated collection, which is different from one with partial collection that covers some entities but not others. ARGUS-LACUNA grades coverage on a continuous scale, not a binary present/absent flag.

### 1.4 Explicit Out of Scope

- **Tasking collection.** ARGUS-LACUNA characterises gaps; TIS/DIKE routes them as collection requirements. ARGUS-LACUNA does not have collection authority.
- **Assessing classification constraints on collection.** Whether collection is absent because it is prohibited, technically infeasible, or simply not tasked is an operational intelligence question, not an ARGUS-LACUNA determination.

---

## 2. Core Responsibilities

### 2.1 Primary Function

CGS/ARGUS-LACUNA assesses collection coverage for analytical requirements, domains, and specific entities — drawing on KCS/ARGUS coverage maps and corpus source density analysis — to produce a structured gap assessment that characterises coverage deficiencies by collection method, entity, and temporal scope. It feeds the epistemic dominance signal to UCS/TYCHE and gap signals to TIS/DIKE and PYTHIA.

### 2.2 Secondary Functions

- Entity-specific gap mapping: for each OGS entity in the analytical scope, characterising which collection methods have coverage and which do not
- Temporal gap identification: periods during which collection coverage was absent or degraded
- Gap priority scoring: ranking gaps by analytical significance (how much would filling this gap change the assessment?)
- Watchlist coverage check: comparing identified gaps against active KCS/ARGUS watchlist entries to identify whether the gap is already being tracked
- GC-2 compliant disclosure: surfacing gap information to analysts in formats consistent with the GC-2 disclosure policy

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
CollectionGap:
  gap_id:                  uuid
  domain:                  str
  entity_id:               uuid | null      # FK → OGS; null for domain-level gaps
  collection_method:       str
  gap_type:                COMPLETE_ABSENCE | PARTIAL_COVERAGE | STALE_COVERAGE |
                           TEMPORAL_GAP | GEOGRAPHIC_GAP | ENTITY_SPECIFIC
  coverage_score:          float            # 0.0 = no coverage, 1.0 = complete coverage
  gap_description:         str              # plain language characterisation
  temporal_scope:          { start: datetime | null, end: datetime | null }
  analytical_significance: HIGH | MEDIUM | LOW
  watchlist_covered:       bool             # is this gap already on a watchlist?
  detected_at:             datetime
  cgr_id:                  uuid | null      # FK → TIS/DIKE CGR if submitted

GapAssessment:
  assessment_id:           uuid
  session_id:              uuid
  domain:                  str
  entity_ids:              [uuid]
  gaps:                    [CollectionGap]
  overall_coverage_score:  float
  epistemic_signal:        float            # 0.0–1.0; used by UCS/TYCHE
  assessment_basis:        str              # what corpus and coverage data was used
  assessed_at:             datetime

CoverageGrade:
  domain:                  str
  collection_method:       str
  current_score:           float
  trend:                   IMPROVING | STABLE | DEGRADING
  last_ingestion:          datetime | null
  source_count:            int
  validity_weighted_count: float            # TVS-weighted
  graded_at:               datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | CollectionGap, GapAssessment, CoverageGrade | Session + 7 years |
| Coverage grade cache | Redis | Current CoverageGrade per domain/method (hot path) | 1h TTL |
| Event store | MOIRAI | Signed gap assessment events | Indefinite |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /assess/session
  Auth:     ATHENA service account | UCS service account
  Request:  {
    session_id:            uuid,
    domain:                str,
    entity_ids:            [uuid],
    collection_methods:    [str] | null    # null = all methods
  }
  Response: {
    assessment_id:         uuid,
    overall_coverage:      float,
    epistemic_signal:      float,          # for UCS/TYCHE
    gaps:                  [{ gap_id, method, coverage_score, significance }],
    high_priority_gaps:    int
  }
  SLA: p99 < 1000ms

GET /gaps/{gap_id}
  Auth:     analyst session token | TIS/DIKE service account
  Response: CollectionGap (GC-2 compliant; collection method details may be redacted)

GET /coverage/{domain}
  Auth:     analyst session token | KCS service account
  Response: [CoverageGrade]               # all methods for this domain

POST /gaps/{gap_id}/submit-cgr
  Auth:     supervisor token | analyst session token
  Response: { cgr_id: uuid }              # triggers TIS/DIKE CGR creation

GET /health
  Response: {
    status, dependencies: { moirai, kcs_argus, tvs, ogs, redis },
    gaps_identified_24h:   int,
    high_priority_gaps:    int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          CGS_GAP_IDENTIFIED
service_id:         "CGS"
session_id:         uuid
classification:     str
event_payload:
  assessment_id:          uuid
  domain:                 str
  gap_count:              int
  high_priority_count:    int
  epistemic_signal:       float
prev_event_hash:    str
signature:          str
timestamp:          datetime
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `CGS_GAP_IDENTIFIED` | Gap assessment completed | MOIRAI, TIS/DIKE (CGR pipeline), PYTHIA (gap signals), UCS/TYCHE |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| KCS/ARGUS | Knowledge Currency | Positive coverage map as baseline | Sync query | Assessment uses corpus density only; less precise |
| TVS/KAIROS | Temporal Validity | Validity-weighted source counts | Sync query | Unweighted source count used |
| OGS/YGGDRASIL | Ontology | Entity-specific gap assessment | Sync query | Domain-level gaps only |

### 5.2 Feeds Into

| Service | Epithet | What CGS provides | How |
|---|---|---|---|
| UCS/TYCHE | Uncertainty Characterization | epistemic_signal for uncertainty profiles | API |
| TIS/DIKE | Tasking Integration | Gap signals for CGR pipeline | `CGS_GAP_IDENTIFIED` event |
| PYTHIA | Anticipatory Surfacing | Collection gap signals for active sessions | `CGS_GAP_IDENTIFIED` event |
| LACHESIS | Weak Signal Fusion | Coverage gaps as context for signal evaluation | API |

---

## 6. Non-Functional Requirements

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Session gap assessment | 200ms | 600ms | 1000ms |
| Coverage grade lookup (cached) | 5ms | 20ms | 100ms |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| ATHENA / UCS | Session gap assessment | Service account |
| Analyst session | Gap detail (GC-2 compliant view) | Session token |
| TIS/DIKE | Gap signals; CGR submission | Service account |
| Supervisor | Full gap assessment | Supervisor token |

---

## 8. Known Design Risks

- **GC-2 governs what analysts see, creating a disclosure design problem.** The gap characterisation contains information about what collection methods are or are not producing coverage — which is itself sensitive. The GC-2 policy must specify the exact format and content of analyst-facing gap disclosure before Phase 1. Without GC-2, the analyst surface cannot be built.

---

## 9. Implementation Roadmap

### Phase 1 — Gap Assessment and Epistemic Signal (Year 2, Weeks 33–40)

- CollectionGap, GapAssessment schemas
- Coverage assessment from KCS/ARGUS + TVS validity weighting
- Epistemic signal computation for UCS/TYCHE
- Gap assessment endpoint
- TIS/DIKE gap signal integration

**Phase gate criterion:** Epistemic signal fed to UCS/TYCHE correctly. Gap assessment completes within 1 second. GC-2 policy reviewed before analyst surface built.

### Phase 2 — Entity-Specific Gaps, Temporal Analysis, and PYTHIA (Year 2, Weeks 41–48)

- Entity-specific gap mapping via OGS
- Temporal gap identification
- PYTHIA gap signal integration
- CGR submission from gap assessment
- GC-2 compliant analyst disclosure surface

**Phase gate criterion:** Entity-specific gaps correctly computed for test entities. PYTHIA receives gap signals. GC-2 compliant disclosure format implemented. ARB and Cell Lead sign-off.

---

## 10. Policy Dependencies

| Ref | Decision required | Gates |
|---|---|---|
| GC-2 | Retrieval gap indicator disclosure policy — what analysts are told about collection gaps, in what format, with what collection method detail visible | Phase 1 analyst surface — cannot build gap disclosure without this |

---

## 11. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q4 gate review.*
