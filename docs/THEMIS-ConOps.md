# THEMIS — Concept of Operations
## ATHENA Analyst Interface · Intelligence Community Deployment · v1.0
*Trusted Human-AI Enablement for Mission Intelligence and Safety*

---

## 1. Purpose and Scope

This Concept of Operations (ConOps) describes how the THEMIS platform and its ATHENA analyst interface are used in operational intelligence analysis. It defines the operational context, user roles and responsibilities, system overview, session workflows, alert and warning handling, collection gap procedures, and oversight protocols that govern platform use.

This document is intended for: analysts transitioning to ATHENA-assisted work, supervisors overseeing AI-assisted analytical sessions, oversight personnel (IOB, ARB), and organisational leadership making deployment decisions.

This ConOps covers the H1 platform (23 services through Phase 5) as the baseline, with intelligence layer capabilities noted where they add operational procedures in Year 2+.

---

## 2. Operational Context

### 2.1 The Problem THEMIS Addresses

Intelligence analysts using AI assistance have no mechanism to:
- Verify that AI-cited sources actually exist and say what the AI claims
- Know whether the AI's confidence is calibrated to its actual accuracy in this domain
- Track what AI said, on what basis, with what sources, across sessions
- Surface the nature of the uncertainty they are working with (inherent vs. collection gap vs. model limitation)
- Produce an auditable record of analytical process that can satisfy oversight requirements

The result is that AI assistance in intelligence analysis is ungoverned: outputs are plausible, sources are unverified, confidence is uncalibrated, methodology is invisible, and oversight is impossible.

THEMIS makes AI assistance governable. It does not restrict what analysts can do — it makes what analysts do accountable and auditable.

### 2.2 What THEMIS Is Not

THEMIS is not a replacement for analytical judgment. Every THEMIS service is advisory: it surfaces signals, flags concerns, characterises uncertainty, and records process. Every consequential analytical decision belongs to the analyst.

THEMIS is not a classification or access control system. PCES/AEGIS enforces clearance-based access control; it does not make original classification decisions.

THEMIS is not a collection system. It identifies collection gaps and routes them to the requirements management system; it does not task collection.

THEMIS is not an intelligence production system. Analysts use ATHENA to develop assessments; DPS/CODEX tracks the document; the intelligence production workflow remains as it is.

---

## 3. User Roles and Responsibilities

### 3.1 Analyst

The primary ATHENA user. Conducts AI-assisted analytical sessions, verifies AI claims through the ATHENA verification queue, resolves uncertainty characterisation questions, contributes to collection gap identification, and associates sessions with intelligence documents.

**Key responsibilities:**
- Engage genuinely with the verification queue — not performatively
- Review memory context at session start; flag stale or incorrect established facts
- Resolve UCS/TYCHE uncertainty questions from domain knowledge, not default acceptance
- Associate ATHENA sessions with the documents they contribute to (DPS/CODEX)
- Act on PYTHIA anticipatory signals for HIGH priority findings

### 3.2 Supervisor

Oversees the analytical team's ATHENA use. Reviews calibration state for team members, approves reserve access requests (FGS), confirms corrections with high analytical stakes (FGTS), reviews cross-requirement connection alerts (CRF/JANUS in Year 2+), and receives SENTINEL warning notifications (Year 3+).

**Key responsibilities:**
- Weekly review of team calibration states in TCS/MIMIR supervisor dashboard
- Approve or deny FGTS correction confirmations for high-stakes assessments
- Act on CRF/JANUS cross-requirement connection alerts within 72 hours
- Approve FGS reserve access requests with documented justification
- Review HADES adversarial exposure summary monthly (Research & Red Team liaison)

### 3.3 Requirements Officer

Works with TIS/DIKE-generated Collection Gap Requests (Year 2+). Reviews CGRs in the requirements management system, confirms collection tasking against THEMIS gaps, and provides feedback to TIS on CGR quality.

### 3.4 Research & Red Team

The operational intelligence function for THEMIS security. Accesses HADES to analyse adversarial patterns, develops and updates IAS/SCUDO threat catalog, evaluates capability benchmarks for CPS/APORIA (proposed), validates LACHESIS precursor patterns (Year 3+), and produces quarterly adversarial exposure briefings for the IOB.

