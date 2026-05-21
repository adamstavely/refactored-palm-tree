# THEMIS Briefing Deck — Outline

## Trusted Human-AI Enablement for Mission Intelligence and Safety

**Deck purpose:** Executive briefing — the problem, the solution, the capability, and the investment required  
**Audience:** Senior leadership / decision-makers  
**Recommended length:** 24 slides + appendix  
**Tone:** Urgent, specific, credible — not aspirational  
**Last updated:** May 2026 — reflects Platform Design v1.0 + Addenda A–F

-----

## SECTION 1: THE PROBLEM

*Slides 1–5 · Establish the burning platform before presenting the solution*

-----

### Slide 1 — Title Slide

**THEMIS**  
Trusted Human-AI Enablement for Mission Intelligence and Safety

*Governing AI Across the Full Intelligence Cycle*

[Agency / Organization]  
[Date]  
[Classification marking]

-----

### Slide 2 — The Situation on the Ground

**AI is already in the analytical workflow. The governance question is not whether. It is how fast the infrastructure catches up.**

- Analysts are using AI-assisted research tools, LLMs for drafting finished intelligence, and AI-summarized source review **daily** — with or without authorization
- No current mechanism to distinguish AI-retrieved claims from AI-hallucinated claims
- No current mechanism to surface whether underlying sources have been superseded
- No current mechanism to measure whether analyst reliance on AI is appropriate for actual AI accuracy
- No current mechanism to govern coalition partner intelligence handling within AI-assisted workflows
- No feedback loop from real-world outcomes back to calibration

> **Speaker note:** This is not a future risk. Every production slide in this deck describes a failure mode that is happening now, in this workflow, without detection. The question is not whether to address it. The question is what the cost of delay is.

-----

### Slide 3 — The Six Failure Modes

**These are not theoretical. They emerge structurally from how LLMs behave in analytical workflows.**

|Failure Mode                   |Description                                                                           |Current Detection|
|-------------------------------|--------------------------------------------------------------------------------------|-----------------|
|**Compartment Leakage**        |RAG tools surface higher-classification content in lower-classification sessions      |None             |
|**Fabricated Source Reporting**|AI generates plausible-sounding source references that do not exist                   |None             |
|**Stale Intelligence**         |AI presents superseded assessments with current-intelligence confidence               |None             |
|**Uncalibrated Trust**         |Junior analysts accept AI output at higher rates than accuracy warrants               |None             |
|**Coalition Handling Failure** |Partner-sourced intelligence enters AI workflows without NOFORN/REL TO enforcement    |None             |
|**Uncertainty Collapse**       |Aleatory, epistemic, and model uncertainty reported as a single undifferentiated score|None             |


> **Speaker note:** Each is a structural property of how LLMs work without governance infrastructure — not a product defect that can be patched. They require infrastructure, not configuration.

-----

### Slide 4 — The Adversarial Dimension

**Foreign intelligence services know we are using AI-assisted analysis. They are designing against it.**

- **Corpus poisoning:** Seeding open-source and third-party reporting with fabricated intelligence likely to be ingested into analytical AI systems
- **Prompt injection:** Embedding adversarial instructions in documents likely to be retrieved in AI-assisted workflows
- **Adversarial competence probing:** Querying the AI on known capability gaps to generate confident confabulations, then introducing them into the analytical record as AI-sourced intelligence
- **Anchoring exploitation:** Targeting AI’s tendency to anchor analytical outputs on early retrieved information — a predictable and exploitable bias

**An ungoverned AI analytical workflow is a collection vector, not just an analytical risk.**

> **Speaker note:** Adversarial competence probing is distinct from corpus poisoning. It doesn’t attack the data — it attacks the model’s tendency to generate confident output in its gap areas. An adversary who has mapped the model’s capability limits can reliably generate useful confabulation. IAS/SCUDO and the proposed Competence axis address this directly.

-----

### Slide 5 — The Cost of Inaction

**Every day of ungoverned AI use generates compounding liability.**

