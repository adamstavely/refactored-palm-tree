# MNEMOSYNE — Institutional Knowledge Graph Service
### MNEMOSYNE · *"Greek Titan of memory — mother of the Muses, the source from which all creative and analytical insight draws; the personification of the collective memory that makes learning possible"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MNEMOSYNE` |
| **Epithet** | `MNEMOSYNE` |
| **Full name** | Institutional Knowledge Graph Service |
| **Namespace** | `themis-knowledge` |
| **Layer** | Intelligence Layer — Knowledge Foundation |
| **Build phase** | Year 2 · Q3 |
| **Build priority** | 4 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Trust — extracts and surfaces the institutional knowledge embedded in the platform's analytical record |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MNEMOSYNE answers: What has this analytical community learned — from corrections, from outcomes, from verified assessments, from methodological patterns — that is relevant to this session's analytical question?**

### 1.2 Why This Service Exists

The THEMIS platform accumulates a record of what has been analytically tried and what happened. FGTS/ALETHEIA holds the correction history. OFS/NEMESIS holds the outcome record. ERAS/LOGOS holds the reasoning audit. TCS/MIMIR holds the calibration history. Individually these are quality signals. Together they contain something more valuable: institutional knowledge — what this analytical community has learned about how the AI performs on specific claim types in specific domains, what analytical patterns reliably precede confirmed outcomes, what common errors recur, what methodological approaches work.

This knowledge exists in the platform but it is not surfaced. An analyst working on a technical programme assessment does not automatically know that the community has made the same systematic error on similar assessments six times. A junior analyst does not know that a specific claim type in a specific domain has a well-established pattern that senior analysts have refined over years.

MNEMOSYNE extracts this implicit knowledge and makes it explicit, queryable, and surfaced at the point where it is most useful — in the session context assembled by MOS/SAGA, before the analyst commits to an analytical approach.

### 1.3 Design Principles