**Key responsibilities:**
- Monthly HADES adversarial intelligence review
- Quarterly IAS/SCUDO threat catalog update derived from HADES findings
- Semi-annual MNEMOSYNE knowledge quality review
- Adversarial robustness red team exercises against live platform

### 3.5 IOB (Intelligence Oversight Board)

The governance authority for the THEMIS platform. Approves GC policy items, reviews platform compliance reports, receives MOIRAI chain audit certificates, sets tokenomics model parameters (FGS), and is the escalation point for all control gate decisions.

### 3.6 AI Trust Cell (Platform Operations)

The operational team responsible for THEMIS platform health, service deployment, monitoring, and first-level escalation. Distinct from the Research & Red Team. Responsible for service uptime, MOIRAI chain integrity monitoring, alert response, and ARB liaison.

---

## 4. System Overview

### 4.1 Platform Architecture Summary

THEMIS comprises 42 approved services organised across six platform namespaces and five intelligence layer namespaces:

**Platform namespaces (H1, Weeks 1–66):**
- `themis-gates`: PCES/AEGIS · PGS/NOMOS · MGS/TERMINUS (Safety and interaction governance)
- `themis-core`: MOIRAI · TCS/MIMIR · DPS/CODEX · OFS/NEMESIS · UCS/TYCHE (Core infrastructure)
- `themis-quality`: FGTS/ALETHEIA · TVS/KAIROS · RQS/HERMES · CVS/VERITAS · IAS/SCUDO · MAS/EIDOLON · MDS/KRONOS · KCS/ARGUS · ERAS/LOGOS · HADES · FGS/PLUTUS (Quality and adversarial)
- `themis-agent`: SCBS/SENTINEL-CAP · CBS/BROKER · RSS/ROLLBACK (Agentic safety)
- `themis-interaction`: PRS/PROMETHEUS · SKS/DAEDALUS (Prompt and skill governance)

**Intelligence layer namespaces (H2–H3, Year 2–4):**
- `themis-knowledge`: OGS/YGGDRASIL · MOS/SAGA · SCRIBE · MNEMOSYNE
- `themis-research`: STOA · MIRROR · ORACLE · PYTHIA
- `themis-warning`: TRS/CHRONOS · ADS/CASSANDRA · CGS/ARGUS-LACUNA · WSF/LACHESIS · CRF/JANUS · SWS/SENTINEL
- `themis-dissemination`: PCS/IRIS
- `themis-requirements`: TIS/DIKE

### 4.2 The Four Accountability Axes

Every THEMIS service contributes to one or more of four accountability axes. Three are approved and operational across H1–H3. A fourth is proposed pending ARB approval.

**Origin** — Where did this AI output come from?
Source type badges (GRND / PARAM / SYNTH) tag every claim by its provenance type. CVS/VERITAS verifies that cited sources exist and say what the AI claims. ERAS/LOGOS captures the full reasoning chain behind every response. DPS/CODEX links sessions to finished intelligence documents. MOIRAI makes the complete provenance record tamper-evident and auditable. When an assessment is questioned, the Origin axis answers: precisely this source, retrieved by this session, cited in this claim, at this time, under this clearance.

**Currency** — Is the intelligence still valid?
TVS/KAIROS tracks source validity through configurable decay models and propagates invalidation through the MOIRAI provenance graph. KCS/ARGUS monitors for external events that supersede corpus sources and tracks the staleness of the model's own parametric knowledge by domain. SCRIBE detects analytically significant changes between document versions and notifies analysts whose prior work is affected. CGS/ARGUS-LACUNA characterises collection gaps — not vaguely, but precisely enough to generate collection requirements through TIS/DIKE. The Currency axis answers: not just "is this source old" but "has the world changed in a way that makes this intelligence wrong."

**Trust** — Is analyst reliance calibrated?
TCS/MIMIR maintains per-analyst, per-domain, per-claim-type Bayesian calibration posteriors — tracking whether each analyst's reliance on AI is consistent with the AI's actual accuracy in that specific combination. FGTS/ALETHEIA accumulates the weighted correction corpus that feeds those posteriors. UCS/TYCHE characterises the nature of uncertainty (aleatory / epistemic / model) rather than collapsing it to a single confidence score. The gaming probability score detects performative verification that would corrupt the calibration record. The Trust axis answers: not "is this AI confident" but "is this analyst's reliance on AI in this domain calibrated to how accurate the AI actually is."

