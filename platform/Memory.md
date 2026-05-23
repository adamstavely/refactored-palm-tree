# THEMIS Platform — Addendum C: Memory Architecture

**Document series:** THEMIS Platform Design · Addendum C  
**Companion to:** Platform Design v1.0 · Addendum A: New Services · Addendum B: Access Control  
**Status:** Draft for review  
**Introduces:** MOS/SAGA — Memory Orchestration Service · MEMORY source type badge

-----

## Overview

This addendum defines MOS/SAGA — the Memory Orchestration Service — and introduces the MEMORY source type badge as a new first-class ATHENA signal. Memory in THEMIS is not a single concept. It has five distinct types with different operational characteristics, and a single architectural principle that governs all of them.

### The Governing Principle

> Memory must be access-controlled at the same level as the evidence it was derived from. Memory derived from HC_AEO evidence is itself HC_AEO. Memory is not a bypass mechanism for the protective order tier model. Memory artifacts appear in the `AccessRevocationEvent` blast radius when source evidence access is revoked. These constraints are architecturally enforced — not reliant on documentation or convention.

### Memory Type Taxonomy

|Type                    |Description                                                                                         |Owner            |Phase    |
|------------------------|----------------------------------------------------------------------------------------------------|-----------------|---------|
|Session working memory  |Active context window. Already handled by inference architecture.                                   |Inference Gateway|Live     |
|Cross-session continuity|Session summaries enabling analysts to resume work without re-establishing context.                 |MOS/SAGA         |Phase 3-4|
|Matter knowledge layer  |Graduated confirmed claims forming a matter-specific knowledge corpus distinct from raw evidence.   |MOS/SAGA         |Phase 5-6|
|Agent task memory       |Persistent state for multi-step STOA and PYTHIA tasks. Redis during execution, MOIRAI at milestones.|MOS/SAGA         |Year 2   |
|Calibration memory      |TCS/MIMIR analyst accuracy patterns and FGTS correction corpus. Already operational.                |TCS/MIMIR        |Live     |

-----

## MOS/SAGA — Memory Orchestration Service

SAGA in Norse mythology is the Ásynja associated with seeing, recounting, and preserving history — keeper of the accumulated record of all that has passed. The service name follows the platform’s Norse naming convention and reflects the function: MOS/SAGA holds the firm’s accumulated analytical memory across the lifecycle of every matter it handles.

### Architectural Position

MOS/SAGA produces artifacts to MOIRAI. It does not own storage. Every session summary, matter knowledge entry, and task milestone is a signed MOIRAI event subject to the same cryptographic attestation, access control enforcement, and blast radius traversal as all other THEMIS events.

**MOS/SAGA owns the logic of what memory means** — when to create it, when to invalidate it, how to surface it, and what threshold a claim must meet to graduate into the matter knowledge layer. **MOIRAI owns the record.**

### What MOS/SAGA Owns

- Session summary lifecycle — generation, versioning, storage as signed MOIRAI artifacts, retrieval with PCES tier enforcement
- Matter knowledge layer — graduation logic, conflict resolution, MIMIR indexing as a distinct retrieval source
- Agent task memory — Redis task state for active tasks, MOIRAI milestone promotion, task handoff access validation
- MEMORY source type metadata — provenance attribution for MEMORY-sourced claims surfaced in ATHENA

### What MOS/SAGA Does Not Own

- **Storage** — MOIRAI holds all memory artifacts as signed events in the provenance graph
- **Access control enforcement** — PCES/AEGIS enforces tier access on memory retrieval as it does for evidence retrieval
- **Confidence scoring** — TCS/MIMIR owns calibration for MEMORY-sourced claims using MEMORY as a new source type dimension
- **Blast radius enumeration** — MOIRAI’s existing reverse DAG traversal reaches MOS/SAGA artifacts through derivation edges MOS/SAGA writes at creation time

### Integration Map

