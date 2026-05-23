# THEMIS Briefing Deck — Outline
## Trusted Human-AI Enablement for Mission Intelligence and Safety

**Deck purpose:** Executive briefing — the problem, the solution, the capability, and the investment required
**Audience:** Senior leadership / decision-makers
**Length:** 25 slides
**Tone:** Urgent, specific, credible — not aspirational
**Last updated:** May 2026 — 42-service platform + Addenda A–F + Intelligence Layer + HADES + FGS/PLUTUS
**Key references:** Taddeo et al. (CETaS, September 2022) · Knack, Carter & Babuta (CETaS, December 2022) · Gruber (Daring Fireball, May 2026)

---

## SECTION 1: THE PROBLEM
*Slides 1–7 · Establish the burning platform before presenting the solution*

---

### Slide 1 — Title Slide

**THEMIS**
Trusted Human-AI Enablement for Mission Intelligence and Safety

*Governing AI Across the Full Intelligence Cycle*

[Agency / Organization]
[Date]
[Classification marking]

---

### Slide 2 — AI Is Infrastructure. Ungoverned Infrastructure Is a Liability.

**The question is not whether AI will pervade analytical work. It already does. The question is whether the infrastructure governing it will keep up.**

John Gruber, writing at Daring Fireball in May 2026, draws the framing analogy precisely:

> Wireless networking is pervasive. But [no organisation] has "a killer wireless networking product." Wireless networking simply pervades everything… There was a time, not too long ago, when [organisations] didn't make a single product with wireless connectivity. Now it's pervasive in all their devices. That's more what AI is going to be like. There's not going to be one "killer AI device." Everything is going to be an AI device, to some extent, just like how everything today is a wireless connectivity device, to some extent.
> — Gruber, J. "AI Is Technology, Not a Product." *Daring Fireball*, 16 May 2026. https://daringfireball.net/2026/05/ai_is_technology_not_a_product

The THEMIS proposition: **AI will pervade every stage of the analytical workflow as infrastructure, not a product.** When wireless networking became pervasive, organisations built security protocols, encryption standards, and access controls. When AI becomes pervasive in intelligence analysis — as it already is — organisations need provenance infrastructure, calibration systems, and adversarial defence. Not a killer AI product. The governance layer for AI-as-technology.

**That governance layer is THEMIS. The AI is already there. The infrastructure is not.**

Current ungoverned conditions:
- Analysts using AI-assisted research, LLM drafting, and AI-summarized source review **daily** — with or without authorization
- No mechanism to distinguish AI-retrieved claims from AI-hallucinated claims
- No mechanism to surface whether underlying sources have been superseded
- No mechanism to measure whether analyst reliance on AI is appropriate for actual AI accuracy
- No feedback loop from real-world outcomes back to calibration
- No AI contribution record defensible to oversight bodies

> **Speaker note:** Use the Gruber framing early when briefing audiences fatigued by AI product hype. The pitch is: "We are not here to pitch an AI product. We are here to pitch the governance infrastructure for the AI technology that is already in your analysts' hands." This reframes from "are you ready to bet on AI?" to "are you governing the AI your analysts are already using?" Citation: Gruber, J. (2026, May 16). AI is technology, not a product. *Daring Fireball.* https://daringfireball.net/2026/05/ai_is_technology_not_a_product

---

### Slide 3 — Research Foundation I: The Predictability Problem

**CETaS Research Report · Taddeo, Ziosi, Tsamados, Gilli & Kurapati · September 2022**
*Artificial Intelligence for National Security: The Predictability Problem*
Centre for Emerging Technology and Security, The Alan Turing Institute · https://cetas.turing.ac.uk

**The dual predictability problem — precise definition:**

| Definition | The Problem | THEMIS Response |
|---|---|---|
| **Maximal** | Given the multi-faceted processes of design, development, and deployment of AI systems — their opaqueness, adapting capabilities, and complex deployment environments — it is not possible to account for all sources of errors, manipulation, or emergent behaviours, whether beneficial or not | MDS/KRONOS model drift · ODS/LETHE OOD detection (proposed) · HADES adversarial pattern repository |
| **Minimal** | Even assuming no design or development errors, once deployed an AI system may still produce formally correct but unwanted outcomes that were not foreseeable at the time of deployment | TCS/MIMIR calibration from what actually happens · FGTS correction corpus · OFS/NEMESIS outcome feedback |

*Crucially: the predictability problem refers to both correct and incorrect outcomes — the issue is not whether outcomes follow logically from the system's workings, but whether they can be foreseen at deployment.*

**The governance unit must be the human-machine team, not the AI system alone:**
Taddeo et al. find that HMT-AI exacerbates the predictability problem by increasing the number and types of interactions between artificial and human agents and their environment — multiplying perturbation sources. Recommendation 3 is explicit: governance must focus on the team, not the AI system in isolation. THEMIS governs the full analytical interaction — every session, every claim, every correction.

**"Trust and forget" dynamics — documented operational risk:**
Over-trust generates dynamics in which analysts no longer supervise AI performance — overlooking potentially erroneous actions, disregarding capability limits, accepting outcomes uncritically. The report cites a 2016 study in which nearly 90% of 42 participants in a simulated fire emergency followed a robot guide blindly even as it committed repeated fatal mistakes for which it offered no explanation or warning. In national security analysis, these dynamics produce the same failure mode at far higher stakes.

**The duty of care principle:**
Taddeo et al. state it directly: the higher the impact of the decisions an AI system supports, the greater the duty of care on those designing, developing, and using it — and the lower the acceptable risk threshold. Intelligence analysis is among the highest-stakes decision contexts. The duty of care ceiling is correspondingly low.

**The known knowns / known unknowns / unknown unknowns risk taxonomy:**
The report proposes that building robustness against risks we are aware of but cannot predict (known unknowns) should precede addressing risks we can predict, and that resilience against potentially catastrophic unknown-unknown failures should be the first priority. This maps to the THEMIS warning layer: SWS/SENTINEL converts unknown unknowns into known unknowns; CGS/ARGUS-LACUNA distinguishes absence-of-collection from absence-of-evidence; UCS/TYCHE decomposes uncertainty by type — aleatory (unknown unknowns), epistemic (known unknowns), model (known knowns with limits).

