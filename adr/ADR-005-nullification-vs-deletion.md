# ADR-005 — Nullification vs. Node Deletion for Data Retention

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, Legal Counsel, Data Protection Officer

## Context

The Provenance Service accumulates data continuously. Data protection obligations — GDPR Article 17 right to erasure, firm retention policies, matter close procedures — require that data be deleted when its legal basis for retention expires. However, deleting a node from the provenance graph breaks the lineage paths that downstream nodes depend on. A deleted chunk node leaves orphaned OutputChunk nodes with no traceable ancestry. This destroys the chain of custody for downstream artifacts.

## Decision

Use **nullification rather than node deletion** as the mechanism for satisfying data retention and erasure obligations. Nullification preserves graph structure while satisfying deletion obligations: the node remains in the graph with its ID and all edges intact, but content fields (raw text, document content, personal identifiers) are replaced with a nullification marker. Structural fields (chunk_id, spatial metadata, edge relationships, timestamps) are retained.

A NullificationRecord is written for each nullified node, capturing: which fields were erased, the legal basis for erasure, the timestamp, and the authorizing user.

## Alternatives Considered

### Option A — Node deletion with tombstone edges
Delete the node; replace incoming edges with a tombstone edge pointing to a placeholder node that records the deletion event. Downstream nodes can still be queried but their lineage terminates at the tombstone. Cleaner from a storage perspective. However: any query that asks "what was the full ancestry of this document?" now returns an incomplete answer at the tombstone. The chain of custody is broken in a way that is semantically meaningful — and potentially harmful in litigation, where the absence of a provenance record could be interpreted as evidence of tampering. Rejected: breaks chain of custody integrity.

### Option B — Full deletion with cascade
Delete the node and cascade deletion to all downstream dependent nodes. Completely removes the deleted content and everything derived from it. However: this is disproportionate — if a client requests erasure of their PII from one exhibit, cascading would delete all AI outputs, document sections, and legal documents that drew from that exhibit. This far exceeds the scope of the erasure right and destroys legitimate work product. Rejected: disproportionate; destroys work product.

### Option C — Separate content store with pointer model
Store content separately from the graph (e.g., in an S3 bucket); the graph contains only pointers. Nullification deletes the content object; the graph pointer remains and is flagged as nullified. Clean separation of structure from content. However: introduces a second storage tier with its own operational complexity; requires content retrieval to go through two systems; complicates the event log replay model. The simplicity benefit of keeping content inline with the graph is worth more than the clean separation. Rejected: operational complexity without sufficient benefit.

## Consequences

**Positive:**
- Graph structural integrity is preserved: all node IDs, edge relationships, and timestamps remain queryable
- Downstream provenance paths are complete (though they indicate that a source node's content was erased)
- NullificationRecord provides a clear audit trail of what was erased, when, and on what legal basis
- Consistent with the append-only philosophy of the event log: erasure is a recorded event, not an absence

**Negative:**
- Storage is not immediately reclaimed: nullified nodes still occupy space in Neo4j. Mitigation: periodic compaction job removes nullified content fields and reclaims storage without touching structural fields.
- The phrase "the content of this source was erased" in a provenance report may require explanation in legal proceedings. Mitigation: ERAS and documentation capture the legal basis for erasure in the NullificationRecord.

**Risks:**
- GDPR compliance of nullification: does replacing content with a nullification marker satisfy "erasure" under Article 17? Legal analysis: the personal data (the actual text) is removed; what remains is metadata about the existence and structure of the document. This is consistent with the approach taken by most large-scale data systems. The DPO has reviewed and accepted this approach.

## Related Decisions
- ADR-001 — Neo4j's node-relationship model makes nullification natural; structural fields (IDs, edges) and content fields can be independently managed
