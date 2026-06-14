# ADR-007 — HITL Orchestration as PGS Capability, Not Standalone Service

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, Product Owner

## Context

The PGS (Policy & Guardrails Service) rule engine includes a REQUIRE_APPROVAL action that holds an AI interaction pending human review. This action requires an operational workflow: a hold record must be created, a reviewer must be assigned, an SLA must be tracked, and a resolution must be recorded. The question is whether this workflow should be implemented as a standalone HITL (Human-in-the-Loop) Orchestration Service or as a capability extension within PGS.

## Decision

Implement HITL orchestration as a **capability extension within PGS, not a standalone service**. The Hold Queue, reviewer assignment logic, SLA tracking, and HoldResolution record generation are owned by PGS. HoldResolution records are written to the Provenance Service as immutable turn-level events.

The rationale: the HITL workflow is inseparable from the policy rule that triggered it. The same service that evaluates whether an interaction requires approval is the natural owner of the approval workflow. A standalone service would create a tight coupling between PGS (which triggers holds) and the HITL service (which resolves them) without the benefit of separation of concerns — the concerns are not actually separate.

## Alternatives Considered

### Option A — Standalone HITL Orchestration Service
A dedicated service owns the Hold Queue, reviewer assignment, SLA tracking, and resolution workflows. PGS emits a HOLD event; the HITL service processes it. Clean service boundary; HITL service could theoretically serve other consumers beyond PGS. However: the primary (and currently only) trigger for HITL is PGS. A standalone service for a single consumer creates operational overhead without architectural benefit. The "other consumers" argument is speculative. Rejected: premature service extraction; operational overhead without benefit.

### Option B — Client-side HITL (analyst tool handles holds)
The analyst tool manages the hold state, surfaces the hold to the reviewer, and signals PGS when the hold is resolved. No server-side hold queue. However: this puts business logic (reviewer assignment, SLA enforcement) in the client; hold state is lost if the analyst closes their browser; SLA enforcement cannot happen server-side if hold state is client-side. Rejected: business logic in client; reliability concerns.

## Consequences

**Positive:**
- Single service owns the complete policy evaluation and enforcement lifecycle
- No cross-service coordination required for hold creation and resolution within PGS
- PGS can enforce SLA deadlines and escalation without depending on another service's availability

**Negative:**
- PGS becomes more complex: it owns both policy evaluation and workflow orchestration
- If a future use case requires HITL outside of PGS (e.g., TVS conflict resolution), the Hold Queue is not available as a shared service. Mitigation: if this use case materializes, extract the Hold Queue into a shared library or service at that time.

## Related Decisions
- ADR-005 — HoldResolution records use the nullification-compatible provenance model
- ADR-004 — Turn records are incomplete until HoldResolution is written; this is enforced by PGS
