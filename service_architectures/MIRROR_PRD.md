# MIRROR — Requirement Similarity Service
### MIRROR · *"That which shows you what has been done before — not the thing itself, but the reflection that makes the pattern visible"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MIRROR` |
| **Epithet** | `MIRROR` |
| **Full name** | Requirement Similarity Service |
| **Namespace** | `themis-research` |
| **Layer** | Intelligence Layer — Research |
| **Build phase** | Year 2 · Q2 |
| **Build priority** | 6 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust — surfaces what the analytical community already knows from prior similar work |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MIRROR answers: Has the THEMIS platform worked on requirements substantially similar to this one before — and if so, what analytical approaches were used, what did those assessments look like, and what were their outcomes?**

### 1.2 Why This Service Exists

The analytical community does not work in isolation. Requirements recur — not identically, but with recognisable structural similarities. An assessment of adversary programme maturity has structural similarities to prior assessments of the same adversary's other programmes, or to similar assessments of other adversaries' programmes in the same technology domain. The specific intelligence is different; the analytical problem structure is not.

Without MIRROR, ATHENA approaches every new requirement as if no prior work is relevant. With MIRROR, it identifies the closest prior requirements and surfaces: what analytical approaches were used (methodology inheritance), what sources were most informative (retrieval priming), what the outcomes were (accuracy calibration), and what errors were commonly made (correction pattern inheritance from MNEMOSYNE).

The data floor is real: fifty requirements with structured profiles is the minimum for meaningful similarity matching. MIRROR's value compounds as the requirement portfolio grows.

### 1.3 Design Principles

- **Similarity is multi-dimensional.** Domain similarity alone is insufficient. A requirement about Adversary A's missile programme in a geopolitical domain may be more similar to a requirement about Adversary B's missile programme than to a requirement about Adversary A's cyber programme in a different domain. MIRROR computes similarity across domain, entity type, claim type, and collection method dimensions.
- **Outcome data is the gold standard.** A prior requirement with an OFS/NEMESIS CONFIRMED outcome is a richer signal than one without outcome data. MIRROR weights outcome-confirmed requirements more heavily in similarity results.
- **The data floor is explicit, not hidden.** When MIRROR cannot find similar requirements (data floor not yet reached), it says so explicitly rather than surfacing low-confidence matches as meaningful. "No similar requirements found with sufficient confidence" is a useful and honest output.
- **Similarity results are advisory, not determinative.** The analyst decides whether the suggested prior requirements are genuinely analogous to their current work. MIRROR surfaces; the analyst judges relevance.

### 1.4 Explicit Out of Scope

- **Cross-compartment similarity matching.** MIRROR matches within the accessible compartment scope of the requesting session. Cross-compartment similarity — which may reveal the most valuable analogies — requires IOB authority and is not available by default.
- **Predicting outcomes.** ORACLE makes predictions. MIRROR surfaces historical analogies. The distinction is intentional: MIRROR shows what happened; ORACLE predicts what will happen.
- **Recommending collection.** MIRROR shows what collection was useful on prior similar requirements; TIS/DIKE manages collection requirements.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MIRROR maintains structured profiles for each completed analytical requirement that passed through ATHENA, indexes these profiles for multi-dimensional similarity matching, and returns ranked similar requirements with their analytical approaches, collection effectiveness signals, and outcome classifications when queried with a new requirement context.

### 2.2 Secondary Functions

- Requirement profiling: building and updating structured RequirementProfile records at session completion
- Collection effectiveness scoring: computing which collection methods produced the most analytically useful intelligence for each requirement type
- Analytical approach inheritance: identifying which SKS skills were most effective on similar prior requirements
- Correction pattern linkage: linking similar requirement clusters to MNEMOSYNE knowledge nodes about common errors in this requirement type
- Outcome calibration: when OFS/NEMESIS classifies an outcome, updating the RequirementProfile with the outcome classification
- Data floor monitoring: tracking whether the requirement portfolio has reached the threshold for meaningful similarity matching by domain

### 2.3 What This Service Does Not Decide