- Analytical assessments built on AI output with no provenance record cannot be defended to oversight
- Oversight bodies are actively developing AI accountability frameworks — agencies without infrastructure will be in remediation mode under scrutiny when those frameworks arrive
- Analysts building calibration intuitions on ungoverned systems are learning the wrong model — miscalibration is difficult to correct later
- The data powering the intelligence layer only accumulates if the governance infrastructure is capturing it from day one
- Real-world outcome feedback that would improve calibration is lost permanently for every requirement processed without OFS/NEMESIS

**The cost of building THEMIS in Year 3 is higher than the cost of building it in Year 1 — not just because of incidents, but because of calibration data and institutional intelligence that is irretrievably lost.**

-----

## SECTION 2: THE FRAMEWORK

*Slides 6–7 · Three accountability axes plus a proposed fourth*

-----

### Slide 6 — Three Accountability Axes + A Proposed Fourth

**Every AI-assisted analytical interaction must be accountable across three independent dimensions — with a fourth proposed.**

```
┌────────────────┬──────────────────┬─────────────────┬──────────────────────┐
│   ORIGIN       │    CURRENCY      │     TRUST        │  COMPETENCE          │
│                │                  │                  │  (proposed)          │
│ Where did this │ Is this          │ Is the analyst's │ Can this AI system   │
│ claim come     │ intelligence     │ reliance         │ reliably perform     │
│ from?          │ still valid?     │ calibrated?      │ this task at all?    │
│                │                  │                  │                      │
│ Retrieved?     │ Source current?  │ Right reliance   │ In-distribution?     │
│ Parametric?    │ Model unchanged? │ for actual       │ Within capability    │
│ Synthesized?   │ Corroborated?    │ AI accuracy?     │ envelope?            │
│ Derived?       │ Outcome tested?  │                  │ Known boundary?      │
└────────────────┴──────────────────┴─────────────────┴──────────────────────┘
```

**The first three are instrumented. The fourth is proposed — hard enforcement gated on open research questions in adversarial ML and OOD detection.**

-----

### Slide 7 — Why Independence Matters

**Each pairing failure mode illustrates a distinct and real risk.**

> **Origin without Currency:** Provenance is perfect. The source was superseded eight months ago. Provenance-perfect and lethally stale.

> **Currency without Trust:** Sources are current. The analyst is systematically over-reliant on AI because their calibration has never been measured. Current picture; degraded judgment.

> **Trust without Origin:** The analyst is well-calibrated. The interface hides that this claim was synthesized across partial evidence. The calibration had nothing to operate on.

> **All three without Competence:** Provenance-perfect, current, well-calibrated — the analyst never knew the task exceeded the model’s reliable capability. The assessment is confident and wrong.

**THEMIS instruments the first three axes continuously. The Competence axis is in advisory deployment pending research maturity.**

-----

## SECTION 3: THE PLATFORM — THEMIS

*Slides 8–11 · 22 services across 7 namespaces*

-----

### Slide 8 — THEMIS: 22 Services Across 7 Namespaces

**22 platform services governing every AI-assisted analytical interaction from collection requirements through policymaker delivery.**

```
SAFETY GATES              CORE INFRASTRUCTURE        QUALITY LAYER
─────────────────         ──────────────────────     ──────────────────────────
PCES / AEGIS              Provenance / MOIRAI        FGTS / ALETHEIA
Compartment &             Cryptographic              Feedback & Ground Truth
Classification            Provenance Backbone
                                                     TVS / KAIROS
PGS / NOMOS               TCS / MIMIR                Temporal Validity
Analytic Standards        Trust Calibration
& Policy                                             RQS / HERMES
                          DPS / CODEX                Retrieval Quality
CLS / PROTEUS             Assessment Provenance
Coalition Liaison                                    CVS / VERITAS
                          OFS / NEMESIS              Source Corroboration
                          Outcome Feedback
                                                     IAS / SCUDO
                          UQS / TYCHE                Adversarial Input Screening
                          Uncertainty
                          Quantification             MAS / EIDOLON
                                                     Media Authenticity
INTELLIGENCE LAYER        AGENT-NATIVE INFRA
──────────────────        ──────────────────         MDS / KRONOS
KCS / ARGUS               SCBS / SENTINEL-CAP        Model Drift Detection
Knowledge Currency        Session Capability
                          Bounding                   DISSEMINATION
ERAS / LOGOS                                         ─────────────
Reasoning Audit           CBS / BROKER               PCS / IRIS
                          Credential Broker          Policymaker
                                                     Communication
                          RSS / ROLLBACK             
                          Reversibility &            REQUIREMENTS INTEGRATION
                          State Snapshot             ─────────────
                                                     TIS / DIKE
                                                     Tasking Integration
```

