# THEMIS Briefing Deck — Outline

## Trusted Human-AI Enablement for Mission Intelligence and Safety

**Deck purpose:** Executive briefing — the problem, the solution, the capability, and the investment required  
**Audience:** Senior leadership / decision-makers  
**Recommended length:** 20 slides + appendix  
**Tone:** Urgent, specific, credible — not aspirational

-----

## SECTION 1: THE PROBLEM

*Slides 1–5 · Establish the burning platform before presenting the solution*

-----

### Slide 1 — Title Slide

**THEMIS**  
Trusted Human-AI Enablement for Mission Intelligence and Safety

*Governing AI in the Intelligence Analysis Workflow*

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

> **Speaker note:** This is not a future risk. Every production slide in this deck describes a failure mode that is happening now, in this workflow, without detection. The question is not whether to address it. The question is what the cost of delay is.

-----

### Slide 3 — The Four Failure Modes

**These are not theoretical. They emerge structurally from how LLMs behave in analytical workflows.**

|Failure Mode                   |Description                                                                     |Current Detection|
|-------------------------------|--------------------------------------------------------------------------------|-----------------|
|**Compartment Leakage**        |RAG tools surface higher-classification content in lower-classification sessions|None             |
|**Fabricated Source Reporting**|AI generates plausible-sounding source references that do not exist             |None             |
|**Stale Intelligence**         |AI presents superseded assessments with current-intelligence confidence         |None             |
|**Uncalibrated Trust**         |Junior analysts accept AI output at higher rates than accuracy warrants         |None             |


> **Speaker note:** Each of these is a structural property of how LLMs work without governance infrastructure — not a product defect that can be patched. They require infrastructure, not configuration.

-----

### Slide 4 — The Adversarial Dimension

**Foreign intelligence services know we are using AI-assisted analysis. They are designing against it.**

- **Corpus poisoning:** Seeding open-source and third-party reporting with plausible but fabricated intelligence that is likely to be ingested into analytical AI systems
- **Prompt injection:** Embedding adversarial instructions in documents likely to be retrieved in AI-assisted research workflows
- **Anchoring exploitation:** Targeting AI’s tendency to anchor analytical outputs on early retrieved information — a predictable and exploitable bias

**An ungoverned AI analytical workflow is a collection vector, not just an analytical risk.**

> **Speaker note:** IAS/SCUDO (the adversarial input screening service in THEMIS) was designed specifically because deadline-pressure periods — when AI adoption is highest — are also when adversary D&D operations are most active. These are not independent phenomena.

-----

### Slide 5 — The Cost of Inaction

**Every day of ungoverned AI use generates compounding liability.**

- Analytical assessments built on AI output with no provenance record cannot be defended to oversight
- Oversight bodies are actively developing AI accountability frameworks — agencies without infrastructure will be in remediation mode under scrutiny when those frameworks arrive
- Analysts building calibration intuitions about AI behavior on ungoverned systems are learning the wrong model — and that miscalibration is difficult to correct later
- The data needed to power the intelligence layer (assessment history, source reliability corpus, correction patterns) only accumulates if the governance infrastructure is capturing it

**The cost of building THEMIS in Year 3 is higher than the cost of building it in Year 1 — not just because of incidents, but because of the data and calibration value that is irretrievably lost.**

-----

## SECTION 2: THE FRAMEWORK

*Slides 6–7 · Three questions every AI-assisted analytical interaction must answer*

-----

### Slide 6 — Three Accountability Axes

**Every AI-assisted analytical interaction must be accountable across three independent dimensions.**

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ORIGIN              CURRENCY              TRUST               │
│                                                                 │
│   Where did this      Is this intelligence  Is the analyst's    │
│   claim come from?    still valid?          reliance calibrated? │
│                                                                 │
│   Retrieved?          Source current?       Right reliance       │
│   Parametric?         Model unchanged?      for actual           │
│   Synthesized?        Corroborated?         AI accuracy?         │
│   Derived?                                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**These axes are independent. Passing two does not protect against failing the third.**

