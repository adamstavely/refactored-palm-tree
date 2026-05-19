# The Question the Three Axes Don't Ask

*A proposed fourth accountability dimension for AI-assisted intelligence analysis.*

---

The three accountability axes in THEMIS ask questions about the quality of AI outputs. Origin: where did this claim come from? Currency: is this intelligence still valid? Trust: is the analyst's reliance appropriately calibrated?

These are the right questions. They address real failure modes. The platform built around them represents a meaningful step toward accountable AI in high-stakes analytical work.

But there is a prior question none of the three axes asks: can this AI system reliably perform this task at all?

This is the Competence axis. It is different from the other three in kind, not just degree. Origin, Currency, and Trust assume the model is capable of producing reliable outputs and ask how well it has done so on a given claim. Competence asks whether the model is capable of producing reliable outputs on this task type. An assessment can pass all three existing axes — provenance-perfect, current, well-calibrated — and still be unreliable if the task exceeds the model's actual capability.

## The Failure Pattern

Large language models produce outputs whether or not they are capable of producing reliable outputs. This is not a property that can be configured away. It is a consequence of how these systems are trained.

The training objective — next-token prediction, optimized through reinforcement learning from human feedback — systematically rewards confident-sounding responses and does not reward "I cannot reliably answer this." Refusal is penalized relative to apparent helpfulness. The result is a system that will generate fluent, confident, structurally coherent output on tasks it has no reliable competence to perform. The output is indistinguishable, at the interface level, from output the system is genuinely capable of producing.

The analyst has no signal. Origin tells them whether the claim was retrieved or parametric. Currency tells them whether the source is still valid. Trust tells them whether their reliance is calibrated to the model's accuracy on this type of claim. None of these axes catches the failure mode where the task itself was beyond the model's reliable capability. A model asked to reason about a novel adversary behavior pattern with no training-data precedent will produce the same format of confident response as a model asked about a well-documented historical pattern. The analyst cannot distinguish them.

## Six Ways This Fails

The Competence axis addresses six distinct failure modes, each with a different structure.

**Knowledge boundary violations** are the most recognizable. The model is asked about events after its training cutoff, classified programs it was never trained on, proprietary organizational knowledge that does not exist in any training corpus. These are not retrieval failures — retrieval-augmented generation addresses the data availability problem for sources you have. Knowledge boundary violations occur when neither retrieval nor training can provide grounding.

**Task-capability mismatch** is less recognized. A text model asked to perform precise mathematical verification. A general-purpose reasoning model performing cryptographic analysis. A language model mapping source networks from indirect indicators. These models will produce output. It will look like analysis. It may even be occasionally correct. But the model has no reliable mechanism to distinguish its correct outputs from its incorrect ones in domains where it has not been systematically evaluated.

**Out-of-distribution queries** are the most operationally dangerous because they are the most invisible. Novel adversary behavioral patterns with no historical precedent. Geopolitical configurations without analogues in the training data. Technical programs in domains where training data was sparse or deliberately withheld. The model has no internal signal that it is operating outside its training distribution. It generates the same confident output it would generate on an in-distribution query. There is no flag.

**Compound capability degradation** is poorly understood even by the research community. A model may perform reliably on each component of a complex analytical task and fail reliably on their combination in ways that are not predictable from component performance. A model that accurately characterizes adversary technical programs and separately performs adequate geopolitical analysis may produce unreliable outputs when asked to integrate the two into a timeline-linked operational assessment. The failure mode emerges from the combination, not from any individual capability gap.

**Epistemic non-declaration** is a training problem masquerading as an interface problem. A model that recognizes it is at its capability limits and says so is operating correctly. A model trained to minimize non-answers will generate confident output instead. The bias is baked in: expressing uncertainty is treated as a failure of helpfulness during training, which means the model's self-reported uncertainty is not a reliable indicator of its actual epistemic state.

**Adversarial competence probing** is the failure mode that should most concern anyone working in intelligence. It is a denial-and-deception technique: an adversary who has mapped the model's capability gaps queries those gaps deliberately to generate confident-sounding confabulations, then introduces those AI-generated confabulations into the analytical record as apparently authoritative "AI-sourced" intelligence. This is not a hypothetical. Any adversary sophisticated enough to study how AI-assisted analysis works is sophisticated enough to study how to exploit it. Corpus poisoning attacks the retrieval layer. Prompt injection attacks the instruction-following layer. Adversarial competence probing attacks the model's tendency to generate confident output in areas where it has no reliable knowledge to draw on.

