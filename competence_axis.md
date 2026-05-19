# THEMIS Platform — Addendum D: The Competence Axis
## A Proposed Fourth Accountability Dimension

**Document series:** THEMIS Platform Design · Addendum D  
**Companion to:** Platform Design v1.0 · Addendum A: New Services · Addendum B: Access Control · Addendum C: Memory Architecture  
**Status:** Proposed — forward-looking design. No implementation timeline.  
**Classification:** Working design document

---

## Overview

Addenda A through C addressed services, access control, and memory within the existing three-axis accountability framework. This addendum proposes a fourth axis — Competence — that addresses a class of failure mode the first three axes do not cover.

The three existing axes ask questions about what the AI produces:
- **Origin** — Where did this claim come from?
- **Currency** — Is this intelligence still valid?
- **Trust** — Is the analyst's reliance calibrated?

The Competence axis asks a prior question: **Can this AI system reliably perform this task at all?**

This is a different kind of accountability. Origin, Currency, and Trust evaluate the quality of outputs. Competence evaluates whether reliable output is achievable on this task by this system. An assessment can pass all three existing axes — provenance-perfect, current, well-calibrated — and still be unreliable if the task it was generated for exceeds the model's actual capability.

### Why This Is a Distinct Axis

The Competence axis cannot be collapsed into the existing three.

It is not a subset of Trust. Trust measures whether the analyst's reliance is calibrated to the AI's accuracy. Competence asks whether the AI has any reliable accuracy to calibrate to for this task type. An analyst can be perfectly calibrated to a model's performance on tasks within its capability — and have no calibration signal at all for tasks outside it, because there is no meaningful accuracy to measure.

It is not a subset of Origin. A parametric claim (PARAM) may still be within the model's reliable capability domain. Origin describes where knowledge came from. Competence describes whether the task is one the model can perform. These are independent.

It is not a subset of Currency. Currency addresses whether the intelligence picture has changed. Competence addresses whether the model can reason reliably about the current picture regardless of when the sources were last updated.

### The Specific Failure Pattern

Large language models produce outputs whether or not they are capable of producing reliable outputs. The training objective — next-token prediction optimized through RLHF — systematically rewards confident-sounding responses and does not reward "I cannot reliably answer this." The result is a system that will generate fluent, confident, structurally coherent output on tasks it has no reliable competence to perform.

Without the Competence axis, the analyst has no signal that the task they submitted is one the system cannot handle — until the downstream consequences make the failure visible.

---

## Six Failure Modes the Competence Axis Addresses

### 1. Knowledge Boundary Violations

The model is asked questions that require knowledge it definitively does not have: events after its training data cutoff, classified source material it was never trained on, proprietary organizational knowledge that does not appear in any training corpus, real-time situational awareness the model cannot have.

These are not retrieval failures — retrieval-augmented generation addresses the data availability problem. Knowledge boundary violations occur when the analyst asks the model to reason about something where neither retrieval nor training can provide a grounding. The model's response is not retrieved and not synthesized from partial evidence; it is generated from the statistical patterns of adjacent training data, which may bear no reliable relationship to the actual answer.

### 2. Task-Capability Mismatch

The deployed model is asked to perform a task it was not designed or evaluated for. Precise mathematical verification. Cryptographic analysis. Complex geospatial reasoning from imagery metadata. Signals processing. Source network mapping from indirect indicators.

A text model generating confident mathematical conclusions, a language model performing complex network topology inference, a general-purpose reasoning model performing specialized technical analysis outside its evaluated domain — these are task-capability mismatches. The output is generated. It may even appear correct. But the model has no reliable mechanism to distinguish its correct outputs from its incorrect ones in domains where it has not been evaluated.

### 3. Out-of-Distribution Queries

The query falls outside the distribution the model was trained on. Novel adversary behavioral patterns with no historical precedent. Geopolitical configurations without analogues in the training data. Technical programs in domains where the model's training data was sparse, deliberately withheld, or systematically biased.

Out-of-distribution inference is the most operationally dangerous failure mode because it is invisible. The model has no internal signal that it is operating outside its training distribution. It generates the same confident output it would generate on an in-distribution query. The analyst has no interface signal. The downstream assessment looks identical to one built on solid training-data coverage.

### 4. Adversarial Competence Probing

A deliberate denial-and-deception technique: querying the AI with questions designed to elicit confident-sounding confabulations in domains where the model's reliable capability is absent, then using those AI-generated confabulations as apparently authoritative "AI-sourced" intelligence to mislead analysts or pollute the analytical record.