-----

### Slide 7 — Why Independence Matters

**Each pairing failure mode illustrates a distinct and real risk.**

> **Origin without Currency:**  
> Provenance is perfect. The claim is retrieved from a verified source. The source was superseded eight months ago. The assessment is provenance-perfect and lethally stale.

> **Currency without Trust:**  
> Sources are current and corroborated. The analyst is systematically over-reliant on AI in this domain because their calibration has never been measured. The picture is current; the judgment is degraded.

> **Trust without Origin:**  
> The analyst is well-calibrated and appropriately skeptical. The interface does not show that this specific claim was synthesized across partial evidence with no single source. The calibration had nothing to operate on.

**THEMIS instruments all three axes, independently, continuously.**

-----

## SECTION 3: THE SOLUTION — THEMIS

*Slides 8–10 · What the platform is and how it is structured*

-----

### Slide 8 — THEMIS: The Governance Platform

**13 integrated services across four namespaces, governing every AI-assisted analytical interaction from ingestion to dissemination.**

```
SAFETY GATES          CORE INFRASTRUCTURE       QUALITY LAYER
──────────────        ───────────────────────   ──────────────────────────────────
PCES / AEGIS          Provenance / MOIRAI       FGTS / ALETHEIA
Compartment &         Cryptographic             Feedback & Ground Truth
Classification        Provenance                (Calibration Engine)
Enforcement
                      TCS / MIMIR               TVS / KAIROS
PGS / NOMOS           Trust Calibration         Temporal Validity
Analytic Standards    Service                   Service
& Policy
                      DPS / CODEX               RQS / HERMES
                      Assessment                Retrieval Quality
                      Provenance                Service
                                                
INTELLIGENCE LAYER    CVS / VERITAS             IAS / SCUDO
──────────────        Source                    Adversarial Input
KCS / ARGUS           Corroboration             Screening
Knowledge Currency
                      MDS / KRONOS              MAS / EIDOLON
ERAS / LOGOS          Model Drift               Media Authenticity
Reasoning Audit       Detection                 Service
```

-----

### Slide 9 — The Three Accountability Axes: Service Mapping

**Each axis is served by a dedicated set of platform services.**

|Axis        |Primary Services          |What They Do                                                                                                              |
|------------|--------------------------|--------------------------------------------------------------------------------------------------------------------------|
|**ORIGIN**  |MOIRAI · PCES · ERAS · DPS|Capture and surface the provenance of every claim — retrieved, parametric, synthesized, or derived                        |
|**CURRENCY**|TVS · KCS · MDS · CVS     |Track temporal validity, detect external invalidation, monitor model version drift, corroborate sources at generation time|
|**TRUST**   |TCS · FGTS · RQS          |Measure per-analyst calibration, accumulate weighted ground truth, monitor retrieval quality                              |


> **Speaker note:** The services shown here are not a product catalog. They are the minimum viable infrastructure to instrument the three axes. The absence of any one of them creates an accountable gap in one of the three dimensions.

-----

### Slide 10 — What Makes THEMIS Defensible

**Cryptographic attestation: the difference between audit logs and audit records.**

- Every THEMIS event is signed with a per-service cryptographic key
- SHA-256 hash chaining: any retroactive modification breaks the chain from that point forward and is immediately detectable
- RFC 3161 timestamp authority anchoring every 24 hours: external, independently verifiable reference point
- Full chain audit certificate: oversight-submissible proof that the provenance record has not been altered since inception

**An ungoverned AI analytical record can be questioned. A THEMIS record can be verified.**

> **Speaker note:** This is the specific capability that transforms THEMIS from a governance tool into an accountability infrastructure. When an oversight body asks “what did the AI produce, and has that record been modified,” THEMIS can answer that question with cryptographic certainty. No other current approach can.

-----

## SECTION 4: THE INTERFACE — ATHENA

