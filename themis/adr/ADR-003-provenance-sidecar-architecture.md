# ADR-003 — Provenance Service Sidecar vs. Dedicated Service Architecture

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, DevOps Lead

## Context

Every AI model interaction must produce a provenance event (TurnInitiated, TurnCompleted). These events must reach the Provenance Service reliably. The API gateway is the natural interception point. The question is how to decouple event emission from event processing to avoid adding latency to the model request path and to prevent provenance failures from affecting AI availability.

## Decision

Use a **sidecar buffer + dedicated Provenance Service** architecture. The API gateway emits events to a sidecar container (fire-and-forget, < 1ms) that buffers to local disk and forwards to the Provenance Service asynchronously with retry. The Provenance Service is a standalone microservice that all gateways call through the sidecar.

The gateway's only obligation is to emit the event. All provenance logic, storage, and querying lives in the dedicated service.

## Alternatives Considered

### Option A — Sidecar only (no dedicated service)
Provenance data is stored locally per gateway pod. Simple deployment; no cross-service network calls. However: provenance data is siloed per pod — querying a full session's lineage requires fan-out across all gateway instances. Scaling the gateway independently of provenance is not possible. Real-time cross-session queries (e.g., "find all turns that retrieved this chunk") are expensive. Rejected: query performance and operational complexity at scale.

### Option B — Dedicated service, direct async call (no sidecar)
Gateway calls Provenance Service directly, asynchronously. No local buffer. Simpler deployment. However: if the Provenance Service is unavailable, events are lost — there is no buffer. An outage leaves a gap in the chain of custody that cannot be retroactively filled. Rejected: data loss risk is unacceptable for a chain-of-custody system.

### Option C — Kafka only (no sidecar)
Gateway publishes directly to Kafka; Provenance Service consumes. Kafka provides durability. However: Kafka client configuration complexity at the gateway layer; all gateways need Kafka connectivity; single broker failure requires careful Kafka cluster management. The sidecar approach provides the same durability guarantee with simpler gateway code. Kafka remains in the architecture as the inter-service event bus, but between the sidecar and the Provenance Service rather than directly from the gateway.

## Consequences

**Positive:**
- Gateway adds < 1ms to the request path for provenance emission
- Provenance Service failure does not affect AI availability
- Local disk buffer survives gateway restarts; events are replayed on recovery
- Provenance Service scales independently of gateway; query workload does not affect generation latency

**Negative:**
- Two components to operate (sidecar + service) rather than one
- Sidecar disk buffer requires monitoring; if disk fills, events are lost (alert threshold: 80% capacity)
- Eventual consistency: there is a brief window between event emission and provenance graph update

**Risks:**
- Sidecar crash during stream: TurnInitiated is written but TurnCompleted may not be. Mitigation: orphaned turn detection job marks incomplete turns as `status: incomplete` after a timeout and alerts.