-----

### Slide 9 — The Accountability Axes: Service Mapping

**Each axis is served by dedicated platform services. Competence axis services are in advisory deployment.**

|Axis                       |Primary Services                |Function                                                                                                                |
|---------------------------|--------------------------------|------------------------------------------------------------------------------------------------------------------------|
|**ORIGIN**                 |MOIRAI · PCES · ERAS · DPS · CLS|Provenance of every claim — retrieved, parametric, synthesized, derived, or coalition-sourced                           |
|**CURRENCY**               |TVS · KCS · MDS · CVS · OFS     |Temporal validity, external invalidation, model drift, source corroboration, real-world outcome feedback                |
|**TRUST**                  |TCS · FGTS · RQS · UQS          |Per-analyst calibration, weighted ground truth, retrieval quality, decomposed uncertainty (aleatory / epistemic / model)|
|**COMPETENCE** *(proposed)*|CPS/APORIA · ODS/LETHE          |Capability taxonomy (Green/Amber/Red zones), out-of-distribution screening. Advisory signals only.                      |

-----

### Slide 10 — What Makes THEMIS Defensible

**Cryptographic attestation: the difference between audit logs and audit records.**

- Every THEMIS event is signed with a per-service cryptographic key
- SHA-256 hash chaining: retroactive modification breaks the chain from that point forward — detectable without access to the original
- RFC 3161 timestamp authority anchoring every 24 hours: external, independently verifiable reference point
- Full chain audit certificate: oversight-submissible proof that the provenance record has not been altered since inception

**An ungoverned AI analytical record can be questioned. A THEMIS record can be verified.**

-----

### Slide 11 — Uncertainty Decomposition: UQS / TYCHE

**The single confidence score hides the most important distinction decision-makers need to make.**

Three kinds of uncertainty require three qualitatively different responses:

|Uncertainty Type|Meaning                                                           |Reducible by                                                 |ATHENA Color|
|----------------|------------------------------------------------------------------|-------------------------------------------------------------|------------|
|**Aleatory**    |Inherent world-state indeterminacy — the adversary has not decided|Nothing — acknowledge and express as probability distribution|Purple      |
|**Epistemic**   |Collection or analytical gaps                                     |Better collection, better coverage                           |Amber       |
|**Model**       |AI capability limitations                                         |Better capability, human substitution                        |Blue        |
|**Partner**     |Coalition source opacity                                          |Partner relationship development                             |Grey        |

**Collapsing these into one score misleads both analysts and decision-makers about what kind of uncertainty they face and what would reduce it.**

> **Speaker note:** A decision-maker who sees HIGH confidence needs to know whether that means “we have excellent current collection” or “the AI is calibrated in this domain” or “the adversary’s intent is genuinely probabilistic.” These are different epistemic situations requiring different responses. UQS/TYCHE makes that distinction visible.

-----

## SECTION 4: THE INTERFACE — ATHENA

*Slides 12–13 · What analysts actually see and interact with*

-----

### Slide 12 — ATHENA: 31 Interventions Across 8 Categories

**ATHENA is the analyst-facing interface that operationalizes the accountability axes through 31 targeted behavioral interventions.**