> **Speaker note:** Taddeo et al. Recommendation 8 proposes an ALARP (as low as reasonably practicable) framework — adapted from safety-critical engineering — for assessing AI risk in national security. Its four prescriptions are: identify hazards; analyse the hazard and determine the risk; take actions to reduce the risk by designing it out or providing controls; demonstrate to regulators that residual risks are tolerable and ALARP. THEMIS's 9 control gates, fail-closed architecture, and IOB threshold authority are the operational implementation of this framework.

*(Taddeo, M., Ziosi, M., Tsamados, A., Gilli, L., & Kurapati, S. (2022). Artificial Intelligence for National Security: The Predictability Problem. CETaS Research Report. The Alan Turing Institute. September 2022.)*

---

### Slide 4 — Research Foundation II: System-Level Trust Requirements

**CETaS Research Report · Knack, Carter & Babuta · December 2022**
*Human-Machine Teaming in Intelligence Analysis: Requirements for developing trust in machine learning systems*
Centre for Emerging Technology and Security, The Alan Turing Institute · https://cetas.turing.ac.uk

18 in-depth interviews with operational intelligence analysts, legal experts, behavioural scientists, and human factors engineers in UK national security. The operational design specification for THEMIS.

| CETaS Finding | What It Requires | THEMIS Implementation |
|---|---|---|
| **Finding 4 — System-level trust** | Trust in ML requires trust in the output *and* the whole system — prior performance, formal approval, task nature all matter | Full 42-service governance platform |
| **Finding 12 — System-level approach** | Technical explainability alone is insufficient — governance, training, and infrastructure together | AI Trust Cell + 9 control gates + training programme |
| **Section 3.2.3 — Audit trail** | Oversight bodies need to know who accessed what data, when, for what reason, and how conclusions were reached | MOIRAI cryptographic chain — every event signed, hashed, tamper-evident |
| **Finding 10 — Standardised language** | Confidence language must be standardised across the national security community | UCS/TYCHE structured uncertainty decomposition · PGS/NOMOS ICD 203 compliance |
| **Section 3.2.1 — Data drift** | Model performance degradation when deployed on data outside training distribution must be tracked | MDS/KRONOS model drift · KCS/ARGUS model knowledge currency |
| **Finding 5 — Workflow integration** | ML must be designed into the analyst's natural workflow from the outset — not bolted on | ATHENA 31-intervention architecture embedded in every session |
| **Section 2.2 — Memory reinstatement** | An ML teammate should "reinstate memory and help the analyst recall strands of the big picture after the weekend" | MOS/SAGA session and matter memory (Year 2) |
| **Finding 8 — Analyst involvement** | Analysts must be included in prototyping and testing — mandatory, not optional | R&RT evaluation programme · STOA mandatory analyst review checkpoints |

**The study found no evidence of systems meeting these requirements currently in operational use.**

*(Knack, A., Carter, R.J., & Babuta, A. (2022). Human-Machine Teaming in Intelligence Analysis: Requirements for developing trust in machine learning systems. CETaS Research Report. The Alan Turing Institute. December 2022.)*

---

### Slide 5 — The Six Structural Failure Modes

**These are not theoretical. They are structural properties of how LLMs behave without governance infrastructure.**

| Failure Mode | Description | Current Detection |
|---|---|---|
| **Compartment Leakage** | RAG tools surface higher-classification content in lower-classification sessions | None |
| **Fabricated Source Reporting** | AI generates plausible-sounding source references that do not exist | None |
| **Stale Intelligence** | AI presents superseded assessments with current-intelligence confidence | None |
| **Uncalibrated Trust** | Junior analysts accept AI output at higher rates than accuracy warrants | None |
| **Adversarial Manipulation** | Adversarially crafted content enters AI context windows undetected | None |
| **Uncertainty Collapse** | Aleatory, epistemic, and model uncertainty reported as a single undifferentiated score | None |

Each failure mode is a structural property, not a product defect. They require governance infrastructure, not configuration.

**The Taddeo et al. technical root cause framework maps directly to each:**

*Fabricated Source Reporting* — a consequence of model overconfidence. Taddeo et al. note that deep neural networks have been proven to be overconfident, potentially leading to high-confidence mistakes. When a model produces a fabricated citation with high expressed confidence, the analyst has no indication that the confidence is unjustified. CVS/VERITAS instruments this directly.

*Stale Intelligence* — what Taddeo et al. term data drift and technical debt: "updates to the system will change the performance of the tool, which may cause it to become unpredictable." The underlying corpus data diverges from the model's training distribution without any signal to the analyst. TVS/KAIROS and MDS/KRONOS instrument this.

*Uncalibrated Trust* — Taddeo et al.'s "trust and forget" dynamics in operational form. Once established, over-trust is self-reinforcing: the analyst who has learned not to scrutinise AI outputs brings that posture to every subsequent interaction. TCS/MIMIR detects and measures this continuously.

*Uncertainty Collapse* — Taddeo et al. document that model confidence is often not statistically robust, and that conflating different uncertainty types misleads both operators and oversight. UCS/TYCHE resolves this by decomposing uncertainty at the claim level.

*Adversarial Manipulation* — Taddeo et al. identify adversarial examples as a root cause of the maximal predictability problem: specification gaming, reward hacking, and pixel-level manipulation that causes AI systems to misidentify objects. IAS/SCUDO and HADES address this.

*(Taddeo et al., CETaS, September 2022 — Sections 3.1, 3.2, 3.3, and 4.3–4.4)*

---

### Slide 6 — The Adversarial Dimension

**Foreign intelligence services know we are using AI-assisted analysis. They are designing against it.**

- **Corpus poisoning:** Seeding open-source and third-party reporting with fabricated intelligence likely to be ingested into analytical AI systems
- **Prompt injection:** Embedding adversarial instructions in documents likely to be retrieved in AI-assisted workflows
- **Adversarial competence probing:** Querying the AI on known capability gaps to generate confident confabulations, then introducing them into the analytical record
- **Anchoring exploitation:** Targeting AI's tendency to anchor analytical outputs on early retrieved information — a predictable and exploitable bias