This is distinct from corpus poisoning (which attacks the retrieval layer) and from prompt injection (which attacks the instruction-following layer). Adversarial competence probing attacks the model's tendency to generate confident output in its gap areas. An adversary who has mapped the model's capability gaps can systematically query those gaps to generate useful confabulation.

### 5. Epistemic Non-Declaration

The model fails to declare that it does not know. This is not a knowledge gap — it is an interface failure. A model that appropriately says "I cannot reliably answer this" is operating correctly within its competence boundaries. A model trained to minimize non-answers and maximize apparent helpfulness will generate confident output instead of declaring uncertainty.

Current RLHF training systematically discourages non-answers. The result is that models are not reliably calibrated to produce "I do not know" as a first-class response, even when that is the correct response. The Competence axis must account for this training-induced bias.

### 6. Compound Capability Degradation

Complex analytical tasks require multiple capabilities in combination: temporal reasoning, technical domain knowledge, geopolitical inference, adversary intent modeling. A model may have partial competence in each component but fail reliably on their combination in ways that are not predictable from performance on the individual components.

A model that accurately characterizes adversary technical programs and separately performs adequate geopolitical analysis may produce unreliable outputs when asked to integrate the two into a timeline-linked operational assessment. Compound capability degradation is difficult to detect through component-level capability assessment.

---

## Proposed Services

The Competence axis cannot be fully instrumented with current technology. The open research questions are substantial (see below). What follows is a set of services that address the tractable portions of the problem now, while preserving design space for more complete solutions as the research matures.

---

### New Service: CPS / APORIA
**Capability Profiling Service**

APORIA in Greek philosophy is the state of productive unknowing — the recognition that a question has reached the limits of what can be reliably answered, which Socrates treated as the necessary precondition for genuine inquiry. The Capability Profiling Service surfaces when the AI system has reached its own APORIA: the boundary of what it can reliably do.

**Function:** CPS/APORIA maintains a version-controlled capability taxonomy for each deployed model, classifying analytical task types into three zones:

- **Green zone:** Task types where the model has been evaluated and found reliable within defined confidence bounds. Standard confidence presentation applies.
- **Amber zone:** Task types where the model has partial capability — accurate on some instances, unreliable on others, with no reliable discriminator available to the system. Capability warning surfaces to the analyst.
- **Red zone:** Task types where the model has been evaluated and found unreliable, or which have been designated beyond-capability by the red team. Hard warning surfaces; supervisory notification fires.

The capability taxonomy is maintained by the Research and Red Team and updated as new failure modes are discovered through red team evaluation. Every taxonomy version is version-controlled. Changes require Cell Lead sign-off. The taxonomy is referenced at query classification time — before inference.

**Integration:**
- Receives task type classification from the query classifier (PGS/NOMOS extension — see below)
- Publishes CapabilityWarningEvents to MOIRAI for every amber or red zone query
- Feeds capability zone into TCS/MIMIR as a context variable for confidence ceiling adjustment — red zone queries carry a hard confidence ceiling regardless of TCS calibration score
- Exposes capability taxonomy API to ATHENA for analyst-facing warning surface

**ATHENA surface:** A **Competence Zone Indicator** appears when a query is classified as amber or red zone: amber — "This query type is in a partially validated capability area. Verify outputs independently." Red — "This query type is outside this system's validated capability. A supervisor has been notified. Do not rely on AI output for this task without independent collection."

**Namespace:** themis-quality  
**Phase:** Forward — pending capability taxonomy development by the Research and Red Team. No deployment until the taxonomy has at least 60 days of red team evaluation data.

---

### New Service: ODS / LETHE
**Out-of-Distribution Screening Service**

LETHE in Greek mythology is the river of forgetfulness — what lies beyond the boundary of what can be recalled or known. ODS/LETHE surfaces queries that fall outside the model's training distribution.

**Function:** ODS/LETHE applies a lightweight out-of-distribution screening layer at query ingestion, before retrieval and before inference. Current approach: embedding distance from the centroid of training-distribution query clusters, with configurable threshold by interaction class. Queries whose embedding distance exceeds the threshold for their interaction class are flagged as potentially out-of-distribution.

**Design constraints and limitations:** OOD detection for autoregressive generative models is an open research problem (see Open Research Questions). The embedding-distance approach is a heuristic, not a reliable detector. High false positive rates are expected in early deployment, particularly for novel but legitimate query types. The service must be deployed in advisory mode initially — surfacing a signal to the analyst rather than blocking — with extensive monitoring before any hard constraints are applied.