The interventions are not features. Each targets a specific cognitive failure mode documented in the human factors and intelligence analysis literature.

|Category                   |Interventions|Failure Mode Targeted                               |
|---------------------------|-------------|----------------------------------------------------|
|Session Configuration      |4            |Chat mode, corpus version, parameter transparency   |
|Session Scaffolding        |6            |Anchoring, context collapse, strategic framing drift|
|Pre-Response               |2            |Anchoring, confirmation bias pre-emption            |
|Response Interventions     |5            |Automation bias, source type opacity                |
|Verification Infrastructure|3            |Vigilance decrement, unverified claim accumulation  |
|Behavioral Telemetry       |6            |Invisible miscalibration, session integrity         |
|Session Integrity          |2            |Gaming detection, cross-analyst divergence          |
|**Pressure Response**      |**3**        |**Calibration degradation under deadline pressure** |

-----

### Slide 13 — ATHENA: Key Interface Elements

**Five elements representing the core axes at the analyst interface.**

**The Intention Gate** *(Origin + Trust)*  
Analyst declares prior belief and falsification condition before the AI response renders. Counteracts anchoring — the prior is set before the AI output becomes the reference point.

**Source Type Badges** *(Origin)*  
Every claim carries a visible badge: GRND, PARAM, SYNTH, TRANSCRIPT, VIDEO, AUDIO, IMAGE, OCR, MEMORY. Each shows where the claim came from and what verification path applies.

**Uncertainty Decomposition Indicator** *(Trust — UQS/TYCHE)*  
Three-component indicator replacing the single confidence score: aleatory (purple), epistemic (amber), model (blue), with reducibility vector on expand.

**The Verification Queue** *(Trust)*  
Unverified claims accumulate in a persistent visible queue. Dissemination is blocked when high-confidence unverified claims remain.

**Deadline-Critical Hard Constraints** *(Trust + adversarial defense)*  
Non-dismissable under DC pressure: PARAM claims block dissemination, IAS/SCUDO elevates one tier. Deadline periods are when adversary D&D operations are most active.

-----

## SECTION 5: THE INTELLIGENCE LAYER

*Slides 14–16 · From governance to institutional intelligence*

-----

### Slide 14 — From Governance to Intelligence

**The governance layer makes AI use safe. The intelligence layer makes the investment compound.**

13 Year 2–4 services transform THEMIS from a governance platform into an institutional intelligence asset:

|Service            |Capability                                                                            |Data Floor                    |
|-------------------|--------------------------------------------------------------------------------------|------------------------------|
|**OGS / YGGDRASIL**|Semantic ontology foundation — canonical entity model for cross-INT entity resolution |Prerequisite for all below    |
|**SCRIBE**         |Semantic document version diff with intelligence significance                         |MOIRAI + TVS                  |
|**STOA**           |Multi-step research orchestration with documented methodology trail                   |RQS + ERAS                    |
|**ORACLE**         |Predictive intelligence from historical requirement outcomes                          |200+ requirements             |
|**MIRROR**         |Similar requirement identification across 6 dimensions                                |50+ requirements              |
|**MNEMOSYNE**      |Institutional knowledge graph from corrections and outcomes                           |FGTS + ORACLE + MIRROR        |
|**PYTHIA**         |Anticipatory research surfacing before the analyst knows to ask                       |MIRROR + MNEMOSYNE + STOA     |
|**TRS / CHRONOS**  |Temporal reasoning — trajectory modeling, leading indicator monitoring, scenario space|OFS/NEMESIS + full intel stack|

**MOS/SAGA** manages memory architecture across the intelligence layer.

-----

### Slide 15 — Addressing Unknown Unknowns: The Rumsfeld Layer

**The intelligence layer as designed reasons about known unknowns. Five additional services address what we don’t know we don’t know.**