**Competence (proposed — pending ARB approval)** — Can the AI reliably perform this specific task?
The three approved axes do not address a fourth failure mode: an AI model operating on a claim type in a domain where it is systematically incapable, producing confident outputs the calibration system cannot yet flag because no calibration data exists for this analyst-domain cell.

CPS/APORIA addresses this through empirically-derived capability zone profiles (Green / Amber / Red) for each model version × domain × claim type combination, derived from Research & Red Team evaluation benchmarks — not from model self-report. Red zone assignments apply a hard confidence ceiling regardless of TCS calibration. ODS/LETHE extends this to the case level, detecting in real time when a specific query or content item falls outside the model's training distribution even within a nominally Green-zone domain.

The Competence axis answers: even before calibration data exists for this analyst-domain combination — and even when the AI sounds confident — is this claim type one where this model has demonstrated reliable performance? If not, what constraints apply?

> *Note for operational users:* CPS/APORIA and ODS/LETHE are not yet deployed. Until ARB approval and deployment, the Competence axis is addressed through UCS/TYCHE's model_dominance component (which receives its signal from ARGUS-LACUNA capability coverage data as a proxy) and through the PRIOR ONLY calibration state in TCS/MIMIR, which explicitly surfaces that no empirical calibration exists for this analyst-domain combination. Analysts should treat these as the current operational substitute for a dedicated Competence axis signal.

### 4.3 The Provenance Chain

Every analytically significant event in every THEMIS session produces a signed event in MOIRAI's cryptographic provenance chain. The chain is tamper-evident: any retroactive modification breaks it detectably. The chain records:
- Who accessed what intelligence, when, under what clearance
- What AI outputs were produced, from what sources, with what claims
- How analysts responded — what they verified, corrected, and confirmed
- How the assessment was disseminated and what consumer package accompanied it

The MOIRAI chain is the foundation of THEMIS's accountability claims. Without it, all other governance is assertion. With it, every claim can be verified.

---

## 5. Session Workflow

### 5.1 Standard Analytical Session

**Session Initialization (< 30 seconds)**

1. Analyst authenticates; PCES/AEGIS validates clearance and compartment, issues session privilege grant
2. ATHENA displays session scope (compartments accessible, any CoI flags, pressure mode)
3. FGS/PLUTUS queries team account balance; displays utilisation indicator in session header
4. MOS/SAGA assembles memory context from prior sessions on this matter (Year 2+); displays matter summary panel
5. PYTHIA checks for pending anticipatory signals; surfaces HIGH priority signals in ambient panel (Year 2+)
6. Analyst configures session: selects prompt from PRS/PROMETHEUS catalog; sets session parameters

**Analytical Interaction**

For each analyst query:

1. PGS/NOMOS screens input; classifies interaction type; checks ICD 203 compliance
2. IAS/SCUDO screens for adversarial injection patterns
3. Retrieval pipeline assembles context from corpus (TVS/KAIROS validity weighted; RQS/HERMES quality scored)
4. IAS/SCUDO screens retrieved chunks; blocked chunks excluded from context
5. PRS/PROMETHEUS mutation hint applied if this is a retry
6. AI generates response using selected prompt version
7. PGS/NOMOS screens output; blocks or flags standards violations
8. CVS/VERITAS checks all citations; populates GRND/PARAM/SYNTH badges with verification status
9. MAS/EIDOLON assesses any media artifacts; risk badges displayed
10. ERAS/LOGOS captures reasoning chain; extracts claims; flags unsupported claims in verification queue
11. UCS/TYCHE produces uncertainty profile; three analyst questions displayed for resolution
12. TCS/MIMIR confidence signal displayed based on analyst-domain calibration state
13. FGS/PLUTUS records token transaction against team account

**Verification Queue Engagement**

After each AI response, the analyst processes the verification queue:
- ERAS unsupported claim flags: verify independently or document as COULD_NOT_VERIFY
- CVS VERIFIED_INACCURATE citations: do not use; examine source directly
- CVS SOURCE_NOT_FOUND: do not rely without independent verification
- TVS FLAGGED or EXPIRED sources: check currency before relying
- MAS HIGH risk media: do not base assessment solely on this artifact

