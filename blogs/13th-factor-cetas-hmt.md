# The Analyst Doesn't Trust Your Model. They Trust Your Process.

### What CETaS's study of intelligence practitioners reveals about how AI trust actually works in high-stakes environments

---

There is a gap between what the explainable AI research community is building and what operational intelligence analysts actually need. The research community is building better attribution methods, more interpretable attention maps, more sophisticated SHAP plots. Analysts are asking a different question: has this system been signed off?

That finding sits at the center of a 2022 research report from the Centre for Emerging Technology and Security (CETaS) at The Alan Turing Institute. The report is "Human-Machine Teaming in Intelligence Analysis: Requirements for Developing Trust in Machine Learning Systems," authored by Anna Knack, Richard J. Carter, and Alexander Babuta. It draws on in-depth interviews and focus groups with 18 practitioners across the UK national security community, including operational intelligence analysts, legal experts, behavioural scientists, and human factors engineers.

The report set out to examine how to calibrate appropriate trust in machine-generated insights and how best to integrate ML capabilities into the analyst's decision-making process. What it found upends some common assumptions about where the hard design problems actually live.

---

## The governance finding

The most counterintuitive result in the report is this: analysts do not primarily ground their trust in ML systems through technical explanations of how the model works. They ground it through organizational governance.

If an internal authority has tested, accredited, and approved an ML capability for operational use, the model explanation becomes substantially less important to the analyst. The system has been vouched for by someone with the standing to do so. That vouching matters more than the SHAP plot.

This is not analyst irrationality. It is rational behavior in an environment where analysts cannot themselves fully evaluate the technical properties of large ML models, where the time pressure of operational contexts makes deep technical review impossible, and where the consequences of getting it wrong are serious. Governance is a trust proxy. Accreditation is a signal that someone with relevant expertise has already done the evaluation work. When that signal exists and is credible, it reduces the cognitive burden on the analyst and makes the model usable.

The design implication is significant. Organizations deploying ML for analysis are investing heavily in XAI techniques designed to make model behavior more transparent to end users. This report suggests that investment may be partially misplaced. Transparency to the senior responsible owner, the person authorizing and governing the deployment, may matter more than transparency to the analyst. The analyst needs to know the system has been approved, what it is designed to do, and what its known limitations are. They do not necessarily need to understand the internal mechanisms of the model.

That does not mean explanation is irrelevant to analysts. It means the type and depth of explanation required is different from what the XAI literature typically targets, and that organizational trust infrastructure is load-bearing in a way that most AI product teams are not building for.

---

## Context is not a variable. It is the design constraint.

The second major finding is that ML explainability requirements cannot be specified at the model level. They can only be specified at the deployment context level.

The same ML capability can be deployed for low-priority retrospective analysis or for high-priority, time-sensitive operations. The same model can support decisions with relatively low individual impact or decisions that directly affect someone's liberty. The explanation requirements appropriate for one context are inappropriate for another. A system that provides the right level of explanation for a routine triage task will provide too much noise in an urgent operational situation, and too little grounding for a decision that will face legal scrutiny.

Interviewees were explicit about this. An analyst working a time-critical operation doesn't want a detailed explanation of why the model produced an alert. They want to know whether to act on it. An analyst preparing analysis that will underpin an arrest warrant needs to be able to reconstruct the reasoning chain that led to the conclusion. These are different cognitive situations with different information needs, and a single explanation design serves neither well.

The practical consequence is that context-sensitivity cannot be bolted on after the fact. It has to be a first-order design requirement. The questions that need answers before any design work begins include: what are the distinct deployment contexts for this system, how do explanation requirements differ across them, and how does the interface shift accordingly?

One practitioner framed the layered approach well: most information should be available to everyone, with the ability to drill down to more granular detail for those who need it, and highly technical information accessible only to those who require it for governance or oversight purposes. That's a progressive disclosure model, not a one-size explanation model, and it requires deliberate information architecture rather than a single explanation widget.

---

## Faster horses, not cars

The report's practitioners were clear-eyed about where ML adds value in the near term and where it doesn't.

The highest-value, lowest-risk application of ML in intelligence analysis is in the early stages of the pipeline: filtering, characterizing, and triaging large volumes of data to reduce what analysts have to manually review. One interviewee described it as producing "piles of hay" so the analyst doesn't have to search "the entire field." The needle-finding stays with the analyst. The field-reduction goes to the machine.

This is a meaningful distinction with direct product implications. Analysts who participated in the research were comfortable with ML-assisted triage. They were substantially less comfortable with ML-generated assessments, predictions, or conclusions. As one practitioner put it, the near-term value is in "procuring faster horses rather than cars." Faster horses are systems that get through more data and surface information of potential interest. Cars are systems that take the analyst somewhere new. Analysts, in 2022, wanted faster horses.

This is partly about trust development. Practitioners described a progression where acceptance of ML in low-stakes tasks generates the confidence that makes higher-stakes applications conceivable. You earn the right to the sensemaking loop by demonstrating value in the foraging loop first. Organizations that try to deploy ML directly into the assessment and conclusion-generation stages, without the trust-building foundation of demonstrated value in data reduction, are skipping steps that cannot be skipped.