|Service          |Relationship to MOS/SAGA                                                                                                   |
|-----------------|---------------------------------------------------------------------------------------------------------------------------|
|Provenance/MOIRAI|Receives all memory artifacts as signed events. Blast radius traversal reaches MOS/SAGA artifacts through derivation edges.|
|PCES/AEGIS       |Enforces tier access on all memory retrieval requests. Memory inherits the most restrictive tier of contributing evidence. |
|FGTS/ALETHEIA    |Provides verified correction signals that drive matter knowledge graduation decisions.                                     |
|ERAS/LOGOS       |Provides reasoning records and session analytics that inform session summary generation.                                   |
|TCS/MIMIR        |Calibrates confidence for MEMORY-sourced claims as a new source type dimension.                                            |
|RQS/HERMES       |Indexes matter knowledge layer entries in MIMIR as a distinct retrieval source with its own quality scoring.               |
|STOA / PYTHIA    |Consume MOS/SAGA for agent task memory state management during multi-step research orchestration.                          |
|ATHENA           |Surfaces session summaries in the context loading UI. Displays MEMORY source type badge with sub-type and provenance link. |

-----

## Session Summary Lifecycle

Session continuity is the immediate operational gap. An analyst working on a complex matter across 40 sessions over three months currently re-establishes context manually at the start of each session. Session summaries solve this without introducing opacity — the analyst explicitly loads prior context and sees exactly what source badge each loaded claim carries.

### Generation

Session summaries are generated at session close or on explicit analyst command. They are not narrative transcripts — they are structured artifacts with discrete fields that can be retrieved selectively.

Free-form narrative summaries are easy to generate but impossible to retrieve precisely. A structured summary with discrete fields is harder to generate but far more useful: the analyst can choose to load only the verified claims from session 12, or only the open questions from sessions 8 through 14, without loading the entire session history into context.

### Session Summary Schema

```typescript
interface SessionSummary {
  summary_id:     UUID;
  session_id:     UUID;           // Source session in MOIRAI
  matter_id:      UUID;
  analyst_id:     AnalystID;
  generated_at:   Timestamp;
  session_date:   Timestamp;
  version:        number;         // Summaries can be revised

  // Structured fields — not narrative
  verified_claims:    VerifiedClaim[];   // From FGTS outcomes this session
  open_questions:     string[];          // Explicitly unresolved
  evidence_reviewed:  UUID[];            // Evidence IDs accessed
  key_findings:       Finding[];         // Analyst-confirmed conclusions
  next_steps:         string[];          // Explicit continuation notes

  // Access control — inherits most restrictive tier of session evidence
  effective_tier:     ProtectiveOrderTier;
  contributing_evidence_tiers: Record<UUID, ProtectiveOrderTier>;

  // Provenance
  signed:           true;
  prev_event_hash:  SHA256;
  moirai_node_id:   string;
}
```

### Explicit Loading — Not Automatic Context Injection

Session summary loading is analyst-controlled. Automatic injection of prior session context without analyst awareness introduces opacity that conflicts with ATHENA’s trust calibration philosophy. When an analyst opens a new session on a matter, ATHENA surfaces a “Prior Sessions” panel showing available summaries scoped to their current access tier. The analyst chooses what to load. All claims from a loaded session summary carry the MEMORY·SES badge with the session date and a provenance link to the originating MOIRAI session node.

### Versioning

A session summary may be revised if new verification outcomes arrive after generation — for example, if a supervisory review later updates the confidence weight of a claim that appeared in the summary. MOS/SAGA maintains a version chain per session stored as linked MOIRAI events. ATHENA shows the most recent version by default and provides access to version history.

-----

## Matter Knowledge Layer

The matter knowledge layer is a matter-specific corpus of graduated, confirmed claims — distinct from the raw evidence corpus and surfaced as a separate retrieval source in RQS/HERMES.

Where raw evidence retrieval answers “what do the documents say”, the matter knowledge layer answers “what has this matter team established”. The two retrieval sources are distinct and labelled differently in ATHENA — claims from the matter knowledge layer carry the MEMORY·MKL badge, not GRND.

### Graduation Logic

Claims graduate from the FGTS correction corpus into the matter knowledge layer when they meet a configurable confirmation threshold. Graduation is a promotion event, not a copy — the graduated claim retains its full provenance chain back through FGTS to the original session and evidence.

**Default graduation threshold:**

|Condition                                   |Required?                                                                |
|--------------------------------------------|-------------------------------------------------------------------------|
|Supervisory acknowledgment                  |**Mandatory** — no graduation without it                                 |
|Domain RAI above threshold                  |**Mandatory** — analyst must be calibrated in this domain                |
|High FGTS correction weight score           |**Mandatory** — low-weighted corrections cannot graduate                 |
|No active dispute flag                      |**Mandatory** — disputed claims route to supervisor, not graduation queue|
|Peer agreement (second analyst confirmation)|**Recommended** — substantially increases graduation confidence          |

