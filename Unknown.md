# THEMIS Platform — Addendum F

## Intelligence Cycle Integration and Analytical Architecture Extensions

**Document series:** THEMIS Platform Design · Addendum F  
**Predecessor documents:** Addenda A–E (Platform Services, Access Control, Memory, Competence Axis, Agent-Native Infrastructure)  
**Status:** Draft — For Architecture Review Board  
**Author:** Technical Director / Principal AI Architect  
**Date:** May 2026  
**Classification:** CONTROLLED

-----

## 1. Purpose and Scope

Addenda A through E extended the THEMIS platform within the boundaries of the AI-assisted analytical session: what the analyst and AI do together, how that interaction is governed, how capability is bounded, and how the intelligence layer learns from accumulated analytical work.

This addendum addresses a different class of gap — one that becomes visible only when the platform is evaluated against the full intelligence cycle rather than against the analyst session.

THEMIS currently governs the analysis phase well. It does not govern:

- The **upstream** of analysis: collection requirements and tasking that determine what intelligence is available
- The **downstream** of analysis: how finished assessments reach decision-makers and how real-world outcomes flow back to improve calibration
- The **epistemics** of analysis: how different kinds of uncertainty should be distinguished and surfaced, and how reasoning about future states differs from reasoning about current states
- The **multi-partner** dimension: how intelligence from coalition partners enters and is governed within the analytical workflow

Six services are proposed to close these gaps. Together they complete THEMIS’s coverage of the intelligence cycle from collection tasking through consumer delivery and outcome feedback — the forward and backward loops that the current platform treats as out of scope.

-----

## 2. The Intelligence Cycle Coverage Map

The IC intelligence cycle runs: Planning → Collection → Processing → Analysis → Dissemination → Feedback

Current THEMIS coverage:

- **Processing:** MAS/EIDOLON (media authenticity), IAS/SCUDO (adversarial screening), RQS/HERMES (retrieval quality)
- **Analysis:** Full coverage — 14 governance services + intelligence layer
- **Dissemination (partial):** DPS/CODEX tracks document provenance through dissemination

Not currently addressed:

- Planning and collection **tasking** (upstream of processing)
- **Consumer delivery** to decision-makers (downstream of dissemination)
- **Outcome feedback** from decisions back to calibration
- **Uncertainty decomposition** across the analytical workflow
- **Coalition partner** handling throughout the cycle
- **Temporal forecasting** as a distinct analytical mode

This addendum proposes one service per gap.

-----

## 3. OFS / NEMESIS — Outcome Feedback Service

**Namespace:** themis-core  
**Phase:** Year 2 · Q2  
**Depends on:** ORACLE/MIRROR (intelligence layer), MOIRAI (provenance), TCS/MIMIR, FGTS/ALETHEIA

### 3.1 The Gap

Calibration in THEMIS currently updates from analyst verification behavior: what the analyst checks, how they verify it, whether they correct the AI’s output. This is necessary. It is not sufficient.

The highest-weight ground truth signal in intelligence analysis is not analyst verification. It is outcome: the adversary did or did not do what the assessment said they would. The program was or was not in the development phase the assessment asserted. The capability was or was not demonstrated. Real-world events are the ultimate arbiter of analytical accuracy — and THEMIS has no mechanism to capture them.

ORACLE tracks requirement outcomes. What it lacks is a pipeline from real-world events to assessment closure and a connection from that closure back to calibration. The feedback loop from the world to the platform does not exist.

### 3.2 Purpose

NEMESIS in Greek mythology is the goddess of retributive justice — the force that corrects imbalance and ensures consequences follow actions. The Outcome Feedback Service corrects the imbalance between what the platform assesses and what the world reveals: every outcome event is a reckoning that flows back to improve future calibration.

OFS/NEMESIS captures real-world outcome events linked to active and closed requirements, assesses their relationship to prior analytical assessments, generates outcome records that close the calibration loop, and feeds high-weight ground truth signals to FGTS/ALETHEIA and ORACLE.

### 3.3 Core Responsibilities

**Outcome event ingestion.** OFS monitors designated outcome feeds: collection reporting that confirms or contradicts prior assessments, watchlist event matches from KCS/ARGUS, human-submitted outcome declarations from analysts and supervisors. Every event is processed against the active and recently closed requirement set.

