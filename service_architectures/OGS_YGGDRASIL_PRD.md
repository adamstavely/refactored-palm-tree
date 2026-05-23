# OGS — Ontology & Graph Service
### YGGDRASIL · *"The Norse world tree — the cosmic axis connecting all realms of existence; the semantic foundation that relates everything to everything else across all domains"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `OGS` |
| **Epithet** | `YGGDRASIL` |
| **Full name** | Ontology & Graph Service |
| **Namespace** | `themis-knowledge` |
| **Layer** | Intelligence Layer — Knowledge Foundation |
| **Build phase** | Year 2 · Q1 |
| **Build priority** | 1 of 15 intelligence layer services — prerequisite for all others |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — provides the canonical entity model for cross-INT source resolution |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**OGS/YGGDRASIL answers: What is this entity — and how does it relate to every other entity across collection types, analytical domains, and source vocabulary that uses different names for the same thing?**

### 1.2 Why This Service Exists

Intelligence collection describes the world using inconsistent vocabulary. SIGINT reports use one set of identifiers for an adversary programme; HUMINT reports use another; open-source reports use a third; previous analytical products use a fourth. Without entity resolution, the AI retrieves these as separate things. An analyst asking about a specific programme receives fragmented results because the retrieval system does not know that four different labels refer to the same entity.

OGS/YGGDRASIL is the canonical entity model that resolves this fragmentation. It maintains the authoritative entity graph: who is connected to whom, what organisations control what capabilities, what programmes are at what facilities, what events are related to what programmes. All other intelligence layer services — STOA, ORACLE, MIRROR, MNEMOSYNE — query OGS when they need to reason about entities rather than surface forms.

This is the first intelligence layer service for the same reason that a foundation is the first thing you build: nothing meaningful can stand without it.

### 1.3 Design Principles

- **Canonical identity is the foundation; aliases are derivations.** Every entity has one canonical_id and one canonical_name. All other names and identifiers are aliases or external_id mappings. Retrieval, reasoning, and analysis operate on canonical_ids — not on the surface form that appeared in a specific document.
- **The ontology is version-controlled and IOB-governed.** What entity types and relationship types exist in the canonical model is an analytical and policy decision, not a technical one. OGS implements the ontology; the analytic standards authority governs it.
- **Confidence is first-class.** Entity records carry confidence scores derived from the evidence supporting their existence and relationships. A low-confidence entity is not suppressed — it is surfaced with its confidence clearly expressed.
- **The graph is the analytical artefact.** The entity graph produced by OGS is not a search index — it is an analytical artefact that encodes what the analytical community believes about the relationships between entities in the world. It compounds in value as it grows.

### 1.4 Explicit Out of Scope

- **Open-source entity databases.** OGS is the canonical model for IC-specific entities. Commercial entity databases (corporate registries, public records) may inform it but are not part of it.
- **Natural language entity extraction.** OGS stores the canonical model. Extracting entity mentions from text and resolving them to canonical entities is a pipeline function that consumes OGS — not OGS itself.
- **Source provenance.** MOIRAI tracks what sources mention an entity. OGS tracks the entity and its relationships — not the full evidence base for each relationship.

---

## 2. Core Responsibilities

### 2.1 Primary Function

OGS/YGGDRASIL maintains the canonical entity model and relationship graph for the intelligence domain — a versioned, confidence-weighted graph of entities (persons, organisations, locations, capabilities, programmes, facilities, events) and their relationships, backed by a version-controlled ontology approved by the analytic standards authority. It provides entity resolution (surface form → canonical_id), relationship querying, and subgraph extraction for all consuming services.

### 2.2 Secondary Functions

- Entity resolution from text: mapping arbitrary surface forms (names, identifiers, aliases) to canonical entity IDs
- Cross-INT entity merging: identifying when SIGINT, HUMINT, and OSINT references are the same real-world entity and merging them under a single canonical ID
- Relationship evidence tracking: associating each relationship with the source corpus evidence that supports it
- Ontology version management: maintaining the history of ontology versions with IOB-approved change records
- Entity confidence scoring: computing and updating confidence scores as evidence accumulates
- Dead reckoning for entity state: inferring likely current entity state from known relationships and historical patterns (Year 2 Q2+ capability)

### 2.3 What This Service Does Not Decide

