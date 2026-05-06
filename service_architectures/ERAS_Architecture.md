# ERAS — Explainability & Reasoning Audit Service
### LOGOS · *"Greek for 'reason' / 'word'"*
*Part of the THEMIS Platform · Strategic Capability · Build Priority: 9*

---

## Design Philosophy

The Provenance Service answers: **where did this content come from?** The TVS answers: **does this evidence still hold?** The ERAS answers a third, distinct question: **why did the AI say what it said?**

In legal practice, the "why" question is professionally significant. An attorney has a duty of competence — they must understand the basis for any advice or strategy they adopt. AI that produces a recommendation without an articulable reasoning chain creates professional responsibility risk.

> **Why ERAS Is Last:** ERAS requires everything else to be trustworthy before its output is meaningful. An explanation of reasoning derived from improperly retrieved content, evaluated without privilege enforcement, with miscalibrated analyst trust is an explanation of a compromised output. Build the conditions for trustworthy AI first; then audit the reasoning.

---

## Reasoning Capture

```yaml
ReasoningCapture:
  capture_id:           uuid
  turn_id:              uuid          # FK → Provenance Service
  interaction_class:    str
  matter_type:          str
  chain_of_thought:     str
  claims:               [Claim]
  confidence_signals:   [Signal]
  alternatives:         [str]         # alternative framings model considered
  evidence_citations:   [chunk_id]    # sources model explicitly cited
  unsupported_claims:   [Claim]       # claims with no retrieval backing

Claim:
  claim_id:             uuid
  text:                 str
  claim_type:           factual | legal | strategic | procedural
  supporting_chunks:    [chunk_id]
  confidence:           high | medium | low | uncertain
  validity_at_capture:  float         # TVS score of supporting chunks at capture
```

---

## Confidence Signal Taxonomy

| Level | Example Language | Attorney Guidance |
|---|---|---|
| **Definitive** | "The statute clearly requires..." / "unambiguous" | High — strong claim; verify evidence currency |
| **Strong** | "Courts have consistently held..." / "well-established" | Medium-high — verify citation currency via TVS |
| **Qualified** | "It appears that..." / "arguably" / "likely" | Medium — flag for attorney review |
| **Uncertain** | "This is a developing area..." / "contested" | Low — do not rely without independent research |

---

## Completeness Scoring

```yaml
CompletenessScore:
  supported_claim_ratio:    float   # claims with evidence / total claims
  confidence_coverage:      float   # claims with explicit confidence / total
  alternative_coverage:     bool    # were alternatives considered?
  evidence_validity_mean:   float   # mean TVS score of cited evidence at capture
  composite_completeness:   float   # weighted combination
```

**Hallucination flags:** Unsupported claims — present in reasoning with no corresponding retrieved evidence — are flagged for attorney verification. ERAS does not label these as hallucinations definitively (the model may draw on valid parametric knowledge) but they require verification before use in work product.

---

## Query Patterns

```
# Attorney query — why did the AI reason this way?
GET /explain/{turn_id}
→ {
    summary: "The recommendation is based on 3 primary claims...",
    chain_of_thought: "...",
    key_claims: [
      { text: "...", confidence: "high", supporting_evidence: [...],
        validity_scores: [0.87, 0.91] },
    ],
    unsupported_claims: [...],
    alternatives_considered: ["...", "..."],
    confidence_summary: { high: 4, medium: 2, low: 1, uncertain: 0 }
  }

# Compliance query — aggregate unsupported claim rates
GET /audit/{case_id}/claim-quality?class=evidence_analysis
→ unsupported claim rates by analyst, by interaction class, by matter type

# Litigation query — full reasoning record for specific work product
GET /document/{doc_id}/reasoning-record
→ all ReasoningCaptures contributing to document, linked to provenance
```

---

## Professional Responsibility Surface

**Client disclosure report:** Plain-language AI usage summary generated from structured reasoning record. Appropriate for client delivery — does not reveal privileged prompt content, system configuration, or analyst identities.

**Bar association readiness:** ERAS produces structured disclosure data in jurisdiction-configurable formats. As bar associations issue AI disclosure requirements, THEMIS schema updates configure outputs without engineering changes.

> **Policy note:** ERAS generates the data. What is disclosed, to whom, and when is a policy decision owned by ethics counsel and general counsel. The system is designed to be policy-neutral on disclosure obligations while ensuring the data exists to satisfy whatever policy applies.

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | Every ReasoningCapture linked to its turn_id; attached to provenance graph |
| TVS / KAIROS | Evidence validity scores at capture time frozen in reasoning record |
| TCS / MIMIR | Unsupported claim rates as additional calibration signal |
| PGS / NOMOS | Completeness scores feed output screening threshold; low completeness tightens screening |
| HADES | Adversarial reasoning records indexed as distinct ERAS category — highest-value content for understanding model failure modes |

---

## Implementation Roadmap

### Phase 1 — Capture Foundation (Weeks 61–62)
- Structured reasoning prompts in system prompt templates
- ReasoningCapture schema; raw chain-of-thought stored per turn
- Basic claim extraction: factual and legal assertion identification
- Turn_id linkage to Provenance Service

### Phase 2 — Indexing & Quality Model (Weeks 63–64)
- Claim-to-evidence mapping: supporting_chunks for each claim
- Unsupported claim detection and flagging
- Confidence signal extraction: linguistic uncertainty normalization
- CompletenessScore computation per turn
- Unsupported claim surface in document editor

### Phase 3 — Professional Responsibility & Audit (Weeks 65–66)
- Attorney /explain query: structured response with claim breakdown
- Client disclosure report generator
- Aggregate audit queries: unsupported claim rates by class and matter type
- TCS integration: unsupported claim rates as calibration signal
- Bar association readiness export

---

**Depends on:** PROV / MOIRAI, TVS / KAIROS, TCS / MIMIR, PGS / NOMOS
**Feeds into:** TCS / MIMIR (reasoning quality signals), PGS / NOMOS (output screening threshold), CLIENT (disclosure reports)