**Assessment linkage.** For each outcome event, OFS traverses the MOIRAI provenance graph to identify all assessments, claims, and sessions that bear on the event. It generates an outcome linkage record connecting the event to every assessment it confirms or disconfirms, with a confidence weighting for the strength of the evidentiary connection.

**Outcome classification.** Each linked assessment receives an outcome classification: CONFIRMED (the assessment’s central claim has been borne out by events), DISCONFIRMED (events contradict the claim), AMBIGUOUS (events are consistent with the claim but do not confirm it), or SUPERSEDED (the world changed in a way that makes the claim’s truth value undeterminable).

**Calibration feed.** Outcome records are submitted to FGTS/ALETHEIA as high-weight corrections. The weighting scheme: CONFIRMED and DISCONFIRMED outcomes from real-world events carry substantially higher weight than analyst verification corrections, because they are independent of analyst behavior and represent the actual truth rather than analyst judgment about truth. AMBIGUOUS outcomes carry moderate weight. SUPERSEDED outcomes update TVS/KAIROS validity tracking but do not update calibration.

**ORACLE integration.** Every outcome record is also published to ORACLE, which uses outcome data to improve its predictions about future requirements. Over time, the combination of outcome feedback to calibration (through FGTS) and outcome feedback to predictive modeling (through ORACLE) creates a compounding improvement loop: better calibration improves current analysis, and better outcome models improve anticipatory intelligence.

### 3.4 Service Interfaces

```
outcome.ingest(event: OutcomeEvent) → OutcomeRecord
  — Ingests a real-world outcome event and begins assessment linkage

outcome.classify(outcomeId: string, assessment: AssessmentRef) → OutcomeClassification
  — Classifies the relationship between an outcome and a prior assessment

outcome.submit(outcomeId: string) → CalibrationFeedEntry, OracleUpdateEvent
  — Submits the completed outcome record to FGTS and ORACLE

outcome.query(requirementId: string) → OutcomeHistory
  — Returns all outcome records linked to a specific requirement

outcome.acknowledge(outcomeId: string, analystAuth: AuthContext) → AcknowledgmentRecord
  — Analyst or supervisor acknowledgment that the outcome record is accurate
```

### 3.5 Policy Dependencies

Outcome classification requires a policy definition of what constitutes a “confirmed” or “disconfirmed” intelligence assessment for calibration purposes. This is not a technical question — it is an analytical standards question that the Intelligence Oversight Board must resolve before OFS can be deployed. The edge cases are significant: an assessment that was correct about the adversary’s capability but wrong about their intent; an assessment that was correct at time of dissemination but subsequently overtaken by events; an assessment where the outcome is classified above the level at which the original assessment was made. GC-5 is designated for this policy gap.

-----

## 4. UQS / TYCHE — Uncertainty Quantification Service

**Namespace:** themis-quality  
**Phase:** Year 2 · Q1 (must precede PCS/IRIS)  
**Depends on:** TCS/MIMIR, CPS/APORIA, CGS/ARGUS-LACUNA, MOIRAI

### 4.1 The Gap

THEMIS surfaces a single calibrated confidence score. Confidence collapses three fundamentally different kinds of uncertainty into one number — and those three kinds require qualitatively different responses.

**Aleatory uncertainty** is inherent randomness in the world. The adversary has not decided. The outcome is genuinely probabilistic, not because we lack collection but because the subject of the assessment has not yet resolved. No collection improves this. No better AI resolves it. The right analytical response is a probability distribution over outcomes, not an assertion.

**Epistemic uncertainty** is lack of knowledge — collection gaps, processing limitations, analytical coverage gaps. ARGUS-LACUNA identifies these. Epistemic uncertainty is reducible: better collection, better processing, more analytical coverage. The right response is a collection requirement or a qualified judgment.

**Model uncertainty** is the AI system’s capability limitations. The Competence axis (Addendum D) addresses this. The right response is human judgment where the AI cannot reliably operate, or constraint of dissemination where capability limits apply.