Analysts submit verification outcomes to FGTS/ALETHEIA. The five-factor weighting system records genuine verification separately from performative clicking.

**Session Close**

1. ATHENA prompts: "Did this session contribute to a finished analytical product?" — analyst links or declines
2. MOS/SAGA updates matter memory with new established facts and open questions (Year 2+)
3. SCBS/SENTINEL-CAP produces session summary (actions taken, spend consumed, unused capabilities)
4. FGS/PLUTUS final spend reconciliation; account balance updated

### 5.2 STOA Multi-Step Research Session (Year 2+)

For complex requirements requiring structured multi-step research:

1. Analyst states requirement to ATHENA; invokes STOA research orchestration
2. STOA proposes decomposition based on MIRROR prior requirements and MNEMOSYNE knowledge (Year 2+)
3. **Analyst review checkpoint:** analyst reviews, approves, or modifies sub-questions before research begins
4. For each sub-question: STOA invokes SKS skill, retrieves corpus, screens through IAS, verifies through CVS
5. Partial answer produced and surfaced; **analyst review checkpoint** before next sub-question
6. After all sub-questions: STOA proposes synthesis
7. **Analyst review checkpoint:** analyst compares synthesis against partial answers; approves or requests revision
8. Methodology trail produced: MOIRAI-attested record of every step, skill, source, and intermediate conclusion

### 5.3 Agent Task Session

For multi-step agentic tasks (autonomous research, document processing, source aggregation):

1. Analyst defines task with declared intent; SCBS/SENTINEL-CAP creates session capability envelope
2. CBS/BROKER issues minimum viable handles based on declared intent
3. Before each write action: SCBS triggers RSS/ROLLBACK pre-action snapshot; debit pre-call cost estimate
4. Agent executes task within envelope bounds; all credentialed calls proxied through CBS
5. Any MCP server calls validated by MGS/TERMINUS; responses screened by IAS/SCUDO
6. If spend approaches bound: analyst receives notification; action paused pending decision
7. Agent session close: rollback points surface in ATHENA reviewer panel for analyst review
8. Analyst reviews agent actions; exercises rollback if any action was unintended

---

## 6. Alert and Warning Handling

### 6.1 Platform-Level Alerts (All Phases)

| Alert | Source | Analyst action | Supervisor action |
|---|---|---|---|
| PGS output blocked | PGS/NOMOS | Do not requery to replicate blocked content; escalate if appears erroneous | Review if analyst escalates |
| IAS input blocked | IAS/SCUDO | Do not reformulate to replicate blocked content | Review repeated blocks in same session |
| CVS SOURCE_NOT_FOUND | CVS/VERITAS | Do not rely on citation; verify independently | Review if claim is analytically central |
| TVS source expired | TVS/KAIROS | Check currency through independent means | — |
| MAS HIGH risk media | MAS/EIDOLON | Seek independent corroboration; do not rely solely | Review before dissemination approval |
| TCS PRIOR ONLY state | TCS/MIMIR | Apply additional independent verification | Consider supervised session for new domain |
| FGS account approaching limit | FGS/PLUTUS | Notify supervisor; request reserve if needed | Approve or deny reserve request |

### 6.2 SCRIBE Document Update Alerts (Year 2+)

When a CRITICAL significance diff is detected on a source an analyst relied upon:
- ATHENA displays: "A source used in [prior/current session] has been significantly updated."
- **Analyst action:** Review the specific change before the affected assessment is disseminated. If the change materially affects the assessment, revise or caveat explicitly.
- **Supervisor action:** If the assessment is already in dissemination, initiate recall review.

### 6.3 SENTINEL Strategic Warning Notifications (Year 3+)

When a SENTINEL strategic warning is generated:

**IMMEDIATE priority:**
- Notification delivered to senior analyst and supervisor simultaneously
- Acknowledgment required within 4 hours
- PCS/IRIS consumer package generated for policymaker notification
- IOB notified

**URGENT/ROUTINE priority:**
- Notification delivered to supervisor
- Acknowledgment within 24/72 hours respectively
- No immediate policymaker notification; analyst-initiated if warranted