OGS maintains the entity model as it is built from corpus evidence and analyst contributions. Whether an entity relationship is analytically significant, whether a low-confidence entity warrants collection attention, and whether two entities that appear different are actually the same real-world entity are analytical judgments. OGS stores the model; analysts build and refine it.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
Entity:
  entity_id:               uuid
  entity_type:             PERSON | ORGANIZATION | LOCATION | CAPABILITY | PROGRAM |
                           FACILITY | VESSEL | PLATFORM | TECHNOLOGY | EVENT | UNKNOWN
  canonical_name:          str
  aliases:                 [str]
  external_ids:            { source_name: external_id }   # cross-reference to external systems
  classification:          str
  confidence:              float                # 0.0–1.0; strength of evidence for entity existence
  first_seen:              datetime
  last_confirmed:          datetime
  status:                  ACTIVE | INACTIVE | MERGED | UNCERTAIN
  merged_into:             uuid | null          # if MERGED; points to canonical entity

EntityRelationship:
  relationship_id:         uuid
  source_entity_id:        uuid
  target_entity_id:        uuid
  relationship_type:       str                  # ontology-defined; e.g., MEMBER_OF, CONTROLS,
                                               # LOCATED_AT, PRODUCES, PARTICIPATES_IN, etc.
  direction:               DIRECTIONAL | BIDIRECTIONAL
  confidence:              float
  evidence_chunk_ids:      [str]               # corpus chunk IDs supporting this relationship
  valid_from:              datetime | null
  valid_until:             datetime | null
  last_confirmed:          datetime

OntologyVersion:
  version_id:              uuid
  version_string:          str
  entity_types:            [str]
  relationship_types:      [{ name, from_types, to_types, description }]
  iob_approval_ref:        str
  effective_from:          datetime
  change_summary:          str

EntityResolutionRecord:
  record_id:               uuid
  surface_form:            str                  # the text as it appeared
  source_type:             str                  # SIGINT | HUMINT | OSINT | etc.
  resolved_entity_id:      uuid | null
  confidence:              float
  resolution_method:       EXACT_MATCH | ALIAS_MATCH | FUZZY_MATCH | UNRESOLVED
  session_id:              uuid | null
  resolved_at:             datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Entity store | Neo4j | Canonical entity and relationship graph | Indefinite |
| Resolution index | Elasticsearch | Surface form → canonical entity fast lookup | Mirrors Neo4j |
| Provenance store | PostgreSQL | EntityResolutionRecord, OntologyVersion audit history | Indefinite |
| Event store | MOIRAI | Signed entity creation and relationship events | Indefinite |
| Graph cache | Redis | Frequently queried subgraphs; entity lookup cache | 1h TTL |

Neo4j is the authoritative store for the entity graph. Elasticsearch is derived from it and supports fast text-based resolution. PostgreSQL holds the audit and governance records.

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| Entity records | Inherits from primary evidence | Compartment-gated; cross-compartment entity resolution requires IOB authority |
| EntityRelationship | Inherits higher of source/target entity classification | Strict compartment enforcement on graph traversal |
| OntologyVersion | Controlled Unclassified | Platform-wide |

### 3.4 Retention and Purge Policy

Entity records and relationships retained indefinitely — the canonical model accumulates value over time and must not be purged without IOB authority. MERGED entity records are retained with their merge history. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /entities/resolve
  Auth:     any service account | ATHENA session token
  Request:  {
    surface_form:          str,
    source_type:           str,
    context_session_id:    uuid | null
  }
  Response: {
    entity_id:             uuid | null,
    canonical_name:        str | null,
    entity_type:           str | null,
    confidence:            float,
    method:                str
  }
  SLA: p99 < 100ms (Elasticsearch index)

GET /entities/{entity_id}
  Auth:     session token (PCES-scoped)
  Response: Entity with top-N relationships

GET /entities/{entity_id}/relationships
  Auth:     session token (PCES-scoped)
  Params:   relationship_types: [str], depth: int (max 3)
  Response: { entity: Entity, relationships: [EntityRelationship], related_entities: [Entity] }
  SLA: p99 < 500ms (Neo4j traversal)

POST /queries/subgraph
  Auth:     STOA service account | ORACLE service account | session token
  Request:  {
    seed_entity_ids:       [uuid],
    relationship_types:    [str] | null,
    max_depth:             int,           # 1-3
    min_confidence:        float,
    classification_ceiling:str
  }
  Response: {
    nodes:                 [Entity],
    edges:                 [EntityRelationship]
  }
  SLA: p99 < 2000ms

POST /entities
  Auth:     corpus ingestion service account | analyst session token
  Request:  Entity (without entity_id)
  Response: { entity_id: uuid, action: CREATED | MERGED_INTO, merged_into_id: uuid | null }

POST /entities/{entity_id}/relationships
  Auth:     corpus ingestion service account | analyst session token
  Request:  EntityRelationship (without relationship_id)
  Response: { relationship_id: uuid }