When these are collapsed into a single confidence score, analysts and decision-makers cannot distinguish which kind of uncertainty they are facing. A HIGH confidence score because we have excellent current collection is epistemically and analytically different from a HIGH confidence score because TCS/MIMIR says this AI is well-calibrated in this domain, which is different from a MEDIUM confidence score because the adversary’s intent is genuinely probabilistic. Each requires a different response. Without the decomposition, all three look the same.

### 4.2 Purpose

TYCHE is the Greek goddess of fortune and chance — chance in the sense of genuine aleatory probability, events not yet determined by any prior cause. UQS/TYCHE is the service that distinguishes what is unknown because chance has not resolved it (aleatory), what is unknown because we have not looked (epistemic), and what is unknown because our tools cannot reliably assess it (model).

### 4.3 Core Responsibilities

**Uncertainty decomposition.** For each claim or session-level assessment, UQS queries TCS/MIMIR for the calibrated confidence signal, CPS/APORIA for the capability zone, CGS/ARGUS-LACUNA for the collection coverage status, and the MOIRAI provenance graph for the evidentiary basis of the claim. It decomposes the overall uncertainty into its three components and produces a structured uncertainty profile.

**Uncertainty profile structure.** Each profile contains:

- `aleatory_weight`: proportion of uncertainty attributable to genuine world-state indeterminacy, not reducible by analysis or collection. Estimated from the claim type — adversary intent claims carry higher aleatory weight than capability status claims, which carry higher aleatory weight than technical specifications.
- `epistemic_weight`: proportion attributable to collection or analytical gaps. Sourced from ARGUS-LACUNA collection coverage data. High epistemic weight means the uncertainty is reducible by better collection.
- `model_weight`: proportion attributable to AI capability limitations. Sourced from CPS/APORIA capability zone data and TCS/MIMIR calibration confidence intervals.
- `reducibility_vector`: what actions would reduce each component — specific collection approaches for epistemic, capability development or human substitution for model, explicit acknowledgment for aleatory.

**ATHENA surface.** In the ATHENA interface, the single confidence score is replaced by a three-component uncertainty indicator. Color coding: aleatory (purple — cannot be reduced), epistemic (amber — reducible by collection), model (blue — reducible by capability). The analyst can expand the indicator to see the reducibility vector and the specific basis for each component.

**Policymaker translation.** The decomposed profile feeds PCS/IRIS, which translates the technical structure into plain-language consumer indicators (see Section 5).

### 4.4 Design Constraints

Uncertainty decomposition at the claim level is a research problem, not a solved engineering problem. The aleatory weight estimation in particular — distinguishing inherent indeterminacy from epistemic gap — requires domain expertise encoded in the claim type taxonomy. The initial deployment should treat the decomposition as advisory, not authoritative. Analysts should be able to override the decomposition with a documented rationale. The Research and Red Team should run a pre-registered validation of the decomposition model against outcome data within the first year of deployment.

-----

## 5. PCS / IRIS — Policymaker Communication Service

**Namespace:** themis-dissemination  
**Phase:** Year 2 · Q3  
**Depends on:** UQS/TYCHE, MOIRAI, DPS/CODEX, OFS/NEMESIS

### 5.1 The Gap

ATHENA is designed for analysts. Intelligence products from ATHENA go to policymakers. Policymakers do not know what a source type badge means. They do not understand calibrated confidence. They receive finished assessments with no visibility into how confident to be, why the confidence level is what it is, what would change the assessment, or what kind of uncertainty they are working with.

The accountability chain ends at dissemination. Everything that happens after — how the policymaker interprets the assessment, what weight they give it, how they act on it — is outside the platform’s governance. This is not a marginal gap. The purpose of intelligence analysis is to inform decisions. The governance architecture governs the analytical process but not the point at which analysis meets decision.

### 5.2 Purpose

IRIS is the goddess of the rainbow — the messenger who traveled between Olympus and the human world, translating between divine knowledge and mortal action. The Policymaker Communication Service translates the technical provenance and calibration metadata of a THEMIS-governed assessment into a consumer package that decision-makers can read, interpret, and act on.

PCS/IRIS does not simplify. It translates without loss. The goal is to give the policymaker the same epistemic standing as the analyst — to make legible, in their terms, what the analyst knows about the quality and limitations of the assessment they are receiving.

