# THEMIS Platform — Addendum B: Access Control Architecture

**Document series:** THEMIS Platform Design · Addendum B  
**Companion to:** THEMIS Platform Design v1.0, Addendum A (New Services)  
**Status:** Draft for review  
**Addresses:** Three access control gaps identified in security design review

-----

## Overview

Addendum A introduced five new THEMIS services and three architectural extensions. Addendum B addresses access control architecture — specifically three problems identified in security design review that the existing service set does not handle in its current form.

**No new services are introduced.** Each problem is addressed through targeted enhancements to existing services. The three problems are:

1. **Derived information persistence** — what happens to AI-synthesized outputs when the source evidence access is subsequently revoked
1. **Retrieval coverage gaps** — how the system surfaces the existence of relevant evidence that an analyst cannot access
1. **Query-layer access control** — how access control applies to the query itself, not just to the evidence retrieved

These problems are architecturally distinct. Problem 1 is retroactive — it concerns artifacts already generated. Problem 2 is prospective — it concerns what the analyst doesn’t know they’re missing. Problem 3 is preventive — it concerns control at the point of query submission.

### Services Enhanced in This Addendum

|Service   |Problems Addressed                                                                                 |
|----------|---------------------------------------------------------------------------------------------------|
|PCES/AEGIS|1 (revocation event production), 3a (query matter scope), 3b (cross-matter contamination detection)|
|MOIRAI    |1 (reverse DAG traversal and blast radius enumeration)                                             |
|DPS/CODEX |1 (AccessRevocationEvent handler, document marking)                                                |
|ERAS/LOGOS|1 (AccessRevocationEvent handler, reasoning record marking)                                        |
|PGS/NOMOS |3a (query privilege classification policy), 3c (query-type authorization)                          |
|IAS/SCUDO |3b (privilege contamination classifier extension)                                                  |
|RQS/HERMES|2 (gap coverage metric, shadow retrieval)                                                          |

-----

## Problem 1: Derived Information Persistence

### Problem Statement

THEMIS enforces access control at evidence ingestion and retrieval. An analyst’s access to specific evidence is governed by PCES/AEGIS based on matter scope, privilege classification, and conflict of interest status.

However, access control decisions are point-in-time. When an analyst has access to Evidence E at session time T1 and the model produces a synthesized response drawing on E, that synthesis exists as a downstream artifact — in ERAS/LOGOS reasoning records, in DPS/CODEX document provenance, and potentially in exported work product. If access to E is subsequently revoked at T2 — through privilege reclassification, matter scope change, conflict of interest discovery, or inadvertent production clawback under FRCP 26(b)(5)(B) — those downstream artifacts remain in the system unchanged.

The current system has no mechanism to:

- Enumerate which downstream artifacts were derived from the revoked evidence
- Flag those artifacts for mandatory attorney review
- Track remediation status

This is the **derived information problem** in access control theory. The technical remediation is bounded: the system can enumerate and flag artifacts, but cannot undo the analyst having read the synthesis. The professional responsibility implications — particularly in inadvertent production clawback scenarios — require General Counsel input before the remediation procedure is finalized. See Open Items GC-1.

### Solution: Derivation-Aware Access Revocation

This enhancement introduces a new first-class event type — `AccessRevocationEvent` — and handlers across four existing services. The pattern follows the existing event-driven architecture: PCES/AEGIS produces the event, MOIRAI orchestrates the blast radius enumeration, DPS/CODEX and ERAS/LOGOS respond with artifact marking.

#### New Event Type: AccessRevocationEvent

```
AccessRevocationEvent {
  evidence_id:       UUID           — Evidence object losing access
  matter_id:         UUID
  revocation_reason: RevocationReason
  revocation_type:   full | analyst_scoped
  affected_analyst_ids: AnalystID[] — For analyst_scoped revocations
  effective_at:      Timestamp
  produced_by:       PCES/AEGIS
  signed:            true           — Cryptographic attestation required
  prev_event_hash:   SHA256
}

RevocationReason:
  privilege_reclassification
  matter_scope_change
  conflict_of_interest
  inadvertent_production_clawback   — FRCP 26(b)(5)(B) scenarios
  administrative
```

