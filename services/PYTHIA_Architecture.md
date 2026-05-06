# PYTHIA — Predictive Research Intelligence
### *"The Prophet — surfaces what you need before you ask"*
*Part of the THEMIS Intelligence Layer · Build Priority: Year 3, Q2+*

---


## Design Philosophy

PYTHIA anticipates what research, evidence, and intelligence a matter team will need before they know to ask for it. As a matter progresses through its lifecycle — complaint, discovery, motion practice, trial preparation — PYTHIA surfaces the research and evidence most likely to be needed at each stage, based on what MIRROR identified as the most similar historical matters and what MNEMOSYNE knows about how those matters developed.

PYTHIA is the difference between an analyst pulling information when asked and a system that anticipates what you need next.

---

## Core Capabilities

### Lifecycle Stage Modeling
Identifies the current stage of each active matter and models what typically happens next:

| Stage | Trigger Signals | PYTHIA Output |
|---|---|---|
| Pre-litigation | Matter opened; no complaint filed | Jurisdiction survey, similar dispute research, opposing party profile |
| Pleadings | Complaint filed; answer pending | Motion to dismiss research, affirmative defense landscape |
| Discovery | Scheduling order entered | Document review prioritization, deposition preparation research |
| Motion practice | MSJ briefing window | Summary judgment standards, analogous outcome research via ORACLE |
| Trial preparation | Trial date set | Expert witness preparation, evidentiary motion research, jury instruction research |
| Post-trial | Verdict/settlement | Appeal research if applicable; outcome recorded for ORACLE |

### Proactive Research Surfacing
Before an attorney requests research, PYTHIA identifies the research most likely to be relevant and pre-executes it via STOA. Results are ready when the attorney is — not minutes later.

```yaml
ProactiveResearchRequest:
  matter_id:        uuid
  stage:            str
  predicted_need:   str            # what research PYTHIA predicts is needed
  confidence:       float          # based on MIRROR similarity and MNEMOSYNE patterns
  source_matters:   [matter_id]    # which historical matters drive this prediction
  stoa_trail_id:    uuid           # pre-executed research trail
  surfaced_at:      ISO8601
  accepted:         bool | null    # did the attorney use it?
  feedback:         str | null     # analyst feedback on prediction quality
```

Analyst feedback (accepted/rejected + reason) feeds back to MNEMOSYNE to improve future predictions.

### Evidence Gap Detection
Compares the current matter's evidence corpus against what similar matters had assembled at the same stage. Surfaces evidence categories typically present at this stage but absent in the current matter.

```
Discovery stage — similar matters typically had by this point:
  ✓ Financial statements (present)
  ✓ Email communications (present)
  ✗ Expert report — typically retained at this stage in similar matters
  ✗ Third-party documents — subpoenas typically issued by now
```

### Matter Intelligence Brief
Daily or weekly synthesis per active matter:

```
MATTER INTELLIGENCE BRIEF — Smith v. Jones
Generated: 2026-05-05

EXTERNAL DEVELOPMENTS (KCS)
  ∙ New circuit court opinion on personal jurisdiction — affects ¶12 of complaint
    TVS validity of Exhibit 3 updated: 0.82 → 0.71

EVIDENCE VALIDITY (TVS)
  ∙ 3 sources now below 0.60 threshold since last brief
  ∙ 1 open conflict unresolved (12 days — SLA at 5 days)

ORACLE TRAJECTORY
  ∙ Based on 8 similar matters: median settlement at day 185 (currently day 142)
  ∙ Opposing counsel (Hargreaves LLP) settled in 6/7 similar matters; avg at 38% of demand

PYTHIA RECOMMENDATIONS
  ∙ Expert retention typically occurs at this stage — not yet on record
  ∙ Dispositive motion research pre-execution ready (STOA trail ST-0842)
  ∙ Deposition outline for Chen typically prepared 3 weeks before deposition date
```

---

## Feedback Loop

Analyst engagement with PYTHIA predictions is tracked and fed back:
- Accepted proactive research → confirms prediction; weight increased for similar future matters
- Rejected proactive research → reason captured; pattern adjusts
- Ignored brief items → lower confidence on similar predictions

This feedback accumulates in MNEMOSYNE as knowledge about which prediction categories are reliable for which matter types.

---

## Integration Points

| Service | Role |
|---|---|
| MIRROR | Similar matter set grounds all lifecycle predictions |
| MNEMOSYNE | Institutional knowledge about how similar matters developed |
| STOA | Executes proactive research PYTHIA identifies |
| ORACLE | Outcome trajectory modeling feeds intelligence brief |
| KCS / ARGUS | External developments feed brief and trigger evidence gap reassessment |
| TVS / KAIROS | Evidence validity status in brief; validity changes trigger re-assessment |
| INTELLECT | Matter intelligence feed with drill-down to source data |
| ANALYST TOOL | Matter sidebar: PYTHIA intelligence brief embedded in primary work surface |

---

**Build Phase:** Year 3, Q2+ · Requires MIRROR + MNEMOSYNE + STOA all operational
**Depends on:** MIRROR, MNEMOSYNE, STOA, ORACLE, KCS / ARGUS, TVS / KAIROS
**Feeds into:** ANALYST TOOL (matter sidebar), INTELLECT (matter intelligence feed), CLIENT
