# ATHENA User Training Design

## Governing Principles · Curriculum Architecture · Delivery Framework

**Document purpose:** Design specification for the ATHENA analyst training program  
**Audience:** Training designers, program managers, analytical supervisors  
**Companion documents:** ATHENA Intervention Catalog v3.2 · THEMIS Platform Design v1.0  
**Status:** Version 1.0

-----

## 1. The Training Problem

ATHENA’s interventions are designed to counteract automatic cognitive processing — the heuristic shortcuts that make AI output dangerous when presented without friction. Training that is itself absorbed automatically defeats the purpose. An analyst who can recall what the Intention Gate does is not the same as an analyst who genuinely activates their prior belief before reading AI output.

This places specific demands on training design: conceptual understanding must precede procedural training. Feature tutorials produce compliance. Conceptual grounding produces genuine engagement.

### 1.1 Why This Problem Is Different from Standard Software Training

Standard software training: teach users what the buttons do and when to click them.

ATHENA training: teach users why the cognitive failure modes exist, why the interventions counteract them, and why genuine engagement produces better analytical outcomes than performative compliance.

The distinction matters because the platform includes compliance gaming detection (TCS/MIMIR). Training must explain this not as surveillance but as a consequence of the calibration architecture: performative verification produces inaccurate calibration, which produces unreliable confidence signals, which degrades the analyst’s ability to work effectively. Gaming the system produces worse outcomes for the analyst, not just the organization.

### 1.2 The Adversarial Dimension

The platform exists in an adversarially manipulated information environment. Training must explain not just the cognitive failure modes but the adversarial exploitation of those failure modes — corpus poisoning, prompt injection, adversarial competence probing, and deadline-targeted denial-and-deception operations. Analysts who do not understand the adversarial dimension will treat the platform’s guardrails as organizational caution rather than operational necessity.

-----

## 2. Target Audiences

### 2.1 Intelligence Analysts

Primary users of ATHENA. Range from junior analysts (first use of AI-assisted analysis) to senior analysts (established workflows and existing calibration intuitions). Training must address both ends:

Junior analysts need foundational epistemic habits built before AI-assisted analysis becomes their default mode of working. If the habits are built on the platform rather than before it, analysts will learn how to use ATHENA rather than why to use it correctly.

Senior analysts present a different problem: established domain expertise creates domain overconfidence. An analyst who is genuinely expert in their area may be systematically over-reliant on AI in that domain precisely because they cannot easily distinguish AI errors from AI accuracy — their expertise tells them the output sounds right. Calibration needs to be grounded in measured accuracy, not experience alone.

### 2.2 Analytical Supervisors

Supervisors access the calibration dashboard and supervisory visibility surface. They need to understand:

- The calibration state machine and how to interpret calibration differences across their analyst pool
- How to conduct calibration review sessions as developmental rather than evaluative tools
- What the supervisory override means for FGTS weighting and when to use it
- How the monthly AI governance report maps to accountability obligations

### 2.3 Intelligence Oversight Board and Program Management

Governance users who consume accountability records and the provenance audit surface. Need to understand what the cryptographic provenance chain can and cannot prove, how to interpret AI governance reports, and how policy changes to analytic standards guardrails are proposed and approved.

-----

## 3. Training Design Principles

### 3.1 Why Before What

Every intervention is introduced with its cognitive failure mode before its operational mechanics. An analyst who understands why the Intention Gate exists will engage with it differently than an analyst who understands how to fill it in.

The sequence is never: “here is the feature, here is when you use it.” It is always: “here is the failure mode, here is why it occurs in AI-assisted analysis specifically, here is what the platform does to counteract it.”

### 3.2 Calibration-Aware Scaffolding

Training intensity scales with calibration state. New analysts receive more scaffolding and more supervised practice. Training does not end at onboarding — it continues as analysts enter new domains where calibration reverts toward PRIOR ONLY, and when model version changes require recalibration.

### 3.3 Practice Over Declarative Knowledge

Conceptual understanding without practice does not transfer to operational behavior. Scenario-based exercises with calibration feedback are the most important training element. Modules that are not followed by structured practice with feedback produce declarative knowledge that does not change behavior.

### 3.4 Genuine Engagement Over Compliance

