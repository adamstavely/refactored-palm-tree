# TVS — Temporal Validity Service
### KAIROS · *"Greek for 'the right moment'"*
*Part of the THEMIS Platform · Quality Feedback Loop · Build Priority: 6*

---

## Design Philosophy

The Provenance Service answers: where did this content come from? The TVS answers a different question: **does this content still hold?** These are independent concerns requiring separate services. Provenance is event-driven — records grow only forward. Validity is time-driven and contradiction-driven — it continuously re-evaluates existing records.

> **Two Distinct Invalidation Mechanisms:** Passive decay (time passing reduces confidence that a source still reflects current reality) and active invalidation (new evidence explicitly contradicts an older one) are legally distinct situations. An unchallenged 18-month-old expert report and one directly contradicted last week may both score low under a naive decay model — but they require different attorney responses.

---

## Validity Scoring Model

```yaml
ValidityRecord:
  chunk_id:          sha256           # FK → Provenance Service
  decay_score:       float 0.0–1.0   # passive time-based component
  contested:         bool             # true if open ConflictEvent exists
  invalidated:       bool             # true if hard invalidation resolved
  invalidated_by:    chunk_id | null
  composite_score:   float 0.0–1.0
  score_history:     [(ISO8601, score)]  # append-only; never truncated
  decay_profile:     str
  ingested_at:       ISO8601
  last_scored_at:    ISO8601
  model_cutoff:      ISO8601 | null   # AI outputs only
  retrieval_freshness: float | null   # AI outputs only; frozen at generation
```

**Composite score formula:**
```python
if invalidated:
    composite_score = 0.0
elif contested:
    composite_score = decay_score * 0.5
else:
    composite_score = decay_score

# For AI outputs — compound penalty:
if model_cutoff is not None:
    cutoff_age_penalty = decay_f(now - model_cutoff, profile='ai_knowledge')
    composite_score = composite_score * min(cutoff_age_penalty, retrieval_freshness)
```

---

## Decay Functions by Content Type

| Content Type | Function | Volatility | Notes |
|---|---|---|---|
| statute | Step function | High | Valid until repeal date; drops to 0.0 on repeal event |
| case_law | Step function | High | Valid until overturned; overturning is an active invalidation event |
| expert_report | Exponential | Medium | Half-life configurable per practice area (default: 18 months) |
| financial_statement | Linear + step | High | Linear decay within fiscal year; drops at fiscal year close |
| deposition | Very low decay | Low | Testimony doesn't expire; recantation = active invalidation |
| news/media | Aggressive exponential | High | Contextual background only; not authoritative source |
| ai_generated | Model vintage | Medium | Three independent temporal bounds (see below) |
| regulatory | Step + slow decay | Medium | Valid until superseded; interpretation drifts with agency guidance |

**Decay profile configuration:**
```yaml
decay_profiles:
  expert_report:
    function:       exponential
    half_life_days: 548          # ~18 months
    floor:          0.15         # never fully expires while case is live

  statute:
    function:       step
    default_score:  0.97
    on_repeal:      0.0

  ai_generated:
    function:       model_vintage
    cutoff_source:  turn.model_cutoff
    compound_penalty: true
```

---

## The AI-Specific Temporal Problem

AI outputs have three independent temporal bounds that compound multiplicatively:

| Bound | Source | Frozen? |
|---|---|---|
| Model training cutoff | Model provider training data cutoff date | Yes — at generation time |
| Retrieval context freshness | Min TVS score of chunks at generation time | Yes — frozen at generation |
| Output age | Time since generation | No — decays continuously |

```python
# Example: AI output generated 2024-06-01
# Model cutoff: 2024-01-01 (5 months before generation)
# Retrieval context: exhibit ingested 2022-09-01 (21 months at gen time)
# Evaluation date: 2026-04-01 (22 months since generation)

cutoff_penalty      = decay_f(days=120, profile='ai_knowledge')   # → 0.88
retrieval_freshness = decay_f(days=630, profile='expert_report')  # → 0.51 (frozen)
output_age_score    = decay_f(days=670, profile='ai_generated')   # → 0.73

composite = 0.88 × 0.51 × 0.73 = 0.33  ← LOW VALIDITY
```

> **Display principle:** Never collapse the three AI temporal bounds into a single score. An attorney needs to know which dimension is driving the low score — the remediation differs for each.

---

## DAG Propagation Engine

Score changes propagate forward through the provenance DAG when a source chunk's score changes.

