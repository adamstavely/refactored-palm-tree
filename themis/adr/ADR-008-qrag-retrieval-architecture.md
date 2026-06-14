# ADR-008 — Q-RAG Multi-Step Retrieval for Legal Research

**Status:** Accepted
**Date:** 2026-05-05
**Deciders:** Platform Architect, ML Engineering Lead

## Context

Legal research questions rarely have single-step answers. "What is the standard for personal jurisdiction given our client's contacts?" decomposes into multiple sub-questions requiring sequential retrieval. The standard RAG approach — single embedding query → retrieve top-k chunks → generate — retrieves content that addresses one facet of the question and leaves others without supporting evidence. This produces confident-sounding but incomplete answers that RQS cannot distinguish from correct answers without ground-truth signals.

Q-RAG (Kirchenbauer et al., 2025, ICLR 2026 oral acceptance) proposes training only the retrieval embedder using reinforcement learning (temporal difference learning), keeping the LLM frozen. The embedder learns to make sequential retrieval decisions — each step informed by what was retrieved in prior steps — enabling multi-hop reasoning over long-context corpora (tested up to 10M tokens).

## Decision

**Adopt Q-RAG as the retrieval architecture for STOA (Legal Research Orchestration) and as the underlying mechanism for RQS retrieval quality measurement.** This decision has two implications: (1) the retrieval embedder will be trained using RL on the FGTS ground truth corpus rather than relying on an off-the-shelf embedding model, and (2) the MOIRAI provenance schema is extended with a RetrievalTrajectory node to capture multi-step retrieval paths.

Key considerations:
- Q-RAG's published benchmarks are on open-domain QA and synthetic long-context tasks. Legal evidence retrieval has different characteristics (specialized citation patterns, privilege constraints, temporal validity). Domain adaptation via FGTS corpus is required before production deployment.
- The embedder-only training approach is significantly cheaper than full LLM fine-tuning and aligns with the FGTS pipeline's existing infrastructure.
- 10M token context handling makes large evidence productions tractable in ways that standard RAG cannot match.

## Alternatives Considered

### Option A — Standard single-step RAG
Query → embed → retrieve top-k → generate. Simple; well-understood; all current RQS measurement infrastructure applies directly. However: consistently fails on multi-hop legal questions; retrieval miss rate is highest for questions that require reasoning across multiple sources. FGTS correction patterns already show systematic correction of answers that would have been correct with better retrieval. Rejected: does not solve the core quality problem.

### Option B — Iterative RAG with LLM-generated sub-queries
LLM generates sub-questions; each sub-question is a separate RAG call; LLM synthesizes the results. Does not require embedder training. However: relies on the LLM to decompose questions correctly (LLMs are inconsistent at this); each sub-question is still single-step; no learned value function means the retrieval stops when the LLM says to stop rather than when evidence is actually sufficient. More expensive per query (multiple LLM calls). Retained as a fallback if Q-RAG embedder training is delayed; not the primary approach.

### Option C — Commercial multi-hop retrieval (Perplexity, Exa, etc.)
Outsource retrieval to a specialized provider. No embedder training required. However: commercial providers cannot access private matter content; the retrieval corpus is the firm's own privilege-sensitive evidence; all retrieval must happen on firm-controlled infrastructure. Rejected: data governance requirement.

## Consequences

**Positive:**
- Multi-step retrieval dramatically improves answer completeness for complex legal research questions
- Embedder training on FGTS corpus produces domain-specific retrieval tuned to the firm's practice mix
- RetrievalTrajectory provenance enables STOA to document the full research methodology, not just the final retrieved set
- 10M token context handling enables evidence review at production scale

**Negative:**
- Embedder training requires the FGTS corpus to reach sufficient size (estimated: Phase 6, ~18 months into platform operation) before training is viable. Standard embedding model is used until then.
- MOIRAI schema requires the RetrievalTrajectory node from Phase 2 (the schema must be designed early even if Q-RAG is not yet in use)
- RQS precision measurement must be extended for multi-step retrieval: terminal precision and trajectory efficiency are both required metrics

**Risks:**
- Domain adaptation gap: Q-RAG's benchmark performance on open-domain QA does not directly predict performance on legal evidence retrieval. Mitigation: internal evaluation against FGTS ground truth corpus before production deployment; staged rollout starting with STOA before extending to all retrieval.

## Related Decisions
- ADR-004 — RetrievalTrajectory is an extension of the turn-level DAG model