**An ungoverned AI analytical workflow is a collection vector, not just an analytical risk.**

**What the research documents:** Taddeo et al. establish adversarial examples as a formally documented root cause of the maximal predictability problem. AI systems have been shown to be susceptible to minor input changes — down to the pixel level — causing them to misidentify 3D-printed turtles as rifles or road markings as school buses with high confidence. These vulnerabilities have been exploited in industrial settings and multi-domain defence operations. In intelligence analysis, the same attack surface exists wherever AI processes visual, textual, or audio inputs derived from any source an adversary can influence.

Critically, Taddeo et al. find that model overconfidence can *conceal* ongoing adversarial attacks: deep neural networks express high confidence even while being manipulated, giving no signal to the analyst that the output is compromised.

*THEMIS addresses this across four services:* IAS/SCUDO screens every input and retrieved chunk against an R&RT-maintained adversarial threat catalog. CVS/VERITAS detects corpus poisoning indicators. MAS/EIDOLON assesses media authenticity at ingestion. HADES preserves every adversarial event as institutional intelligence for future defence improvement — including the adversarial reasoning record, the AI's full chain-of-thought during sessions where adversarial content reached the context window.

As AI-as-technology becomes pervasive across the analytical workflow (Gruber, 2026), the adversarial surface expands in proportion. HADES is the institutional infrastructure for building adversarial resilience as that surface grows.

> **Speaker note:** The concealment point is significant for the leadership audience: adversarial manipulation is hardest to detect precisely when the system appears most confident. An analyst relying on expressed confidence as a signal of reliability is most vulnerable at the moment of most successful attack. This is why IAS/SCUDO screens inputs independently of the model's own confidence signals.

---

### Slide 7 — The Cost of Inaction

**Every day of ungoverned AI use generates compounding liability across three dimensions.**

**Institutional liability:**
- Analytical assessments built on AI output with no provenance record cannot be defended to oversight
- Oversight bodies are developing AI accountability frameworks — agencies without infrastructure will be in remediation mode when those frameworks arrive

**Calibration liability:**
- Analysts building calibration intuitions on ungoverned systems are learning the wrong model
- Taddeo et al. warn that "trust and forget" dynamics — once established — are difficult to reverse; operators who have learned to over-trust AI outputs bring that bias to any new system
- Knack et al. Finding 13: data literacy training must precede ML deployment; ungoverned use builds the wrong intuitions first

**Data compounding liability:**
- The intelligence layer accumulates only if governance infrastructure captures data from day one
- Real-world outcome feedback is lost permanently for every requirement processed without OFS/NEMESIS
- MIRROR (50+ requirements), ORACLE (200+ with outcomes), MNEMOSYNE (18 months FGTS data) — these floors cannot be compressed; they can only be started earlier

**The cost of building THEMIS in Year 3 is higher than Year 1** — not just from incidents but from irretrievably lost calibration data and institutional intelligence.

If AI is technology (Gruber, 2026) — already pervasive, already ungoverned — then the governance infrastructure must be built now. You do not retrofit security standards onto a deployed wireless network. You build them in.

---

## SECTION 2: THE FRAMEWORK
*Slides 8–9 · The four accountability axes*

---

### Slide 8 — Four Accountability Axes

**Every AI-assisted analytical interaction must be accountable across three dimensions — with a fourth proposed.**

```
┌─────────────────┬──────────────────┬─────────────────┬──────────────────────┐
│   ORIGIN        │    CURRENCY      │     TRUST        │  COMPETENCE          │
│                 │                  │                  │  (proposed)          │
│ Where did this  │ Is this          │ Is the analyst's │ Can this AI system   │
│ claim come      │ intelligence     │ reliance         │ reliably perform     │
│ from?           │ still valid?     │ calibrated?      │ this task at all?    │
│ Retrieved?      │ Source current?  │ Right reliance   │ In-distribution?     │
│ Parametric?     │ Model unchanged? │ for actual       │ Within capability    │
│ Synthesized?    │ Corroborated?    │ AI accuracy?     │ envelope?            │
│ Derived?        │ Outcome tested?  │                  │ Known boundary?      │
└─────────────────┴──────────────────┴─────────────────┴──────────────────────┘
```

**Why independence is not optional — each pairing failure illustrates a distinct operational risk:**

> **Origin without Currency:** Provenance is perfect. The source was superseded eight months ago. Provenance-perfect and lethally stale.

> **Currency without Trust:** Sources are current. The analyst is systematically over-reliant on AI — Taddeo et al.'s "trust and forget" dynamic in operational form. A 2016 study cited in the report found that nearly 90% of participants in a simulated emergency followed a robot guide blindly even as it committed repeated fatal mistakes. In intelligence analysis at scale, this dynamic is invisible without calibration measurement. Current picture; degraded judgment.

> **Trust without Origin:** The analyst is well-calibrated. The interface hides that this claim was synthesized across partial evidence. The calibration had nothing to operate on.

> **All three without Competence:** Provenance-perfect, current, well-calibrated — the analyst never knew the task exceeded the model's reliable capability envelope. The assessment is confident and wrong.

**The first three axes are instrumented from deployment day one. The Competence axis provides advisory signals (Green/Amber/Red zones) pending ARB approval of hard ceiling enforcement.**

> **Speaker note:** The Knack et al. three-stakeholder framework (Senior Responsible Owner, Intelligence Analysts, Oversight Bodies) maps onto the axes: the Senior Responsible Owner needs Origin and Competence signals; Analysts need Trust and Uncertainty signals; Oversight Bodies need the MOIRAI audit trail that spans all four.

---

## SECTION 3: THE PLATFORM
*Slides 9–14 · 42 services across 11 namespaces*

---

### Slide 9 — THEMIS: 42 Services. Three Layers. One Governance Chain.

**42 approved platform services governing every AI-assisted analytical interaction — from compartment enforcement through strategic warning synthesis.**