#### PCES/AEGIS Enhancement: Revocation Event Production

PCES/AEGIS gains a revocation production function. When any access control decision changes for a specific evidence object in a way that reduces access for one or more analysts, PCES produces and publishes a signed AccessRevocationEvent to Kafka. All downstream services consume this event via their existing Kafka consumer groups.

**New function:** Revocation event production  
**Trigger:** Any change to privilege classification, matter scope, or conflict of interest status that reduces access to a previously accessible evidence object  
**Output:** Signed AccessRevocationEvent published to Kafka  
**Phase:** Phase 3-4 — AccessRevocationEvent schema design must occur alongside MOIRAI Phase 3-4, as MOIRAI is the primary consumer

#### MOIRAI Enhancement: Reverse DAG Traversal

MOIRAI adds a reverse traversal function that fires on AccessRevocationEvent consumption. The provenance graph already encodes which turns, sessions, documents, and claims were derived from which evidence objects. On AccessRevocationEvent receipt:

1. Locate the node for the revoked evidence_id in the Neo4j provenance graph
1. Traverse all outbound derivation edges recursively (produced, retrieved, derived_from, attributed_to)
1. Enumerate the blast radius: sessions, turns, claims, ERAS reasoning records, DPS document provenance records with a derivation path through the revoked evidence
1. Emit a `RevocationBlastRadiusEvent` containing the enumerated artifact IDs
1. Write a blast radius record to the revocation registry

**Storage — blast radius registry (PostgreSQL, append-only):**

|Field              |Type     |Purpose                                             |
|-------------------|---------|----------------------------------------------------|
|revocation_event_id|UUID     |Source AccessRevocationEvent                        |
|artifact_type      |string   |session / turn / claim / document / reasoning_record|
|artifact_id        |UUID     |Affected artifact identifier                        |
|remediation_status |enum     |pending / reviewed / acknowledged / escalated       |
|reviewed_by        |AnalystID|Supervising attorney who reviewed                   |
|reviewed_at        |Timestamp|Review timestamp                                    |

**Latency:** Async. Not in the critical path for active sessions. SLA: blast radius enumeration complete within 60 minutes of AccessRevocationEvent receipt.  
**Phase:** Phase 3-4 alongside MOIRAI. Reverse DAG traversal requires a fully operational provenance graph.

#### DPS/CODEX Enhancement: Document Marking

DPS adds a handler for `RevocationBlastRadiusEvent`. For each document_id in the blast radius:

- Document record marked: `source_access_revoked: true`, `revocation_event_id`, `review_required: true`
- On next DocumentOpenedEvent for the marked document, ATHENA surfaces a blocking notice to the analyst: *“A source document used in this work product is no longer accessible to you. A supervising attorney must review this document before it can be exported or filed.”*
- The DPS provenance panel highlights affected paragraphs — those whose provenance chain includes the revoked evidence
- **Export gate hardening:** Documents with `review_required: true` from an access revocation cannot be exported through the ATHENA export gate until a supervising attorney provides a signed acknowledgment. The acknowledgment is recorded as a MOIRAI event.

**Phase:** Phase 3-4 alongside DPS. DPS document lifecycle events must be live before this handler is meaningful.

#### ERAS/LOGOS Enhancement: Reasoning Record Marking

ERAS adds a handler for `RevocationBlastRadiusEvent`. For each reasoning record in the blast radius:

- Record marked: `source_access_revoked: true`, `revocation_event_id`
- Marked records display a revocation indicator in the ATHENA session view
- Professional responsibility export of marked reasoning records requires supervising attorney acknowledgment before export
- Marked records are flagged in the ERAS professional responsibility export surface as potentially affected

**Phase:** Phase 7-8 alongside ERAS.