Training must never frame ATHENA’s interventions as compliance requirements. The Intention Gate only works cognitively if the prior belief is genuinely committed before reading the response. An analyst who fills it in after reading has undermined the one thing it does — set the anchor before the AI output sets it for them. Training should make this concrete with a worked example showing the difference between pre-committed and post-hoc Intention Gate entries.

### 3.5 Management Reinforcement Required

Training content cannot override management behavior. If supervisors use the compliance gaming detection score punitively rather than developmentally, the training message is undermined immediately. Supervisory training must precede the rollout of analyst training, and supervisory behavior must be explicitly aligned with the developmental framing of the calibration system.

-----

## 4. Knowledge Domains

Four knowledge domains must be covered. They are sequenced: Domain 1 is the prerequisite for understanding Domains 2 and 3, and Domain 4 depends on all three.

### Domain 1: The Fluency Trap

**Core concept:** AI output is uniformly fluent regardless of epistemic status. A retrieved claim and a confabulated claim sound identical. This is structural, not a quality deficiency — it is how large language models work. The generation process treats retrieved text and parametric knowledge as inputs to the same probability distribution over next tokens. The claim that emerges does not carry a label saying where it came from.

**What analysts must understand:**

The distinction between grounded (GRND), parametric (PARAM), and synthesized (SYNTH) claims at the conceptual level — before meeting the badges. Grounded claims can be verified against the source. Parametric claims have no specific source to check; the only verification path is independent collection or acknowledgment that verification was not possible. Synthesized claims require checking that the underlying sources actually support the synthesized conclusion, not merely that the sources exist.

Why fluency is a credibility signal for human speakers but not for AI systems. With human speakers, articulate delivery often correlates with genuine knowledge and preparation. With AI systems, the fluency of the output is a function of the architecture, not the accuracy of the claim.

Why this is not fixed by better AI models. A more accurate model still produces confident-sounding errors, and those errors will look identical to its accurate outputs. The fluency signal is structurally uninformative about accuracy.

**Key misconception to address:** “The AI will tell me if it is uncertain.” AI models frequently produce hedging language (“I believe,” “it appears,” “it is likely”) that does not reliably correspond to lower accuracy. The calibrated confidence signals in ATHENA are grounded in measured accuracy for this analyst on this claim type in this domain — not model self-report.

**Learning objective:** The analyst can articulate, without reference to the platform, why AI confidence signals cannot be taken at face value and what they would need to know about a claim to evaluate it properly.

### Domain 2: The Three Accountability Axes

**Core concept:** Every AI-assisted analytical interaction must be accountable across three independent dimensions. An assessment that passes two axes can fail catastrophically on the third. The independence is the critical point — passing Origin and Currency provides no protection against failing Trust.

**Origin — Where did this claim come from?** The verification path for a retrieved claim is completely different from the verification path for a parametric claim. An analyst who does not know the source type cannot apply the right verification action. The most dangerous case is treating a parametric claim as retrieved — it looks like the source can be checked, but there is no specific source.

**Currency — Is this intelligence still valid?** The model does not know the intelligence picture has changed. A claim from a source ingested two years ago looks exactly like a claim from a source ingested this morning. Currency failure is silent — there is no signal in the output that a previously valid assessment has been superseded. This is the failure mode that accumulates over time rather than appearing as an acute error.

**Trust — Is my reliance calibrated?** The hardest domain for experienced analysts. Calibration is not a property of the model — it is a relationship between a specific analyst, a specific claim type, and a specific domain. An analyst who is accurate in their primary domain may be systematically over-reliant on AI in adjacent domains where they cannot catch model errors. Without measurement, this variation is invisible to supervisors and often to the analysts themselves.

**The three failure mode pairings:**

Origin without Currency: The claim is provenance-perfect, retrieved from a verified source. The source was superseded eight months ago when the situation changed dramatically. The assessment is provenance-perfect and lethally stale.

Currency without Trust: Sources are current and corroborated. The analyst is systematically over-reliant on AI in this domain because their calibration has never been measured and the model’s actual accuracy in this domain is lower than they believe. The picture is current; the judgment is degraded.