### 5.3 Core Responsibilities

**Consumer package generation.** At dissemination, PCS/IRIS generates a consumer package alongside the finished intelligence product. The package contains:

- **Assessment basis summary**: A plain-language description of what sources informed the assessment, when they were collected, and whether they are still current. Not source names — source characteristics (recent human reporting, current signals collection, imagery from [timeframe], open-source corroboration).
- **Confidence plain-language translation**: A translation of the UQS/TYCHE uncertainty profile into three short sentences. “We assess [X] with [high/medium/low] confidence. The primary uncertainty in this assessment is [uncertainty type described in plain language]. Confidence would [increase/decrease] if [specific observable condition].”
- **Key assumptions**: The analytical assumptions the assessment depends on, stated explicitly. Drawn from the session’s Intention Gate records and analyst annotations in MOIRAI.
- **Falsification indicators**: What observable events would call this assessment into question. This is the single most important element of the consumer package — it transforms the assessment from an assertion into a hypothesis with testable implications, and it gives the policymaker a framework for updating their understanding as events develop.
- **Prior outcome record**: If ORACLE has outcome data on similar assessments (from MIRROR’s comparable requirement set), the consumer package includes a plain-language summary: “Assessments of this type from this analytical community have been confirmed at [X]% in recent years, with an average error of [Y] in the [dimension].”

**Provenance certificate.** The consumer package includes a provenance certificate: a cryptographically attested summary of the analytical record, readable to a non-technical audience. “This assessment was produced with AI assistance. The AI’s contributions were reviewed and verified by [analyst role]. The assessment and verification record are maintained in an auditable chain.”

**Decision support interface.** For high-profile assessments requiring policymaker briefings, PCS/IRIS generates a briefing deck summary that positions the assessment within a scenario space: under what conditions each plausible alternative outcome would materialize, and what indicators to watch. This is not scenario planning — it is the consumer-facing translation of the temporal reasoning outputs from TRS/CHRONOS (Section 8).

### 5.4 Policy Dependencies

The consumer package content must be approved by the Intelligence Oversight Board before deployment. Specifically: what information can appear in a consumer package (the falsification indicators in particular carry analytical risk if widely distributed), what classification level the consumer package carries relative to the underlying assessment, and who has the authority to modify a consumer package between generation and delivery. GC-6 is designated for this policy gap.

-----

## 6. CLS / PROTEUS — Coalition Liaison Service

**Namespace:** themis-gates  
**Phase:** Year 2 · Q2  
**Depends on:** PCES/AEGIS, PGS/NOMOS, MOIRAI, TCS/MIMIR, UQS/TYCHE

### 6.1 The Gap

PCES/AEGIS enforces US compartment classification. It does not understand coalition handling caveats: NOFORN (Not Releasable to Foreign Nationals), REL TO (releasable to specified countries), FVEY (Five Eyes), bilateral agreement constraints, or partner-specific handling requirements. A significant portion of IC analytical work depends on liaison reporting from partner services, and that reporting carries handling requirements that exist outside the US classification framework.

When partner-sourced intelligence enters the corpus, three things need to happen that THEMIS does not currently support. First, the handling constraints attached to that intelligence need to propagate to every derivative analysis product. Second, the confidence calibration for that intelligence needs to account for the partner service’s reliability profile in the relevant domain — a partner with an excellent track record on technical intelligence may have a poor track record on political reporting, and those profiles should be captured and applied. Third, the releasability of any analysis product that draws on partner sources needs to be enforced: an assessment citing FVEY-controlled source material cannot be released to a non-FVEY consumer without a formal sanitization process.

### 6.2 Purpose

PROTEUS is the shape-shifting sea deity of Greek mythology — the deity who could only be caught and made to reveal truth through extraordinary persistence, because he constantly changed form to escape. Coalition intelligence is PROTEUS: the same intelligence may have different handling requirements, different releasability, different confidence profiles depending on which partner it came from and under what arrangement. CLS/PROTEUS makes that multiplicity of forms governable within the THEMIS architecture.

### 6.3 Core Responsibilities