**Storage:** OOD screening event log (PostgreSQL, append-only). Query embeddings, computed OOD distance scores, thresholds applied, flags triggered. All events signed and published to MOIRAI.

**Integration with CPS/APORIA:** OOD flags from LETHE feed into CPS/APORIA as evidence when updating the capability taxonomy — a cluster of OOD-flagged queries that also produced analyst-flagged incorrect outputs is strong evidence for a red zone designation.

**Namespace:** themis-quality  
**Phase:** Forward — research prototype first. Do not deploy in any enforcing capacity until false positive rate is characterized on at least 500 operational queries.

---

### Enhancement: IAS / SCUDO — Capability Probe Detection Layer

IAS/SCUDO currently screens for adversarial injection at the retrieval layer. A new detection layer adds capability probe pattern detection at the query layer.

**Function:** A capability probe classifier added to IAS/SCUDO's query ingestion screening, running alongside the adversarial injection classifiers. The capability probe classifier is trained on patterns characteristic of adversarial competence probing: queries designed to elicit model output in known capability gap areas, queries that reference entities or events specifically absent from the model's training distribution, queries constructed to make the AI appear authoritative on topics it cannot reliably address.

**Initial approach:** Signature matching against a red-team-maintained probe taxonomy, plus embedding similarity against a curated corpus of known capability-gap query patterns. The ML-based classifier is a Year 2 capability, requiring labeled training data from red team evaluation.

**Disposition:** Flagged capability probe queries are not blocked. They surface to the analyst with a signal: "This query pattern has characteristics associated with adversarial probing of AI capability boundaries. The AI response to this query may be less reliable than normal. Consider independent verification and supervisor consultation." The event is logged to MOIRAI and included in the AI Governance/IOB monthly report.

---

### Enhancement: ERAS / LOGOS — Competence-Limit Reasoning Pattern Detection

ERAS/LOGOS currently captures reasoning structure and detects unsupported claims. A new detection module flags reasoning patterns characteristic of capability-limit operation.

**Patterns to detect:**

- **Circular reasoning:** The model's conclusion restates its premises without adding evidential content — a characteristic pattern when the model has no real grounding for its conclusion
- **Excessive hedging followed by confident conclusion:** Paragraphs of uncertainty language that resolve into a confident assertion — characteristic of a model that "knows" it doesn't know but generates a conclusion anyway
- **Self-contradiction within response:** The model asserts X early in a response and contradicts X later — characteristic of confabulation without coherent knowledge to anchor on
- **Specificity inversion:** The model is more specific and confident about the conclusion than about any of the evidence it cites — characteristic of reasoning backward from a generated conclusion rather than forward from evidence

These patterns are heuristics, not definitive indicators. ERAS surfaces them as advisory signals in the analyst view — "This response contains reasoning patterns that may indicate the model is operating at its capability limits" — rather than as hard flags.

---

### Enhancement: TCS / MIMIR — Capability-Stratified Calibration

TCS/MIMIR currently maintains calibration posteriors per (analyst, claim type, domain, model version). A Competence axis extension adds a fourth stratification variable: **capability zone** (green / amber / red, sourced from CPS/APORIA).

The calibration model for amber-zone tasks is maintained separately from the calibration model for green-zone tasks on the same domain. An analyst's green-zone accuracy on geopolitical assessment does not inform their amber-zone calibration on the same topic — the reliability properties of the model are different enough that pooling the calibration data would produce misleading confidence signals.

Red-zone tasks carry a hard confidence ceiling regardless of TCS calibration score: TCS cannot assign HIGH or MEDIUM confidence to a claim generated on a red-zone task, because the model's reliability in that zone is not a function of analyst calibration — it is a function of the model's fundamental capability boundary.

---

## ATHENA Competence Axis Surface

Three new ATHENA interface elements for the Competence axis. All are advisory in Phase 1 — no hard blocks until false positive rates are characterized.

**Competence Zone Indicator:** Badge displayed on sessions that include amber or red zone queries. Green zone queries produce no badge (no cognitive overhead for in-scope tasks). Amber zone queries produce an amber indicator. Red zone queries produce a red indicator with a supervisor notification link.

**OOD Distance Signal:** An optional panel, collapsed by default, showing the out-of-distribution distance score for the current query. Designed for power users and supervisors rather than general analyst use. Collapsed by default to avoid alert fatigue.