Trust without Origin: The analyst is well-calibrated with strong verification habits. The interface does not make visible that this specific claim was synthesized across partial evidence with no single source. The analyst accepted a SYNTH claim believing it was GRND. The calibration was good; the information needed to apply it was hidden.

**Learning objective:** Given a specific AI claim, the analyst can identify which axis failure modes apply, what the correct verification action is, and what a genuine verification looks like for that source type.

### Domain 3: Verification as Craft

**Core concept:** Verification actions are not checkbox compliance. Different source types have qualitatively different verification paths, and the epistemic weight of different verification outcomes differs substantially.

“Confirmed against source” means locating the specific retrieved document and confirming the AI characterized it accurately. This is verifiable and carries the most weight in FGTS calibration.

“Verified independently” means the analyst located independent evidence supporting the claim without reference to the AI’s output. The epistemic act is different — the analyst is not checking the AI’s characterization of a source, they are confirming the claim holds in the world.

“Could not verify” is not failure. It is honest accounting. An unverified claim that is explicitly marked unverified is far less dangerous than an unverified claim that appears verified. The dissemination gate handles explicitly unverified claims correctly. The system cannot handle claims that were accepted without genuine verification.

**Verification paths by source type:**

GRND: Locate the source document. Confirm the AI characterized it accurately. Confirm the source is current and from a credible collector.

PARAM: No specific source to check. Verification requires independent collection, subject matter expert consultation, or explicit acknowledgment that verification was not achievable through normal analytical workflow.

SYNTH: The specific assertion does not appear verbatim in any single source. Verification requires confirming that the underlying sources actually support the synthesized conclusion — not merely that the sources exist and are current.

TRANSCRIPT: Verify against the original recording, not the AI transcript. ASR accuracy varies substantially by audio quality, speaker characteristics, and technical terminology. The confidence score on the TRANSCRIPT badge reflects ASR accuracy, not claim accuracy.

IMAGE / VIDEO / AUDIO: Verify the MAS/EIDOLON authenticity assessment before accepting analytical conclusions derived from the artifact. A high deepfake risk score or poor metadata integrity creates a hard confidence ceiling on derivative claims regardless of how confident the AI’s analysis appears.

OCR: The confidence score represents text extraction accuracy for that specific scan. Low OCR confidence requires returning to the original physical or high-quality digital document before analytical use.

**Why verification quality matters for calibration:** FGTS/ALETHEIA weights corrections before they enter the ground truth corpus. A correction submitted with long dwell time on the source document, under standard (not deadline-critical) conditions, by an analyst with a low gaming probability score, carries substantially more calibration weight than a correction submitted in three seconds. Analysts who verify genuinely build more accurate calibration faster than analysts who click through.

**Learning objective:** For each source type badge, the analyst can explain the correct verification path, what a genuine verification looks like in practice, and why verification quality affects their calibration accuracy.

### Domain 4: Calibration Literacy

**Core concept:** Calibration is not a score the platform assigns to the analyst. It is a continuously updated statistical model of the relationship between this analyst’s reliance on AI and the AI’s actual accuracy on this claim type in this domain with this model version. It changes as any of those variables change.

**The calibration state machine:**

PRIOR ONLY: Calibration posteriors are initialized from benchmark data on model performance. Confidence signals reflect general model accuracy, not this analyst’s verified accuracy in this specific domain. Treat with explicit humility — the signals are informative but not personalized.

CALIBRATING: Operational data is accumulating. The posterior is updating but has not yet stabilized. Confidence signals are improving but carry higher uncertainty than the CALIBRATED state.

CALIBRATED: Empirically grounded calibration for this analyst on this claim type in this domain with this model version. Confidence signals can be relied upon within the confidence bounds of the calibration model itself — which are visible in the session manifest.

**What triggers calibration state changes:**

A model version change resets calibration toward PRIOR ONLY for affected claim types. The analyst’s prior intuitions about the model’s behavior may not apply to the updated model. MDS/KRONOS detects version changes and flags calibration baselines as potentially stale.

Entering a new domain requires domain-specific calibration data before the analyst can reach CALIBRATED in that domain. Expertise in one domain does not transfer.

Extended absence degrades confidence in the posterior. The system handles this by widening confidence intervals rather than resetting the state.

