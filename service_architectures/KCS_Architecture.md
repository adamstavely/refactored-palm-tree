# KCS — Knowledge Currency Service
### ARGUS · *"The all-seeing — monitors the external world"*
*Part of the THEMIS Platform · Operational Intelligence · Build Priority: 8*

---

## Design Philosophy

The TVS re-evaluates evidence you already have. The KCS watches the world outside your corpus and tells you when it changes in ways that matter. **TVS is a re-evaluation engine. KCS is a sensor network.** Neither is sufficient without the other.

> **KCS–TVS Relationship:** KCS is the event source. TVS is the processor. KCS monitors external sources and generates active invalidation events when something material changes. TVS receives those events and propagates validity score changes through the provenance DAG.

---

## Source Monitoring

```yaml
SourceRecord:
  source_id:         uuid
  name:              str
  type:              court_docket | regulatory_feed | legal_db | news | scholarly | agency_guidance
  jurisdiction:      str | [str]
  practice_areas:    [str]
  polling_interval:  duration
  push_endpoint:     url | null
  reliability_score: float         # 0.0–1.0; affects confidence of generated events
  last_checked:      ISO8601
  auth_config:       { ... }       # stored in Vault
```

**Source integrations:**

| Source | Type | Reliability | Notes |
|---|---|---|---|
| PACER | court_docket | 1.0 | Official docket entries |
| Federal Register | regulatory_feed | 0.95 | Rule amendments; effective date extraction |
| Westlaw KeyCite | legal_db | 0.95 | Negative treatment signals |
| Lexis Shepard's | legal_db | 0.95 | Citation analysis |
| Bloomberg Law | legal_db | 0.90 | Case law and regulatory alerts |
| News aggregators | news | ≤ 0.65 | Early-warning signals only; never direct invalidation |

---

## Development-to-Corpus Mapping

Three-stage pipeline when a new external development is detected:

```
Stage 1: Entity Extraction
  Extract from incoming development:
  - Case names and docket numbers
  - Statute and regulation citations (USC, CFR, etc.)
  - Regulatory body and agency identifiers
  - Key legal concepts and holdings
  - Named parties and witnesses

Stage 2: Corpus Impact Scoring
  For each extracted entity:
    query corpus chunk metadata for matching citations
    compute semantic similarity between development and candidate chunks
    score: impact = citation_match_weight × semantic_similarity × source_reliability

Stage 3: Matter Scoping
  Filter impact-scored chunks to active matters only
  (no alerts for inactive/closed matters unless litigation hold active)
  Rank by impact score; only generate events above confidence threshold
```

---

## Confidence Threshold Routing

| Confidence | Action | Example |
|---|---|---|
| 0.90–1.0 | Auto-forward to TVS as active invalidation | Court docket ruling directly citing corpus exhibit |
| 0.70–0.90 | Forward with attorney review flag; 48h review window | Regulatory amendment with citation overlap |
| 0.50–0.70 | Flag in matter dashboard; attorney decision required | News article discussing related case law |
| < 0.50 | Log only; no alert or TVS event | Low-confidence semantic match |

---

## KCS Invalidation Event → TVS

```yaml
KCSInvalidationEvent:
  event_id:           uuid
  source_id:          uuid
  development_url:    str
  development_type:   ruling | amendment | repeal | new_filing | guidance | contradictory_testimony
  affected_chunks:    [{ chunk_id, impact_score, impact_type }]
  confidence:         float
  detected_at:        ISO8601
  status:             pending_review | forwarded_to_tvs | dismissed
  human_reviewed:     bool
  reviewer_id:        uuid | null
```

TVS response on receiving KCS event:
- Updates validity records for affected chunks
- Propagates through provenance DAG
- Raises ConflictEvents for contradicted chunks (human resolution required)
- Returns: `{ affected_downstream_count, conflict_events_raised }`

---

## Matter Watch Lists

```yaml
WatchListEntry:
  entry_id:          uuid
  matter_id:         uuid
  entity_type:       case_citation | party | statute | regulation | witness
  entity_value:      str
  alert_threshold:   float    # confidence threshold (default: 0.50)
  notify:            [user_id, ...]
  created_by:        user_id
  active:            bool
```

**Monitoring lifecycle:** Active monitoring for open matters. Paused on matter close (unless litigation hold active). Indefinite monitoring for matters under hold.

---

## Integration Points

| Service | Role |
|---|---|
| TVS / KAIROS | Primary consumer of KCS invalidation events; propagates validity changes through DAG |
| PROV / MOIRAI | KCS development events logged; affected chunk metadata updated |
| HADES | KCS external threat intelligence feeds adversarial library expansion (AI adversarial research) |
| INTELLECT | KCS alert dashboard; matter watch list management; development feed |

---

## Implementation Roadmap

### Phase 1 — Source Infrastructure (Weeks 45–48)
- SourceRecord schema, auth configuration, reliability scoring
- PACER integration: court docket polling
- Federal Register feed: rule amendment detection
- Basic corpus mapping: citation-level matching
- Manual review queue for all events in Phase 1

### Phase 2 — Mapping Intelligence & TVS Integration (Weeks 49–54)
- Entity extraction pipeline: case citations, statute references, party names
- Semantic similarity scoring for impact assessment
- Confidence threshold routing: auto-forward, review flag, log-only
- TVS ActiveInvalidationRequest integration
- Matter-specific watch lists

### Phase 3 — Expanded Sources & Automation (Weeks 55–60)
- Legal database citator integration: KeyCite and Shepard's negative treatment
- News feed monitoring: early-warning signals
- Automated confidence calibration from outcome data

---

**Depends on:** TVS / KAIROS, PROV / MOIRAI
**Feeds into:** TVS / KAIROS (ActiveInvalidationRequests), HADES (adversarial library expansion)