**Capability-Limit Reasoning Flag:** Inline annotation on responses where ERAS detects capability-limit reasoning patterns. Displayed as a subtle icon adjacent to the claim, expanding on click to show which patterns were detected and what they indicate about the reliability of this specific response.

---

## Open Research Questions

The following questions must be substantially answered before the Competence axis can be implemented as a hard enforcement capability rather than an advisory signal. They represent the current boundary of the field.

---

### 1. Reliable Epistemic Self-Assessment in Large Language Models

**The problem:** Models do not reliably know what they do not know. The training objective — maximizing reward for helpful-sounding responses — systematically discourages epistemic non-declaration. A model trained on RLHF to be helpful will not reliably say "I cannot answer this" because refusal is penalized relative to confident-sounding output.

**Current state:** Calibration research has made progress on models' ability to express calibrated confidence in their outputs on tasks within their training distribution. The harder problem — reliable declaration of capability boundaries — remains unsolved. Models that appear to express uncertainty often do so performatively: they hedge in the preamble and then produce a confident conclusion anyway.

**Research direction:** Constitutional approaches, capability-aware fine-tuning, and explicit capability declaration training objectives are active areas. No production-ready solution exists for intelligence-domain deployment.

---

### 2. Out-of-Distribution Detection for Autoregressive Generative Models

**The problem:** OOD detection is well-developed for discriminative models — classifiers that output a label or a probability. For autoregressive generative models, every output is "in distribution" from the model's perspective: it is always generating the most probable next tokens given its training. There is no reliable internal signal that the query is outside the training distribution — the generation process is the same regardless.