|Service               |Mechanism                                                                                                                                             |Data Floor                             |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|
|**ADS / CASSANDRA**   |Anomaly detection — surfaces behavioral deviations from established baselines. What should be in the collection picture but isn’t.                    |ORACLE maturity + 200 reqs             |
|**CGS / ARGUS-LACUNA**|Collection gap mapping — distinguishes absence-of-evidence from absence-of-collection. Makes the shape of the unknown visible.                        |KCS + ARGUS-LACUNA integration         |
|**WSF / LACHESIS**    |Weak signal fusion across requirements — clusters that exceed significance threshold their individual components don’t reach. Supervisory-facing only.|30+ simultaneous active reqs           |
|**CRF / JANUS**       |Cross-requirement semantic correlation — connections neither requirement alone could see. PCES-gated. Supervisory-facing only.                        |OGS/YGGDRASIL + PCES clearance         |
|**SWS / SENTINEL**    |Strategic warning — indicators of things that would fundamentally change the analytical picture.                                                      |Full intel stack + CASSANDRA + LACHESIS|

**These services do not eliminate unknown unknowns. They convert more of them into known unknowns faster than an ungoverned platform would.**

-----

### Slide 16 — The Flywheel

**Every requirement processed through THEMIS adds to the institutional intelligence base.**

```
Every assessment  → MIRROR grows the comparable requirement set
Every correction  → FGTS enriches the source reliability corpus
Every outcome     → OFS/NEMESIS feeds ORACLE + FGTS calibration
Every session     → MNEMOSYNE extracts tacit knowledge
Every anomaly     → CASSANDRA refines behavioral baselines
Every gap signal  → TIS/DIKE converts to collection requirements
```

**At Year 4, the agency has a platform that gets smarter with every requirement it handles — and a gap that widens every year relative to any competitor using ungoverned AI tools.**

**That gap cannot be closed by any external vendor or adversary because it is built on the agency’s own analytical history.**

-----

## SECTION 6: AGENT-NATIVE INFRASTRUCTURE

*Slide 17 · Governing AI agents, not just AI outputs*

-----

### Slide 17 — Agent-Native Infrastructure: Governing AI Agents

**As AI moves from generating outputs to taking actions, three new services bound what agents can do.**

The existing 14 governance services govern what AI outputs say and how analysts use them. As agentic AI systems begin taking actions on behalf of analysts — retrieving from live systems, writing to documents, executing multi-step research — the threat surface changes. The critical risk is not a bad output. It is a bad action that cannot be undone.

|Service                |Function                                                                                                            |Failure Without It                                  |
|-----------------------|--------------------------------------------------------------------------------------------------------------------|----------------------------------------------------|
|**SCBS / SENTINEL-CAP**|Session capability bounding — defines and enforces spend cap, resource scope, TTL. Fails closed on bound exceedance.|Agent executes unbounded actions; no escalation path|
|**CBS / BROKER**       |Credential broker — agents receive scoped handles, never raw credentials. All credentialed calls proxied.           |Raw credential exposure; no revocation path         |
|**RSS / ROLLBACK**     |Reversibility and state snapshots — pre-action snapshots, 72h retention, single-command rollback.                   |Irreversible agent actions with no recovery path    |

**These three services are the minimum viable infrastructure for any agentic deployment. No agent-assisted analysis should be deployed without all three operational.**

-----

## SECTION 7: THE INTELLIGENCE CYCLE

*Slide 18 · Closing the loops the platform previously left open*

-----

### Slide 18 — Closing the Intelligence Cycle

**Addendum F extends THEMIS from governing the analysis phase to governing the full intelligence cycle.**

```
                    TASKING
                    TIS / DIKE
                    Collection Gap Requests
                    upstream to requirements officers
                         ↓
FEEDBACK         COLLECTION           PROCESSING
OFS / NEMESIS    CLS / PROTEUS        MAS / EIDOLON
Outcome events   Coalition partner    Media authenticity
close the        handling,            IAS / SCUDO
calibration      NOFORN / REL TO      Adversarial screening
loop back to     enforcement
FGTS + ORACLE         ↓
     ↑           ANALYSIS             DISSEMINATION
     │           Full THEMIS          PCS / IRIS
     └───────────platform             Consumer packages
                 (22 services)        for decision-makers
```