**What calibration is not:** A performance evaluation. A high calibration score is not a proxy for analytical quality. Calibration measures the accuracy of the analyst’s reliance on AI, not the quality of their independent analytical judgment. These are different things. An analyst who never relies on AI has no calibration record and that is fine — the calibration system only governs the AI-assisted portion of their work.

**What compliance gaming does to calibration:** An analyst who submits minimal Intention Gate entries, clicks verify without reading sources, and batch-dismisses the verification queue builds a calibration record that does not reflect genuine verification behavior. The confidence signals produced by that record are unreliable in proportion to the gaming behavior. The analyst receives worse signals. Their analytical outputs are degraded by signals they cannot trust. Gaming is self-defeating.

**Learning objective:** The analyst can explain their current calibration state in their primary domain, describe what changed the last time their state changed, and identify which of their domains remain in CALIBRATING or PRIOR ONLY.

-----

## 5. Curriculum Architecture

### Module 0: Foundation (Pre-Platform)

**Target:** All analysts, before any platform access  
**Duration:** Half day  
**Format:** Facilitated session with case studies  
**Prerequisite:** None — this is the prerequisite for everything else

**Content:**

1. The fluency trap, presented with examples before any ATHENA reference. Walk through specific examples of identical-sounding outputs with different epistemic statuses. Ask analysts to identify which sounds more credible. The correct answer is that they are indistinguishable.
1. The three accountability questions as analytical habits, not platform features. Introduce them as questions any rigorous analyst should be asking — ATHENA is the instrumentation that makes asking them systematic.
1. The adversarial information environment. What corpus poisoning, prompt injection, and adversarial competence probing look like. Why deadline periods are specifically targeted by denial-and-deception operations.
1. What AI-assisted analysis failure looks like. Case studies of analytical failures attributable to fluency trap acceptance, stale intelligence, and uncalibrated trust. Use synthetic or sanitized historical cases. Make the failure consequences concrete.

**Assessment:** Analysts articulate the three accountability questions and describe a specific scenario where each axis failure mode would be dangerous in their analytical domain. No ATHENA feature knowledge is assessed or expected.

**Note on senior analysts:** Module 0 is not optional for experienced analysts. They should be specifically challenged: “Given your domain expertise, where are you most likely to accept AI claims without adequate independent verification?” This surfaces the domain expertise overconfidence problem that senior analysts are specifically vulnerable to.

-----

### Module 1: ATHENA Orientation

**Target:** All analysts  
**Duration:** Half day  
**Format:** Structured walkthrough with hands-on exercises on historical cases  
**Prerequisite:** Module 0

**Content:**

1. Session configuration — chat modes, model parameter transparency, prompt repository. How configuration choices affect TCS/MIMIR calibration stratification.
1. The Intention Gate — what it is, what anchoring is and why it matters, what a genuine pre-commitment looks like versus a post-hoc entry. Walk through a specific example: same analytical question, one analyst commits prior belief before reading the response, one fills in the gate after. Show the calibration difference.
1. Source type badges — all eight types. For each: the failure mode it addresses, the correct verification path, one worked example of a genuine verification, one worked example of a performative verification that looks correct but isn’t.
1. Verification Queue — mechanics, what the states mean (pending → confirmed / misrepresents source / could not verify / etc.), the dissemination gate behavior, and what explicit “could not verify” means for accountability.
1. Session integrity indicators — the agreement rate tracker and what it surfaces about analytical behavior in this session. Compliance gaming detection explained: what it measures, why it exists, what it means for calibration.

**Assessment:** Analysts complete a structured ATHENA session using a historical case with a known outcome. Calibration feedback reviewed with the facilitator immediately afterward. The analyst sees their calibration record from the session.

-----

### Module 2: Pressure Mode Operations

**Target:** All analysts  
**Duration:** Half day  
**Format:** Tabletop exercise with pressure simulation  
**Prerequisite:** Module 1

**Content:**

