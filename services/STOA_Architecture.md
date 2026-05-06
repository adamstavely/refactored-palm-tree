# STOA — Legal Research Orchestration
### *"The Forum — where structured legal reasoning is assembled"*
*Part of the THEMIS Intelligence Layer · Build Priority: Year 2, Q1*

---


## Design Philosophy

Where a single RAG call retrieves and synthesizes, STOA orchestrates: query decomposition, iterative refinement, sub-question routing, source triangulation, and synthesis — with a complete research trail showing every step from initial question to final answer.

ERAS captures the reasoning. STOA captures the methodology. The distinction matters when an attorney must explain how they arrived at a legal position.

---

## Core Capabilities

### Query Decomposition
Breaks a legal research question into constituent sub-questions using a structured legal reasoning taxonomy:
- **Threshold question** — does this legal standard apply at all?
- **Factual question** — what does the record establish?
- **Precedential question** — what do courts require?
- **Jurisdiction question** — which law governs?
- **Current-law question** — is this authority still good law?

Each sub-question is routed to the appropriate retrieval strategy.

### Q-RAG Integration
Uses the Q-RAG multi-step retrieval architecture (ICLR 2026 oral) for each sub-question, with full RetrievalTrajectory logging. The research trail captures not just what was found but the embedding steps that led there.

### Source Triangulation
Cross-references retrieved sources across sub-questions to identify:
- Conflicts: two sub-questions retrieve sources in TVS conflict
- Gaps: a sub-question was not adequately addressed by any retrieved source
- Convergences: multiple sub-questions point to the same authoritative source

### Research Trail Documentation
```yaml
ResearchTrail:
  trail_id:             uuid
  turn_id:              uuid
  question:             str
  decomposed_questions: [SubQuestion]
  sources_used:         [{ chunk_id, sub_question_id, validity_at_research: float }]
  conflicts_detected:   [{ chunk_a, chunk_b, conflict_type }]
  synthesis_logic:      str          # how the sub-answers were combined
  validity_snapshot:    ISO8601      # TVS scores frozen at research time
  kcs_checked:          bool         # were retrieved authorities checked for recent developments?
  kcs_alerts:           [str]        # any KCS flags on retrieved authorities
```

### Iterative Refinement
Multi-turn research sessions where each answer refines the next question — with refinement logic logged so the research process is fully reconstructable.

---

## Value Delivered

| Competence Documentation | Research Completeness | Validity-Aware Synthesis |
|---|---|---|
| Auditable research methodology record — not just the answer but the documented process | Multi-step decomposition catches gaps that single-call RAG misses | Every source carries its TVS score at research time, flagging low-validity authorities |

---

## Integration Points

| Service | Role |
|---|---|
| RQS / HERMES | Built on Q-RAG retrieval infrastructure; RetrievalTrajectory records capture multi-step paths |
| PROV / MOIRAI | Research trails are first-class provenance artifacts linked to work product |
| TVS / KAIROS | Validity scores checked at source retrieval; frozen in research trail |
| ERAS / LOGOS | Reasoning captured at each synthesis step |
| KCS / ARGUS | Retrieved authorities checked for recent external developments before synthesis delivery |
| MNEMOSYNE | Research trails feed institutional research methodology knowledge |
| PYTHIA | STOA executes proactive research that PYTHIA identifies as needed |

---

**Build Phase:** Year 2, Q1–Q2 · After RQS Phase 2 and Q-RAG integration
**Depends on:** RQS / HERMES (Q-RAG), PROV / MOIRAI, TVS / KAIROS, ERAS / LOGOS, KCS / ARGUS
**Feeds into:** MNEMOSYNE, PYTHIA, SCRIBE, CLIENT (research trail attestation)