*Slides 11–12 · What analysts actually see and interact with*

-----

### Slide 11 — ATHENA: 31 Interventions Across the Intelligence Lifecycle

**ATHENA is the analyst-facing interface that operationalizes all three accountability axes through 31 targeted behavioral interventions.**

The interventions are not features. Each one targets a specific cognitive failure mode documented in the human factors and intelligence analysis literature.

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

**The most critical: the Intention Gate, Source Type Badges, the Verification Queue, and the Deadline-Critical Hard Constraints.**

-----

### Slide 12 — ATHENA: Key Interface Elements

**Four elements that represent the core of the Origin, Currency, and Trust axes at the analyst interface.**

**The Intention Gate** *(Origin + Trust)*  
Before the AI response renders, the analyst declares their prior belief and the evidence that would change it. The response cannot appear until both are submitted. Counteracts anchoring — the analyst’s prior is set before the AI’s output becomes the reference point.

**Source Type Badges** *(Origin)*  
Every claim in every AI response carries a visible badge: GRND, PARAM, SYNTH, TRANSCRIPT, VIDEO, AUDIO, IMAGE, OCR. Each badge shows where the claim came from and what verification action is appropriate. The analyst cannot mistake a parametric claim for a retrieved one.

**The Verification Queue** *(Trust)*  
Unverified claims accumulate in a persistent visible queue. Dissemination is blocked when high-confidence unverified claims remain. The queue makes accountability structural rather than aspirational.

**Deadline-Critical Hard Constraints** *(Trust + adversarial defense)*  
When DC pressure mode is declared: PARAM claims block dissemination until explicitly actioned; IAS/SCUDO screening sensitivity elevates one tier. Non-dismissable. These constraints exist because deadline periods are when AI-assisted analysis is most operationally critical and most adversarially targeted.

-----

## SECTION 5: THE INTELLIGENCE LAYER

*Slides 13–14 · The compounding advantage*

-----

### Slide 13 — From Governance to Intelligence

**The governance layer makes AI use safe. The intelligence layer makes the investment compound.**

Six Year 2–3 services transform THEMIS from a governance platform into an institutional intelligence asset:

|Service      |Capability                                                         |Requires                 |
|-------------|-------------------------------------------------------------------|-------------------------|
|**SCRIBE**   |Semantic version diff with intelligence significance awareness     |MOIRAI + TVS             |
|**STOA**     |Multi-step research orchestration with documented methodology trail|RQS + ERAS               |
|**ORACLE**   |Predictive intelligence from historical requirement outcomes       |200+ requirements        |
|**MIRROR**   |Similar requirement identification across 6 dimensions             |50+ requirements         |
|**MNEMOSYNE**|Institutional knowledge graph from corrections and outcomes        |FGTS + ORACLE + MIRROR   |
|**PYTHIA**   |Anticipatory research surfacing before the analyst knows to ask    |MIRROR + MNEMOSYNE + STOA|

**MOS/SAGA** manages the memory architecture: session continuity, matter knowledge graduation, and agent task memory across the intelligence layer.

-----

### Slide 14 — The Flywheel

**Every requirement processed through THEMIS adds to the institutional intelligence base.**

```
Every assessment → MIRROR adds a comparable requirement
Every correction → FGTS enriches the source reliability corpus  
Every outcome → ORACLE refines its prediction model
Every analyst session → MNEMOSYNE extracts tacit knowledge
Every similarity match → PYTHIA improves its anticipation accuracy
```

**At Year 4, the agency has a platform that gets smarter with every requirement it handles.**

Peer agencies using generic AI tools start from zero on each requirement.  
The agency using THEMIS starts from the accumulated knowledge of every requirement it has ever handled.

**That gap widens every year. It cannot be closed by any external vendor or adversary — because it is built on the agency’s own data.**