Each loop was previously open. TIS/DIKE closes the upstream loop to collection. OFS/NEMESIS closes the downstream calibration loop from outcomes. PCS/IRIS closes the consumer loop to decision-makers. CLS/PROTEUS governs coalition intelligence throughout.

-----

## SECTION 8: THE TEAM

*Slide 19 · Who builds and sustains THEMIS*

-----

### Slide 19 — The AI Trust Cell: 21 People, Three Teams

**Governing AI in high-stakes analysis is an organizational design problem, not just an engineering problem.**

```
AI TRUST CELL — 21 people

┌──────────────────────┬─────────────────────────┬─────────────────────────┐
│  Research &          │  THEMIS Platform Team   │  Intelligence Layer     │
│  Red Team            │                         │  Team                   │
│  7 people            │  7 people + 1 embedded  │  6 people               │
├──────────────────────┼─────────────────────────┼─────────────────────────┤
│ Pre-registered       │ Synchronized 2-week     │ Sprint stream +         │
│ research cycles      │ sprints                 │ 4–6 week model          │
│                      │                         │ development cycles      │
│ Red team evaluation  │ 3 swim lanes:           │                         │
│ continuous           │ Roadmap / Red Team      │ Deployed on quality     │
│                      │ Response / Tech Health  │ thresholds, not sprint  │
│ Never takes sprint   │                         │ boundaries              │
│ tickets              │ Tech health ≥ 15–20%    │                         │
│                      │ non-negotiable          │                         │
└──────────────────────┴─────────────────────────┴─────────────────────────┘
```

**The Research and Red Team never takes sprint tickets.** Research independence is structural or it does not exist.

**Monthly evaluation report delivered simultaneously to Cell Lead and CTO.** Not filtered. The people accountable for outcomes see quality problems at the same time as the people accountable for fixing them.

-----

## SECTION 9: THE ROADMAP

*Slides 20–21 · Phased implementation*

-----

### Slide 20 — Implementation Roadmap

**Three horizons. Each phase gate requires Cell Lead + Engineering Lead + Intelligence ML Lead joint confirmation.**

|Horizon                              |Timeline   |Deliverable                                                                                              |Services                                                                                                                                     |
|-------------------------------------|-----------|---------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
|**H1: Governance Foundation**        |Months 1–17|Governed AI analytical platform. Full accountability infrastructure. Answerable to any oversight inquiry.|All 14 original governance services + agent-native (SCBS, CBS, RSS)                                                                          |
|**H2: Intelligence Cycle Completion**|Year 2     |Full intelligence cycle governed. Coalition handling. Policymaker interface. Calibration loop closed.    |UQS/TYCHE · CLS/PROTEUS · OFS/NEMESIS · PCS/IRIS · TIS/DIKE + intelligence layer (SCRIBE, STOA, ORACLE, MIRROR, MNEMOSYNE, PYTHIA, YGGDRASIL)|
|**H3: Institutional Intelligence**   |Years 3–4  |Platform that improves with every requirement. Unknown unknown detection. Temporal forecasting.          |TRS/CHRONOS · CASSANDRA · ARGUS-LACUNA · LACHESIS · JANUS · SENTINEL                                                                         |

-----

### Slide 21 — Policy Dependencies: 8 Open Items

**Eight General Counsel / IOB decisions gate specific platform capabilities. None are engineering decisions.**

