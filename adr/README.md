# Architecture Decision Records

This directory captures significant design decisions made for the THEMIS platform. Each ADR records what was decided, what alternatives were considered, and why — so the same debate does not happen twice as the team grows.

---

## ADR Index

| ID | Title | Status | Date |
|---|---|---|---|
| [ADR-001](ADR-001-graph-database-selection.md) | Graph Database Selection for Provenance Service | Accepted | 2026-05-05 |
| [ADR-002](ADR-002-fingerprinting-strategy.md) | AI Output Fingerprinting Strategy | Accepted | 2026-05-05 |
| [ADR-003](ADR-003-provenance-sidecar-architecture.md) | Provenance Service Sidecar vs. Dedicated Service | Accepted | 2026-05-05 |
| [ADR-004](ADR-004-session-manifest-vs-turn-dag.md) | Session Manifests vs. Turn-Level DAG for AI Interaction Provenance | Accepted | 2026-05-05 |
| [ADR-005](ADR-005-nullification-vs-deletion.md) | Nullification vs. Node Deletion for Data Retention | Accepted | 2026-05-05 |
| [ADR-006](ADR-006-iam-pseudonymisation-boundary.md) | Pseudonymisation at the IAM Boundary for GDPR Compliance | Accepted | 2026-05-05 |
| [ADR-007](ADR-007-pgs-hitl-as-capability-not-service.md) | HITL Orchestration as PGS Capability Not Standalone Service | Accepted | 2026-05-05 |
| [ADR-008](ADR-008-qrag-retrieval-architecture.md) | Q-RAG Multi-Step Retrieval for Legal Research | Accepted | 2026-05-05 |
| [ADR-009](ADR-009-regional-topology.md) | Regional Data Plane Topology for Data Sovereignty | Accepted | 2026-05-05 |
| [ADR-010](ADR-010-themis-no-standalone-ui.md) | THEMIS Has No Standalone Application UI | Accepted | 2026-05-05 |

---

## ADR Status Definitions

| Status | Meaning |
|---|---|
| **Proposed** | Under discussion; decision not yet made |
| **Accepted** | Decision made; implementation follows this decision |
| **Deprecated** | Was accepted; superseded by a later decision |
| **Superseded by ADR-NNN** | Explicitly replaced; link to replacement |

---

## How to Create an ADR

1. Copy the template below
2. Number it sequentially (ADR-011, ADR-012, ...)
3. Fill in all sections
4. Add it to the index table above
5. Reference it from any relevant service architecture doc

### ADR Template

```markdown
# ADR-NNN — [Short Decision Title]

**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-NNN
**Date:** YYYY-MM-DD
**Deciders:** [Names or roles of people involved in this decision]

## Context

[What is the situation that requires a decision? What forces are at play?
What constraints exist? Keep this factual — not the decision, just the context.]

## Decision

[What was decided? State it clearly in the first sentence.
Then explain the rationale.]

## Alternatives Considered

### Option A — [Name]
[Description. Why was it not chosen?]

### Option B — [Name]
[Description. Why was it not chosen?]

## Consequences

**Positive:**
- [What becomes easier or better because of this decision?]

**Negative:**
- [What becomes harder or what do we accept as a cost?]

**Risks:**
- [What could go wrong? How is it mitigated?]

## Related Decisions
- [Links to other ADRs that this one depends on or influences]
```