MIRROR surfaces similar requirements. Whether those requirements are genuinely analogous to the current analytical problem, how much weight to give their outcomes, and whether their analytical approaches should be inherited are analyst decisions. MIRROR provides structured historical context; the analyst judges its applicability.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
RequirementProfile:
  profile_id:              uuid
  requirement_id:          str              # external requirement identifier
  domain:                  str
  sub_domain:              str | null
  claim_types:             [str]
  entity_types:            [str]            # OGS entity types involved
  entity_ids:              [uuid]           # specific OGS entities (compartment-appropriate)
  collection_methods_used: [str]
  collection_effectiveness:[CollectionEffectivenessRecord]
  skills_used:             [uuid]           # SKS skill IDs
  session_count:           int
  analyst_pool_size:       int              # how many distinct analysts worked this requirement
  outcome_classification:  str | null       # from OFS/NEMESIS
  outcome_confidence:      str | null
  profile_completeness:    float            # 0.0–1.0; how complete this profile is
  created_at:              datetime
  last_updated:            datetime

CollectionEffectivenessRecord:
  record_id:               uuid
  profile_id:              uuid
  collection_method:       str
  retrieval_rate:          float           # how often sources from this method were retrieved
  reliance_rate:           float           # how often retrieved sources were relied upon in assessments
  outcome_correlation:     float | null    # correlation with confirmed outcome if available

SimilarityResult:
  result_id:               uuid
  query_context_hash:      str             # hash of the query context (not stored plain)
  session_id:              uuid
  similar_profiles:        [SimilarProfile]
  data_floor_met:          bool
  query_domain:            str
  queried_at:              datetime

SimilarProfile:
  profile_id:              uuid
  similarity_score:        float           # overall 0.0–1.0
  dimension_scores:        {
    domain:                float,
    entity_type:           float,
    claim_type:            float,
    collection_method:     float
  }
  outcome_available:       bool
  outcome_classification:  str | null
  collection_suggestions:  [str]          # collection methods that were effective
  skill_suggestions:       [uuid]         # skills that were used
  confidence:              HIGH | MEDIUM | LOW
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Profile store | PostgreSQL | RequirementProfile, CollectionEffectivenessRecord | Indefinite |
| Similarity index | Elasticsearch (vector index) | Multi-dimensional profile embeddings for similarity search | Mirrors profile store |
| Event store | MOIRAI | Signed profile and similarity events | Indefinite |
| Similarity cache | Redis | Recent similarity results for active STOA sessions | 4h TTL |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| RequirementProfile | Inherits requirement classification | Compartment-gated; entity_ids restricted to session scope |
| SimilarityResult | Inherits session classification | Session-compartmented |
| CollectionEffectiveness | Controlled Unclassified (aggregate) | Platform-wide; de-identified |

---

## 4. API Contract

### 4.1 Endpoints

```
POST /similarity/query
  Auth:     STOA service account | PYTHIA service account | analyst session token
  Request:  {
    session_id:            uuid,
    requirement_context:   str,
    domain:                str,
    claim_types:           [str],
    entity_types:          [str],
    max_results:           int            # default 5
  }
  Response: {
    result_id:             uuid,
    data_floor_met:        bool,
    similar_profiles:      [SimilarProfile],
    domain_requirement_count:int,        # total requirements in this domain
    minimum_for_confidence:int           # data floor threshold
  }
  SLA: p99 < 1000ms

POST /profiles
  Auth:     STOA service account | ATHENA service account (session close)
  Request:  {
    requirement_id:        str,
    domain:                str,
    claim_types:           [str],
    entity_types:          [str],
    collection_methods:    [str],
    skills_used:           [uuid],
    session_count:         int
  }
  Response: { profile_id: uuid }

PATCH /profiles/{profile_id}/outcome
  Auth:     OFS/NEMESIS service account
  Request:  {
    outcome_classification:str,
    outcome_confidence:    str
  }
  Response: { profile_id: uuid, updated: bool }

GET /profiles/{profile_id}
  Auth:     analyst session token (compartment-scoped) | IOB token
  Response: RequirementProfile

GET /effectiveness/{domain}/{collection_method}
  Auth:     analyst session token | TIS/DIKE service account
  Response: {
    domain:                str,
    collection_method:     str,
    retrieval_rate_mean:   float,
    reliance_rate_mean:    float,
    outcome_correlation:   float | null,
    requirement_count:     int,
    data_floor_met:        bool
  }

GET /data-floor/status
  Auth:     analyst session token | platform operator
  Response: {
    by_domain:             [{ domain, requirement_count, threshold, met: bool }]
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces, elasticsearch, redis },
    total_profiles:        int,
    profiles_with_outcomes:int,
    domains_above_floor:   int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          MIRROR_PROFILE_CREATED
service_id:         "MIRROR"
session_id:         null
classification:     str
event_payload:
  profile_id:             uuid
  domain:                 str
  claim_types:            [str]
  session_count:          int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          MIRROR_OUTCOME_UPDATED
event_payload:
  profile_id:             uuid
  outcome_classification: str
  outcome_confidence:     str
```

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| STOA | `STOA_RESEARCH_COMPLETE` | Triggers RequirementProfile creation/update |
| OFS/NEMESIS | `OFS_OUTCOME_CLASSIFIED` | Updates RequirementProfile outcome classification |
| SKS/DAEDALUS | `SKS_SKILL_INVOKED` | Adds skill to RequirementProfile.skills_used |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Profile and outcome events | Async event | Events buffered; profiles still created |
| OGS/YGGDRASIL | Ontology | Entity type classification for profiles | Sync query | Entity type similarity dimension unavailable |

