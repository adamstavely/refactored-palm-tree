# Introducing THEMIS
## Post 3 of 9 — Designing for Doubt

*The platform: what it is, what it does, and how its 14 services fit together.*

---

The first two posts in this series made a case about a problem and a framework. The problem: AI systems produce uniformly fluent output regardless of whether a claim is grounded in retrieved intelligence or confabulated from training weights, and most analytical interfaces give analysts no signal to tell the difference. The framework: every AI-assisted analytical interaction should be accountable across three independent dimensions — Origin (where did this claim come from), Currency (is this intelligence still valid), and Trust (is the analyst's reliance appropriately calibrated).

This post introduces the platform built to answer those three questions across the intelligence lifecycle. Its name is THEMIS: **Trusted Human-AI Enablement for Mission Intelligence and Safety**.

THEMIS is not a product. It is a governance architecture — fourteen integrated services that instrument the three accountability axes from the moment source intelligence enters the system to the moment a finished assessment leaves it.

## What THEMIS Is Not

Before describing what THEMIS does, it is worth being direct about what it is not, because the field is full of things that look similar and are not.

THEMIS is not a prompt guardrail system. Guardrails intercept problematic outputs after the model has generated them. THEMIS instruments the full lifecycle — ingestion, retrieval, inference, verification, and dissemination — not just the generation step.

THEMIS is not an audit logging system. Logging captures what happened. THEMIS captures what happened, who verified what, what the calibration state was, whether the sources were current, and it stores all of that in a cryptographically tamper-evident chain that can be produced for oversight. Logs are records. THEMIS produces evidence.

THEMIS is not a model evaluation framework. Model evaluation assesses model performance in controlled conditions. THEMIS measures analyst-AI calibration in operational conditions — a different problem with different solutions.

## The Fourteen Services

THEMIS is organized into five functional groups. Together they cover the Origin, Currency, and Trust axes.

---

### Safety Gates

The first two services are the entry point for every analytical interaction. Nothing passes them without explicit enforcement.

**PCES / AEGIS — Compartment and Classification Enforcement**
Enforces compartment boundaries on what source intelligence can enter a given analytical session. An analyst working at one classification level cannot receive retrieval results from a higher-classification corpus. Conflict of interest detection, collection authority validation, and classification log generation run here. PCES is the first service in every request path and the only one with no fallback — if PCES is unavailable, the session does not proceed.

**PGS / NOMOS — Analytic Standards Policy**
Enforces analytic standards — ICD 203 compliance, PII detection, interaction classification, output screening — as a policy rule engine. Every privilege-cleared interaction passes PGS before retrieval or inference. Policy rules are version-controlled and require Intelligence Oversight Board approval to change. The policy version applied to every session is logged for retrospective audit.

---

### Provenance and Accountability

These four services ensure that every AI-assisted analytical act has a traceable, tamper-evident record.

**Provenance / MOIRAI — The Provenance Backbone**
The center of gravity for THEMIS accountability. Every event from every other service flows into MOIRAI as a cryptographically signed event in a hash-chained provenance graph. Any retroactive modification of any record breaks the hash chain at the modification point and propagates visibly forward. RFC 3161 external timestamp anchoring every 24 hours provides an externally verifiable reference point. MOIRAI is what transforms THEMIS from a governance platform into an accountability infrastructure — the difference between records that might have been altered and records that demonstrably have not been.

**DPS / CODEX — Assessment Provenance**
Tracks provenance at the document level: when AI-generated content is inserted into a finished intelligence product, when an analyst modifies that content, when the document is opened, and when it is disseminated. DPS connects the AI assistance trail to the output — so that a finished assessment can be traced not just to a session but to the specific AI claims that contributed to specific paragraphs. DPS and MOIRAI must be designed concurrently; the document-level event schema depends on the provenance graph node structure.

**ERAS / LOGOS — Reasoning Audit**
Captures the reasoning process behind every AI response. Per-turn reasoning records, claim-level indexing, unsupported claim detection — ERAS surfaces when the AI produced a conclusion that has no retrieval backing in the context window. The reasoning records are the audit artifact for tradecraft compliance disclosure: an analyst can produce an ERAS record demonstrating what intelligence the AI analyzed, what reasoning it applied, and whether the claims it made were supported by retrieved sources.

**CVS / VERITAS — Source Corroboration**
Fires at generation time — before the analyst sees the response — checking every source reference the AI cites against current intelligence databases. If a cited source exists and is currently accessible, the claim is delivered with a corroborated signal. If the source cannot be resolved, the claim is flagged. If the source does not exist in any accessible database, the response is blocked. This is the most acute operational intervention in the quality layer: a claim that cites a nonexistent source is blocked before the analyst ever encounters it.

---

### Currency and Validity

These three services track whether the intelligence is still accurate in the world as it currently exists.

**TVS / KAIROS — Temporal Validity**
Applies configurable decay functions to every source in the corpus by source type — current reporting decays faster than foundational assessments, which decay faster than authoritative technical judgments. The validity score for every source decreases continuously from ingestion and can be refreshed by new corroborating collection. When KCS identifies an invalidation event, TVS propagates validity changes through the MOIRAI provenance graph to every claim derived from the invalidated source.

**KCS / ARGUS — Knowledge Currency**
Monitors external feeds — OSINT aggregators, allied reporting channels, collection system status dashboards — for events that invalidate or supersede sources currently in the corpus. When an external event indicates that a source has been compromised, a program has been cancelled, or an assessment has been superseded by new collection, KCS fires an ActiveInvalidationRequest to TVS. Target watch lists register entities of interest per active requirement for priority monitoring.

**MDS / KRONOS — Model Drift Detection**
Polls model version endpoints every four hours. When the underlying AI model changes — a new version deployed, a different checkpoint serving inference — MDS fires immediately. Analysts on active requirements are notified. TCS calibration baselines established for the prior model version are flagged as potentially stale. The intuitions an analyst developed about a prior model's behavior may no longer apply to the updated model, and MDS is the mechanism that makes that visible.

---

### Calibration and Trust

These three services measure and improve the human-AI reliance relationship.

**TCS / MIMIR — Trust Calibration**
The calibration engine. Maintains domain-specific Bayesian posteriors per analyst per claim type per model version. Tracks the Reliance-Accuracy Index — whether each analyst is trusting the AI at a level appropriate for its actual accuracy in this domain and at this task. Calibration state progresses from PRIOR ONLY (initialized from benchmark data) through CALIBRATING (accumulating session outcomes) to CALIBRATED (empirically grounded). TCS is what makes the confidence signals in ATHENA meaningful rather than decorative.

**FGTS / ALETHEIA — Feedback and Ground Truth**
Accumulates analyst verification outcomes as a weighted ground truth corpus that continuously updates TCS calibration. Not all corrections are equal — a correction submitted under Deadline-Critical pressure by an analyst with a high gaming probability score carries less weight than a supervisory-confirmed peer-agreed correction in a standard session. Five factors weight every correction before it enters the corpus. The ground truth corpus is the proprietary institutional knowledge that makes THEMIS calibration specific to this agency's analysts on this agency's requirements — not a generic benchmark.

**RQS / HERMES — Retrieval Quality**
Observes the retrieval pipeline for precision degradation, miss rates, and embedding drift. When retrieved chunks consistently fail to support the claims the model generates — a signal that the retrieval layer is not surfacing the relevant intelligence — RQS surfaces this as a quality alert. TVS-retrieval intersection monitoring identifies when low-validity sources were retrieved and contributed to synthesis even though their validity scores should have deprioritized them.

---

### Adversarial Defense

These two services assume a hostile information environment — because in intelligence analysis, the information environment always is.

**IAS / SCUDO — Input Adversarial Screening**
Screens every retrieved chunk for adversarial injection content before it enters the context window. Three detection layers: signature matching against the injection taxonomy, embedding similarity against the adversarial pattern corpus, and heuristic scoring for imperative-directive linguistic patterns. IAS sits at the retrieval ingestion boundary — before the model sees the content. During Deadline-Critical sessions, IAS sensitivity elevates one tier, because deadline pressure periods are specifically targeted by adversary denial-and-deception operations.

**MAS / EIDOLON — Media Authenticity**
Assesses the authenticity of every media artifact before AI analysis begins — video (deepfake detection, temporal manipulation, metadata integrity), audio (voice synthesis, speaker verification, acoustic continuity), image (GAN artifact detection, EXIF integrity, steganographic scan), and scanned documents (per-region OCR confidence scoring). MAS risk scores create hard confidence ceilings on claims derived from high-risk artifacts: a claim from a video with a high deepfake risk score cannot carry HIGH confidence regardless of how confidently the model asserted it.

---

## ATHENA: The Analyst Interface

THEMIS is the governance infrastructure. ATHENA is what the analyst actually uses.

ATHENA is a calibrated intelligence analysis interface with 31 targeted behavioral interventions designed to counteract the specific cognitive failure modes that make AI-assisted analysis dangerous: anchoring, automation bias, confirmation bias, and vigilance decrement. Each intervention targets one of these failure modes with a specific design response.

The most important four: the **Intention Gate** (the analyst declares their prior belief and falsification condition before the AI response renders — anchoring countermeasure), **Source Type Badges** (every claim carries a visible badge showing whether it was retrieved, parametric, synthesized, transcript-derived, or media-analyzed — origin visibility), the **Verification Queue** (unverified claims accumulate visibly; dissemination is blocked until they are actioned — accountability enforcement), and **Deadline-Critical Hard Constraints** (non-dismissable constraints under time pressure, including IAS screening elevation — adversarial defense during the periods when analysts are most vulnerable).

ATHENA is where the three accountability axes become analyst-facing signals. The Origin axis surfaces as source type badges. The Currency axis surfaces as validity indicators and No Corroboration hard interrupts. The Trust axis surfaces as calibrated confidence scores and the behavioral telemetry that informs them.

## The Intelligence Layer

The 14 THEMIS services govern the analytical workflow. Six additional services, built in years two and three, transform the platform from governance infrastructure into institutional intelligence.

SCRIBE tracks semantic changes across document versions with intelligence significance awareness. STOA orchestrates multi-step research with documented methodology trails. ORACLE surfaces predictive intelligence from historical requirement outcomes. MIRROR identifies similar historical requirements and what they can tell you about the current one. MNEMOSYNE builds the institutional knowledge graph from accumulated corrections and outcomes — the tacit knowledge that usually walks out the door when experienced analysts leave. PYTHIA anticipates what research and collection the analytic team will need before they know to ask for it.

MOS/SAGA — the Memory Orchestration Service — underpins all of this: session continuity, matter knowledge graduation, and agent task memory across the intelligence layer lifecycle.

The intelligence layer services are not day-one capabilities. ORACLE needs 200 historical requirements before its predictions are statistically meaningful. MNEMOSYNE cannot produce useful institutional knowledge until FGTS has accumulated enough weighted corrections and both ORACLE and MIRROR are operational. The data floor is real. But the data that will power the intelligence layer only accumulates if the governance infrastructure is capturing it from day one. Every month of ungoverned AI use is a month of institutional knowledge that is not being captured in a queryable form.

## The Roadmap

THEMIS is implemented in four phases across 17 months. Phase 1-2 (weeks 1-8) establishes the safety gates — PCES and PGS — so that no AI interaction touches source intelligence without compartment enforcement and analytic standards guardrails. Phase 3-4 (weeks 9-28) builds the provenance backbone, calibration engine, source corroboration, model drift detection, and assessment provenance — the complete accountability infrastructure. Phase 5-6 (weeks 29-46) activates the quality loops: ground truth accumulation, validity propagation, adversarial screening, and media authenticity. Phase 7-8 (weeks 47-66) completes the intelligence layer: knowledge currency monitoring, reasoning audit, and full tradecraft compliance surface.

The organization that builds and sustains THEMIS is the AI Trust Cell: 21 people across three teams — Research and Red Team, THEMIS Platform, and Intelligence Layer — with a structural rule that the research function never takes sprint tickets. Research independence is not a cultural norm in the AI Trust Cell. It is an organizational constraint.

## What This Series Covers From Here

The next three posts go deep on each accountability axis in turn — Origin, Currency, Trust — with the specific design decisions and failure modes each axis addresses. Then the synthesis: how the 31 ATHENA interventions and the 14 platform services work together as an integrated system. Then the intelligence layer and the organizational design that sustains all of it.

The detail that follows is substantial. This post is the map. The rest of the series is the territory.

---

*Previously: Three Questions Every High-Stakes AI Interface Should Answer*  
*Next: Where Did That Come From? — The Origin Axis*

---

*The 13th Factor · Designing for Doubt · Post 3 of 9*
