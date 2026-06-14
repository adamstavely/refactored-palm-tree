# ADR-004 — Session Manifests vs. Turn-Level DAG for AI Interaction Provenance

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect

## Context

Multi-turn AI conversations present a lineage challenge. The naive approach is a flat session manifest — recording model, user, timestamp, and retrieval context at the session level. The question is whether session-level capture is sufficient for the chain-of-custody requirements of legal work product provenance.

## Decision

Use **turn-level provenance records organized as a directed acyclic graph (DAG)** within each session. A session manifest is a container, not the unit of provenance. Each turn record captures exactly which prior turn outputs were in the context window, which retrieval chunks were used for that specific turn, and the exact messages array sent to the model API.

The key insight: in a multi-turn conversation, the model at Turn 5 is conditioned on the outputs of Turns 1–4. Those prior outputs are ancestors of Turn 5's output. A flat session manifest loses this intra-session lineage. The turn-level DAG preserves it.

## Alternatives Considered

### Option A — Flat session manifest
Single record per session capturing: model version, user identity, timestamp, all retrieval chunks across all turns, and a hash of the system prompt. Simple to implement; low storage overhead. However: retrieval context changes turn by turn — a flat record loses which chunks informed which output. Prior AI outputs in the context window are themselves ancestors of later outputs — a flat manifest cannot represent this. Branching (edit-and-regenerate) creates parallel histories that a flat record cannot represent. Rejected: loses the lineage information that matters most for failure attribution.

### Option B — Turn-level records with post-hoc DAG reconstruction
Store turn records as a flat list; reconstruct the DAG at query time by matching assistant message content in each turn's context window snapshot against the turn registry. Lower write complexity; DAG is derived rather than stored. However: reconstruction at query time is expensive for long sessions; exact matching of assistant message content to turn records requires the full content to be stored and compared. Rejected: query performance; storage cost of content duplication for matching.

## Consequences

**Positive:**
- Failure attribution across four root causes (model, retrieval, prompt instruction, analyst intent) is tractable only with turn-level lineage
- Intra-session DAG reveals when a low-quality output at Turn 2 influenced outputs at Turns 4, 6, and 8
- Branching and regeneration create explicit branch_parent edges rather than silent history forks
- Context window truncation is detectable via input_token_count and flagged as weak_ancestry

**Negative:**
- Higher write volume: a 20-turn session produces 20 TurnInitiated + 20 TurnCompleted events rather than 1 session event
- resolve_prior_turns() logic adds complexity: assistant messages in the context window must be matched to prior turn records by content hash

**Risks:**
- The messages_snapshot field (exact context window) grows with conversation length. For very long sessions this may approach storage limits. Mitigation: store a hash of the messages_snapshot alongside the full content; the full content is stored in the append-only event log and can be dropped from the graph node after a defined retention period.

## Related Decisions
- ADR-007 — Prompt Lineage extends the turn-level model with PromptTemplate, AnalystInput, and PromptAssembly nodes