**Partner registry.** CLS maintains a registry of intelligence-sharing partners with associated handling profiles. Each partner entry specifies: applicable sharing agreements, default handling caveats, domain-specific confidence calibration parameters, releasability tiers, and the policy authority responsible for managing the relationship. Partner registry changes require approval from the designated authority (GC-7).

**Source tagging extension.** Partner-sourced intelligence entering the corpus receives a CLS source tag alongside the standard THEMIS source type badge. The CLS tag carries: partner identifier (encoded — not plain-text partner names in the operational interface), applicable handling caveats, partner-specific confidence ceiling for this domain, and the originating collection date under the partner’s reporting calendar.

**Confidence adjustment.** When TCS/MIMIR computes a calibration-adjusted confidence signal for a claim with partner-source provenance, CLS supplies the partner reliability coefficient for the relevant domain. A claim from a partner with an excellent track record on technical intelligence carries a higher confidence ceiling than the same claim from a partner with a poor track record in that domain. This coefficient is maintained by the partner registry and updated as outcome feedback from OFS/NEMESIS accumulates.

**Releasability enforcement.** Before any analysis product is disseminated, CLS checks the provenance graph for all partner-sourced contributing evidence. If any contributing source carries handling constraints that restrict the proposed dissemination, CLS blocks dissemination and routes to the analyst and supervisor for a formal sanitization decision. The analyst may: sanitize the product (remove the partner-sourced contribution and note the removal), seek formal release authorization through the partner channel, or release a version of the product that excludes the restricted contribution.

**UQS/TYCHE integration.** Partner-sourced uncertainty is added as a fourth uncertainty dimension in the UQS profile: `partner_weight` — the proportion of uncertainty attributable to reliance on partner-sourced intelligence with unknown internal collection methodology. This is distinct from epistemic uncertainty (which concerns our own collection gaps) and is surfaced specifically to analysts and policymakers who need to understand that confidence in partner-sourced intelligence is bounded by the opacity of the partner’s collection and analytical processes.

### 6.4 Design Constraints

The partner registry is policy infrastructure, not technical infrastructure. Every decision about what appears in a partner’s profile — their confidence coefficients, their handling requirements, their releasability tiers — is a policy and legal decision that requires coordination with the relevant authorities, general counsel, and the partner service itself through formal liaison channels. CLS provides the technical framework. The policy authority to populate and maintain that framework must be explicitly chartered before any partner entries are created.

-----

## 7. TIS / DIKE — Tasking Integration Service

**Namespace:** themis-requirements  
**Phase:** Year 2 · Q4  
**Depends on:** CGS/ARGUS-LACUNA, ORACLE, MOIRAI, PGS/NOMOS

### 7.1 The Gap

The intelligence cycle begins with collection requirements. Someone determines that intelligence on Topic X is needed, tasks collection assets to collect it, and the resulting intelligence eventually reaches the analyst. THEMIS enters the cycle only after collection, processing, and ingestion. The upstream — who tasked collection, what was asked for, whether the tasking was effective — is invisible to the platform.

This invisibility has two consequences. First, the collection gaps that ARGUS-LACUNA identifies have no path upstream. They accumulate as known gaps but cannot trigger collection requirements without a human manually translating the gap signal into a requirements request through a separate system. The feedback loop from analysis back to collection is broken. Second, ORACLE’s outcome data — what collection approaches produced accurate assessments, what collection profiles were correlated with analytical success — has no path upstream to inform requirements management decisions. The most actionable intelligence the platform produces about its own performance cannot reach the people who make collection decisions.

### 7.2 Purpose

DIKE is the Greek goddess of justice and moral order — specifically the justice that comes from things being in their proper place and sequence. The intelligence cycle has a proper sequence: requirements drive collection, collection informs analysis, analysis identifies new requirements. TIS/DIKE restores that sequence by connecting the THEMIS platform to the upstream requirements management infrastructure.

### 7.3 Core Responsibilities

**Requirements registry integration.** TIS maintains a live connection to the organization’s requirements management system. Active requirements from the requirements system are registered in TIS, which links them to the ATHENA sessions, MOIRAI provenance records, and ORACLE tracking entries associated with each requirement. This creates a requirement-level view of the complete analytical record for each tasking.