```python
def propagate(chunk_id, new_score):
    downstream = provenance_graph.descendants(chunk_id)
    for node in topological_sort(downstream):
        parents = provenance_graph.parents(node)
        inherited_scores = []
        for parent in parents:
            hop_weight = 1.0 - (edge.hop_distance × 0.07)
            inherited = validity_index[parent].composite_score × hop_weight
            inherited_scores.append(inherited)
        
        node_inherited = min(inherited_scores)      # weakest ancestor constrains
        node_own = validity_index[node].decay_score
        new_composite = min(node_own, max(node_inherited, FLOOR))
        
        if abs(new_composite - validity_index[node].composite_score) > 0.01:
            validity_index.update(node, new_composite, trigger='propagation')
```

**Propagation triggers:**

| Trigger | Behavior |
|---|---|
| Scheduled decay run | Batch recalculation; propagate only on delta > threshold |
| Active invalidation | Immediate targeted propagation; high priority |
| Conflict resolution | Immediate forward propagation; contested flag update |
| KCS invalidation event | Immediate; triggered by external legal developments |

**Truncated lineage:** Context window truncation during AI generation creates `weak_ancestry` edges. These are discounted to 60% of normal inheritance weight during propagation.

---

## Conflict Detection & Resolution

### Tier 1 — Structural Contradiction (fast, on ingest)
Same entity, opposing factual assertions detectable from metadata. Runs synchronously on every new chunk ingestion. Cost: O(indexed lookups).

### Tier 2 — Semantic Contradiction (async, queued)
Vector embedding comparison. High-similarity pairs classified by LLM for direct contradiction. Runs asynchronously. Cost: O(n) embeddings per ingest. Phase 7 implementation.

### Conflict State Machine

```
[open]
  │  contested flag set on chunk_a and chunk_b
  │  downstream citations inherit contested flag
  │  alert raised if unresolved past SLA (default: 5 business days)
  │
  ├─► [resolved]   attorney decision + required rationale
  │     a_supersedes_b → chunk_b validity = 0.0
  │     b_supersedes_a → chunk_a validity = 0.0
  │     both_valid     → contested flag cleared; scores restored
  │     both_discarded → both set to 0.0
  │
  └─► [dismissed]  false positive; detector retraining flagged
```

```yaml
ConflictEvent:
  conflict_id:        uuid
  chunk_a:            chunk_id
  chunk_b:            chunk_id
  conflict_type:      direct_contradiction | supersession | partial_overlap
  detection_method:   semantic | structural | metadata
  contradiction_score: float
  detected_at:        ISO8601
  status:             open | resolved | dismissed
  resolution:
    actor:            user_id
    decision:         a_supersedes_b | b_supersedes_a | both_valid | both_discarded
    rationale:        str   # required
```

---

## Point-in-Time Reconstruction

Critical for litigation. The score_history array is append-only — every update is timestamped, never truncated.

```
GET /validity/{chunk_id}/at?date=2025-03-15
→ score as of that date; what triggered the last update before that date

GET /case/{case_id}/snapshot?date=2025-03-15
→ all chunk validity scores for every chunk in the case as of that date

GET /document/{doc_id}/validity-at?date=2025-03-15
→ per-paragraph validity scores as they stood at filing date
```

---

## Document Editor Integration

Score bands displayed as margin indicators:

| Band | Score Range | Color | Meaning |
|---|---|---|---|
| HIGH | > 0.85 | Green | Current, unchallenged source |
| MEDIUM | 0.60–0.85 | Blue | Aging but not contested |
| LOW | 0.40–0.60 | Amber | Significant decay; verify before filing |
| CRITICAL | < 0.40 | Red | Highly stale or near-invalidated |
| CONTESTED | any | Purple | Open conflict event — resolution required |

---

## Implementation Roadmap

### Phase 1 — Validity Index & Decay (Weeks 35–36)
- ValidityRecord schema; ChunkIngested handler
- Decay Engine v1: exponential, linear, step functions
- model_vintage profile: model_cutoff + retrieval_freshness frozen at generation
- score_history append-only log
- Basic TVS Query API

### Phase 2 — DAG Propagation & Conflicts (Weeks 37–38)
- DAG propagation engine; hop_weight and inheritance floor
- weak_ancestry edge discount (60%)
- Tier 1 structural conflict detection on ingest
- ConflictEvent state machine; resolution UI
- Document editor integration: score band annotations
- Brief Validity Report

---

**Depends on:** PROV / MOIRAI
**Feeds into:** KCS / ARGUS (receives ActiveInvalidationRequests), RQS / HERMES (validity-aware retrieval), INTELLECT (validity dashboards), Document Editor (score bands)