1. The pressure state machine — Standard, Elevated, Deadline-Critical. What changes under each mode and why.
1. The hard constraints under Deadline-Critical — which are non-dismissable, why they are non-dismissable, and why “non-dismissable” is a deliberate design choice rather than a technical limitation.
1. The adversarial timing insight — why deadline periods are when corpus poisoning and prompt injection attempts increase. Why the constraints are calibrated specifically to the failure mode, not to general caution.
1. Verification queue prioritization under time pressure — highest-risk claims first, explicit “Reordered for time-constrained verification” indicator, what the prioritization criteria are.

**This module must include a live simulation.** Analysts work a time-constrained analytical problem in ATHENA under Deadline-Critical conditions. The constraints will feel obstructive. This is intentional. The debrief is not “how did you manage the constraints” — it is “what would have happened without the constraints, given the time pressure you were under.”

**Assessment:** Tabletop debrief. Analysts explain why each Deadline-Critical constraint is non-dismissable and describe the failure mode that each constraint prevents. Speed of completion is explicitly not assessed.

-----

### Module 3: Advanced Calibration

**Target:** Analysts with at least 30 days of operational ATHENA use who have completed Module 1  
**Duration:** Half day  
**Format:** Individual calibration review session with supervisor  
**Prerequisite:** Module 1, 30+ days operational use

**Content:**

1. Review of the analyst’s calibration record across their active domains — domain-by-domain calibration state, confidence interval widths, correction weight patterns.
1. Identification of calibration gaps — domains in PRIOR ONLY or CALIBRATING after extended use (may indicate domain overconfidence or gaming behavior).
1. Identification of calibration concerns — patterns in verification behavior that suggest systematic issues (consistent batch dismissal, very low dwell times on source documents, high semantic similarity across Intention Gate entries).
1. Plan for calibration development — specific domains and claim types to focus verification attention on for the next 90 days.

**Module 3 is a recurring module, not a one-time event.** Calibration review sessions should occur at minimum quarterly, and immediately following any model version change or domain transition.

-----

### Module 4: Supervisory Operations

**Target:** Analytical supervisors  
**Duration:** Full day  
**Format:** Facilitated workshop  
**Prerequisite:** Modules 0 and 1 (supervisors must understand analyst experience before managing it)

**Content:**

1. The supervisory dashboard — reading calibration state and gaming probability across the analyst pool. What the indicators mean and how to distinguish genuine calibration gaps from gaming signals.
1. Conducting calibration review sessions developmentally — how to make them useful rather than evaluative. Specific questions that surface useful insight versus questions that produce defensive responses.
1. Supervisory override — when to use it, what it means for FGTS weighting (supervisory-confirmed corrections carry the highest calibration weight in the system), how to document the reasoning.
1. The AI governance report — what the monthly report covers, what it does not cover, what questions to ask when something looks anomalous.
1. Multi-analyst session management — supervisory review mode mechanics, matter handoff procedures, collaborative session attribution requirements.

**Module 4 must be completed before Module 3 is rolled out to analysts.** Supervisors need to be prepared to run calibration review sessions before analysts start receiving calibration data.

-----

### Module 5: Intelligence Oversight Board Operations

**Target:** IOB members, program managers  
**Duration:** Half day  
**Format:** Facilitated briefing  
**Prerequisite:** None (does not require prior ATHENA use)

**Content:**

1. The provenance chain — what it records, what it proves, and what it explicitly does not prove. The chain proves the record has not been altered since creation. It does not prove the analytical judgments in the record were correct.
1. The cryptographic attestation — what “demonstrably unaltered” means in practice, how to verify the chain, what a broken chain means.
1. The AI governance report — what to look for, what questions to raise, what escalation paths exist for concerns.
1. Policy change process — how analytic standards guardrails are modified, what requires IOB approval, what the version control mechanism looks like.

-----

## 6. Hard Training Problems

### 6.1 Pressure Mode Resistance

The most difficult training challenge. Deadline-Critical constraints will feel obstructive to analysts under genuine operational pressure. Training must address this directly rather than pretending the friction is seamless.

**The cognitive preparation component:** Analysts must understand the asymmetric cost argument before they experience the constraints. The cost of a miscalibrated high-stakes assessment under deadline pressure substantially exceeds the cost of the constraints. This must be established as a stated design premise — not discovered through experience.