**Current state:** Embedding-distance heuristics (like ODS/LETHE's approach) have partial validity but high false positive rates in practice. Perplexity-based detection is unreliable for instruction-tuned models. Semantic similarity to training data is difficult to compute at inference time. Conformal prediction approaches show theoretical promise but have not been demonstrated at scale on intelligence-domain queries.

**Research direction:** Mixture-of-experts architectures with explicit routing confidence, retrieval-augmented uncertainty estimation, and ensemble disagreement as an OOD signal are under active research. None are production-ready for this use case.

---

### 3. Ground Truth Development for Capability Boundary Assessment

**The problem:** To know where the model's capability boundaries are, you need ground truth — verified correct answers — for a representative sample of queries in each candidate boundary zone. In intelligence domains, ground truth is often classified, delayed, or impossible to produce without real-world events providing the answer.

**Current state:** The Research and Red Team can develop capability taxonomies through synthetic benchmarking and red team evaluation. These approaches provide evidence for capability boundaries but are necessarily incomplete. The capability taxonomy for CPS/APORIA is only as reliable as the red team's ability to generate representative test cases for each zone — a process that is labor-intensive and always behind the frontier of novel query types.

**Research direction:** Automated red teaming for capability gap discovery, automated benchmark generation for novel domains, and transfer of capability boundary evidence across model versions.

---

### 4. Compound Capability Interaction

**The problem:** Capability assessment for individual task types does not reliably predict capability on their combination. A model may perform reliably on technical program assessment and on adversary intent modeling separately, and fail reliably on their integration in a complex multi-step analytical task. The interaction effects are not predictable from component performance.

**Current state:** Essentially unknown for intelligence-domain analytical tasks. Compositional generalization research in NLP provides some theoretical grounding but has not been applied to this specific problem.

**Research direction:** Systematic compositional capability evaluation — testing all relevant combinations of task types at the complexity levels that appear in operational analytical workflows. This is a large-scale evaluation effort, not a research question with near-term answer.

---

### 5. Capability Boundary Shifts Under Adversarial Information Conditions

**The problem:** In intelligence analysis, the information environment is adversarially manipulated. Capability boundaries that hold under benign conditions may shift when the model is operating on adversarially selected information. An adversary with knowledge of the model's capability gaps can select or fabricate information designed to push the model into those gaps — making the model appear more capable than it is on specific tasks while engineering failures on others.

**Current state:** This problem has no established research program. It sits at the intersection of adversarial ML, capability evaluation, and intelligence tradecraft — a combination that has not been studied systematically.

**Research direction:** Red team simulation of adversarial capability manipulation as an explicit evaluation scenario. This requires access to the deployed model, a capable red team, and a framework for designing adversarial capability-manipulation scenarios — all of which the AI Trust Cell is positioned to develop, but the research has not been scoped.

---

### 6. Training Objective Conflict

**The problem:** The RLHF training objective rewards helpfulness and penalizes refusal. A model trained to maximize this objective will generate confident-sounding output on capability-limit tasks rather than declaring incompetence. Any Competence axis surface that relies on the model's own uncertainty expression is working against the training objective that shaped the model's behavior.

**Current state:** This is a known alignment problem with no clean solution at the deployment layer. Constitutional AI approaches and capability-declaration fine-tuning can partially address it, but both require access to the training pipeline — not available at the deployment layer. Inference-time approaches (eliciting uncertainty expression through prompting, chain-of-thought reasoning before conclusion) have partial effectiveness but are not reliable enough for hard-constraint enforcement.

**Research direction:** This is fundamentally a training-time problem. Solving it at the deployment layer is a holding action. The Research and Red Team should monitor alignment research for training-time solutions that can be specified as requirements for future model procurement or fine-tuning partnerships.

---

## Implementation Posture

Given the maturity of the research, the Competence axis must be implemented in three phases with explicit gates between them.

**Phase 1 — Advisory only:** CPS/APORIA deployed with a red-team-developed capability taxonomy. All signals are advisory — surfaced to analysts as information, not as enforcement. ODS/LETHE deployed in monitoring mode with no analyst-facing surface. ERAS competence-limit reasoning detection deployed as opt-in for supervisors and the Research and Red Team. False positive rates characterized. No hard blocks. Duration: minimum 6 months of operational data.

**Phase 2 — Soft enforcement:** CPS/APORIA red zone queries trigger supervisory notification. ODS/LETHE amber zone flags surface to analysts (collapsed indicator, not blocking). IAS/SCUDO capability probe detection deployed as an advisory classifier. TCS/MIMIR capability-stratified calibration active. Gate: false positive rate below 15% on amber zone classification, red team confirmation of probe detection taxonomy adequacy.

**Phase 3 — Hard constraints (research-dependent):** Red zone queries produce hard constraints on dissemination — equivalent to the Deadline-Critical hard constraints in the existing Trust axis. Gate: research questions 1 and 2 above must have tractable partial solutions validated on intelligence-domain queries before hard constraints are applied. This gate is not on a timeline. It is on research maturity.

---

## Relationship to Existing THEMIS Architecture

The Competence axis does not require modifications to the existing 14 THEMIS services beyond the enhancements described for IAS/SCUDO, ERAS/LOGOS, and TCS/MIMIR. CPS/APORIA and ODS/LETHE are new services added to the themis-quality namespace alongside CVS, IAS, MAS, FGTS, TVS, and RQS.

The cryptographic attestation architecture applies to both new services without modification. CPS capability zone assignments and ODS distance scores are signed events in the MOIRAI provenance graph. All CapabilityWarningEvents and OOD screening events appear in the audit chain.

The ATHENA intervention catalog would require a new category — **Competence Response** — for the three new interface elements. The intervention count increases from 31 to 34 when the Competence axis is fully deployed.

The AI Trust Cell operating model requires a specific addition: the Research and Red Team capability taxonomy development work must be formally scoped and resourced as a standing workstream, not an ad hoc project. The taxonomy is the foundational input to CPS/APORIA, and its quality directly determines the quality of every downstream Competence axis capability.

---

## Summary

The Competence axis addresses the failure mode that the three existing axes cannot: the AI system is asked to perform a task it cannot reliably perform, and produces confident output anyway.

Two new services (CPS/APORIA, ODS/LETHE), three enhanced services (IAS/SCUDO, ERAS/LOGOS, TCS/MIMIR), and three new ATHENA interface elements provide partial instrumentation of the axis with current technology. Six open research questions define the boundary between what can be built now and what must wait for the research to mature.

The Competence axis is not optional. It addresses a failure mode that adversaries understand and exploit — adversarial competence probing is a denial-and-deception technique, not a hypothetical edge case. But deploying it before the research is mature enough to produce reliable signals would be worse than deferring — false positives from an unreliable competence detection system would erode analyst trust in legitimate warnings and produce the exact alert-fatigue problem that the intervention design has worked to avoid.

The right posture is: deploy advisory capabilities now, fund the research actively, and gate hard enforcement on research maturity rather than on a calendar.

---

*THEMIS Platform · Addendum D · The Competence Axis: A Proposed Fourth Accountability Dimension*  
*Companion documents: Platform Design v1.0 · Addenda A, B, C*