The threshold is configurable by matter class. AI Governance Committee approval is required for threshold changes. The right threshold is not a technical question — see Design Questions below.

### Conflict Resolution

When two analysts produce graduated claims that contradict each other about the same evidence or fact, MOS/SAGA surfaces the conflict to the matter supervisor rather than resolving it automatically. Automatic conflict resolution would silently lose information. The supervisor reviews both claims with their full provenance chains and either accepts one, accepts both with a reconciliation note, or escalates.

Conflict events are logged to MOIRAI as signed events. Both conflicting claims remain visible to the matter team as disputed facts during supervisor review.

### MIMIR Indexing

Graduated matter knowledge entries are indexed in MIMIR as a named retrieval source distinct from the raw evidence corpus. In ATHENA, analysts can specify whether their query should draw from evidence only, matter knowledge only, or both. Results from the matter knowledge source carry the MEMORY·MKL badge. The RQS/HERMES quality score for matter knowledge retrieval is tracked separately from evidence retrieval quality.

-----

## Agent Task Memory

Multi-step agentic tasks — STOA research orchestration, PYTHIA proactive analysis — require persistent state. They unfold over time, across potentially many API calls. Without persistent task memory, an interrupted task cannot resume; the entire research process restarts. With it, a task can be paused, handed off to another analyst, and resumed without loss of progress.

### Redis During Execution, MOIRAI at Milestones

Active task state lives in Redis — fast reads and writes, task progress visible in real-time to the analyst. At meaningful milestones — sub-task completed, research phase concluded, task paused, task completed — MOS/SAGA promotes a snapshot to MOIRAI as a signed task milestone event.

The Redis state is the operational surface. MOIRAI is the audit record. The two substrates serve different purposes and should not be conflated.

### Task State Schema

```typescript
interface TaskState {
  task_id:             UUID;
  task_type:           'stoa_research' | 'pythia_analysis' | 'custom';
  matter_id:           UUID;
  initiated_by:        AnalystID;
  task_objective:      string;
  status:              'active' | 'paused' | 'completed' | 'failed' | 'handed_off';

  sub_tasks: SubTask[] {
    sub_task_id:     UUID;
    description:     string;
    status:          'pending' | 'in_progress' | 'completed' | 'skipped';
    finding:         string | null;
    sources_used:    UUID[];        // Evidence IDs consulted
    completed_at:    Timestamp | null;
  };

  open_questions:          string[];
  intermediate_findings:   Finding[];

  // Access control — validated on every resume
  access_credential_snapshot: AccessScope;
  credential_valid_until:     Timestamp;
}
```

### Task Handoff Access Validation

When a task is handed off to a different analyst, MOS/SAGA validates that the receiving analyst has access to all evidence referenced in the task state — including evidence retrieved during completed sub-tasks. If any referenced evidence is outside the receiving analyst’s access scope, the handoff is blocked. The task can proceed with the receiving analyst only after either the evidence scope is resolved or the restricted sub-task findings are explicitly excluded from the handoff state.

### Credential Re-Validation on Every Resume

Access credentials attached to a task state are validated on every resume event, not only at initiation. If an analyst’s access scope changes between task initiation and resume — tier revocation, matter conflict discovered — the task is suspended, not silently continued with stale credentials. The task cannot resume until access is re-validated or the task scope is adjusted.

-----

## MEMORY Source Type Badge

### Why a New Badge Type is Necessary

A claim that enters the current session from a prior session summary or the matter knowledge layer should not carry a GRND badge even if the original claim was GRND. It passed through a derivation step — summarization, graduation — with its own error rate distinct from the original evidence retrieval.

This is precisely the same reasoning that motivated the TRANSCRIPT badge. A transcript is not a retrieved source document — it is a derivative artifact with its own ASR accuracy profile. A session summary is not a retrieved evidence claim — it is a derivative artifact with its own summarization error profile. The MEMORY badge makes this derivation visible at the claim level, exactly as the TRANSCRIPT badge does.

The appropriate verification action for a MEMORY-sourced claim is different from the action for a GRND claim. The confidence ceiling is different. The FGTS routing is different. The analyst needs to know which type of claim they are working with.

### Three Sub-Types