```
PLATFORM LAYER (H1, Months 1–17)

SAFETY GATES              CORE INFRASTRUCTURE        QUALITY LAYER (13 services)
─────────────────         ──────────────────────     ────────────────────────────
PCES / AEGIS              MOIRAI                     FGTS / ALETHEIA · TVS / KAIROS
Classification            Cryptographic              RQS / HERMES · CVS / VERITAS
Enforcement               Provenance Backbone        IAS / SCUDO · MAS / EIDOLON
                                                     MDS / KRONOS · KCS / ARGUS
PGS / NOMOS               TCS / MIMIR                ERAS / LOGOS · HADES
Analytic Standards        Trust Calibration          FGS / PLUTUS
& Policy                  (Bayesian posteriors)
                          DPS / CODEX                INTERACTION LAYER
MGS / TERMINUS            Assessment Provenance      ──────────────────
MCP Gateway                                          PRS / PROMETHEUS (Prompts)
                          OFS / NEMESIS              SKS / DAEDALUS (Skills)
                          Outcome Feedback
                          UCS / TYCHE                AGENT-NATIVE
                          Uncertainty                ──────────────
                          Characterization           SCBS · CBS / BROKER · RSS

INTELLIGENCE LAYER (H2–H3, Year 2–4): 15 services across 5 namespaces — see Slide 17
PROPOSED: CPS/APORIA (Capability Profiling) · ODS/LETHE (OOD Screening)
```

**The accountability axis service mapping:**

| Axis | Primary Services |
|---|---|
| **Origin** | MOIRAI · PCES · ERAS · DPS |
| **Currency** | TVS · KCS · MDS · CVS · OFS · SCRIBE |
| **Trust** | TCS · FGTS · RQS · UCS · PRS · SKS |
| **Competence** *(proposed)* | CPS/APORIA · ODS/LETHE |

---

### Slide 10 — What Makes THEMIS Defensible: The Cryptographic Provenance Chain

**The difference between audit logs and audit records — between records that can be questioned and records that can be verified.**

**MOIRAI — the backbone:**
- Every THEMIS event is signed with a per-service HMAC-SHA256 key stored in Vault
- SHA-256 hash chaining: retroactive modification breaks the chain from that point forward — detectable without access to the original
- RFC 3161 timestamp authority anchoring every 24 hours: external, independently verifiable reference point
- Full chain audit certificate: oversight-submissible proof that the provenance record has not been altered since inception

**Why this matters for the predictability problem:**
Taddeo et al. Recommendation 8 defines an ALARP-based framework for AI risk assessment. Its second element is explicit: an assessment of the traceability of the design, development, and procurement steps leading to deployment. Its third element: an assessment of conditions of deployment — level of training of operators, level of transparency of the interface, level of human control. Its sixth element: protocols for human overriding and redress mechanisms.

MOIRAI is the operational implementation of elements two, three, and six together. Every analytical step — what was retrieved, how the AI reasoned, what the analyst verified, what conclusion was drawn, what corrections were applied — is permanently recorded in a chain that cannot be altered retroactively. The audit certificate is the oversight-submissible evidence that the chain is intact.

**An ungoverned AI analytical record can be questioned. A THEMIS record can be verified.**

---

### Slide 11 — Uncertainty Characterization: UCS / TYCHE

**A single confidence score hides the most important distinction decision-makers need to make.**

Three kinds of uncertainty require three qualitatively different analytical and policy responses:

| Uncertainty Type | What It Means | Reducible by | ATHENA Indicator |
|---|---|---|---|
| **Aleatory** | Inherent world-state indeterminacy — the adversary has not decided | Nothing — express as probability distribution | Purple |
| **Epistemic** | Collection gaps that better intelligence could close | Better collection → TIS/DIKE gap pipeline | Amber |
| **Model** | AI capability limitations at this claim type in this domain | Human substitution · CPS zone warning | Blue |
| **Uncharacterised** | Analyst has not yet answered the three UCS questions | Analyst engagement | Grey |

Collapsing these into one score misleads both analysts and decision-makers about what kind of uncertainty they face and what would reduce it.

**The Taddeo et al. connection:** the known/unknown taxonomy directly informs uncertainty type. Aleatory uncertainty represents unknown unknowns — genuinely irreducible. Epistemic uncertainty represents known unknowns — gaps we know exist and can target with collection. Model uncertainty is the competence-axis signal — the capability boundary of the system itself.

**Knack et al. Finding 10:** language for communicating ML confidence should be standardised across the national security community. UCS/TYCHE implements that standardisation at the claim level; PGS/NOMOS enforces ICD 203 compliance throughout.

---

### Slide 12 — Financial Governance and the Adversarial Record

**Two quality-layer services that address distinct but equally critical governance gaps.**

---

**FGS / PLUTUS — Financial Governance & Tokenomics**

AI inference at analytical community scale is a significant, variable operational cost. Without FGS, the organisation cannot answer: How much did the HUMINT analysis team consume last quarter? Which requirement types are most inference-intensive? Are consumption patterns consistent with the analytical workload?

The Tokenomics Model:
- Every team receives a periodic token allocation; every inference call attributed to team, session, analyst, interaction class, and requirement in real time
- SCBS per-session envelopes derive from team account balances — session-level bounds consistent with organisational budgets
- STOA multi-step research: 12–18× baseline single-query cost — FGS makes this visible from session one

**An organisation that cannot answer "what did our AI assistance cost and what did we get for it" cannot sustain the investment. FGS makes the investment legible.**

---

**HADES — Adversarial Intelligence Repository**

