# SCRIBE — Document Version & Semantic Diff Intelligence
### *"The Recorder — keeper of what changed and why"*
*Part of the THEMIS Intelligence Layer · Build Priority: Year 2, Q1*

---


## Design Philosophy

SCRIBE tracks substantive changes across document versions with AI-assisted semantic understanding. It distinguishes edits that changed legal meaning from edits that changed style, flags provisions that disappeared without explanation, and identifies where AI-generated content in earlier versions was silently modified in later ones.

This is not track-changes. It is semantic diff with legal meaning awareness, integrated with the THEMIS provenance graph.

---

## Core Capabilities

### Semantic Diff Engine
Compares document versions at the clause and provision level, classifying each change as:
- **Substantive** — alters legal meaning or obligation
- **Structural** — reorganizes content without meaning change
- **Stylistic** — phrasing only; no legal impact

Uses an LLM with legal domain fine-tuning from the FGTS corpus.

### AI Contribution Tracking
For each document version, shows which clauses contain AI-generated content (from MOIRAI provenance), whether that content changed between versions, and whether the change moved toward or away from the AI's original output.

### Provision Disappearance Detection
Flags provisions present in a prior version that are absent in a later version without a corresponding replacement or deletion note. Surfaces these for attorney review as potential unintentional omissions.

### Cross-Matter Clause Library
Builds a queryable library of clause variations across matters:
- How has the firm's standard indemnification clause evolved across 50 transactions?
- What does TVS score as the most currently valid formulation of a specific provision type?

### Version Validity Inheritance
Tracks how validity scores change across versions. If v4 has a lower composite TVS score than v3, SCRIBE identifies which changes caused the decline and which source chunks drove it.

---

## Version Diff Schema

```yaml
VersionDiff:
  diff_id:              uuid
  document_id:          uuid
  version_from:         int
  version_to:           int
  timestamp:            ISO8601
  analyst_id:           uuid
  changes: [
    {
      clause_id:        uuid
      change_type:      substantive | structural | stylistic
      ai_contributed:   bool        # was AI-generated content in this clause?
      ai_content_retained: bool     # if yes — did the change preserve the AI output?
      validity_delta:   float       # change in TVS validity score for this clause
      provision_status: retained | modified | removed | added
    }
  ]
  provisions_removed:   [clause_id, ...]   # disappeared between versions
  ai_contribution_shift: float             # net change in AI contribution %
```

---

## Value Delivered

| Quality Control | Institutional Clause Intelligence | Audit Readiness |
|---|---|---|
| Catches silent degradation of AI-generated content — the most dangerous editing pattern because it is invisible to standard review | Searchable library of how the firm drafts specific provision types across matters and jurisdictions | Complete version-by-version accountability record linking changes to THEMIS services involved at each stage |

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | Reads AI-contributed content at chunk level per version; writes VersionDiff records as edges between document node versions |
| TVS / KAIROS | Validity scores inform semantic diff classifier; clause changes referencing low-validity sources flagged differently |
| ERAS / LOGOS | Reasoning captures from drafting sessions linked to specific version changes |
| FGTS / ALETHEIA | FGTS corpus fine-tunes the semantic diff LLM on legal meaning classification |
| MNEMOSYNE | Cross-matter clause library feeds into institutional knowledge graph |

---

**Build Phase:** Year 2, Q1–Q2 · After MOIRAI Phase 2 is stable
**Depends on:** PROV / MOIRAI, TVS / KAIROS, ERAS / LOGOS, FGTS / ALETHEIA (corpus)
**Feeds into:** MNEMOSYNE, CLIENT (version attestation), STOA (research context)