> **Speaker note:** The data floor is real. ORACLE needs 200+ requirements before its predictions are statistically meaningful. The organizations that start building the governance infrastructure now will have that data when the intelligence layer is ready to use it. The organizations that wait will be starting from zero on the data AND the infrastructure simultaneously.

-----

## SECTION 6: THE TEAM

*Slide 15 · Who builds and sustains THEMIS*

-----

### Slide 15 — The AI Trust Cell: 21 People, Three Teams

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

**Monthly evaluation report delivered simultaneously to Cell Lead and CTO.** Not filtered. Not delayed. The people accountable for outcomes see quality problems at the same time as the people accountable for fixing them.

-----

## SECTION 7: THE ROADMAP

*Slides 16–17 · Phased implementation*

-----

### Slide 16 — 17-Month Implementation Roadmap

**Four phases. Each phase gate requires Cell Lead + Engineering Lead + Intelligence ML Lead joint confirmation before the next phase is funded.**

|Phase        |Weeks|Services                                                  |Phase Gate                                                                                                                                                                               |
|-------------|-----|----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**Phase 1–2**|1–8  |PCES/AEGIS · PGS/NOMOS                                    |No AI interaction touches source intelligence without compartment enforcement and analytic standards guardrails. Classification log searchable. Intelligence Oversight Board constituted.|
|**Phase 3–4**|9–28 |MOIRAI · TCS · CVS · MDS · DPS + cryptographic attestation|Complete tamper-evident chain of custody for every AI analytical interaction. Answerable to any oversight inquiry.                                                                       |
|**Phase 5–6**|29–46|FGTS · TVS · RQS · IAS · MAS                              |Quality feedback loops live. Adversarial input screening active. Media authenticity across all source modalities.                                                                        |
|**Phase 7–8**|47–66|KCS · ERAS + TVS scope extension                          |Full ICD 203 analytic standards audit surface. Oversight-submissible provenance from platform inception.                                                                                 |

**Year 2–3:** Intelligence layer services deployed as data floor is reached.

-----

### Slide 17 — Access Control and Memory Architecture

**Two addenda address critical capabilities that span the full implementation timeline.**

**Addendum B: Access Control and Compartmentalization**  
Five access control problems addressed through enhancements to existing services — no new services required:

- Derived information persistence (what happens when access to source evidence is revoked after it has been synthesized)
- Retrieval coverage gaps (surfacing the existence of relevant evidence the analyst cannot access)
- Query-layer access control (queries as analytic work product, cross-requirement contamination detection, query-type authorization)
- Within-requirement access granularity (protective designation tiers, privilege log generation)
- Temporal and situational access (time-limited grants, break-glass emergency access, historical access reconstruction)

**Addendum C: Memory Architecture — MOS/SAGA**  
Cross-session continuity, matter knowledge layer, and agent task memory — the analytical working relationship persists across sessions, requirements, and teams.

-----

## SECTION 8: THE ASK

*Slides 18–20 · Investment, risk, and next steps*

-----

### Slide 18 — Investment Summary

**Three-horizon investment profile aligned to the phased roadmap.**

|Horizon      |Timeline   |Primary Investment                             |Output                                                                                                          |
|-------------|-----------|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------|
|**Horizon 1**|Months 1–17|Team assembly + Phase 1–8 platform build       |Governed AI analytical platform. All 13 services operational. Full accountability infrastructure.               |
|**Horizon 2**|Years 2–3  |Intelligence layer services + data accumulation|Institutional intelligence flywheel. ORACLE, MIRROR, MNEMOSYNE, PYTHIA delivering compounding value.            |
|**Horizon 3**|Year 4+    |Sustained operation + continuous research      |Platform improving with every requirement. Calibration accuracy increasing. Institutional knowledge compounding.|

**The most significant cost is not the technology. It is the team.** 21 specialized roles across research, platform engineering, ML engineering, media forensics, and calibration science are not commodity hires. The hiring timeline should begin immediately and treated as a pre-Phase-1 dependency.