## Why the Other Three Axes Don't Cover This

It is worth being explicit about what the existing axes do and do not address.

Trust calibration — the third axis — measures whether analyst reliance is appropriate for the model's actual accuracy. But calibration requires a stable accuracy rate to calibrate to. A model operating within its competence domain has such a rate. A model operating at its capability limits does not — its accuracy in that zone is not a function of the task type and domain in general, but of whatever the model happens to confabulate on this specific query. You cannot calibrate to random noise.

Origin visibility — the first axis — tells the analyst that a claim is parametric: drawn from training weights with no retrieval backing. But a parametric claim within the model's competence domain is different in kind from a parametric claim beyond its competence boundary. Both show up as PARAM. The analyst has no signal that one is a claim the model can make with reasonable reliability and the other is a claim the model is inventing without any reliable knowledge basis.

Currency tracking — the second axis — addresses whether the intelligence picture has changed since the source was ingested. It does not address whether the model can reliably reason about the current intelligence picture regardless of when the sources were last updated.

The Competence axis is genuinely prior to the other three. Before asking how good the output is, you need to ask whether good output is achievable on this task.

## What Can Be Designed Now

The honest answer is that the Competence axis cannot be fully instrumented with current technology. There are six open research questions that define the boundary between what can be built now and what must wait for the research to mature. I will name them rather than bury them.

Reliable epistemic self-assessment in large language models. Out-of-distribution detection for autoregressive generative models — a fundamentally different problem than OOD detection for classifiers. Ground truth development for capability boundary assessment in intelligence domains. Compound capability interaction and degradation. Capability boundary shifts under adversarial information conditions. And the training objective conflict that makes epistemic non-declaration systematically less likely: the RLHF reward function penalizes "I don't know" as a response, which means every model trained to be helpful is trained to be overconfident at its capability limits.

None of these have clean answers today.

What can be built now are the tractable partial solutions. A capability taxonomy developed by the red team — a version-controlled registry of what the deployed model can reliably do, what it can partially do, and what it cannot do — provides the basis for a query classification service. Call it CPS/APORIA: the Capability Profiling Service. APORIA in Greek philosophy is the productive state of recognizing that a question has reached the limits of what can be reliably answered. That is exactly the signal the service needs to surface.

A lightweight out-of-distribution screening service — ODS/LETHE — can apply embedding-distance heuristics at query ingestion. Not reliable enough for hard enforcement. Reliable enough for an advisory signal that asks the analyst to pause before accepting outputs on a query that looks different from anything in the training distribution. And the existing adversarial input screening service, IAS/SCUDO, can be extended with a capability probe detection layer — a pattern classifier trained on query types characteristic of adversarial competence probing, which is a distinct pattern from standard adversarial injection.

These partial solutions should be deployed as advisory signals, not as hard controls. The false positive risk from imprecise capability boundary detection is significant — flagging legitimate analytical queries as out-of-scope would erode analyst trust in legitimate warnings and produce exactly the alert-fatigue problem that the intervention design has worked to avoid. Hard enforcement must wait for research question one and two to have tractable answers validated on intelligence-domain queries.

## Why This Matters Before It Can Be Solved

There is a temptation to defer this discussion until the research is mature. The logic: if we cannot reliably detect capability-limit failures, why define a governance axis for them?

The answer is that defining the axis is itself a governance act. It forces the question into the design process before the first deployment. Without a Competence axis, there is no design space for capability boundary signals, no red team mandate for capability taxonomy development, no calibration architecture that accounts for the qualitative difference between in-envelope and out-of-envelope model behavior. The axis is a commitment to solving the problem as the research matures, not a claim to have solved it already.

More urgently: adversarial competence probing does not wait for the research. An adversary designing denial-and-deception operations against AI-assisted analysis does not need us to have solved OOD detection. They only need the model's capability gaps to exist and to be exploitable. They exist. They are exploitable. The governance response to that fact cannot wait for perfect detection capability.

The Competence axis is proposed, not implemented. But the proposal is serious — because the failure mode it addresses is already operational.

---

*This post outlines a fourth accountability axis proposed for the THEMIS governance framework. The axis is described in full in Addendum D of the THEMIS platform design documentation, including the two new proposed services (CPS/APORIA and ODS/LETHE), enhancements to three existing services, and a detailed treatment of the six open research questions that must be answered before hard enforcement capability can be deployed.*

---

*The 13th Factor · Designing for Doubt · Coda: The Question the Three Axes Don't Ask*