Every adversarial event — blocked, bypassed, or identified retrospectively — is preserved permanently. The adversarial reasoning record (the AI's full chain-of-thought during sessions where adversarial content reached the context window) is the crown jewel: it describes not just that manipulation was attempted but *how the model reasoned when being deceived*.

HADES ingests from IAS/SCUDO (blocked injections + retrospective bypasses), ERAS/LOGOS (adversarial reasoning records), CVS/VERITAS (fabrications), MAS/EIDOLON (high-risk media), ADS/CASSANDRA (Year 3). All air-gapped. Research & Red Team and IOB access only.

**HADES produces:** updated IAS/SCUDO threat catalogs, CPS/APORIA capability zone recalibration evidence, de-identified MNEMOSYNE knowledge nodes. All Research & Red Team mediated — no autonomous writes to other services.

---

### Slide 13 — ATHENA: The Analyst Interface

**ATHENA operationalises the four accountability axes through targeted behavioural interventions embedded in every analytical session.**

Each intervention targets a specific cognitive failure mode — including the "trust and forget" dynamic identified by Taddeo et al.

| Category | Count | Failure Mode Targeted |
|---|---|---|
| Session Configuration | 4 | Chat mode, corpus version, parameter transparency |
| Session Scaffolding | 6 | Anchoring, context collapse, strategic framing drift |
| Pre-Response | 2 | Anchoring, confirmation bias pre-emption |
| Response Interventions | 5 | Automation bias, source type opacity |
| Verification Infrastructure | 3 | Vigilance decrement, unverified claim accumulation |
| Behavioural Telemetry | 6 | Invisible miscalibration, gaming detection |
| Session Integrity | 2 | Cross-analyst divergence, accountability |
| **Pressure Response** | **3** | **Calibration degradation under deadline pressure** |

**Five key interface elements and their research grounding:**

| Element | Axis | Research Grounding |
|---|---|---|
| **Intention Gate** — prior belief declared before AI response renders | Origin + Trust | Counteracts anchoring; analyst hypothesis formed before AI output becomes the reference point |
| **GRND / PARAM / SYNTH badges** — on every claim | Origin | Knack et al. Finding 6: accessible plain-English certainty indication so analysts can calibrate confidence |
| **Uncertainty Decomposition Indicator** — three-component display | Trust | Knack et al. Finding 10: standardised confidence language; Taddeo et al.: uncertainty typed not collapsed |
| **Verification Queue** — dissemination blocked on unverified high-confidence claims | Trust | Knack et al.: analysts should never wholly trust AI output without verifying against other information |
| **Memory Context Panel** *(Year 2 — MOS/SAGA)* — prior work summary at session start | Origin + Trust | Knack et al. Section 2.2: ML teammate should "reinstate memory and help the analyst recall the big picture after the weekend" |

---

### Slide 14 — The Intelligence Layer: From Governance to Institutional Intelligence

**The governance layer makes AI use safe. The intelligence layer makes the investment compound. 15 services across five namespaces.**

| Namespace | Services | Function |
|---|---|---|
| **themis-knowledge** | OGS/YGGDRASIL · SCRIBE · MNEMOSYNE · MOS/SAGA | Semantic entity ontology, document change detection, institutional knowledge graph, session memory. Prerequisite for all other namespaces. |
| **themis-research** | STOA · ORACLE · MIRROR · PYTHIA | Multi-step research orchestration with documented methodology trails, outcome prediction, similar requirement matching, anticipatory surfacing. Data floors: 50 reqs (MIRROR), 200+ with outcomes (ORACLE). |
| **themis-warning** | TRS/CHRONOS · ADS/CASSANDRA · CGS/ARGUS-LACUNA · WSF/LACHESIS · CRF/JANUS · SWS/SENTINEL | Temporal trajectory modelling, anomaly detection, collection gap mapping, weak signal fusion, cross-requirement correlation, strategic warning synthesis. Year 3–4. |
| **themis-dissemination** | PCS/IRIS | Policymaker consumer packages — plain-language source basis, falsification indicators, prior outcome record, provenance certificate. GC-6 required. |
| **themis-requirements** | TIS/DIKE | Gap-to-requirement pipeline, ORACLE effectiveness feedback to requirements officers, full CGR lifecycle tracking. |

**The flywheel:** every session → MNEMOSYNE extracts knowledge → ORACLE/MIRROR refine predictions. Every correction → FGTS enriches corpus → calibration improves. Every outcome → OFS/NEMESIS closes the loop → ORACLE + calibration update together.

**The data floor is real and cannot be engineered around.** Agencies that start now will have the data to run all five namespaces. Agencies that wait start from zero on both data and infrastructure simultaneously.

---

### Slide 15 — The Research Teammate: STOA and the Warning Layer

**Two capabilities that directly implement the CETaS research recommendations.**

---

**STOA — Research Orchestration: The ML Teammate, Not Just a Tool**

Knack et al. Section 2.2 distinguishes an ML *tool* (triage, filtering — narrow task) from an ML *teammate* (joint problem-solving across a sustained analytical engagement). Taddeo et al. Recommendation 2 calls for a certification scheme for HMT-AI covering effective communication, performance consistency, and adapting to new teammates.

STOA is the teammate interface. For a complex requirement:
1. STOA proposes decomposition from MIRROR prior requirements + MNEMOSYNE knowledge → **Analyst review checkpoint** (mandatory before research begins)
2. Each sub-question executed with versioned SKS skill, IAS screened, CVS verified → **Analyst reviews each partial answer** before the next sub-question
3. STOA proposes synthesis → **Final analyst review** → Methodology trail generated (MOIRAI-attested)

The methodology trail is the accountability artefact Taddeo et al. require under Recommendation 8: traceability of every process step.

---

**The Warning Layer — Converting Unknown Unknowns**

Taddeo et al. identify the known/unknown taxonomy as the foundation for risk prioritisation. The six themis-warning services operationalise this directly:

| Service | Risk Category Addressed | Mechanism |
|---|---|---|
| CGS/ARGUS-LACUNA | Known unknowns | Characterises collection gaps precisely enough to drive requirements |
| ADS/CASSANDRA | Known unknowns → emerging known knowns | Baseline behavioural monitoring; anomaly detection |
| WSF/LACHESIS | Unknown unknowns → known unknowns | Weak signal fusion matching historical precursor patterns |
| CRF/JANUS | Known unknowns (cross-silo) | Surfaces analytical connections between separate requirements |
| TRS/CHRONOS | Known unknowns → probabilistic known | Trajectory modelling with scenario spaces and leading indicators |
| SWS/SENTINEL | Warning synthesis | Multi-source corroboration before strategic warning issued |

**These services do not eliminate unknown unknowns. They convert more of them into known unknowns faster than an ungoverned platform.**

---

## SECTION 4: INFRASTRUCTURE AND GOVERNANCE
*Slides 16–19 · Completing the platform*

---

### Slide 16 — Agent-Native Infrastructure and the Intelligence Cycle

**Two governance layers that are non-negotiable before agentic or full-cycle deployment.**

---

**Agent-Native: Three Services That Make Agentic AI Safe**

As AI moves from generating outputs to taking actions, the threat surface changes. The critical risk is not a bad output — it is an irreversible action.

| Service | Function | Without It |
|---|---|---|
| **SCBS / SENTINEL-CAP** | Capability bounding — enforces spend cap, resource scope, TTL. Pre-call cost estimates before every state-changing action. Fails closed. | Agent executes unbounded actions |
| **CBS / BROKER** | Credential broker — agents receive time-limited operation-scoped handles, never raw credentials. Revoked within 100ms on session close. | Raw credential exposure; no revocation |
| **RSS / ROLLBACK** | State snapshots — pre-action snapshot before every write, 72-hour retention, single-command rollback. | Irreversible agent actions; no recovery |

FGS/PLUTUS integrates with SCBS: per-session envelopes cannot exceed team allocation. Every agent inference call attributed in real time.

---

**Closing the Intelligence Cycle**

```
TASKING                     ANALYSIS                DISSEMINATION
TIS / DIKE               Full THEMIS platform       PCS / IRIS
Gap signals →            (42 services)          Consumer packages:
Requirements         ↑                              · Plain-language source basis
management           │       FEEDBACK               · Falsification indicators
system               └─── OFS / NEMESIS ────────   · Prior outcome record
                          Outcomes close the        · Provenance certificate
                          calibration loop
                          back to TCS + ORACLE
```

- **TIS/DIKE** closes the upstream loop: ATHENA gaps → collection requirements
- **OFS/NEMESIS** closes the downstream calibration loop: real-world outcomes → TCS/MIMIR posteriors
- **PCS/IRIS** closes the consumer loop: structured, auditable packages to policymakers

---

### Slide 17 — The AI Trust Cell and Nine Control Gates

**Governing AI in high-stakes analysis is an organisational design problem, not just an engineering problem.**

```
AI TRUST CELL

┌──────────────────────┬─────────────────────────┬─────────────────────────┐
│  Research &          │  THEMIS Platform Team   │  Intelligence Layer     │
│  Red Team            │                         │  Team                   │
├──────────────────────┼─────────────────────────┼─────────────────────────┤
│ HADES access —       │ 2-week synchronized     │ Deployed on quality     │
│ adversarial analysis │ sprints                 │ thresholds, not sprint  │
│ + IAS catalog        │                         │ boundaries              │
│ updates              │ 3 swim lanes:           │                         │
│                      │ Roadmap / Red Team      │ Feeds MNEMOSYNE and     │
│ Never takes sprint   │ Response / Tech Health  │ ORACLE data floors      │
│ tickets              │                         │                         │
│                      │ FGS consumption         │ STOA methodology trail  │
│ Monthly adversarial  │ reports to leadership   │ quality review          │
│ summary → IOB        │ monthly                 │                         │
└──────────────────────┴─────────────────────────┴─────────────────────────┘
```

**The Research and Red Team never takes sprint tickets.** Research independence is structural or it does not exist. The monthly adversarial exposure summary is delivered simultaneously to Cell Lead and IOB — not filtered through management.

**Taddeo et al. Recommendation 1:** government funding should be allocated to longitudinal studies on HMT-AI, focusing on team conventions, training protocols, and risk management standards. The AI Trust Cell's pre-registered research cycles are the institutional implementation.

**Nine Control Gates — proportionate governance through the build:**

| Gate | Timing | Key requirement |
|---|---|---|
| Gate-0 | Before Week 1 | IOB Charter · GC-3, GC-4 · Tokenomics model approved |
| Gate-1 | Week 8 | Cell Lead + Engineering Lead + IOB review |
| Gate-2–4 | Weeks 28–46 | ARB (agent surfaces) · MOIRAI chain audit · R&RT adversarial assessment |
| **Gate-5 (H1 Close)** | Week 66 | **Full ARB + IOB formal assessment. H2 authorised here.** |
| Gate-6–9 | Year 2–4 | Intelligence cycle gates · Warning governance confirmation |

---

## SECTION 5: THE ASK
*Slides 18–25 · Investment, validation, risk, and next steps*

---

### Slide 18 — Policy Dependencies: Eight Open GC Items

**Eight governance decisions gate specific platform capabilities. None are engineering decisions.**

| Ref | Decision Required | Gates | Initiate By |
|---|---|---|---|
| **GC-2** | Retrieval gap indicator disclosure — what analysts see about collection gaps, in what format, with what method detail | CGS/ARGUS-LACUNA analyst surface | Year 2 Q3 |
| **GC-3** | Query-type authorisation taxonomy | PCES classifier training | Gate-0 |
| **GC-4** | Within-requirement role-tier access policy | Phase 1 tier deployment | Gate-0 |
| **GC-5** | Definition of "confirmed" / "disconfirmed" for outcome classification — partial accuracy, overtaken-by-events, outcomes above assessment classification | OFS/NEMESIS | Year 2 Q1 |
| **GC-6** | Consumer package content policy — falsification indicator handling, package classification, modification authority | PCS/IRIS | Year 2 Q1 |
| **GC-7** | Forecast product governance — labelling, accuracy measurement for probabilistic assertions, dissemination authority | TRS/CHRONOS | Year 3 |
| **GC-8** | Tokenomics model governance — allocation basis, reserve fraction, interaction class weights, consumption data governance | FGS/PLUTUS Phase 1 | Gate-0 |
| **IOB Charter** | IOB composition, decision authority, quorum, meeting cadence, relationship to ARB | All gate decisions | Before Gate-0 |

**GC-3, GC-4, GC-8, and IOB Charter must be resolved before Phase 1 begins. GC-5 and GC-6 must be initiated before Year 2 Q1.**

> **Speaker note:** Taddeo et al. Recommendation 4 calls for an independent body to assess the predictability of AI systems deployed in national security contexts — distinct from the deploying organisation. The IOB charter must define this independence explicitly.

---

### Slide 19 — Implementation Roadmap: Three Horizons

| Horizon | Timeline | Services | Output |
|---|---|---|---|
| **H1: Governance Foundation** | Months 1–17 · 5 build phases · Gate-5 | 27 platform services including HADES and FGS/PLUTUS | Governed AI platform. Full accountability infrastructure. Adversarial institutional memory. Tokenomics operational. Answerable to any oversight inquiry. |
| **H2: Intelligence Cycle + Layer** | Year 2 · Gate-6 through Gate-8 | 15 intelligence layer services + 5 intelligence cycle completion services | Full cycle governed from requirements through policymaker. Calibration loop closed. STOA teammate capability. Institutional intelligence flywheel operational. |
| **H3: Warning Layer** | Year 3–4 · Gate-9 | 6 warning layer services + ORACLE | Platform that improves with every requirement. Strategic warning. Unknown unknown detection. |

**Data floors that cannot be engineered around:**

| Service | Data floor | Realistic availability |
|---|---|---|
| TCS/MIMIR (CALIBRATED state) | 30+ weighted corrections per analyst-domain cell | Year 2+ per cell |
| MIRROR | 50+ requirements with profiles | Year 2 Q1–Q2 |
| MNEMOSYNE | 18 months FGTS + ERAS data | Year 2 Q3 |
| ORACLE | 200+ requirements with classified outcomes | Year 3–5 realistic |
| SWS/SENTINEL (meaningful warnings) | Full warning layer mature | Year 4 |

**The gap between agencies that start now and agencies that wait widens every year — because it is built on that agency's own analytical history, which no external vendor or adversary can replicate.**

---

### Slide 20 — Investment Summary

| Horizon | Timeline | Output |
|---|---|---|
| **H1** | Months 1–17 | 42 platform services. Full accountability infrastructure. Adversarial institutional memory. Tokenomics governance. Answerable to oversight. |
| **H2** | Years 2–3 | Full cycle governed. Calibration loop closed. STOA teammate capability. Institutional intelligence flywheel. |
| **H3** | Year 4+ | Platform that improves with every requirement. Strategic warning. Gap widens every year vs. ungoverned alternatives. |

**The most significant cost is not the technology. It is the team.** Calibration scientists, media forensics engineers, adversarial ML specialists — not commodity hires. Hiring begins immediately as a pre-Phase-1 dependency.

**FGS/PLUTUS makes the investment legible.** Monthly cost-per-analytical-output data from session one. The investment is not a black box.

---

### Slide 21 — What the Research Validates

**Three anchoring sources. Eight recommendations. Thirteen findings. One infrastructure.**

**Taddeo et al. (CETaS, September 2022) — all 8 recommendations reflected in THEMIS:**

| Recommendation | THEMIS Implementation |
|---|---|
| R1: Longitudinal HMT-AI research — team conventions, training protocols, risk management standards | AI Trust Cell R&RT pre-registered research cycles; THEMIS as a longitudinal HMT-AI dataset |
| R2: HMT-AI certification scheme — performance consistency, traceability, trust auditing; disincentivise over-trust | 9 control gates + IOB Gate-5 certification; TCS/MIMIR continuous trust measurement; FGTS corpus as auditable trust record |
| R3: Govern HMT-AI teams, not AI systems alone | 42 services govern the full analytical interaction — session, claim, correction, outcome |
| R4: CBA of HMT-AI including predictability assessment; independent third-party assessor | FGS/PLUTUS cost-per-analytical-output; CPS/APORIA predictability score (proposed); IOB as assessor independent from deploying organisation |
| R5: Predictability trade-offs, not more/less predictability — gains on one level can cost on another | UCS/TYCHE decomposes uncertainty by type, surfacing trade-offs at the claim level |
| R6: Link trustworthiness to an amendable predictability score including CBA of unwanted behaviour risks | CPS/APORIA Green/Amber/Red zones as amendable score (proposed); TCS/MIMIR Bayesian calibration as trustworthiness measure |
| R7: Risk thresholds mapping unpredictability severity to risk predictability level (known knowns/unknowns) | SENTINEL warning layer taxonomy; CGS/ARGUS-LACUNA (known unknowns); LACHESIS (unknown unknowns → known); PGS/NOMOS thresholds per interaction class |
| R8: ALARP framework — quantitative predictability assessment, traceability, deployment conditions, CBA, scenario analysis, human override protocols | MOIRAI chain (traceability) · FGS CBA data · SCBS fail-closed + RSS rollback (human override) · 9 control gates (scenario-gated progression) · IOB threshold authority |

**Knack et al. (CETaS, December 2022) — 13 findings, all reflected:**

| CETaS Finding | THEMIS Implementation |
|---|---|
| System-level trust — the output AND the whole system (Finding 4) | 42-service governance platform |
| Audit trail for oversight — who, what, when, why (Section 3.2.3) | MOIRAI cryptographic chain — tamper-evident from session one |
| Stratified explanations by stakeholder (Section 3.2) | PCS/IRIS · ATHENA layered interface · MOIRAI audit certificate |
| Standardised confidence language across the national security community (Finding 10) | UCS/TYCHE · PGS/NOMOS ICD 203 compliance |
| Data drift tracked across the deployment lifecycle (Section 3.2.1) | MDS/KRONOS · KCS/ARGUS model knowledge currency |
| ML teammate reinstates memory — "the big picture after the weekend" (Section 2.2) | MOS/SAGA session and matter memory |

**Gruber (Daring Fireball, May 2026):** AI is pervasive technology infrastructure — not a product. Governance infrastructure is the correct investment category. The AI is already in the workflow; the governance layer is not.

---

### Slide 22 — Risk Comparison

| Risk | Without THEMIS | With THEMIS |
|---|---|---|
| Fabricated source reaches policymaker | Detected post-dissemination or never | Blocked at CVS/VERITAS before analyst sees it |
| Analyst acts on superseded intelligence | No currency signal | TVS/KAIROS decay + KCS supersession propagated |
| Adversary corpus poisoning reaches analysis | No detection layer | IAS/SCUDO screening at ingestion |
| "Trust and forget" — analyst over-trusts AI | No measurement; no detection | TCS gaming probability · FGTS five-factor weighting |
| Adversarial manipulation documented for future defence | No record | HADES adversarial repository — permanent institutional asset |
| Oversight body requests AI usage audit | No defensible record | MOIRAI cryptographic chain from session one |
| AI agent takes irreversible action | No recovery path | RSS/ROLLBACK 72h snapshot + single-command rollback |
| Intelligence failure traced to AI calibration gap | No feedback loop | OFS/NEMESIS closes loop from real-world outcomes |
| Unknown unknowns not surfaced | No detection mechanism | SENTINEL warning layer converts unknowns to known unknowns |
| AI inference costs unattributed | No visibility | FGS/PLUTUS token attribution from session one |
| Analyst using model beyond reliable capability | No signal | CPS/APORIA capability zone advisory (proposed) |
| Policymaker receives unexplained confidence score | No consumer translation | PCS/IRIS with falsification indicators |

---

### Slide 23 — Four Decisions Required

**These four decisions initiate the programme. Nothing else is needed to begin.**

**Decision 1 — Programme Authorization**
Authorize THEMIS with H1 funding. H1 closes with a formal IOB assessment (Gate-5) before H2 funding is required — a discrete, auditable commitment with a decision point.

**Decision 2 — Team Assembly**
Begin hiring the AI Trust Cell immediately. The Research and Red Team must be in place before IAS/SCUDO and HADES are operational — their analytical work is the operational content of those services. This is a pre-Phase-1 dependency, not a parallel track.

**Decision 3 — GC / IOB Engagement**
Initiate GC-3, GC-4, GC-8, and the IOB Charter before Gate-0. Initiate GC-5 and GC-6 before Year 2 Q1. Designate IOB chair and constitute the board before Phase 1 closes. The IOB is the governance authority for every control gate — without it, the gate decisions have no authority.

**Decision 4 — Organisational Structure**
Three structural requirements before Phase 1:
- **R&RT charter protection:** ring-fenced headcount, not available for sprint work, requires above-Cell-Lead authority to override. Research independence is structural or it does not exist. (Taddeo et al. Recommendation 1: longitudinal research programmes require structural independence.)
- **FGS tokenomics model approval:** IOB approves the allocation basis before Phase 1 — token allocation periods cannot open without an approved model.
- **Cross-organisational governance authority:** THEMIS must govern AI use across the full analytical organisation, not just the platform team that builds it.

---

### Slide 24 — Research References

**Three anchoring sources for the THEMIS design:**

---

Taddeo, M., Ziosi, M., Tsamados, A., Gilli, L., & Kurapati, S. (2022). *Artificial Intelligence for National Security: The Predictability Problem.* CETaS Research Report. Centre for Emerging Technology and Security, The Alan Turing Institute. September 2022.
https://cetas.turing.ac.uk

*The Predictability Problem* · Dual definition (maximal and minimal) · HMT-AI as the governance unit · "Trust and forget" dynamics · Known/unknown unknowns risk taxonomy · ALARP framework for AI risk assessment · 8 recommendations

---

Knack, A., Carter, R.J., & Babuta, A. (2022). *Human-Machine Teaming in Intelligence Analysis: Requirements for developing trust in machine learning systems.* CETaS Research Report. Centre for Emerging Technology and Security, The Alan Turing Institute. December 2022.
https://cetas.turing.ac.uk

*18 in-depth interviews with UK national security practitioners* · System-level trust requirements · Stratified explanation requirements by stakeholder · Audit trail for oversight bodies · Memory reinstatement as teammate capability · 13 findings

---

Gruber, J. (2026, May 16). AI is technology, not a product. *Daring Fireball.*
https://daringfireball.net/2026/05/ai_is_technology_not_a_product

*AI as infrastructure, not product* · Wireless networking analogy · Pervasive technology requires pervasive governance infrastructure · The governance layer is the right investment category

---

Supporting literature: MITRE (2018), *Human-Machine Teaming Systems Engineering Guide* · DARPA Explainable AI Programme · Mitchell et al. (2019), model cards · Babuta & Oswald (2019), data analytics in policing · UK MoD (2022), *Defence AI Strategy* · GCHQ (2021), *Ethics of Artificial Intelligence*

---

### Slide 25 — THEMIS at a Glance

**42 services · 11 namespaces · 9 control gates · 4 accountability axes · 3 implementation horizons**

```
H1 — GOVERNANCE FOUNDATION (Months 1–17)
  27 platform services · Full accountability infrastructure
  MOIRAI chain · TCS calibration · HADES adversarial memory
  FGS tokenomics · IAS/SCUDO adversarial defence · ATHENA interface
  Agent-native: SCBS · CBS · RSS

H2 — INTELLIGENCE CYCLE + LAYER (Year 2)
  15 intelligence layer services
  Full cycle: TIS/DIKE (tasking) → analysis → OFS/NEMESIS (outcomes) → PCS/IRIS (policymaker)
  STOA teammate orchestration · MOS/SAGA memory · MNEMOSYNE institutional knowledge

H3 — WARNING LAYER (Year 3–4)
  6 warning services · ORACLE outcome intelligence
  Unknown unknown detection · Strategic warning synthesis
  Platform improves with every requirement processed

PROPOSED (pending ARB)
  CPS/APORIA — Green/Amber/Red capability zones
  ODS/LETHE — Out-of-distribution detection

Research grounding:
  Taddeo et al. (CETaS, Sep 2022) — The Predictability Problem
  Knack, Carter & Babuta (CETaS, Dec 2022) — HMT Trust Requirements
  Gruber (Daring Fireball, May 2026) — AI Is Technology, Not a Product
```

**The AI is already in the workflow. The governance infrastructure is not. THEMIS is the infrastructure.**