|Ref     |Decision Required                                                                                                                                             |Gates                       |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|
|**GC-1**|Derived information remediation procedure                                                                                                                     |Access control Phase 3–4    |
|**GC-2**|Retrieval gap indicator disclosure acceptability                                                                                                              |ARGUS-LACUNA analyst surface|
|**GC-3**|Query-type authorization taxonomy                                                                                                                             |PCES classifier training    |
|**GC-4**|Within-requirement role-tier access policy                                                                                                                    |Phase 1–2 tier deployment   |
|**GC-5**|Definition of “confirmed” / “disconfirmed” for calibration — edge cases: partial accuracy, overtaken by events, outcomes above assessment classification level|OFS/NEMESIS deployment      |
|**GC-6**|Consumer package content policy — falsification indicator handling, classification relative to underlying product                                             |PCS/IRIS deployment         |
|**GC-7**|Partner registry governance — creation authority, partner agreement requirements, review cadence                                                              |CLS/PROTEUS deployment      |
|**GC-8**|Forecast product governance — labeling, accuracy measurement for probabilistic assertions, outcome feedback framework                                         |TRS/CHRONOS deployment      |

**GC-1 through GC-4 must be initiated before Phase 3–4 begins. GC-5 through GC-8 must be initiated before Year 2 Q2.**

-----

## SECTION 10: THE ASK

*Slides 22–24 · Investment, risk, and next steps*

-----

### Slide 22 — Investment Summary

**Three-horizon investment profile.**

|Horizon      |Timeline   |Primary Investment                                     |Output                                                                                                                            |
|-------------|-----------|-------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
|**Horizon 1**|Months 1–17|Team assembly + Phase 1–8 governance build             |22 platform services operational. Full accountability infrastructure. Agent-native capability bounding. Answerable to oversight.  |
|**Horizon 2**|Years 2–3  |Intelligence cycle completion + intelligence layer     |Full cycle governed from requirements through policymaker. Institutional intelligence flywheel beginning. Calibration loop closed.|
|**Horizon 3**|Year 4+    |Intelligence layer maturity + unknown unknown detection|Platform that improves with every requirement. Strategic warning capability. Gap widens every year vs. ungoverned alternatives.   |

**The most significant cost is not the technology. It is the team.** 21 specialized roles — calibration scientists, media forensics engineers, adversarial ML specialists, coalition liaison policy architects — are not commodity hires. Hiring begins immediately and is treated as a pre-Phase-1 dependency.

-----

### Slide 23 — Risk of Delay

**The argument is not that THEMIS prevents all AI-related analytical failures. It is that the alternative is ungoverned exposure with accumulating cost.**

|Risk                                                  |Without THEMIS                           |With THEMIS                                            |
|------------------------------------------------------|-----------------------------------------|-------------------------------------------------------|
|Fabricated source citation reaches policymaker        |Detected post-dissemination or not at all|Blocked at CVS/VERITAS before analyst sees it          |
|Analyst acts on superseded intelligence               |No currency signal                       |TVS/KAIROS decay + KCS invalidation propagated         |
|Adversary corpus poisoning reaches analysis           |No detection layer                       |IAS/SCUDO screening at ingestion                       |
|Adversarial competence probing generates confabulation|No detection                             |IAS/SCUDO probe detection layer (Year 2)               |
|Coalition partner content mishandled                  |No handling constraint enforcement       |CLS/PROTEUS releasability enforcement                  |
|Oversight body requests AI usage audit                |No defensible record                     |Cryptographic chain audit from inception               |
|Junior analyst systematically over-reliant            |No measurement                           |TCS calibration → supervisory dashboard                |
|AI agent takes irreversible action                    |No recovery path                         |RSS/ROLLBACK 72h snapshot + single-command rollback    |
|Intelligence failure traced to AI calibration gap     |No feedback loop                         |OFS/NEMESIS closes calibration loop from outcomes      |
|Policymaker receives unexplained confidence score     |No consumer translation                  |PCS/IRIS consumer package with falsification indicators|

-----

### Slide 24 — Recommended Next Steps

**Four decisions required to initiate this program.**

**Decision 1 — Program Authorization**  
Authorize THEMIS with Horizon 1 funding and a commitment to review Horizon 2 at the Phase 7–8 gate.

**Decision 2 — Team Assembly**  
Begin hiring the AI Trust Cell immediately. Target full assembly before Phase 1 begins. Treat as a critical path dependency, not a parallel track.