**The experiential component:** The live simulation in Module 2 is not optional. Conceptual understanding of deadline pressure dynamics does not transfer to operational behavior without an experience that activates the pressure response. The debrief consolidates understanding from a concrete rather than abstract starting point.

**The management component:** Supervisors who pressure analysts to produce quickly in ways that implicitly endorse circumventing Deadline-Critical constraints undermine this module immediately. Supervisory training must address this directly.

### 6.2 The Intention Gate — Genuine vs. Performative Engagement

Analysts who have been briefed on compliance gaming detection may approach the Intention Gate strategically: write substantive-sounding entries that avoid triggering pattern-matching detection while still filling them in after reading the AI response.

Training must address this directly: “You cannot outrun the calibration model. Over time, genuine pre-commitment and post-hoc simulation produce different calibration records. The difference is not immediately obvious in individual sessions but becomes clear in aggregate verification behavior patterns.”

More importantly: the training goal is not to produce analysts who write convincing Intention Gate entries. It is to produce analysts who understand that the gate only does what it does — counteract anchoring — if the prior belief is committed before the anchor is set. The entry is a byproduct of genuine pre-commitment, not the thing itself.

### 6.3 Gaming Detection as Developmental Rather Than Punitive

The compliance gaming detection score being visible to supervisors will feel adversarial to some analysts. This is a management design problem, not only a training content problem.

Training content should say explicitly: “Some of you will feel that this score makes ATHENA a surveillance system. Here is what it actually is: calibration is only useful if it reflects genuine verification behavior. An analyst whose calibration is built on gaming behavior gets worse confidence signals, which produces worse analytical outcomes for them. The score exists to help you and your supervisor understand whether your calibration data is reliable — and to protect you from acting on confidence signals that do not actually reflect verified accuracy.”

Supervisors must then behave consistently with this framing. If the first time a supervisor mentions a gaming detection score is in a performance review, the training message is undermined. The first mention should be in a calibration development conversation.

-----

## 7. Assessment Framework

### Formative Assessments (During Training)

Module 0: Structured discussion. Analysts articulate the three accountability questions and one specific scenario per axis. No platform knowledge required.

Module 1: Structured ATHENA session on a historical case with known outcome. Calibration feedback reviewed immediately. The assessment is whether genuine verification occurred, not whether the analyst selected correct button labels.

Module 2: Tabletop debrief. Analysts explain the rationale for each non-dismissable Deadline-Critical constraint. Speed of completion not assessed.

### Summative Assessment (Ongoing)

The calibration record is the persistent summative assessment. Module assessments are point-in-time snapshots. The calibration record accumulates continuously and reflects operational behavior.

Calibration review sessions every 90 days, or on domain change or model version change, provide the structured summative review. Supervisors hold the accountability for ensuring these sessions occur and are documented.

### What Is Not Assessed

Whether analysts like the interventions. Whether analysts find the platform intuitive. Whether analysts feel the constraints are appropriate. These are feedback questions for the platform team, not training assessments.

-----

## 8. Development Priorities

The modules must be developed in a specific sequence reflecting their dependency structure.

**Priority 1 — Module 0 (Foundation).** This module is the prerequisite for everything. Without it, all other training produces compliance at best. Must be developed and piloted before any other content.

**Priority 2 — Module 4 (Supervisory Operations).** Supervisors must be trained before analysts. Their behavior shapes the environment in which training either takes root or is undermined.

**Priority 3 — Module 1 (ATHENA Orientation) plus the supervised onboarding calibration session.** Analysts cannot begin operational use without this. The onboarding calibration session provides the cold-start data that makes the PRIOR ONLY → CALIBRATING transition meaningful.

**Priority 4 — Module 2 (Pressure Mode Operations).** Must be completed before analysts encounter any high-stakes time-constrained requirements in production.

**Priority 5 — Module 3 (Advanced Calibration).** Requires 30+ days of operational data. Development can proceed in parallel with Phase 1-2 platform deployment.

**Priority 6 — Module 5 (IOB Operations).** Can be developed in parallel with Phase 3-4 platform work; not required until the accountability surface is operational.

-----

*ATHENA User Training Design v1.0*  
*Companion documents: THEMIS Platform Design v1.0 · ATHENA Intervention Catalog v3.2 · AI Trust Cell Operating Model*
