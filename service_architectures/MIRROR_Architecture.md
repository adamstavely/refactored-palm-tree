# MIRROR — Similar Matter Intelligence
### *"The Reflector — shows you where you have been before"*
*Part of the THEMIS Intelligence Layer · Build Priority: Year 2, Q3*

---


## Design Philosophy

MIRROR identifies the most similar historical matters in the firm's corpus for any active matter — by legal theory, industry sector, opposing party profile, regulatory context, and evidentiary pattern. It shows what worked, what didn't, and what unexpected complications arose.

MIRROR makes institutional experience searchable and comparable rather than dependent on whether the right partner happens to be available to take a call.

---

## Core Capabilities

### Multi-Dimensional Similarity Scoring
Compares active matters to the historical corpus across six dimensions:

| Dimension | Weight (adjustable by matter type) | Signal Source |
|---|---|---|
| Legal theory | 25% | STOA research taxonomy classification |
| Industry sector | 20% | Matter management metadata |
| Opposing party profile | 20% | ORACLE opposing counsel profiles |
| Regulatory context | 15% | KCS regulatory monitoring |
| Evidentiary pattern | 15% | MOIRAI chunk type distribution |
| Geographic jurisdiction | 5% | Matter management metadata |

### Outcome-Linked Comparison
Similar matters displayed with outcomes:
- Settlement amount and timing
- Resource consumption (attorney hours, AI spend from FGS)
- Strategic moves that proved decisive
- Complications that arose and how they were resolved

### Complication Pattern Surfacing
Identifies complications from similar matters not yet apparent in the active matter:
- Discovery disputes that emerged at a certain phase
- Expert witness challenges
- Privilege issues requiring resolution
- Unexpected motion practice

### Attorney Experience Matching
Identifies which attorneys in the firm have prior experience in the most similar matters — not stated practice area but demonstrated matter history.

### Continuous Similarity Refresh
As a matter develops — new facts, evolving legal theories — MIRROR re-runs similarity scoring and updates the comparable matter set. The view of prior experience evolves with the matter.

---

## Similarity Record Schema

```yaml
MatterSimilarityRecord:
  active_matter_id:     uuid
  comparable_matter_id: uuid
  computed_at:          ISO8601
  similarity_scores:    {
    legal_theory:       float,
    industry_sector:    float,
    opposing_party:     float,
    regulatory_context: float,
    evidentiary_pattern: float,
    jurisdiction:       float,
    composite:          float
  }
  outcome_summary:      str
  key_complications:    [str]
  experienced_attorneys: [user_id, ...]
  validity_quality:     float   # mean TVS score of comparable matter documents
```

---

## Value Delivered

| Experience Accessibility | Risk Anticipation | Staffing Intelligence |
|---|---|---|
| Every attorney has access to the firm's full prior experience — not just the experience of whoever is staffed | Complication patterns surface before they occur, enabling proactive strategy | Staffing based on demonstrated matter experience, not availability alone |

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | Full matter history including AI interaction patterns; evidence corpus for comparison |
| RQS / HERMES | Vector embeddings from RQS vector store for matter profile similarity computation |
| TVS / KAIROS | Validity scores on historical matter documents inform comparison quality |
| STOA | Legal theory classification from STOA research taxonomy |
| ORACLE | Similar matter set grounds ORACLE outcome modeling |
| MNEMOSYNE | Similarity scores provide matter-to-knowledge mapping for institutional memory |
| PYTHIA | MIRROR's comparable matter set grounds PYTHIA's predictions |
| INTELLECT | Matter analytics module: similar matters panel |

---

**Build Phase:** Year 2, Q3 · Requires 50+ historical matters; improves continuously
**Depends on:** PROV / MOIRAI, RQS / HERMES, TVS / KAIROS, STOA
**Feeds into:** ORACLE, MNEMOSYNE, PYTHIA, INTELLECT