**Warning lifecycle:** GENERATED → ACKNOWLEDGED → ACTIONED or CLOSED (with outcome: CONFIRMED, FALSE_POSITIVE, or SUPERSEDED)

---

## 7. Collection Gap Procedures

### 7.1 Gap Identification (Year 2+)

Collection gaps are identified by four mechanisms:
- CGS/ARGUS-LACUNA automated assessment at session start (domain and entity coverage)
- RQS/HERMES sparse retrieval detection during session
- TVS/KAIROS source expiry flagging
- UCS/TYCHE epistemic dominance characterisation (analyst-confirmed collection gap)

### 7.2 Gap Request Submission

When an analyst confirms an analytically significant collection gap:

1. ATHENA surfaces the gap details from ARGUS-LACUNA
2. Analyst confirms or modifies the gap characterisation
3. Supervisor reviews (compartment-scoped view of full gap context)
4. Supervisor approves submission; TIS/DIKE creates Collection Gap Request (CGR)
5. CGR submitted to requirements management system; lifecycle tracking begins
6. Analyst notified when collection is tasked and when new intelligence is ingested

### 7.3 Gap Closure

When collection arrives against a prior gap:
- ATHENA notification: "New collection ingested against a gap you identified."
- Analyst re-runs affected assessment with new collection
- TVS/KAIROS and ARGUS-LACUNA re-evaluate coverage; gap marked INGESTED if coverage improved

---

## 8. Oversight Procedures

### 8.1 Routine IOB Visibility

The IOB has continuous read access to:
- MOIRAI chain audit certificates (any session, on demand)
- PGS compliance summary reports (aggregate flag rates by interaction class)
- TCS calibration state distribution (aggregate, de-identified)
- FGTS corpus quality report (monthly)
- HADES adversarial exposure summary (monthly, from Research & Red Team)
- FGS platform utilisation and tokenomics compliance
- DPS AI usage disclosure reports for disseminated products
- SWS/SENTINEL warning lifecycle records (Year 3+)

### 8.2 Control Gate Reviews

At each control gate (Gate-1 through Gate-9), the IOB receives:
- Full gate criteria verification report from the AI Trust Cell
- MOIRAI chain integrity audit certificate
- Research & Red Team adversarial assessment summary
- Open failure modes from HADES and CPS/APORIA (proposed)
- GC item status for gates requiring policy resolution

### 8.3 Analyst Misconduct Investigation

If an analyst is suspected of misconduct related to ATHENA use, the IOB can request:
- Full session provenance record (all MOIRAI events for specified sessions)
- ERAS reasoning captures (what the AI said and why)
- FGTS correction record (what the analyst verified and how)
- TCS gaming probability history (pattern of verification behaviour)
- FGS consumption record (session volume and token usage pattern)

All records are MOIRAI-attested and tamper-evident. The chain proves records have not been altered since recording.

### 8.4 Product Recall

If a disseminated intelligence product must be recalled:
1. Supervisor or IOB initiates recall via PCS/IRIS
2. PCS/IRIS generates `PCS_PACKAGE_RECALLED` MOIRAI event
3. DPS/CODEX marks document as SUPERSEDED
4. Consuming organisations notified through dissemination channel

---

## 9. Training Requirements

### 9.1 Analyst Training Programme (6 Modules)

**Module 1 — Understanding AI Source Types and Provenance (GRND/PARAM/SYNTH)**
Duration: 4 hours. Required before first ATHENA session.
Key outcomes: Analyst understands the three source type badges, what each means, and what verification action is appropriate for each.

**Module 2 — Calibration, Confidence Signals, and Verification**
Duration: 4 hours. Required before first ATHENA session.
Key outcomes: Analyst understands TCS/MIMIR calibration states (PRIOR ONLY → CALIBRATED), why genuine verification produces meaningful confidence signals, and how gaming degrades their own calibration.

**Module 3 — Uncertainty Characterisation (UCS/TYCHE)**
Duration: 2 hours. Required before first assessed assignment.
Key outcomes: Analyst understands the three uncertainty types (aleatory/epistemic/model), how to answer the three ATHENA questions from domain knowledge, and when to override taxonomy defaults with documented rationale.