**Gap-to-requirement pipeline.** When ARGUS-LACUNA identifies a collection gap for an active requirement, TIS generates a Collection Gap Request (CGR) and routes it to the requirements officer responsible for the requirement. The CGR contains: the specific domain or question with no collection coverage, the requirement context (what analytical question the gap affects), the estimated epistemic uncertainty reduction if the gap were closed (from UQS/TYCHE), and a suggested collection approach based on ORACLE outcome data from similar prior requirements.

**Collection effectiveness feedback.** TIS publishes ORACLE outcome data to requirements officers in a requirements management-native format. When a requirement closes with outcome data, requirements officers receive: what collection profile was associated with this requirement, what the analytical accuracy was, and how this requirement compares to similar historical requirements. This gives collection managers empirical data on which collection approaches produce accurate analysis.

**Requirements lifecycle tracking.** TIS tracks the full lifecycle of each requirement from initial tasking through collection through analysis through dissemination through outcome. This lifecycle record is the basis for ORACLE’s predictive modeling and for the requirements-level provenance reports that oversight bodies request.

**PGS/NOMOS integration.** New collection requirements generated through the gap-to-requirement pipeline must be screened against analytic standards and legal authority by PGS/NOMOS before they are forwarded to the requirements management system. A collection gap does not automatically generate a collection requirement — it generates a proposed requirement that must be approved through normal authorities.

### 7.4 Interface with Existing Intelligence Management Systems

TIS is explicitly designed as an integration layer, not a requirements management replacement. The organization’s existing requirements management systems — whatever they are — remain authoritative. TIS consumes their data, enriches it with THEMIS analytical context, and returns gap signals and effectiveness data. This integration requires API agreements with the requirements management system owners that must be established as a prerequisite to Phase 2 Q4 deployment.

-----

## 8. TRS / CHRONOS — Temporal Reasoning Service

**Namespace:** themis-intelligence  
**Phase:** Year 3 · Q1  
**Depends on:** OFS/NEMESIS, UQS/TYCHE, ORACLE, MNEMOSYNE, ADS/CASSANDRA, SWS/SENTINEL

### 8.1 The Gap

The intelligence layer as designed reasons about the present and the recent past: what is the current state of the intelligence picture, what collection is current, what similar requirements looked like. TVS/KAIROS tracks whether current intelligence is still valid. PYTHIA anticipates what intelligence a requirement will need.

None of these address temporal forecasting as a distinct analytical mode. Forecasting — what will the adversary do next, how is a capability evolving, when is a threshold event likely — requires reasoning about future states under uncertainty. That reasoning is analytically and epistemically different from reasoning about current states.

The most important distinction: forecasting outputs are inherently aleatory in a way that current-state assessments are not. When we assess the adversary’s current capability, the answer exists — we may not know it, but it has a determinate value. When we forecast the adversary’s future intent, the answer may not yet exist — the adversary may not have decided, the decision may depend on conditions that have not yet materialized. Treating a forecast with the same confidence framework as a current-state assessment systematically misleads both analysts and decision-makers about the nature of what they are working with.

### 8.2 Purpose

CHRONOS is the personification of time in Greek mythology — not a deity who controls time, but time itself as an active force shaping events. TRS/CHRONOS does not predict the future; it structures how the platform reasons about time, trajectories, and the probability space of future states.

### 8.3 Core Responsibilities

**Trajectory modeling.** For active requirements with sufficient historical data, TRS generates trajectory models: time-indexed probability distributions over key indicators, based on ORACLE historical outcomes, MNEMOSYNE’s institutional knowledge of adversary behavioral patterns, and ADS/CASSANDRA’s anomaly detection outputs. Trajectories are explicitly labeled as projections with confidence intervals, not predictions.

**Leading indicator monitoring.** TRS maintains a set of leading indicators for each active requirement with a trajectory model. Leading indicators are observable collection signatures that historically precede specific threshold events. When current collection matches or deviates from a leading indicator pattern, TRS generates an indicator alert routed to the analyst. The alert includes: which threshold event the indicator pattern historically precedes, how far in advance of the event the indicator typically appears, the historical confirmation rate for this indicator pattern, and the current collection status on the indicator.

