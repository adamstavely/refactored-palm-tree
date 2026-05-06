# ADR-010 — THEMIS Has No Standalone Application UI

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, Product Owner, Chief Experience Officer

## Context

THEMIS produces significant user-facing output: provenance score bands, validity warnings, Hold Queue notifications, Brief Validity Reports, calibration dashboards, HADES failure catalogs, and matter intelligence briefs. The question is whether these surfaces should live in a dedicated THEMIS application or be distributed across existing tools that attorneys and analysts already use.

## Decision

**THEMIS has no standalone application.** User-facing THEMIS surfaces are distributed across three existing platforms:

1. **Analyst tool** — inline provenance annotations, validity warnings, Hold Queue resolution workflows, Brief Validity Reports. Surfaces appear contextually when relevant, not as a destination to navigate to.
2. **Intellect (internal BI platform)** — governance dashboards, matter validity analytics, calibration trends, HADES failure catalog summaries, FGS cost intelligence. THEMIS is a module within Intellect, not a separate application.
3. **Kibana** — platform engineering observability: provenance event log analytics, service health, HADES raw probe logs, infrastructure metrics. Engineering-facing only.

External surfaces (Client Transparency Portal) are a separate application serving clients, not internal users.

## Alternatives Considered

### Option A — Dedicated THEMIS governance application
A standalone web application that attorneys open to access all THEMIS functionality. Centralized; can be designed holistically; no dependency on other teams' roadmaps. However: requires attorneys to context-switch from their primary work tool; a Hold Queue notification that requires opening a separate application introduces friction that will reduce compliance; tool adoption requires behavior change from every user; duplicates navigation structure that already exists in Intellect. Rejected: adoption risk; friction; duplication.

### Option B — Everything in the analyst tool
All THEMIS surfaces — including governance dashboards and calibration analytics — live in the analyst tool. Minimal context switching for analysts; single tool. However: governance dashboards serve the AI Governance Committee and practice group leads who are not primarily analyst tool users; mixing operational analytics with daily work surfaces clutters the analyst experience; Intellect already provides the dashboard mental model for this audience. Rejected: wrong audience for the analyst tool; clutters the primary work surface.

## Consequences

**Positive:**
- Attorneys engage with THEMIS governance naturally within existing workflows rather than as a compliance obligation requiring a separate tool
- Hold Queue resolution happens in the same surface where the blocked interaction was initiated — context is preserved
- Intellect gains THEMIS data alongside financial and people analytics — governance committee has a unified view of matter performance
- No new application to operate, maintain, or drive adoption of

**Negative:**
- THEMIS depends on the analyst tool team and Intellect team to implement their respective integration surfaces; THEMIS cannot ship governance features unilaterally
- Consistent visual language for THEMIS elements (score bands, badge colors, confidence indicators) must be coordinated across multiple host applications

**Risks:**
- If the analyst tool team deprioritizes THEMIS integration, governance features are delayed. Mitigation: THEMIS integration requirements are defined as part of the analyst tool roadmap from the start, not added later. The Query API design must support all integration surfaces without requiring per-surface API changes.