> **Speaker note:** The feasibility assessment is honest: Phase 1–8 in 17 months is achievable but requires the team to be assembled and productive before Phase 1 begins. If hiring takes 6–12 months — a realistic estimate for these roles — the effective start of Phase 1 is that far out. Planning should reflect this.

-----

### Slide 19 — Risk of Delay

**The argument is not that THEMIS prevents all AI-related analytical failures. It is that the alternative is ungoverned exposure with accumulating cost.**

|Risk                                           |Without THEMIS                           |With THEMIS                                         |
|-----------------------------------------------|-----------------------------------------|----------------------------------------------------|
|Fabricated source citation reaches policymaker |Detected post-dissemination or not at all|Blocked at CVS/VERITAS before analyst sees it       |
|Analyst acts on superseded intelligence        |No currency signal available             |TVS/KAIROS decay + KCS invalidation event propagated|
|Adversary corpus poisoning reaches analysis    |No detection layer at ingestion          |IAS/SCUDO screening at retrieval ingestion          |
|Oversight body requests AI usage audit         |No defensible record available           |Cryptographic chain audit from platform inception   |
|Junior analyst systematically over-relies on AI|No measurement, no signal to supervisor  |TCS calibration surfaces to supervisory dashboard   |

**Every month of delay is a month during which these risks are undetected and unmitigated.**

-----

### Slide 20 — Recommended Next Steps

**Three decisions required to initiate this program.**

**Decision 1 — Program Authorization**  
Authorize the THEMIS program with Phase 1–2 funding and a commitment to review Phase 3–4 at the Phase 1–2 gate.

**Decision 2 — Team Assembly**  
Begin hiring the AI Trust Cell immediately. Target full team assembly before Phase 1 begins. Treat hiring as a critical path dependency, not a parallel track.

**Decision 3 — General Counsel / Inspector General Engagement**  
Four policy decisions require General Counsel / IG input before Phase 3–4 begins:

- GC-1: Derived information remediation procedure (inadvertent disclosure recovery)
- GC-2: Retrieval gap indicator disclosure acceptability
- GC-3: Query-type authorization taxonomy
- GC-4: Within-requirement role-tier access policy

These are not engineering decisions. They are institutional policy decisions that must be made before the relevant platform capabilities are built. Engaging GC and IG now — before Phase 3–4 — preserves the Phase timeline.

-----

## APPENDIX

*Reference slides — available for Q&A or follow-on briefings*

-----

### Appendix A — THEMIS Service Reference (All 13)

*Full service descriptions, storage architecture, latency targets, phase assignments, and integration maps. Reference: THEMIS Platform Design v1.0.*

-----

### Appendix B — ATHENA Intervention Catalog (All 31)

*Full intervention descriptions, behavioral science mechanisms, THEMIS service integrations, and THEMIS integration matrix (31 × 20 services). Reference: ATHENA Intervention Catalog v3.2.*

-----

### Appendix C — Intelligence Layer Service Specifications

*Full specifications for SCRIBE, STOA, ORACLE, MIRROR, MNEMOSYNE, PYTHIA, and MOS/SAGA. Reference: THEMIS Intelligence Layer Proposal.*

-----

### Appendix D — Access Control Architecture Detail

*Five access control problems, seven service enhancements, implementation sequencing, and four GC/IG open items. Reference: THEMIS Addendum B.*

-----

### Appendix E — Feasibility and Risk Assessment

*Honest assessment of timeline feasibility, technical risk areas (TCS/MIMIR calibration methodology, IAS/SCUDO classifier training data requirements, MAS/EIDOLON media forensics scope), and recommended de-risking approaches.*

-----

### Appendix F — Competitive and Adversarial Landscape

*Why external vendor solutions cannot replicate the intelligence layer (it is built on agency-specific data). Adversary exploitation of AI analytical workflows: current threat landscape.*

-----

*THEMIS Briefing Deck Outline · Designed for Doubt Series · For further detail see the THEMIS Platform Reference Document*
