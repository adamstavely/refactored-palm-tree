# RQS — Retrieval Quality Service
### HERMES · *"Messenger — who finds and delivers information"*
*Part of the THEMIS Platform · Operational Intelligence · Build Priority: 7*

---

## Design Philosophy

Retrieval-augmented generation is the backbone of legal AI work. When retrieval fails silently — surfacing the wrong exhibit, missing a key deposition passage, ranking an outdated filing above a current one — the model answers confidently from incomplete context. No error is raised. The analyst may never know.

> **The Silent Failure Problem:** A hallucinating model is visible — the output is wrong. A retrieval miss is invisible — the model gives a correct-sounding answer based on incomplete evidence, and neither the model nor the analyst knows what was missing.

The RQS makes retrieval measurable: tracking precision over time, identifying systematic miss patterns, flagging stale content entering AI context windows, and detecting embedding drift.

---

## Core Measurement

```yaml
RetrievalQualityRecord:
  query_turn_id:        uuid
  retrieved_chunks:     [chunk_id, ...]
  confirmed_relevant:   [chunk_id, ...]     # analyst-confirmed or eval-set
  confirmed_missed:     [chunk_id, ...]     # analyst-identified missing
  precision:            float               # relevant ∩ retrieved / retrieved
  recall_estimate:      float               # relevant ∩ retrieved / all relevant
  mean_validity_score:  float               # avg TVS score of retrieved chunks
  stale_chunk_count:    int                 # retrieved chunks with TVS score < 0.6
  query_class:          str
  matter_type:          str
```

---

## TVS-Retrieval Intersection

Pre-generation validity check runs for every retrieval:

```python
# For each retrieved chunk:
validity = TVS.get_current_score(chunk_id)
if validity < 0.6:  annotate as LOW_VALIDITY in context
if contested:       annotate as CONTESTED in context

# Aggregate:
if mean_validity < 0.65:
    attach LOW_CONTEXT_VALIDITY warning to turn record
    surface warning to analyst before response
if any_chunk_invalidated:
    BLOCK generation; require analyst acknowledgment
```

> **Why annotation, not always blocking:** Low validity context should warn, not always block. Researching the history of an outdated regulation legitimately requires low-validity content. The RQS annotates and warns; PGS governs whether to block based on interaction class.

---

## Q-RAG Integration (Multi-Step Retrieval)

Q-RAG (Kirchenbauer et al., ICLR 2026 oral) trains only the retrieval embedder using RL, keeping the LLM frozen. This changes the provenance model for retrieval:

```yaml
RetrievalTrajectory:   # New node for Q-RAG multi-step retrieval
  trajectory_id:     uuid
  turn_id:           uuid
  steps: [
    {
      step_index:      int
      query_embedding: vector
      retrieved_chunks: [chunk_id, ...]
      step_score:      float    # value function estimate
      informed_by:     step_index | null
    }
  ]
  terminal_chunks:   [chunk_id, ...]  # final set passed to model
  total_steps:       int
```

**RQS precision measurement for multi-step retrieval** requires both terminal precision (did the final set contain relevant chunks?) and trajectory efficiency (did the path converge or thrash?).

---

## Miss Pattern Taxonomy

| Pattern | Description | Root Cause Indicator |
|---|---|---|
| Category miss | An entire category of relevant documents systematically not retrieved | Embedding model doesn't represent this content type adequately |
| Recency miss | Recent documents consistently rank below older ones | Embedding drift; newer docs indexed with different distribution |
| Granularity miss | Relevant passages in retrieved docs but chunk boundaries too coarse | Chunking strategy needs adjustment |
| Validity blind | Retrieved chunks have low TVS scores but surfaced anyway | Retrieval ranking not incorporating validity signals |
| Cross-matter ghost | Content from another matter bleeds through PCES boundaries | PCES filter gap; retrieve-then-filter latency issue |

---

## Alerting & Remediation

| Condition | Trigger | Remediation |
|---|---|---|
| Precision below threshold | Rolling precision < 0.60 for a query class | Retrieval model review; embedding audit |
| High stale retrieval rate | > 20% retrieved chunks with TVS score < 0.6 | TVS-aware reranking; corpus freshness review |
| Systematic miss detected | Same miss pattern across 5+ independent queries | Chunking strategy review; corpus gap analysis |
| Embedding drift | Known-pair similarity delta > 0.15 | Reindexing recommendation; model version review |
| Cross-matter ghost | PCES-blocked content appearing in retrieval candidates | Urgent: PCES filter review |

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | Every RetrievalQualityRecord attached to corresponding turn record |
| TVS / KAIROS | Current validity scores read for all retrieved chunks; annotations added to context |
| FGTS / ALETHEIA | RetrievalMissSignal events from FGTS are the primary ground truth miss signal |
| HADES | Surface 2 probes (retrieval poisoning) run against RQS layer in SEE |
| KCS / ARGUS | KCS alerts when new external developments suggest retrieval corpus is stale |

---

## Implementation Roadmap

### Phase 1 — Observability (Weeks 39–40)
- RetrievalQualityRecord schema for every turn with retrieval context
- TVS validity score annotation for all retrieved chunks
- LOW_CONTEXT_VALIDITY warning surfaced to analysts
- Basic quality dashboard: per-query-class precision, stale retrieval rate

### Phase 2 — Miss Detection & Drift (Weeks 41–42)
- FGTS RetrievalMissSignal integration
- Baseline establishment: rolling precision baselines per query class and matter type
- Drift detection: embedding similarity monitoring for known-good pairs
- Miss pattern taxonomy: automated classification of miss types

### Phase 3 — Attribution & Remediation (Weeks 43–44)
- Root cause attribution: embedding vs. indexing vs. chunking failure classification
- Underperforming pattern alerts with remediation playbook routing
- RQS corpus health dashboard
- TVS-aware retrieval reranking (requires retrieval layer integration)

---

**Depends on:** PROV / MOIRAI, TVS / KAIROS, FGTS / ALETHEIA
**Feeds into:** Retrieval layer (reranking signals), HADES (Surface 2 probes)
