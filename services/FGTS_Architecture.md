# FGTS — Feedback & Ground Truth Service
### ALETHEIA · *"Greek for 'truth' or 'disclosure'"*
*Part of the THEMIS Platform · Quality Feedback Loop · Build Priority: 5*

---

## Design Philosophy

Every other THEMIS service measures, governs, or tracks AI behavior. The FGTS is the only one that **learns from outcomes**. When an analyst rejects, edits, or overrides an AI output, that correction is a signal — about model quality, about retrieval gaps, about miscalibration. Without the FGTS, those signals disappear.

> **The Core Insight:** An AI that is wrong 15% of the time and an AI where analysts catch 15% of its errors are very different systems. The FGTS is what makes that distinction visible — and actionable.

---

## Correction Capture Model

```yaml
CorrectionEvent:
  correction_id:     uuid
  turn_id:           uuid          # FK → Provenance Service turn record
  chunk_id:          uuid          # FK → specific output chunk corrected
  analyst_id:        uuid
  timestamp:         ISO8601
  correction_type:   reject | edit | partial_edit | flag | approve_override
  original_content:  str
  corrected_content: str | null    # null for reject/flag
  edit_distance:     float         # normalized Levenshtein 0.0–1.0
  reason_code:       factual_error | outdated | irrelevant | style |
                     privilege_concern | citation_error | other
  reason_note:       str | null
  retrieval_context: [chunk_id]
  matter_type:       str
  interaction_class: str           # from PGS classification
```

### Correction Taxonomy

| Type | Description | Signal Value |
|---|---|---|
| `reject` | Analyst discards AI output entirely | High signal: complete miss |
| `edit` | Analyst modifies AI output | Graded signal: edit_distance indicates severity |
| `partial_edit` | Analyst accepts part, rewrites part | Span-level signal |
| `flag` | Analyst marks as potentially problematic | Weak signal: concern logged |
| `approve_override` | Analyst overrides a PGS warning | Calibration signal: over-reliance |

---

## Ground Truth Corpus

```yaml
CorpusEntry:
  entry_id:          uuid
  correction_id:     uuid
  matter_id:         uuid
  consent_level:     none | anonymised | full    # from ConsentRecord at capture time
  consent_id:        uuid
  input_context:     str
  ai_output:         str
  ground_truth:      str
  correction_type:   str
  quality_score:     float         # 0.0–1.0
  labels:            [str]
  usable_for_ft:     bool          # consent_level in [anonymised, full]
  usable_for_ft_full: bool         # consent_level == full
  version:           int
```

**Quality scoring criteria:** edit distance threshold, reason_code completeness, analyst track record (high-accuracy analysts' corrections score higher), consistency with other analysts on similar content.

---

## Training Data Consent Architecture

### Why Consent Is Required

Client engagement letters almost universally do not address whether client matter content may be used to train AI models. Proceeding with fine-tuning on un-consented matter content exposes the firm to breach of fiduciary duty claims and GDPR Article 6 lawful basis failures.

### Three-Tier Consent Model

| Consent Level | Status | Scope |
|---|---|---|
| `AI_ASSISTED_WORK` | Granted by default | Client consents to AI being used on their matter. Required for THEMIS to operate on this matter at all. |
| `AI_TRAINING_ANONYMISED` | Optional; must be affirmative | Anonymised, de-identified correction patterns used to improve AI systems. No client identity or matter details included. |
| `AI_TRAINING_FULL` | Optional; affirmative; senior attorney sign-off | Full matter content used (with privilege review) for fine-tuning. Requires explicit renewal per matter. |

### Consent Revocation

Revocation immediately sets `usable_for_ft = false` and `usable_for_ft_full = false` on all affected corpus entries. Past training runs cannot be reversed — documented in model registry as a provenance fact about that model version.

### Fine-Tuning Governance Gates

| Gate | Requirement |
|---|---|
| Consent coverage | ≥ 80% of corpus entries with valid consent level |
| Privilege review | Supervising attorney certification that no privileged content included |
| Jurisdiction review | GDPR lawful basis assessment for EU-jurisdiction matters |
| AI Governance Committee approval | Required for all fine-tuning runs |

---

## Analyst Signal Model

```yaml
AnalystSignalRecord:
  analyst_id:         uuid
  correction_count:   int
  correction_accuracy: float     # % confirmed correct by supervisors
  domain_expertise:   { matter_type: accuracy_score }
  catch_rate:         float     # corrections / AI outputs reviewed
  signal_weight:      float     # used when weighting corrections in corpus
  calibration_link:   tcs_analyst_id
```

**Cohort analysis:** When multiple analysts independently correct the same category of AI output in the same matter type, that is a systematic model failure — not individual preference. Cohort-level patterns are the primary input to the fine-tuning pipeline.

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | CorrectionEvents attached to turn_id; corpus entries linked to provenance graph |
| TCS / MIMIR | AIPerformanceUpdate events: correction severity drives Elo rating updates and RAI scores |
| RQS / HERMES | RetrievalMissSignal: analyst-identified missing sources forwarded to RQS quality index |
| HADES | Systematic correction patterns expand adversarial probe library |
| Fine-tuning pipeline | Ground truth corpus with consent gating feeds model training |

---

## Implementation Roadmap

### Phase 1 — Correction Capture (Weeks 29–30)
- CorrectionEvent schema; capture in document editor and chat interface
- Edit distance computation; reason_code taxonomy
- CorrectionEvent → Provenance Service attachment
- Analyst correction dashboard in Intellect

### Phase 2 — Corpus & Signal (Weeks 31–32)
- Curation pipeline: quality scoring, noise filtering, labeling
- AnalystSignalRecord: accuracy tracking, domain expertise scoring
- FGTS → TCS AIPerformanceUpdate event feed
- Consent architecture: ConsentRecord integration; corpus tagging

### Phase 3 — RQS Integration & Fine-Tuning Prep (Weeks 33–34)
- RetrievalMissSignal event feed to RQS
- Cohort analysis: systematic error pattern detection
- Fine-tuning pipeline scaffolding: quality gates, privilege review, model registry handoff

---

**Depends on:** PROV / MOIRAI, TCS / MIMIR, PCES / AEGIS, PGS / NOMOS
**Feeds into:** TCS / MIMIR, RQS / HERMES, HADES, Fine-tuning pipeline