|Badge       |Sub-type        |Source                                               |Confidence Ceiling                                                                                      |
|------------|----------------|-----------------------------------------------------|--------------------------------------------------------------------------------------------------------|
|`MEMORY·SES`|Session memory  |Prior session summary loaded by analyst              |Moderate reduction — summarization error rate. Shows originating session date.                          |
|`MEMORY·MKL`|Matter knowledge|Graduated confirmed claim from matter knowledge layer|Small reduction — multiple confirmations required for graduation. Strongest memory type.                |
|`MEMORY·TSK`|Task finding    |Intermediate finding from STOA or PYTHIA agent task  |Substantial reduction — agent intermediate findings have higher error rate than analyst-verified claims.|

### Confidence Ceilings

TCS/MIMIR maintains a separate calibration stream for MEMORY-sourced claims by sub-type. An analyst’s accuracy on GRND claims does not predict their accuracy on MEMORY claims — the error modes differ fundamentally.

GRND errors are typically hallucination or synthesis failures at the model layer. MEMORY errors are systematic distortions at the derivation layer: summarization compression, selective retention of the more salient claims, and propagation of prior misclassifications from session to session.

|Sub-type  |Ceiling vs. GRND     |Primary error mode                                                 |Ceiling driver                                |
|----------|---------------------|-------------------------------------------------------------------|----------------------------------------------|
|MEMORY·SES|Moderate reduction   |Summarization compression, selective retention                     |Session summary quality score from MOS/SAGA   |
|MEMORY·MKL|Small reduction      |Graduation threshold errors, conflict resolution artifacts         |Confirmation weight of the graduated claim    |
|MEMORY·TSK|Substantial reduction|Agent intermediate reasoning errors, incomplete sub-task completion|Sub-task completion confidence from task state|

### Verification Action Sets

**MEMORY·SES — Session Summary**

- **Confirmed in original session** — analyst reviews the original MOIRAI session record and confirms the claim. Routes to FGTS as a session memory confirmation signal. Strongest verification action for session-sourced memory.
- **Not in original session** — analyst cannot locate the originating claim in the source session. Routes to MOS/SAGA as a summary accuracy failure — the summary introduced content not present in the session.
- **Contradicts current evidence** — the memory-sourced claim conflicts with what current evidence retrieval shows. High-value FGTS signal: a prior confirmed claim is no longer supported by current evidence, indicating either evidence has changed (TVS/KAIROS concern) or the original claim was incorrect.
- **Dismiss** — logged but not incorporated into calibration.

**MEMORY·MKL — Matter Knowledge**

- **Confirmed as established fact** — analyst confirms the graduated claim holds. Increases the confirmation weight of the matter knowledge entry.
- **No longer accurate** — evidence has changed since graduation. Routes to MOS/SAGA for matter knowledge entry invalidation and to TVS/KAIROS for validity investigation.
- **Disputed — requires supervisor review** — analyst has a substantive basis for disputing the established fact. Routes to the matter supervisor with dispute details. Matter knowledge entry marked as disputed pending resolution. Both claims remain visible as disputed facts.
- **Dismiss** — logged.

**MEMORY·TSK — Agent Task Finding**

- **Confirmed against current evidence** — analyst independently verifies the task finding against retrieved evidence. Upgrades the finding from MEMORY·TSK to a FGTS-eligible correction.
- **Task finding was incorrect** — agent intermediate reasoning error. Routes to FGTS as an agent calibration signal and to MOS/SAGA for task state annotation.
- **Dismiss** — logged.

### FGTS Routing for MEMORY-Sourced Corrections

Corrections on MEMORY-sourced claims route differently from corrections on GRND or PARAM claims because the failure mode is at the memory layer rather than the model layer.

|Verification Action          |FGTS Route                                                         |MOS/SAGA Route                                    |
|-----------------------------|-------------------------------------------------------------------|--------------------------------------------------|
|Confirmed in original session|Session memory confirmation signal — informs MEMORY·SES calibration|None                                              |
|Not in original session      |Not routed to FGTS (not a model error)                             |Summary accuracy failure — triggers summary review|
|Contradicts current evidence |High-value calibration signal — prior confirmed claim invalidated  |Summary invalidation flag                         |
|Disputed (MKL)               |Not routed until supervisor resolves                               |Matter knowledge dispute — supervisor review queue|
|No longer accurate (MKL)     |Validity change signal — connects to TVS/KAIROS                    |Matter knowledge invalidation                     |
|Task finding incorrect       |Agent calibration signal for STOA/PYTHIA                           |Task state annotation                             |