**Scenario space generation.** For high-priority requirements, TRS generates a scenario space: a structured set of plausible future states, each with a probability distribution, the conditions under which each would materialize, and the observable indicators that would discriminate between them. The scenario space feeds PCS/IRIS for the consumer package and the ATHENA analyst interface for session framing.

**Temporal uncertainty labeling.** All TRS outputs are processed through UQS/TYCHE with an explicit temporal uncertainty component. The temporal uncertainty field captures: how far into the future the projection extends (longer horizons carry higher uncertainty), whether the aleatory component is high (the subject of the forecast has not yet decided) or low (the trajectory is constrained by technical or physical factors), and the sensitivity of the trajectory to the key assumptions.

**SENTINEL integration.** TRS outputs feed SWS/SENTINEL as precursor signals. A trajectory that is diverging significantly from historical patterns is a strategic warning signal. A leading indicator pattern that has not appeared when historical precedent says it should have is an absence signal for ADS/CASSANDRA. TRS and the unknown unknown services are explicitly coupled: temporal reasoning about what should be happening is one of the primary mechanisms for surfacing unknown unknowns.

### 8.4 The Forecasting Governance Problem

Forecasting in intelligence analysis carries a specific accountability risk that current-state assessment does not. If a current-state assessment is wrong, it is wrong because the evidence was wrong or misinterpreted. If a forecast is wrong, it may be wrong for multiple reasons: the analysis was wrong, the aleatory component was high and the unlikely outcome materialized, or the world changed in a way that invalidated the assumptions. These require different accountability responses.

The consumer package (PCS/IRIS) must clearly distinguish forecast products from current-state assessments. The provenance record (MOIRAI) must capture which type of product an assessment is. The outcome feedback loop (OFS/NEMESIS) must classify outcome events against forecast products differently from current-state assessments — a forecast that said a 30% probability event would not happen that then happened is not a failed forecast.

The governance framework for forecast products — how they are labeled, how their accuracy is measured, how outcome feedback is structured for probabilistic assertions — must be designed before TRS is deployed in production. This is GC-8.

-----

## 9. Cross-Cutting Integration

The six services in this addendum form two functional loops and one enabling layer.

**The Calibration Loop:** OFS/NEMESIS closes the feedback loop from real-world outcomes to TCS/MIMIR calibration. UQS/TYCHE decomposes the calibration signal into its three components. TRS/CHRONOS produces temporal assessments that OFS/NEMESIS can eventually close with outcomes. The three services together create a calibration system that learns not just from analyst verification behavior but from the world.

**The Intelligence Cycle Loop:** TIS/DIKE connects THEMIS to the upstream requirements and collection management infrastructure. CLS/PROTEUS governs multi-partner intelligence throughout the cycle. PCS/IRIS delivers the downstream consumer package to decision-makers. The three services together extend THEMIS from governing the analysis phase to governing the full cycle from requirements through feedback.

**The Enabling Layer:** UQS/TYCHE is a prerequisite for PCS/IRIS (which needs the decomposed uncertainty profile to generate the consumer package), for CLS/PROTEUS (which adds a partner uncertainty component to the profile), and for TRS/CHRONOS (which requires temporal uncertainty as a distinct uncertainty dimension). UQS/TYCHE must be deployed before any of the services that consume its outputs.

-----

## 10. Implementation Sequencing

|Priority|Service      |Rationale                                                                                                                                                                                   |Phase  |
|--------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------|
|1       |UQS / TYCHE  |Prerequisite for PCS/IRIS, CLS/PROTEUS, and TRS/CHRONOS. Enabling layer for the full addendum.                                                                                              |Yr 2 Q1|
|2       |CLS / PROTEUS|Operational requirement for IC deployment. Coalition intelligence handling cannot wait for the intelligence layer services. Depends on UQS/TYCHE.                                           |Yr 2 Q2|
|3       |OFS / NEMESIS|Closes the calibration loop. Every month without outcome feedback is a month of calibration improvement foregone. Depends on ORACLE (intelligence layer, Yr 2 prerequisite).                |Yr 2 Q2|
|4       |PCS / IRIS   |Consumer-facing. Depends on UQS/TYCHE and should incorporate OFS outcome data for prior accuracy records.                                                                                   |Yr 2 Q3|
|5       |TIS / DIKE   |Requires API agreements with requirements management systems. Operational but not prerequisite.                                                                                             |Yr 2 Q4|
|6       |TRS / CHRONOS|Most complex. Depends on OFS/NEMESIS outcome data, UQS/TYCHE temporal uncertainty, and the full intelligence layer stack. Not deployable until the intelligence layer data floor is reached.|Yr 3 Q1|