GET /ontology/current
  Auth:     any service account
  Response: OntologyVersion (entity_types and relationship_types)
  SLA: p99 < 50ms (cached)

GET /health
  Response: {
    status, dependencies: { moirai, pces, neo4j, elasticsearch, redis },
    entity_count:          int,
    relationship_count:    int,
    active_ontology_version:str,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          OGS_ENTITY_CREATED
service_id:         "OGS"
session_id:         uuid | null
classification:     str
event_payload:
  entity_id:              uuid
  entity_type:            str
  confidence:             float
  source_type:            str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          OGS_ENTITIES_MERGED
event_payload:
  source_entity_id:       uuid
  target_entity_id:       uuid        # canonical surviving entity
  relationship_count_merged: int

EventType:          OGS_ONTOLOGY_UPDATED
event_payload:
  version_from:           str
  version_to:             str
  iob_approval_ref:       str
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `OGS_ENTITY_CREATED` | New entity added to the graph | MOIRAI, MNEMOSYNE (new entity triggers knowledge graph update) |
| `OGS_ENTITIES_MERGED` | Two entities resolved to the same real-world entity | MOIRAI, all consuming services (entity ID update cascade) |
| `OGS_ONTOLOGY_UPDATED` | New ontology version activated | MOIRAI, all consuming services (cache invalidation) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| Corpus ingestion | New source ingested | Entity extraction pipeline runs; new entities and relationships submitted |
| PCES/AEGIS | `PCES_SESSION_GRANTED` | Session classification scope used for graph query compartment filtering |
| KCS/ARGUS | `KCS_SOURCE_SUPERSEDED` | Relationships sourced from superseded source flagged for confidence review |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed entity and relationship events | Async event | Events buffered; graph operations continue |
| PCES/AEGIS | Classification Enforcement | Graph query compartment scoping | Sync | Queries proceed with cached scope |

### 5.2 Feeds Into

| Service | Epithet | What OGS provides | How |
|---|---|---|---|
| STOA | Research Orchestration | Entity resolution and relationship context for research queries | API |
| ORACLE | Outcome Intelligence | Entity-based requirement similarity and outcome linking | API |
| MIRROR | Requirement Similarity | Entity overlap scoring for similar requirement identification | API |
| MNEMOSYNE | Institutional Knowledge | Entity and relationship graph for knowledge node anchoring | API |
| MOS/SAGA | Memory Orchestration | Entity context for session memory assembly | API |
| ATHENA | Interface | Entity hover cards; entity-linked source badges | API |
| ARGUS-LACUNA | Collection Gap | Entity-based gap identification (entities with thin coverage) | API |

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 | p95 | p99 |
|---|---|---|---|
| Entity resolution (text → canonical ID) | 10ms | 30ms | 100ms |
| Entity lookup by ID | 5ms | 15ms | 50ms |
| 1-hop relationship query | 20ms | 60ms | 200ms |
| 2-hop subgraph query | 100ms | 400ms | 1000ms |
| 3-hop subgraph query | 500ms | 1500ms | 2000ms |

### 6.2 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — OGS unavailability degrades all intelligence layer services |
| Neo4j durability | 99.9999% — the graph is irreplaceable |
| RTO | 15 minutes |
| RPO | 1 hour |

### 6.3 Graceful Degradation

| Dependency unavailable | Behavior | Impact |
|---|---|---|
| Neo4j | Entity resolution falls back to Elasticsearch (surface forms only; no relationship traversal) | Intelligence layer services degrade to keyword retrieval |
| Elasticsearch | Resolution falls back to Neo4j (higher latency) | Resolution p99 increases to ~500ms |
| MOIRAI | Events buffered; graph operations continue | Provenance gap logged |

---

## 7. Security Model

### 7.1 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Corpus ingestion | Entity and relationship creation | Service account |
| Intelligence layer services | Read (compartment-scoped) | Service account |
| Analyst session | Entity lookup (compartment-scoped) | Session token |
| Analytic standards authority | Ontology management | Authority token + IOB approval ref |
| IOB | Full access | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/ogs/signing-key` | 90 days |
| Neo4j credentials | `themis/ogs/neo4j-credentials` | 30 days |
| Elasticsearch credentials | `themis/ogs/es-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Cross-compartment entity merge error | Low | P0 — entities from different compartments incorrectly merged | Compartment consistency check on merge | Merge operation validates both entities' compartments match; IOB approval for cross-compartment merge |
| Entity resolution ambiguity (multiple entities match a surface form) | High | P2 — wrong entity resolved | Resolution confidence score | Return top-N candidates with confidence scores; analyst resolution for low-confidence cases |
| Graph traversal performance degradation at scale | Medium (Year 3+) | P2 — subgraph queries slow | 3-hop query p99 monitoring | Query depth limit (max 3); caching for common subgraphs; graph partitioning strategy |

### 8.1 Known Design Risks

- **Initial entity graph population is the hardest problem.** The value of OGS is proportional to the quality and completeness of the entity graph. Building the initial graph from scratch requires a combination of automated extraction from the existing corpus, manual analyst contribution, and integration with existing entity registries. The automated extraction pipeline will produce a noisy initial graph with many false merges and missed relationships. The analytic standards authority must plan for a significant graph quality effort in the first 6 months after OGS deployment.
- **Cross-compartment entity resolution is architecturally complex.** The same real-world person may appear in both SECRET and TS/SCI corpus sources. Resolving these to the same canonical entity requires a merge operation that spans compartments — which is itself a classified operation. The current design restricts cross-compartment merge to IOB authority. This means some entity merges that a senior analyst with full access would recognise will not happen automatically. Resolution path: a supervised cross-compartment entity resolution workflow for authorised analysts, with IOB sign-off.

---

## 9. Observability

| Metric | Type | Alert | Severity |
|---|---|---|---|
| `ogs.resolution.latency_p99` | Histogram | `> 100ms for 5m` | P1 |
| `ogs.subgraph.latency_p99` | Histogram | `> 2000ms for 5m` | P1 |
| `ogs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `ogs.resolution.unresolved_rate` | Gauge | `> 30%` | P2 |
| `ogs.graph.entity_count` | Gauge | For monitoring | Informational |

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/ogs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Entity creations and merges are permanently recorded. An oversight body can reconstruct the evolution of the canonical entity model — when each entity was first identified, when merges were performed, and which ontology version governed the model at any point.

---

## 11. Implementation Roadmap

### Phase 1 — Core Entity Graph and Resolution (Year 2, Weeks 1–8)

- Entity and EntityRelationship schemas in Neo4j
- Entity resolution Elasticsearch index
- Core API: resolve, get entity, get relationships, create entity/relationship
- Ontology v1.0 deployment with IOB approval
- MOIRAI event emission
- Initial corpus entity extraction pipeline (automated, noisy — accuracy improves in Phase 2)

**Phase gate criterion:** Entity resolution returns canonical IDs for > 50% of test surface forms. Subgraph query returns correct 2-hop neighbourhood for test entities. Ontology v1.0 covers all primary IC entity types. OGS is accessible to other intelligence layer services.

### Phase 2 — Subgraph Queries, Cross-INT Merging, and Quality Programme (Year 2, Weeks 9–16)

- Subgraph query endpoint for STOA and ORACLE
- Cross-INT entity merge workflow (same-compartment automated; cross-compartment IOB workflow)
- Entity confidence scoring and update pipeline
- Ontology governance workflow (IOB approval for updates)
- Entity graph quality programme with analytic standards authority

**Phase gate criterion:** Subgraph queries serve STOA and ORACLE correctly in integration testing. Cross-INT merge demonstrated on test corpus. Resolution rate > 70% on test surface forms. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate OGS. Ontology changes require IOB approval per the same governance model as PGS/NOMOS policy changes. Cross-compartment entity merge requires IOB case-by-case authority.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In ATHENA, entity mentions in retrieved sources display a hover card showing the canonical entity name, type, confidence, and top relationships from OGS. Low-confidence entities show an amber indicator. Unresolved entity mentions (surface forms that OGS cannot map to a canonical entity) show a grey indicator with an option to contribute a resolution.

### 13.2 What the Analyst Should Do

When OGS cannot resolve an entity mention, the analyst can submit a resolution (linking the surface form to an existing canonical entity or proposing a new entity). These analyst contributions are among the most valuable inputs to graph quality. Unresolved entities are not just a UI annoyance — they represent gaps in the canonical model that affect retrieval quality for everyone working on similar targets.

---

## 14. Open Questions

### 14.1 Technical Open Questions

- **Q1: Automated entity extraction quality at corpus scale.** The accuracy of automated entity extraction from classified intelligence documents varies significantly by document type and collection method. Resolution path: Research & Red Team to benchmark extraction accuracy on sample corpus before Phase 1 gate; set minimum precision/recall thresholds for production deployment.

### 14.2 Operational Assumptions

- **Assumption 1: The analytic standards authority will staff and plan for graph quality work.** Building a high-quality entity graph requires sustained human effort beyond what automated extraction can provide. This must be a planned operational activity, not an afterthought.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 2 Q1 gate review.*