#### ATHENA Surface: Revocation Notice

A new notification type in ATHENA — the **Revocation Notice** — surfaces when an analyst opens a session or document that has been flagged:

> *“Evidence used in this session is no longer accessible to you. [N] claims and [M] document paragraphs in this matter may be affected. A supervising attorney must review before this work product is exported or filed.”*

**Behavior:**

- Blocking for export — no export proceeds until supervisor acknowledgment
- Informational (non-blocking) for continued session work — the analyst may continue working in the session
- The supervising attorney receives a parallel notification with the blast radius summary, not shared with the affected analyst

#### Professional Responsibility Note

THEMIS can enumerate the blast radius of a derived information event and enforce review requirements before export. It cannot remediate the fact that synthesis derived from the now-inaccessible evidence was read by the analyst prior to revocation.

For inadvertent production clawback scenarios specifically — where opposing counsel has produced privileged documents that are now subject to recall under FRCP 26(b)(5)(B) — the firm’s obligations (whether to file corrections, notify the court, or take other remedial steps) require procedures developed with General Counsel. THEMIS provides the technical infrastructure for that procedure. See Open Items GC-1.

-----

## Problem 2: Retrieval Coverage Gaps

### Problem Statement

PCES/AEGIS enforces matter scope and privilege access at the retrieval layer. Evidence outside an analyst’s access scope does not enter the RAG pipeline. From the analyst’s perspective, retrieval appears complete — there is no signal that relevant evidence exists beyond their access boundary.

The risk: an analyst produces a finding that is substantially incomplete or misleading because highly relevant evidence was excluded by access control, and no mechanism exists to surface this gap. The system knows the gap exists. It currently says nothing.

A complete solution does not exist. The system cannot tell the analyst what the inaccessible evidence says without violating access control. The tractable version: tell the analyst that relevant inaccessible evidence exists on this query, without revealing its content, and provide a path to escalation.

### The Privileged MIMIR Channel

This enhancement requires a second retrieval pass against the full corpus, bypassing PCES access scope filtering, for the purpose of relevance comparison only. This is the **Privileged MIMIR Channel**.

**Architectural constraints — all must be enforced before Phase 5-6 implementation:**

1. The channel returns relevance scores and evidence_ids only — never content or text
1. Channel output never enters the context window for synthesis under any circumstances
1. The channel is accessible only to the RQS gap detection function — not reachable from any inference path
1. Every privileged channel query is logged to MOIRAI as a signed event with purpose: `gap_detection_only`
1. AI Governance Committee must formally approve the privileged channel at Phase 5-6 governance review
1. A dedicated security architecture review is required before implementation

Violation of constraint 2 or 3 would create a privilege bypass vulnerability. These constraints must be architecturally enforced, not reliant on documentation or convention.

### RQS/HERMES Enhancement: Gap Coverage Metric

RQS/HERMES adds a **gap coverage metric** as a new retrieval quality dimension alongside precision, miss rate, and embedding drift monitoring.

**Function:** On each retrieval event, RQS:

1. Receives the standard access-scoped retrieval results from MIMIR
1. Issues a second query via the Privileged MIMIR Channel against the full corpus
1. Computes the gap coverage score
1. If gap coverage exceeds the configured threshold, emits a `RelevanceGapEvent`

**Gap coverage score:**

```
gap_score = Σ(relevance_score of excluded chunks above threshold)
          ÷ Σ(relevance_score of all chunks above threshold)
```

A gap_score of 0.0 means all relevant evidence is within the analyst’s access scope. A gap_score of 0.4 means approximately 40% of the relevant retrieval weight is from evidence the analyst cannot access.

**Threshold configuration:** Configurable by PGS/NOMOS per interaction class. Default: gap indicator fires when gap_score > 0.25. Criminal litigation and high-stakes matter classes may warrant lower thresholds. The appropriate threshold configuration for each of the firm’s practice areas requires AI Governance Committee determination.

#### RelevanceGapEvent Schema