-----

## Access Control Integration

### Tier Inheritance

Every memory artifact inherits the most restrictive protective order tier of any evidence contributing to it. A session summary that includes analysis of HC_AEO evidence is itself HC_AEO. PCES/AEGIS enforces this at retrieval time — not at the application layer.

MOS/SAGA computes and stores `effective_tier` for every session summary and matter knowledge entry at creation time by querying PCES/AEGIS for the tier of each contributing evidence object. If contributing evidence tier changes after creation — court order modification, tier re-classification — MOS/SAGA receives an `AccessRevocationEvent` and must recompute `effective_tier` for affected memory artifacts.

### Blast Radius Connection — No New Logic Required

When PCES/AEGIS revokes access to evidence E and MOIRAI performs the reverse DAG traversal, the traversal reaches memory artifacts through derivation edges MOS/SAGA writes at creation time:

```
session_summary  derived_from  session
session          retrieved     evidence_id

matter_knowledge_entry  derived_from  correction_event
correction_event        derived_from  session
session                 retrieved     evidence_id
```

The blast radius therefore includes session summaries and matter knowledge entries derived from the revoked evidence. These memory artifacts receive the same marking and export gate hardening as documents and reasoning records in the blast radius. No new blast radius logic is required — MOS/SAGA artifacts flow through the existing Problem 1 machinery automatically.

### Task Credential Validation

Active task execution credentials are validated on every resume, not only at initiation. If an analyst’s access scope changes while a task is running, the task is suspended and a supervisor notification is sent. The task cannot resume until access is re-validated or the task scope is adjusted to exclude evidence the analyst can no longer access.

-----

## Implementation Sequencing

|Capability                                   |Depends on                                        |Phase    |
|---------------------------------------------|--------------------------------------------------|---------|
|Session summary schema design                |MOIRAI Phase 3-4 — concurrent build required      |Phase 3-4|
|Session summary generation + MOIRAI storage  |MOIRAI provenance graph operational               |Phase 3-4|
|Session summary retrieval + ATHENA UI        |Session summaries in MOIRAI                       |Phase 3-4|
|MEMORY·SES badge + TCS calibration stream    |Session summary retrieval live                    |Phase 3-4|
|Matter knowledge graduation logic            |FGTS 5-factor correction weighting operational    |Phase 5-6|
|Matter knowledge MIMIR indexing + retrieval  |Graduation logic live                             |Phase 5-6|
|MEMORY·MKL badge + TCS calibration stream    |Matter knowledge retrieval live                   |Phase 5-6|
|Agent task memory (Redis + MOIRAI milestones)|STOA / PYTHIA intelligence layer services deployed|Year 2   |
|MEMORY·TSK badge + TCS calibration stream    |Agent task memory operational                     |Year 2   |

**Phase 3-4 co-design constraint:** The session summary schema must be designed concurrently with MOIRAI Phase 3-4, not after. Summary storage depends on MOIRAI node type design. A session summary schema designed after MOIRAI is built may require MOIRAI node type changes that are expensive to retrofit.

-----

## Open Design Questions

### Design Question 1 — Matter Knowledge Graduation Threshold

The graduation threshold table above shows default conditions. The right threshold is not a technical question — it is a professional judgment about how much confirmation is sufficient before a claim is elevated to established institutional fact.

Too low: unverified claims pollute the matter knowledge corpus and receive unwarranted credibility as “established facts”. Too high: nothing ever graduates and the matter knowledge layer provides no practical benefit.

The threshold should be configurable by matter class — criminal litigation may require supervisory confirmation as a hard requirement with peer agreement also mandatory; a routine transactional matter might have a lower bar. This requires AI Governance Committee determination before Phase 5-6 implementation.

### Design Question 2 — Session Summary Schema

The session summary schema above is a starting point. The actual schema must be co-designed with the ATHENA interface team and with practicing attorneys who will load and use session summaries in their actual workflow.

The schema needs to be rich enough to be useful — not just a list of evidence IDs — but structured enough to be precisely retrievable — not a narrative blob that the model re-interprets differently on each retrieval. The co-design work should begin in Phase 3-4 alongside MOIRAI, not after MOIRAI is built.

-----

*THEMIS Platform · Addendum C · Memory Architecture*  
*Companion documents: Platform Design v1.0 · Addendum A: New Services · Addendum B: Access Control Architecture*
