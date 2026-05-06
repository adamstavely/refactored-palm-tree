# ADR-009 — Regional Data Plane Topology for Data Sovereignty

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, General Counsel, DPO

## Context

The firm operates in multiple jurisdictions including EU member states, the UK post-Brexit, and the United States. EU client matter data is subject to GDPR data residency obligations — it cannot leave EU infrastructure. UK data has distinct obligations from EU data post-Brexit. A single Kubernetes cluster deployment cannot satisfy both simultaneously.

This decision must be made before Phase 1 code is written because the PCES matter-scoping enforcement, MOIRAI chunk registry, and TVS validity index all require data_jurisdiction as a first-class attribute. Retrofitting geographic data boundaries onto a system built without them is architecturally expensive.

## Decision

Deploy THEMIS as a **hub-and-spoke regional topology**: a non-data-bearing control plane cluster (handles routing, policy distribution, and anonymised aggregate metrics) and regional data plane clusters (EU, UK, US, APAC) where matter data lives. No matter data transits the control plane.

All data-bearing THEMIS services (MOIRAI, TVS, ERAS, PCES matter registry, FGS cost records) are deployed per region. The control plane holds only routing configuration, PGS rule definitions (no matter content), and anonymised aggregate metrics. Model provider API calls are routed to endpoints within the same regional boundary as the matter data.

## Alternatives Considered

### Option A — Single cluster with logical separation only
Single Kubernetes cluster; matters tagged with jurisdiction; application-layer controls prevent cross-region data access. Lower operational complexity; single deployment to manage. However: application-layer controls are less robust than infrastructure-layer isolation; a misconfiguration in PCES could allow EU data to be processed outside the EU; data regulators may not accept application-layer separation as equivalent to physical data residency. Rejected: insufficient for regulatory compliance.

### Option B — Fully independent regional deployments (no shared control plane)
Each region is a completely independent THEMIS installation with no shared infrastructure. Maximum isolation; clear data residency. However: platform-level governance (PGS rule updates, AI Governance Committee decisions) must be propagated to each region independently; cross-region anonymised analytics (calibration trends, HADES coverage metrics) require separate extraction and anonymisation pipelines per region; operational complexity scales linearly with regions. Rejected: governance and observability complexity.

## Consequences

**Positive:**
- Data residency is enforced at the infrastructure layer, not the application layer
- Cross-region anonymised analytics are possible through the control plane without matter data crossing regional boundaries
- Model provider routing at the gateway level enforces regional API call locality without application-level changes
- Regional clusters can use region-appropriate cloud providers (AWS EU for EU data, AWS GovCloud for US federal data)

**Negative:**
- Significantly higher operational complexity: N+1 clusters to manage, monitor, and upgrade
- Inter-region latency: matter management system integration must handle the routing logic to send events to the correct regional cluster
- New matter intake must include jurisdiction classification before any THEMIS services process matter content

**Risks:**
- Jurisdiction misclassification: if a matter is incorrectly classified as US when it should be EU, data may be processed outside the required region. Mitigation: default to the most restrictive jurisdiction available; require explicit attorney confirmation for cross-border matter classification.

## Related Decisions
- ADR-006 — Pseudonymisation at IAM boundary enables user identity to cross regional boundaries safely