```
RelevanceGapEvent {
  session_id:              UUID
  query_id:                UUID
  matter_id:               UUID
  analyst_id:              AnalystID
  gap_score:               number     — 0.0 – 1.0
  excluded_evidence_count: number     — Count only, not identifiers
  threshold_exceeded:      boolean
  produced_by:             RQS/HERMES
  signed:                  true
}
```

**Note:** `excluded_evidence_ids` are stored in the RQS gap event log for supervisor review purposes. They do not appear in the RelevanceGapEvent schema and are not surfaced to the analyst in any form.

**Storage — gap event log (PostgreSQL, append-only):**

|Field                    |Type   |Purpose                                              |
|-------------------------|-------|-----------------------------------------------------|
|query_id                 |UUID   |Originating query                                    |
|gap_score                |number |Computed coverage gap                                |
|excluded_evidence_ids    |UUID[] |Accessible to supervisors only — never to the analyst|
|threshold_exceeded       |boolean|Whether gap indicator was fired                      |
|privileged_channel_log_id|UUID   |MOIRAI event reference for audit                     |

#### PCES/AEGIS Enhancement: Dual-Scope Retrieval Coordination

PCES/AEGIS adds a coordination function for the privileged channel: it provides RQS with the access-scoped retrieval scope for the current analyst session, and separately authorizes the RQS gap detection function to issue privileged channel queries. The authorization is session-scoped, logged, and enforced at the MIMIR routing layer.

#### ATHENA Surface: Gap Indicator

When a RelevanceGapEvent fires above threshold, ATHENA surfaces a **Gap Indicator** in the session:

> *“Additional evidence relevant to this query exists in this matter that you do not currently have access to. A supervising attorney should review this analysis before it is used in work product.”*

**Indicator displays:** The gap score as a qualitative signal (Low / Moderate / High based on threshold bands) and a one-click escalation option.  
**Indicator does not display:** What the excluded evidence is, how many items, or any characterizing information about the excluded evidence.

#### Escalation Path

The Gap Indicator includes a one-click **“Flag for Supervisor Review”** action. This creates a `SupervisorReviewRequest` in DPS/CODEX that surfaces in the supervising attorney’s ATHENA dashboard. The supervisor — who has full matter access — reviews the flagged analysis in the context of the complete evidence set and provides either an acknowledgment or a revised finding to the analyst team.

Supervisor review acknowledgments are logged to MOIRAI as signed events.

**Phase:** Phase 5-6 alongside RQS/HERMES. Requires operational PCES/AEGIS scope enforcement (Phase 1-2) and completion of the Privileged MIMIR Channel security architecture review. The security review must precede Phase 5-6 development start.

**Open governance question:** Whether the Gap Indicator itself — communicating that relevant inaccessible evidence exists — is acceptable across all matter contexts requires an AI Governance Committee policy determination before threshold configuration is finalized. See Open Items GC-2.

-----

## Problem 3: Query-Layer Access Control

### Problem Statement

Current THEMIS access control governs evidence — what evidence enters the RAG pipeline for a given analyst session. It does not govern the query itself.

Queries are not neutral. A query such as *“what evidence supports our client’s alibi for March 5th?”* reveals legal strategy, theory of the case, and the existence of a specific matter concern. A query such as *“what does the confidential settlement agreement say about the non-disclosure scope?”* reveals the existence of a specific settlement and its subject matter. These are attorney work product and should be governed accordingly.

Three specific problems exist:

**3a. Queries are not treated as privileged work product.** Queries are logged for ERAS/LOGOS and behavioral telemetry but are not classified as attorney work product. They should be, from the first analyst interaction.

**3b. Cross-matter contamination.** An analyst working on two concurrent matters may ask a query in Matter A’s session that encodes strategic framing from Matter B. Evidence scope enforcement prevents Matter B evidence from entering Matter A synthesis. The query itself can still leak Matter B’s strategic framing into a session record associated with Matter A.

