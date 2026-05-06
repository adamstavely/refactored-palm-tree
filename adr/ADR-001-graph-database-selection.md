# ADR-001 — Graph Database Selection for Provenance Service

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, Lead Infrastructure Engineer

## Context

The Provenance Service (MOIRAI) must store and query a directed acyclic graph of content lineage — chunks, turns, sessions, documents, and the relationships between them. The primary query patterns are: ancestor traversal (given a document paragraph, find all source chunks it derives from), descendant traversal (given a source chunk, find all downstream work product that cites it), and path-length queries (what is the hop distance between a source and its derivative?). Secondary query patterns include: node property filters (chunks from a specific matter, turns from a specific session), time-bounded queries (what was the lineage state at a given date), and bulk graph reconstruction from the event log.

## Decision

Use **Neo4j** as the primary graph store for the Provenance Service. The graph is stored natively as nodes and relationships rather than as adjacency lists in a relational database or as documents in a document store.

The rationale: Neo4j's Cypher query language expresses traversal queries that would require complex recursive CTEs in PostgreSQL and cannot be expressed naturally in document stores. The provenance graph's primary value is in its relationships — ancestry chains, citation paths, session DAGs — not in the properties of individual nodes. A graph database treats relationships as first-class citizens; relational databases treat them as foreign keys to be joined.

## Alternatives Considered

### Option A — PostgreSQL with recursive CTEs
Familiar to most engineers; strong operational tooling; ACID guarantees; supports jsonb for flexible schema. Recursive ancestor/descendant queries are possible using `WITH RECURSIVE` CTEs but become expensive as graph depth and branching factor increase. A 10-hop ancestry traversal across a corpus of millions of chunks would require query optimization that a native graph database handles automatically. Relationship traversal performance degrades O(n) with path length in relational databases. Rejected: performance ceiling too low for the query patterns that matter most.

### Option B — Amazon Neptune (managed graph database)
Managed service reduces operational overhead. Supports both Gremlin and SPARQL. However: vendor lock-in to AWS complicates the regional data plane topology, which requires flexibility to deploy in EU and UK cloud regions with different provider characteristics. Neptune's Cypher support is less mature than Neo4j's native implementation. Rejected: regional flexibility requirement; Cypher maturity.

### Option C — PostgreSQL + pgvector for hybrid storage
Use PostgreSQL for structured node/edge data and pgvector for embedding-based similarity queries. Avoids introducing a second database technology. However: this still requires recursive CTEs for traversal and does not solve the fundamental query pattern mismatch. The vector store requirement (for RQS and ERAS) is separate from the lineage graph requirement and should be addressed separately. Rejected: does not solve the graph traversal problem.

## Consequences

**Positive:**
- Native graph traversal: ancestry and descendant queries are Cypher `MATCH (a)-[*]->(b)` patterns rather than recursive CTEs
- Relationship-first data model matches the provenance domain naturally
- Neo4j's APOC library provides graph algorithms (shortest path, PageRank, community detection) that may become useful for provenance analysis
- Well-established in production at scale; strong operational tooling (Neo4j AuraDB, self-hosted)

**Negative:**
- Introduces a database technology that most engineers are less familiar with than PostgreSQL
- Operational complexity: the team must develop Neo4j expertise alongside the service
- The event log (append-only Kafka stream) remains the source of truth; Neo4j is a derived view. Recovery from Neo4j failure requires replaying the event log.

**Risks:**
- Schema migration risk: Neo4j schema changes (new node types, new relationship types) require careful versioning as the graph grows. Mitigation: treat the provenance graph schema as a versioned contract; new node/relationship types are additive (backward compatible); breaking changes require a migration plan.

## Related Decisions
- ADR-003 — Sidecar architecture ensures events reach Neo4j reliably even during graph store downtime
- ADR-005 — Nullification strategy chosen specifically to preserve Neo4j node IDs and edges even when content is deleted