-----

## 11. Policy Dependencies

|Ref |Policy Question                                                                                                                                                                                                                            |Owner                                           |Required Before       |
|----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------|----------------------|
|GC-5|Definition of “confirmed” and “disconfirmed” for intelligence calibration purposes. Edge cases: assessments correct in one dimension and wrong in another; assessments overtaken by events; outcomes classified above the assessment level.|General Counsel / IOB                           |OFS/NEMESIS deployment|
|GC-6|Consumer package content policy: what appears in a consumer package, its classification relative to the underlying assessment, modification authority before delivery.                                                                     |General Counsel / IOB                           |PCS/IRIS deployment   |
|GC-7|Partner registry governance: who has authority to create, modify, and delete partner entries; what requires partner service agreement before entry creation; rotation and review cadence.                                                  |General Counsel / Legal / Senior Agency Official|CLS/PROTEUS deployment|
|GC-8|Forecast product governance: labeling standards, accuracy measurement methodology for probabilistic assertions, outcome feedback framework for forecast products, policymaker guidance on interpreting probabilistic assessments.          |IOB / Senior Analytic Authority                 |TRS/CHRONOS deployment|

-----

## 12. Updated Service and Namespace Registry

This addendum adds six services across three namespaces:

|Service|Epithet|Namespace           |Phase  |
|-------|-------|--------------------|-------|
|OFS    |NEMESIS|themis-core         |Yr 2 Q2|
|UQS    |TYCHE  |themis-quality      |Yr 2 Q1|
|PCS    |IRIS   |themis-dissemination|Yr 2 Q3|
|CLS    |PROTEUS|themis-gates        |Yr 2 Q2|
|TIS    |DIKE   |themis-requirements |Yr 2 Q4|
|TRS    |CHRONOS|themis-intelligence |Yr 3 Q1|

`themis-dissemination` and `themis-requirements` are new namespaces not present in Addenda A through E.

The THEMIS platform now spans seven namespaces: themis-gates, themis-core, themis-quality, themis-intelligence, themis-agent, themis-dissemination, and themis-requirements.

-----

## 13. Architecture Review Board Recommendation

Submit the following to the Architecture Review Board:

**Approve UQS/TYCHE for immediate design.** It is a prerequisite for five other services in this addendum and several in the existing platform. Delay compounds.

**Approve CLS/PROTEUS for design, contingent on GC-7 initiation.** Coalition intelligence handling is an operational gap that cannot wait for the intelligence layer. Initiate GC-7 in parallel with design.

**Approve OFS/NEMESIS for design.** The calibration improvement that outcome feedback enables is the most valuable long-term return in the platform. The delay cost is one month of calibration improvement per month of delay.

**Approve PCS/IRIS for scoping.** The consumer interface is a governance and policy design problem as much as a technical one. Initiate GC-6 immediately to avoid blocking deployment.

**Defer TIS/DIKE pending API agreements.** Technical design can proceed but deployment cannot until requirements management system integration agreements are in place. Begin those negotiations now.

**Defer TRS/CHRONOS pending intelligence layer data floor.** Design can proceed. Deployment requires OFS/NEMESIS operational outcome data and the full intelligence layer stack. Target Yr 3 Q1 is realistic only if intelligence layer deployment begins on schedule.

**Commission Addendum G** to address the remaining gaps identified in the architecture review: analyst workflow outside ATHENA, federated deployment model, model procurement governance, and the empirical validation program for ATHENA interventions.

-----

*THEMIS Platform · Addendum F · Intelligence Cycle Integration and Analytical Architecture Extensions*  
*Companion documents: Platform Design v1.0 · Addenda A, B, C, D, E · ATHENA Intervention Catalog v3.2*