**3c. Query-type authorization gaps.** Certain query types — those implicating settlement exposure, opposing counsel strategy, or firm liability assessment — reveal strategic thinking at a level that should require supervisory authorization before processing. Not all analysts should be able to submit these queries without oversight.

### Enhancement 3a: Query Privilege Classification

**PGS/NOMOS enhancement:**  
PGS/NOMOS adds query privilege classification as a policy default. Every query submitted to ATHENA is classified as attorney work product at ingestion, before any processing occurs. This is a policy default applied to all interaction classes. The AI Governance Committee may override this default for specific interaction classes where work product protection does not apply (e.g., administrative system queries, configuration queries).

**MOIRAI enhancement:**  
The MOIRAI event schema for query records gains a `query_privilege` field:

```
QueryRecord.query_privilege {
  classification:  WORK_PRODUCT | NOT_PRIVILEGED
  matter_id:       UUID
  classified_at:   Timestamp
  policy_version:  string     — PGS/NOMOS policy version applied
}
```

Query records classified as WORK_PRODUCT are subject to the same matter scope access control as evidence objects — they cannot be accessed cross-matter, and access by non-matter personnel requires supervisory authorization.

**Phase:** Phase 1-2 alongside PGS/NOMOS. Queries must be treated as privileged work product from the first analyst interaction. This is the highest-priority enhancement in this addendum and must not wait for later phases.

### Enhancement 3b: Cross-Matter Contamination Detection

**PCES/AEGIS enhancement:**  
PCES/AEGIS gains a cross-matter contamination detection function. At query ingestion, PCES:

1. Identifies the matter scope of the current session (already present)
1. Identifies all other matters the analyst has active access to
1. Checks whether the query content references named entities — party names, case numbers, matter-specific terminology — associated with those other matters
1. If a cross-matter reference is detected above threshold, emits a `CrossMatterContaminationSignal`

Entity detection uses a **matter entity registry** maintained by PCES/AEGIS: a structured list of named entities (parties, case references, key terms) associated with each matter in the analyst’s access scope. The registry is populated at matter onboarding and updated as matter participants change.

**IAS/SCUDO enhancement:**  
IAS/SCUDO adds a **privilege contamination classifier** as a second detection path alongside the existing adversarial injection classifiers. Both run at query ingestion. The privilege contamination classifier uses embedding similarity against a matter-specific entity corpus — distinct from the adversarial injection taxonomy — to identify queries that may encode privileged information from a matter other than the current session scope.

Two distinct detection paths within IAS, both running at query ingestion, both contributing to the CrossMatterContaminationSignal.

**On detection:**  
The CrossMatterContaminationSignal is advisory by default. ATHENA surfaces: *“This query may reference information from another matter. The query has been logged and your supervisor has been notified.”* The query proceeds but the contamination signal is recorded as a signed MOIRAI event.

For high-sensitivity matter classes, PGS/NOMOS can configure the signal as blocking, requiring explicit analyst confirmation before the query is processed.

**Phase:** Phase 3-4. Requires the matter entity registry build and the privilege contamination classifier training. Both require matter history data not available until Phase 1-2 operations have generated sufficient matter records.

### Enhancement 3c: Query-Type Authorization

**PGS/NOMOS enhancement:**  
PGS/NOMOS adds a query-type authorization policy dimension. Specific query types are classified in the policy rule library with an authorization requirement. A lightweight classifier at the PGS/NOMOS policy evaluation layer — running after IAS/SCUDO screening — assigns a query_type label that PGS evaluates against the authorization policy for the current interaction class.

**Default query-type authorization taxonomy:**

|Query Type                                  |Default Authorization Requirement     |
|--------------------------------------------|--------------------------------------|
|Settlement exposure or demand assessment    |Supervisory acknowledgment            |
|Opposing counsel strategy or likely position|Supervisory acknowledgment            |
|Firm internal liability exposure assessment |Supervisory acknowledgment            |
|Privilege waiver risk assessment            |Supervisory acknowledgment            |
|Witness credibility or impeachment strategy |Standard — no additional authorization|
|Legal research and case law                 |Standard — no additional authorization|
|Evidence characterization and summary       |Standard — no additional authorization|
|Document comparison and version analysis    |Standard — no additional authorization|