- **Extraction is inductive, not deductive.** MNEMOSYNE does not encode analytical doctrine from the top down. It extracts patterns from the bottom up — from what the platform has observed, not from what authority has declared. This makes it empirical, not prescriptive.
- **Knowledge nodes require evidence thresholds.** A pattern observed in two sessions is not institutional knowledge. MNEMOSYNE requires a minimum evidence threshold before creating a knowledge node. Early knowledge nodes are tentative; high-evidence nodes are authoritative.
- **Contradiction is surfaced, not resolved.** When the evidence record contains contradictory patterns (sometimes X works, sometimes it doesn't), MNEMOSYNE creates knowledge nodes for both patterns with their respective evidence bases. Resolving the contradiction is an analytical task, not a platform task.
- **Knowledge nodes decay.** Institutional knowledge from five years ago about a programme that has significantly changed is less relevant than current patterns. Knowledge nodes carry freshness signals; stale nodes are flagged, not deleted.
- **Year 2 Q3 means real evidence exists.** MNEMOSYNE is the last knowledge-layer service specifically because it requires at least 18 months of accumulated FGTS, OFS, and ERAS data to extract meaningful patterns. Deploying it earlier would produce knowledge nodes with insufficient evidence, creating false authority.

### 1.4 Explicit Out of Scope

- **Generating analytical conclusions.** MNEMOSYNE surfaces patterns from the record. Interpreting those patterns and deciding how to apply them to a specific analytical question is the analyst's task.
- **Replacing analytical doctrine.** MNEMOSYNE complements the existing analytic standards; it does not generate new doctrine. Where MNEMOSYNE patterns conflict with established standards, the standards take precedence.
- **Real-time extraction.** MNEMOSYNE extracts patterns in batch processes, not in real time. New sessions contribute to knowledge extraction on a scheduled cadence, not immediately.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MNEMOSYNE extracts structured knowledge nodes from the THEMIS analytical record — from FGTS correction patterns, OFS outcome patterns, ERAS reasoning captures, and TCS calibration history — builds a queryable institutional knowledge graph anchored to OGS entities and domain taxonomy, and surfaces relevant knowledge nodes to MOS/SAGA for session context assembly.

### 2.2 Secondary Functions

- Knowledge node confidence scoring: weighting each knowledge node by the evidence volume, recency, and analyst pool diversity that supports it
- Knowledge node contradiction tracking: maintaining parallel nodes for contradictory patterns and surfacing both to the analyst
- Domain expertise pattern extraction: identifying which analyst behaviours and analytical approaches are associated with higher confirmation rates in specific domains
- Freshness monitoring: tracking knowledge node age and flagging nodes whose evidence base is becoming stale
- Knowledge graph query interface: allowing analysts and STOA to query the knowledge graph directly for specific domain patterns
- IOB knowledge quality reporting: aggregate knowledge node quality metrics for oversight

### 2.3 What This Service Does Not Decide

MNEMOSYNE extracts and surfaces patterns. Whether a knowledge node should be acted upon in a specific analytical context, whether a contradicted node represents a genuine ambiguity or an error in the evidence, and whether institutional knowledge from the platform record should override an analyst's domain expertise are human analytical decisions.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
KnowledgeNode:
  node_id:                 uuid
  node_type:               CORRECTION_PATTERN |
                           OUTCOME_PATTERN |
                           CALIBRATION_PATTERN |
                           METHODOLOGY_PATTERN |
                           DOMAIN_HEURISTIC |
                           COMMON_ERROR |
                           ANALYTICAL_APPROACH
  domain:                  str
  claim_types:             [str]
  entity_types:            [str]              # OGS entity types this applies to
  content:                 str               # the knowledge in plain analytical language
  evidence_strength:       HIGH | MEDIUM | LOW | TENTATIVE
  evidence_count:          int               # number of evidence records supporting this
  evidence_source_types:   [str]             # FGTS | OFS | ERAS | TCS
  evidence_refs:           [uuid]            # references to source records
  freshness:               CURRENT | AGING | STALE  # based on evidence recency
  contradicted_by:         [uuid]            # node_ids of contradicting nodes
  analyst_pool_diversity:  int               # number of distinct analysts contributing evidence
  created_at:              datetime
  last_updated:            datetime
  last_confirmed:          datetime | null   # when was this last validated by new evidence

KnowledgeEdge:
  edge_id:                 uuid
  source_node_id:          uuid
  target_node_id:          uuid
  relationship:            SUPPORTS | CONTRADICTS | REFINES | SUPERSEDES | CONTEXTUALISES
  confidence:              float

KnowledgeQuery:
  query_id:                uuid
  session_id:              uuid
  domain:                  str
  claim_types:             [str]
  entity_ids:              [uuid]
  nodes_returned:          int
  relevance_scores:        { node_id: float }
  queried_at:              datetime

ExtractionRun:
  run_id:                  uuid
  run_type:                SCHEDULED | TRIGGERED
  source_type:             FGTS | OFS_NEMESIS | ERAS | TCS | ALL
  period_start:            datetime
  period_end:              datetime
  nodes_created:           int
  nodes_updated:           int
  nodes_contradicted:      int
  run_duration_seconds:    int
  run_at:                  datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Knowledge graph | Neo4j | KnowledgeNode and KnowledgeEdge graph | Indefinite |
| Query index | Elasticsearch | Knowledge node full-text search and relevance ranking | Mirrors Neo4j |
| Extraction records | PostgreSQL | ExtractionRun history; KnowledgeQuery audit | Indefinite |
| Node cache | Redis | Frequently queried nodes by domain (MOS/SAGA hot path) | 4h TTL |
| Event store | MOIRAI | Signed knowledge extraction events | Indefinite |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| KnowledgeNode | Controlled Unclassified (patterns are de-identified) | Platform-wide; no analyst identity in nodes |
| KnowledgeQuery | Inherits session classification | Session-compartmented |
| ExtractionRun | Controlled Unclassified | Platform team and IOB |

*Note: Knowledge nodes are de-identified. They contain analytical patterns (e.g., "assessments of adversary timeline in this domain have overestimated confidence 3 out of 4 times") not attributions to specific analysts or sessions.*

---

## 4. API Contract

### 4.1 Endpoints

```
GET /knowledge/query
  Auth:     MOS/SAGA service account | STOA service account | analyst session token
  Params:   domain: str, claim_types: [str], entity_types: [str], strength_min: str
  Response: {
    nodes:                 [KnowledgeNode],
    contradictions:        [{ node_id, contradicted_by_id }],
    query_id:              uuid
  }
  SLA: p99 < 500ms

GET /knowledge/nodes/{node_id}
  Auth:     analyst session token | supervisor token
  Response: KnowledgeNode with edge detail

GET /knowledge/domains/{domain}/summary
  Auth:     analyst session token | supervisor token | IOB token
  Response: {
    domain:                str,
    node_count:            int,
    by_type:               { node_type: count },
    by_strength:           { strength: count },
    stale_count:           int,
    contradiction_count:   int
  }

POST /extraction/trigger
  Auth:     platform operator token (for manual extraction)
  Request:  { source_type: str, period_start: datetime, period_end: datetime }
  Response: { run_id: uuid, estimated_completion: str }

GET /extraction/runs
  Auth:     platform operator | IOB token
  Response: [ExtractionRun]

GET /health
  Response: {
    status, dependencies: { moirai, neo4j, elasticsearch, fgts, ofs, eras, tcs, redis },
    total_nodes:           int,
    high_strength_nodes:   int,
    stale_nodes:           int,
    last_extraction_at:    datetime,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          MNEMOSYNE_EXTRACTION_COMPLETE
service_id:         "MNEMOSYNE"
session_id:         null
classification:     CONTROLLED_UNCLASSIFIED
event_payload:
  run_id:                 uuid
  source_type:            str
  nodes_created:          int
  nodes_updated:          int
  nodes_contradicted:     int
  extraction_period:      { start, end }
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          MNEMOSYNE_KNOWLEDGE_UPDATED
event_payload:
  domain:                 str
  node_type:              str
  nodes_affected:         int
  update_type:            CREATED | STRENGTHENED | CONTRADICTED | STALED
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `MNEMOSYNE_EXTRACTION_COMPLETE` | Extraction run completed | MOIRAI, MOS/SAGA (cache invalidation for relevant domains) |
| `MNEMOSYNE_KNOWLEDGE_UPDATED` | Significant node changes in a domain | MOIRAI, MOS/SAGA, STOA |

### 4.3 Consumed Events

MNEMOSYNE runs scheduled extraction against FGTS, OFS/NEMESIS, ERAS, and TCS data. It does not receive real-time events from these services — it reads their accumulated records on a scheduled cadence (default: daily).

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| FGTS/ALETHEIA | Ground Truth | Correction pattern source for extraction | Scheduled batch read | Extraction skips FGTS source; partial extraction |
| OFS/NEMESIS | Outcome Feedback | Outcome pattern source for extraction | Scheduled batch read | Extraction skips OFS source; partial extraction |
| ERAS/LOGOS | Reasoning Audit | Reasoning pattern source for extraction | Scheduled batch read | Extraction skips ERAS source; partial extraction |
| TCS/MIMIR | Trust Calibration | Calibration pattern source for extraction | Scheduled batch read | Extraction skips TCS source; partial extraction |
| OGS/YGGDRASIL | Ontology | Entity type and domain anchoring for knowledge nodes | Sync query | Knowledge nodes created without entity anchoring |
| MOIRAI | Provenance | Signed extraction events | Async event | Events buffered; extraction continues |

### 5.2 Feeds Into

| Service | Epithet | What MNEMOSYNE provides | How |
|---|---|---|---|
| MOS/SAGA | Memory Orchestration | Institutional knowledge nodes for session context | API + events |
| STOA | Research Orchestration | Domain knowledge patterns for research decomposition | API |
| ATHENA | Interface | Knowledge surfacing in analytical context panel | API (via MOS/SAGA) |
| IOB Reporting | Oversight | Knowledge quality metrics by domain | Audit endpoint |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Knowledge query (cached domain) | 30ms | 100ms | 500ms |
| Knowledge query (cold domain) | 200ms | 500ms | 1000ms |
| Extraction run (daily batch) | 5 min | 20 min | 60 min |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — MNEMOSYNE unavailability degrades session context quality but does not block sessions |
| RTO | 30 minutes |

---

## 7. Security Model

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| MOS/SAGA | Knowledge query (de-identified nodes) | Service account |
| STOA | Knowledge query | Service account |
| Analyst session | Domain summary; individual node detail (de-identified) | Session token |
| Platform operator | Extraction trigger; run history | Operator token |
| IOB | Full access including extraction history | IOB token |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Knowledge node false positive (pattern extracted from noise) | Medium | P2 — misleading institutional knowledge surfaces | Evidence threshold enforcement | Minimum evidence count = 5 before node creation; TENTATIVE label below threshold |
| Contradicted knowledge graph paralyses analyst | Low | P2 — too many contradictions, analyst cannot act | Contradiction rate monitoring | Contradictions are surfaced not highlighted; analyst chooses which applies |
| Stale knowledge nodes receive new evidence after staling | Medium | P2 — stale node inappropriately revived | Freshness scoring recalculation on new evidence | New evidence updates freshness score; node automatically re-evaluated |

### 8.1 Known Design Risks

- **Extraction requires 18 months of operational data to be meaningful.** Year 2 Q3 deployment means approximately 18 months of accumulated FGTS, OFS/NEMESIS, and ERAS data. At that scale, many domain-claim_type cells will have fewer than the minimum 5 evidence records for node creation. Early MNEMOSYNE will produce TENTATIVE nodes only in well-represented domains. This is correct behaviour — the value compounds as data accumulates.
- **Pattern extraction may perpetuate systematic bias.** If the analytical community has been systematically wrong about a specific domain (e.g., consistently over-rating AI capability confidence), MNEMOSYNE will extract that bias as a pattern. The knowledge node would accurately describe the community's historical behaviour but would reinforce rather than correct the bias. Resolution path: IOB knowledge quality review identifies systematically biased patterns; knowledge nodes from biased evidence can be flagged with an IOB caveat.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `mnemosyne.extraction.age_hours` | Gauge | `> 26h` (overdue) | P2 |
| `mnemosyne.nodes.stale_ratio` | Gauge | `> 20%` of total nodes | P2 |
| `mnemosyne.nodes.contradiction_ratio` | Gauge | `> 15%` of total nodes | P2 |
| `mnemosyne.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/mnemosyne/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every extraction run is permanently recorded — what sources were processed, how many nodes were created, updated, or contradicted, and over what period. An oversight body can verify that the institutional knowledge graph reflects the evidence record it claims to draw from.
- **What it cannot prove:** Knowledge nodes accurately capture the patterns in the evidence. The extraction model may make classification errors. The MOIRAI record attests the extraction was performed and what it produced; it does not attest the extraction was accurate.

---

## 11. Implementation Roadmap

### Phase 1 — Core Extraction Pipeline and Knowledge Graph (Year 2, Weeks 25–32)

- KnowledgeNode and KnowledgeEdge schemas in Neo4j
- Scheduled extraction pipeline (daily) reading FGTS, OFS, ERAS, TCS records
- Evidence threshold enforcement (minimum 5 records before node creation)
- TENTATIVE → LOW → MEDIUM → HIGH strength progression as evidence accumulates
- Knowledge query endpoint
- MOS/SAGA knowledge context integration

**Phase gate criterion:** Extraction pipeline runs daily and produces non-zero nodes in test domains. Knowledge query returns relevant nodes with confidence scores. MOS/SAGA successfully includes knowledge nodes in session context. Minimum evidence threshold enforced.

### Phase 2 — Contradiction Tracking, Freshness, and IOB Reporting (Year 2, Weeks 33–40)

- Contradiction detection and knowledge edge creation
- Freshness scoring and stale node identification
- STOA knowledge context integration
- IOB domain quality reporting
- Analyst knowledge query interface in ATHENA
- De-identification validation (no analyst identity in any node)

**Phase gate criterion:** Contradicted node pairs surface correctly for test evidence records. Freshness scoring updates correctly as evidence ages. IOB domain quality report shows node distribution by strength and freshness. De-identification audit confirms no analyst identity in node content. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate MNEMOSYNE. The IOB knowledge quality review process is an operational oversight function, not a policy dependency.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In the ATHENA session context panel (assembled by MOS/SAGA), institutional knowledge nodes appear as a "what the community has learned" section: "[HIGH strength] In assessments of this claim type in this domain, AI confidence has historically exceeded confirmation rate by approximately 15%. Apply additional epistemic scrutiny." Contradicted nodes show both patterns: "[MEDIUM] Pattern A: X. [MEDIUM] Pattern B: the inverse of X. Both patterns have evidence support — context determines which applies."

### 13.2 What the Analyst Should Do

Treat HIGH and MEDIUM strength knowledge nodes as strong advisory signals — not directives, but patterns with genuine evidential weight. If a knowledge node contradicts your intended analytical approach, investigate why before proceeding. Contradicted nodes require the analyst to consider which applies in the specific context rather than defaulting to either.

### 13.3 What the Signal Does Not Mean

A HIGH strength knowledge node is not a rule. It is an empirical pattern from the platform's accumulated evidence. The pattern may not apply to this specific case. Institutional knowledge is the starting point for analytical reasoning, not the conclusion.

---

## 14. Open Questions

### 14.1 Technical Open Questions

- **Q1: Extraction model classification accuracy.** Classifying FGTS correction records as CORRECTION_PATTERN vs. METHODOLOGY_PATTERN vs. COMMON_ERROR requires semantic understanding of what each correction represents. The classification model must be validated before Phase 1. Resolution path: Research & Red Team to label a sample of FGTS records for supervised training before extraction pipeline development.

### 14.2 Research Dependencies

- **Minimum evidence threshold validation.** The minimum evidence count of 5 before node creation is a conservative design choice. The Research & Red Team should evaluate whether this threshold produces the right balance between suppressing noise (too low a threshold) and suppressing real patterns (too high a threshold). Resolution path: empirical study after 6 months of operational data using a sample of manually validated patterns as ground truth.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q3 gate review.*