**Decision 3 — GC / IOB Engagement**  
Initiate GC-1 through GC-4 before Phase 3–4 begins. Initiate GC-5 through GC-8 before Year 2 Q2. Designate IOB chair and constitute the board before Phase 1 closes.

**Decision 4 — Apps Org Organizational Design**  
THEMIS lives in the Apps Org. Three structural requirements must be established before Phase 1:

- Research and Red Team charter protection (ring-fenced headcount, not available for sprint work, requires above-Apps-Org-VP authority to override)
- Data Org dependency agreements covering shared infrastructure (Kafka, vector stores, graph database)
- Cross-organizational governance authority chartered above both Apps Org and Data Org — THEMIS must be able to govern AI use in the Data Org, not just the Apps Org

-----

## APPENDIX

*Reference slides — available for Q&A or follow-on briefings*

-----

### Appendix A — Full Service Reference (All 22 Platform + 13 Intelligence Layer)

*Service descriptions, namespace assignments, phase sequencing, and integration maps for all 35 THEMIS services. Reference: THEMIS Platform Design v1.0 + Addenda A–F.*

-----

### Appendix B — ATHENA Intervention Catalog (All 31)

*Full intervention descriptions, behavioral science mechanisms, THEMIS integration matrix (31 × 22 services). Reference: ATHENA Intervention Catalog v3.2.*

-----

### Appendix C — Intelligence Layer Specifications

*Full specifications for SCRIBE, STOA, ORACLE, MIRROR, MNEMOSYNE, PYTHIA, OGS/YGGDRASIL, TRS/CHRONOS, MOS/SAGA, and the five Unknown Unknown services. Data floor requirements and sequencing.*

-----

### Appendix D — Access Control Architecture Detail

*Five access control problems, seven service enhancements, implementation sequencing, GC-1 through GC-4 open items. Reference: THEMIS Addendum B.*

-----

### Appendix E — Competence Axis (Proposed Fourth Axis)

*Six failure modes, CPS/APORIA and ODS/LETHE service specifications, three-phase advisory-to-enforcement implementation posture, six open research questions. Reference: THEMIS Addendum D.*

-----

### Appendix F — Agent-Native Infrastructure Detail

*SCBS/SENTINEL-CAP, CBS/BROKER, RSS/ROLLBACK — full specifications, capability envelope design, credential isolation model, rollback architecture. Reference: THEMIS Addendum E.*

-----

### Appendix G — Intelligence Cycle Integration Detail

*OFS/NEMESIS, UQS/TYCHE, PCS/IRIS, CLS/PROTEUS, TIS/DIKE, TRS/CHRONOS — full specifications, two new namespace designs, GC-5 through GC-8 policy dependencies. Reference: THEMIS Addendum F.*

-----

### Appendix H — Training Design

*Six-module ATHENA analyst training program: foundation (pre-platform), orientation, pressure mode operations, advanced calibration, supervisory, and IOB operations. Three hard training problems: pressure mode resistance, Intention Gate genuine vs. performative, gaming detection as developmental. Reference: ATHENA Training Design v1.0.*

-----

### Appendix I — Feasibility and Risk Assessment

*Honest timeline assessment, technical risk areas (TCS/MIMIR calibration methodology, IAS/SCUDO classifier training data, MAS/EIDOLON forensics scope, OFS/NEMESIS outcome definition policy), de-risking approaches.*

-----

### Appendix J — Competitive and Adversarial Landscape

*Why external vendor solutions cannot replicate the intelligence layer (built on agency-specific analytical history). Adversary exploitation of AI analytical workflows: corpus poisoning, prompt injection, adversarial competence probing. Current threat landscape.*

-----

*THEMIS Briefing Deck Outline v2.0 · May 2026 · Reflects Platform Design v1.0 + Addenda A–F*  
*Companion documents: THEMIS trust-cell-reference.html · ATHENA Intervention Catalog v3.2 · Training Design v1.0*