---

## The tool-teammate distinction

The report draws a useful distinction between ML as a tool and ML as a teammate, and argues that these are different categories with different requirements.

A tool performs a narrow task: filter this data, flag this pattern, classify this image. The analyst uses it the way they use any other instrument. They do not need to trust it the way they trust a colleague. A teammate is something different. A teammate is involved in joint problem solving. It helps the analyst recall context, surfaces connections the analyst might have missed, challenges assumptions, and evolves with the analyst's way of working over time.

Practitioners could imagine both. They were skeptical about how quickly the teammate model would arrive and whether current ML capabilities were ready for it. But the description of what a genuine AI teammate would need to do is instructive. One practitioner described it as helping "reinstate memory and helping the analyst recall strands of the big picture after the weekend." Another suggested a future system that could listen to conversations between analysts and flag connections that human analysts might be blind to.

What's notable about these descriptions is that they are cognitive partnership functions, not information retrieval functions. The teammate is helping the analyst maintain the state of the sensemaking loop across time and cognitive load. It is addressing a real problem: analysis work is iterative and extended, cognitive load fluctuates, and the schema the analyst is building can degrade or fragment between sessions. A tool that supports that specific cognitive need is a different design target from a tool that surfaces relevant documents.

One practitioner flagged an important calibration challenge for the teammate model: a system that challenges the analyst's reasoning on every output will get turned off. The friction budget for constructive challenge is finite, and spending it all at once exhausts it quickly. The design implication is that an AI teammate's challenge function has to be selective, targeted, and proportionate. It should push back on the high-stakes hypothesis where the evidence is weak, not on the routine classification where the analyst has domain certainty.

---

## What builders need from this report

**Governance infrastructure is a product requirement, not a compliance checkbox.** If analyst trust in ML systems runs through organizational accreditation, then the design of that accreditation process, its visibility to analysts, and the mechanisms by which model limitations are communicated as part of approval, are all product-adjacent problems. An ML tool deployed without a clear approval signal that analysts can see and reference is a tool that will face an uphill adoption problem regardless of its technical quality.

**Explanation design requires context mapping first.** Before any interface design work on ML explainability, the product team needs a complete map of the deployment contexts in which the system will operate, the decision types each context involves, and the explanation requirements for each. The layered progressive disclosure model is the right structural answer, but the information architecture within that model is context-specific and cannot be designed generically.

**The false negative problem is asymmetric and context-dependent.** The report finds that false negatives are generally more costly in intelligence contexts than false positives, and that risk tolerance for false positives increases under time pressure and high-stakes conditions. These are not fixed parameters. They shift with context. A threshold that is appropriate for routine analysis will be wrong for an urgent operational situation. Building context-aware threshold adjustment into the system, with corresponding transparency about how thresholds shift, is a design requirement that most ML tools do not address.

**The audit trail is not optional.** Analysts operate in an environment where their decisions may face legal challenge, judicial review, or inquiry. The decisions informed by ML-assisted analysis carry an additional accountability burden: not just "why did I make this decision" but "what role did the ML system play and what were its known limitations at the time." That audit trail needs to be built at the system level, not reconstructed after the fact. Decisions about what to log, how to attribute analyst actions to ML-assisted outputs, and how to surface that trail to oversight bodies are design decisions that have to be made during development, not after deployment.

**User testing is not a phase. It is a method.** The report's practitioners were consistent on this: the most effective way to integrate ML into analyst workflows is controlled testing in a sandbox environment, with real analysts, before operational deployment. The lessons from that testing should set the threshold parameters, calibrate the explanation model, and surface the adoption risks. They should also be shared across partner organizations deploying similar systems. The organizational tendency to treat user testing as a launch gate rather than a development method produces systems that fail in deployment for reasons that would have been visible six months earlier.

---

## The larger point

The CETaS report is valuable precisely because it is grounded in practitioner experience rather than technical aspiration. Its practitioners are not describing the ML system they wish existed. They are describing what they need from the systems being deployed now and in the near term, in the operational contexts they actually work in.

What they are describing is a trust system, not just a technical system. Trust in ML for intelligence analysis is built through demonstrated reliability in low-stakes applications, organizational governance that signals approval and accountability, contextually appropriate explanation at the right level of detail for the right audience, and an interface that reduces cognitive burden rather than adding to it.

Most ML products in this space are optimizing for the technical dimension. The governance dimension, the context-sensitivity dimension, and the cognitive load dimension are being underbuilt. That gap is where adoption fails, and it is where the design work still needs to happen.

---

*The 13th Factor covers human-centered design, UX, and human-machine teaming.*

---

**Reference:** Knack, A., Carter, R. J., & Babuta, A. (2022). Human-machine teaming in intelligence analysis: Requirements for developing trust in machine learning systems. CETaS Research Report. Centre for Emerging Technology and Security, The Alan Turing Institute. https://cetas.turing.ac.uk
