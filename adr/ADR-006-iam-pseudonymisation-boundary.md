# ADR-006 — Pseudonymisation at the IAM Boundary for GDPR Compliance

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, Data Protection Officer

## Context

THEMIS stores references to the analysts and attorneys who initiated AI interactions — for chain of custody, calibration, and audit purposes. These are natural persons; their identity is personal data under GDPR. GDPR Article 17 grants individuals the right to erasure of their personal data. The question is: when an employee leaves the firm and requests erasure of their personal data from THEMIS, how is this achieved without destroying the chain of custody records of every AI interaction they ever initiated?

## Decision

**Pseudonymise analyst identity at the IAM boundary.** THEMIS services store only a `user_id` pseudonym — not name, email, or any directly identifying information. The mapping from `user_id` to real identity lives exclusively in the IAM system (Okta, Azure AD), not in THEMIS.

Satisfying a GDPR Article 17 erasure request for an analyst requires: (1) deletion of the identity mapping in the IAM system. No action in THEMIS is required, because THEMIS contains only the pseudonym. The pseudonym remains in the provenance graph — it is no longer linkable to a real person — and chain-of-custody integrity is preserved.

## Alternatives Considered

### Option A — Store full identity in THEMIS; nullify on erasure request
Store name and email in THEMIS turn records. On erasure request, nullify identity fields per ADR-005. Simple to implement initially. However: every erasure request requires identifying all THEMIS records containing that person's data, which is a graph-wide search operation at scale. This is expensive and error-prone. Rejected: operational complexity; risk of incomplete erasure.

### Option B — Store no identity in THEMIS
Anonymous interactions only. No analyst identity anywhere in THEMIS. Simple GDPR compliance — no personal data, no erasure obligation. However: chain of custody requires knowing who initiated each AI interaction. "An analyst reviewed this evidence" is not sufficient for legal work product accountability. Rejected: breaks chain-of-custody requirements.

## Consequences

**Positive:**
- GDPR Article 17 erasure for employees is satisfied by a single IAM action with no THEMIS changes
- THEMIS does not hold directly identifying personal data; it is structurally GDPR-compatible at the data model level
- Chain-of-custody integrity is preserved: the pseudonym remains in the graph and is attributable to a role even after the identity mapping is deleted

**Negative:**
- Analyst dashboards and audit reports that display human-readable names require a join against the IAM system at query time. This adds a network dependency to every user-facing provenance query.
- After identity mapping deletion, historical records show only a pseudonymous user_id with no human-readable attribution. This is intentional (erasure) but may require explanation in audit contexts.

**Risks:**
- If the IAM system is unavailable, THEMIS cannot resolve user_ids to human-readable identities for display purposes. Mitigation: cache display names with a short TTL; display the pseudonym when cache is cold.