The default taxonomy is a starting point. The appropriate query-type list for this firm’s practice areas and risk tolerance requires AI Governance Committee determination before deployment.

**When supervisory authorization is triggered:**  
ATHENA surfaces a hold notice to the analyst: *“This query type requires supervisor acknowledgment before processing. Your supervising attorney has been notified.”*

The query is queued. The supervisor receives a notification containing the query text — not yet processed, not submitted to the model. The supervisor approves or rejects. Approval triggers normal query processing. Rejection returns a notice to the analyst with the option to rephrase.

All authorization events — approvals, rejections, and holds — are logged to MOIRAI as signed events.

**Phase:** Phase 3-4 alongside the query classifier build. Query privilege classification (3a) must be in place before query-type authorization is layered on top.

-----

## Implementation Sequencing

|Enhancement                                   |Depends on                                   |Recommended Phase|
|----------------------------------------------|---------------------------------------------|-----------------|
|Query privilege classification — 3a           |PGS/NOMOS operational                        |Phase 1-2        |
|AccessRevocationEvent schema + PCES production|PCES/AEGIS operational                       |Phase 3-4        |
|MOIRAI reverse DAG traversal                  |MOIRAI provenance graph operational          |Phase 3-4        |
|DPS/CODEX revocation handler                  |DPS document lifecycle operational           |Phase 3-4        |
|Cross-matter contamination detection — 3b     |PCES entity registry + IAS classifier        |Phase 3-4        |
|Query-type authorization — 3c                 |PGS Phase 1-2 + classifier build             |Phase 3-4        |
|Privileged MIMIR Channel security review      |Architecture review + GC input               |Pre Phase 5-6    |
|RQS gap coverage metric — Problem 2           |RQS operational + privileged channel approved|Phase 5-6        |
|ATHENA gap indicator and escalation path      |RQS gap coverage operational                 |Phase 5-6        |
|ERAS/LOGOS revocation handler                 |ERAS operational                             |Phase 7-8        |

-----

## Open Items Requiring General Counsel Input

Three design decisions in this addendum have professional responsibility implications that require General Counsel input before the relevant phase begins.

**GC-1 — Derived information remediation procedure (Problem 1, Phase 3-4):**  
THEMIS can enumerate the blast radius of a derived information event and enforce review requirements before export. It cannot undo synthesis that was read prior to revocation. In inadvertent production clawback scenarios under FRCP 26(b)(5)(B), the firm’s obligations — whether to file corrections, notify opposing counsel, notify the court, or take other remedial steps — must be defined as a firm procedure before Phase 3-4 implementation. THEMIS will implement whatever workflow General Counsel specifies; the workflow itself is a professional judgment the platform does not make.

**GC-2 — Gap indicator disclosure acceptability (Problem 2, pre Phase 5-6):**  
The Gap Indicator communicates to the analyst that relevant inaccessible evidence exists without revealing its content. In some matter contexts, even the existence of certain evidence may be sensitive information. General Counsel must determine whether the gap indicator is appropriate in all matter classes, or whether it should be configurable as disabled for specific high-sensitivity matter types. This determination must occur before RQS gap coverage threshold configuration is finalized.

**GC-3 — Query-type authorization taxonomy (Problem 3, pre Phase 3-4):**  
The default query-type authorization table in Enhancement 3c is a starting point based on general professional responsibility principles. The appropriate taxonomy for this firm’s specific practice areas, risk profile, and supervision structure requires General Counsel determination in collaboration with supervising partners. This must be finalized before the query-type classifier is trained.

-----

*THEMIS Platform · Addendum B · Access Control Architecture*  
*Companion documents: THEMIS Platform Design v1.0 · Addendum A: New Services*