**Module 4 — Agentic Sessions and Rollback**
Duration: 3 hours. Required before first agent task session.
Key outcomes: Analyst understands capability envelopes, how to configure minimum viable envelopes, what the rollback panel shows, and when to exercise rollback.

**Module 5 — Collection Gap Identification and CGR Submission (Year 2+)**
Duration: 2 hours. Required when CGS/ARGUS-LACUNA deploys.
Key outcomes: Analyst can distinguish types of collection gaps, confirm gap assessments from ARGUS-LACUNA, and submit CGRs through the appropriate workflow.

**Module 6 — Multi-Step Research and STOA Orchestration (Year 2+)**
Duration: 4 hours. Required before first STOA session.
Key outcomes: Analyst can decompose complex requirements, review and modify STOA decompositions, exercise partial answer review checkpoints, and interpret methodology trails.

### 9.2 Supervisor Training

In addition to all analyst modules: calibration dashboard interpretation, FGTS correction confirmation procedures, CRF/JANUS cross-requirement alert response, SENTINEL warning acknowledgment protocols, and FGS reserve approval governance.

### 9.3 IOB Training

MOIRAI chain audit certificate interpretation, HADES adversarial exposure summary review, GC item governance process, control gate review procedures.

---

## 10. Performance Monitoring

### 10.1 Platform Health Indicators (Weekly)

| Indicator | Target | Action if missed |
|---|---|---|
| MOIRAI chain integrity | 100% | Incident response; isolate affected sessions |
| PCES session denial rate | < 5% | Review clearance system API; investigate pattern |
| IAS block rate | Baseline ± 3σ | R&RT investigation if spike |
| FGS platform utilisation | < 85% | Leadership notification at > 85% |
| TCS PRIOR ONLY ratio | Declining trend | Review analyst onboarding; consider calibration sessions |
| ERAS unsupported claim rate | < 30% | Review prompt effectiveness; PRS catalog review |

### 10.2 Calibration Quality Indicators (Monthly)

| Indicator | Target | Action if missed |
|---|---|---|
| FGTS corpus weighted record count | Growing trend | Review analyst verification engagement |
| TCS gaming probability spikes | < 2% of sessions | R&RT investigation; supervisor review |
| OFS confirmation rate (Year 2+) | Growing sample size | Review GC-5 implementation; broaden outcome classification |
| MNEMOSYNE high-strength node count | Growing trend | Review extraction pipeline; analytic standards authority review |

### 10.3 Operational Efficiency Indicators (Quarterly)

| Indicator | Target | Notes |
|---|---|---|
| FGS consumption by interaction class | Within tokenomics model | Rebalance allocation if distribution shifts |
| STOA session completion rate (Year 2+) | > 80% not abandoned | Investigate decomposition quality; PRS prompt refinement |
| MIRROR data floor domains (Year 2+) | Growing count | Expect slow growth; not a failure indicator below Year 3 |
| SENTINEL true warning rate (Year 3+) | Track and calibrate | No initial target; build baseline |

---

## 11. Key Operational Constraints

**Analysts cannot:**
- Override a PCES session denial (must contact supervisor or security officer)
- Modify a PCS/IRIS consumer package without documented supervisor approval
- Access HADES content (Research & Red Team and IOB only)
- Exceed team token allocation without supervisor-approved reserve access
- Bypass the STOA analyst review checkpoints

**Supervisors can:**
- Approve reserve access requests (FGS)
- Confirm or override analyst FGTS corrections
- Initiate a product recall (DPS/CODEX + PCS/IRIS)
- View team calibration states (TCS supervisor dashboard)
- Acknowledge SENTINEL warnings

**IOB can:**
- Approve GC policy items
- Approve tokenomics model changes (FGS)
- Access any session's MOIRAI provenance record
- Access HADES adversarial exposure summaries
- Approve corpus rollback (FGTS — dual authorisation)
- Review ARB decisions and escalate platform governance issues

---

*THEMIS ConOps v1.0 · Platform: 42 services · Intelligence Layer: 15 services (Year 2+)*
*This document is reviewed and updated at each control gate.*
*Maintained by: THEMIS Platform Team in consultation with the IOB and Analytic Standards Authority.*