### 5.2 Feeds Into

| Service | Epithet | What MIRROR provides | How |
|---|---|---|---|
| STOA | Research Orchestration | Similar prior requirement context for decomposition | API |
| PYTHIA | Anticipatory Surfacing | Similarity-based anticipatory signals | API |
| ORACLE | Outcome Intelligence | Structured profiles for predictive model training | API |
| TIS/DIKE | Tasking Integration | Collection effectiveness by domain/method | API |
| IOB Reporting | Oversight | Data floor status; outcome-linked profile summary | API |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Similarity query | 100ms | 400ms | 1000ms |
| Profile creation | 200ms | 500ms | 1000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — MIRROR unavailability degrades STOA decomposition quality |
| RTO | 15 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| STOA / PYTHIA | Similarity query (compartment-scoped) | Service account |
| Analyst session | Profile view (compartment-scoped); data floor status | Session token |
| OFS/NEMESIS | Profile outcome update | Service account |
| IOB | Full access | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Data floor not reached (sparse results returned as meaningful) | High initially | P2 — analyst relies on low-confidence similarities | Data floor explicit in all responses | data_floor_met field returned on every query; analyst trained on interpretation |
| Similarity model returns structurally dissimilar requirements | Medium | P2 — poor analytical approach inheritance | Analyst dismissal rate monitoring | Analyst can rate similarity result relevance; feedback used to tune embedding model |

### 8.1 Known Design Risks

- **The similarity model requires intelligence-domain training.** Embedding-based similarity matching for intelligence requirements requires training data from the IC analytical domain. Generic semantic similarity models will not capture domain-specific analytical structure. Resolution path: Research & Red Team to develop IC-specific embedding fine-tuning before Phase 1, using a sample of analyst-labelled requirement pairs.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `mirror.query.latency_p99` | Histogram | `> 1000ms for 5m` | P1 |
| `mirror.profiles.total` | Gauge | For monitoring | Informational |
| `mirror.data_floor.domains_met` | Gauge | For monitoring | Informational |
| `mirror.moirai.emit.failure_rate` | Counter | `> 0.1%` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/mirror/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every requirement profile created and every outcome update is permanently recorded. The similarity results used in STOA decompositions can be verified against the profile record.

---

## 11. Implementation Roadmap

### Phase 1 — Profile Building and Basic Similarity (Year 2, Weeks 9–16)

- RequirementProfile schema and creation at STOA session completion
- Multi-dimensional Elasticsearch vector index
- Similarity query endpoint with data floor enforcement
- Data floor status endpoint
- STOA integration for decomposition suggestions

**Phase gate criterion:** Similarity query returns meaningful results for domains with > 50 profiles. Data floor status correctly reported. STOA receives similarity results.

### Phase 2 — Collection Effectiveness, Outcome Integration, and ORACLE Feed (Year 2, Weeks 17–24)

- CollectionEffectivenessRecord computation per profile
- OFS/NEMESIS outcome integration
- ORACLE profile feed
- TIS/DIKE collection effectiveness API
- Analyst similarity rating feedback

**Phase gate criterion:** Collection effectiveness scores differ meaningfully across collection methods. Outcome-updated profiles correctly reflected in similarity results. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate MIRROR. Cross-compartment similarity matching (a more powerful but restricted capability) requires IOB authority and is not in Phase 1 scope.

---

## 13. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q2 gate review.*
