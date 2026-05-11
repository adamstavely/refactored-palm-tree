# ATHENA — Intervention Catalog
## Calibrated Analysis Interface | THEMIS Integration Reference

**Version 3.2 | THEMIS 9-Service Architecture**

28 Interventions · 9 THEMIS Services · 11 Supporting Services

---

# Executive Summary

ATHENA is the calibrated analysis interface layer of the THEMIS platform. It delivers 28 discrete design interventions organised across eight functional categories, each targeting a specific cognitive or behavioral failure mode in AI-assisted analysis: sycophancy, automation bias, vigilance decrement, anchoring, and confirmation bias.

THEMIS — Trusted Human-AI Enablement for Matter Intelligence and Safety — provides the persistent intelligence layer that transforms ATHENA from a well-designed interface into a learning system. Without THEMIS, ATHENA can display uncertainty but cannot serve calibrated confidence scores. It can prompt verification but cannot measure whether verification behavior changes over time.



# THEMIS Service Reference


## Safety Gates

| Code | Full Name | ATHENA Role |
|---|---|---|
| PCES / AEGIS | Privilege and Consent Enforcement | Gates access to retrieval corpus. Governs what can be exported. Privilege log entries in every session manifest. |
| PGS / NOMOS | Policy and Guardrails | Governs interaction classes. Constrains system prompt authority. Validates prompt repository templates. Governs export permissions. |


## Core Infrastructure

| Code | Full Name | ATHENA Role |
|---|---|---|
| Provenance / MOIRAI | AI Content Provenance | Records every claim, every source badge, every session configuration, every verification action. The audit substrate for all 28 interventions. |
| TCS / MIMIR | Trust Calibration | Powers claim confidence badges with calibrated scores (not model self-report). Computes analyst RAI across seven behavioral input streams. |


## Quality Feedback Loops

| Code | Full Name | ATHENA Role |
|---|---|---|
| FGTS / ALETHEIA | Feedback and Ground Truth | Primary consumer of Verification Queue actions. The feedback loop that makes TCS calibration improve over time. |
| TVS / KAIROS | Temporal Validity | Validity scores flow into Source Type Badges for Grounded claims. Ground truth corpus validity also governed by TVS extension. |
| RQS / HERMES | Retrieval Quality | Retrieval match scores power Claim Confidence Badges and Source Type classification. |


## Operational Intelligence and Strategic Capability

| Code | Full Name | ATHENA Role |
|---|---|---|
| KCS / ARGUS | Knowledge Currency | KCS invalidation events reach TVS, which updates Grounded claim validity scores visible in Source Type Badges. |
| ERAS / LOGOS | Explainability and Reasoning Audit | Claim Confidence Badges surface ERAS confidence signals. Context Window Inspector displays ERAS reasoning captures. Export Gate includes ERAS reasoning record. |



# Intervention Categories


## Category 1: Session Configuration (Interventions 2, 3, 4, 5)

Operates before any query. Mode is constrained by PCES/AEGIS and PGS/NOMOS. System prompt authority is bounded by PGS global policy rules. All configuration is logged to Provenance/MOIRAI for cross-session calibration analysis.



## Category 2: Session Scaffolding (Interventions 1, 6, 16, 22, 26, 27)

Frames the analytical work and resets that framing at intervals. Session Intent provides consequence-of-error weight for TCS/MIMIR RAI scoring. Pressure-Aware Session Mode calibrates the intervention profile to declared pressure context. Multi-Analyst Session Framework extends scaffolding to supervisory review, matter handoff, and collaborative workflows.



## Category 3: Pre-Response Interventions (Interventions 7, 8)

Fire between query submission and response rendering, before the model output can anchor analyst reasoning. Input Type Classification maps to PGS interaction classes. The Intention Gate produces behavioral signals consumed by TCS/MIMIR and FGTS/ALETHEIA.



## Category 4: Response Interventions (Interventions 9, 10, 11, 12, 21)

Deepest THEMIS integration. Claim Confidence Badges are the primary consumer of TCS/MIMIR calibrated scores. Source Type Badges populate Provenance/MOIRAI at claim generation time across eight source types. Counter-Position is governed by PGS/NOMOS as a distinct interaction class.



## Category 5: Verification Infrastructure (Interventions 13, 14, 15)

The primary ATHENA-to-THEMIS feedback channel. Every resolved verification action is a FGTS/ALETHEIA correction event — the ground truth that updates TCS/MIMIR calibration models. Context-sensitive actions ensure FGTS receives correctly typed correction signals across all eight source types.



## Category 6: Behavioral Telemetry, Control, and Output (Interventions 17, 18, 19, 20, 23, 24)

Per-message feedback and annotations provide secondary FGTS/ALETHEIA correction signals. Context Window Inspector makes Provenance/MOIRAI navigable and surfaces RQS/HERMES and ERAS/LOGOS data. Export Gate is governed by PCES/AEGIS and PGS/NOMOS. Session Manifest IS the Provenance/MOIRAI record.



## Category 7: Session Integrity and Multi-Analyst Monitoring (Interventions 25, 27, 28)

Address failure modes visible only at the session-integrity and cross-analyst level. Compliance Gaming Detection requires a TCS/MIMIR analytics extension computing a gaming probability score from four behavioral signal streams. Cross-Analyst Disagreement Detection requires a Provenance/MOIRAI matter-level cross-session query index.



## Category 8: Pressure Response (Intervention 26)

The only intervention that actively changes the behavior of other interventions rather than adding a new one. Under Deadline-Critical mode, the Intention Gate compresses, the Reformulation Gate defers, the Episode Boundary accelerates, and the Export Gate hardens. PGS/NOMOS governs the export gate rules.



---


# Intervention Catalog — 28 Entries

Each entry covers: purpose, mechanism targets, runtime service dependencies, THEMIS integration, supporting service dependencies, and data produced and consumed.



---


## 1. Session Intent Declaration

**Category:** Session Scaffolding



### Purpose

A required pre-session form capturing task description, the decision the session supports, and the consequence of an error. The first query is blocked until completed. Skips are logged. After submission a persistent bar below the header displays all three fields for the duration of the session.



### Mechanism Targets

- Stakes recalibration (Lerner and Tetlock, 1999): motivational maintenance through consequence salience.
- Prospective memory activation: declared intent before any model output creates a cognitive anchor that survives engagement with fluent confident prose.
- Session framing: connecting the analytical task to real-world consequence weight counteracts the flow-state tendency to deprioritise verification over time.


### Runtime Service Dependencies

- Session context store: persists task, decision, and consequence fields keyed to session ID.
- Behavioral telemetry: logs whether intent was set or skipped and captures the content when set.


### THEMIS Integration

PGS/NOMOS may mandate intent declaration as a policy condition for high-sensitivity interaction classes. PCES/AEGIS logs session intent against the privilege scope of the matter. TCS/MIMIR uses consequence-of-error weight from declared intent to modulate RAI scoring.


**Active THEMIS services:** PCES/AEGIS (P) · PGS/NOMOS (P) · TCS/MIMIR (P)



### Supporting Service Dependencies

None at runtime.


**Data produced:** `{task, decision, consequence, set_at, skipped:bool}`

**Data consumed:** None



---


## 2. Chat Mode / Knowledge Source Configuration

**Category:** Session Configuration



### Purpose

Toggle between three knowledge modes: Training Only (parametric knowledge, no retrieval), RAG plus Training (retrieved documents plus model knowledge), and Company Docs Only (restricted to internal corpus). Active mode displayed persistently in the header and logged in the session manifest.



### Mechanism Targets

- Tesler's Law: the epistemic difference between retrieved and parametric knowledge is real complexity. Making it visible externalises it rather than transferring it invisibly to the analyst.
- Mode transparency: trust calibration differs when the model draws on a retrieved document versus training weights.


### Runtime Service Dependencies

- Inference gateway: routes requests with or without retrieval based on mode selection.
- MIMIR retrieval infrastructure: corpus selection and retrieval scope determined by mode.


### THEMIS Integration

PCES/AEGIS determines which corpus partitions are accessible given the analyst matter scope. PGS/NOMOS may restrict certain modes for specific interaction classes. Provenance/MOIRAI logs mode in every session record. TCS/MIMIR maintains mode-stratified calibration. RQS/HERMES only fires in RAG modes.


**Active THEMIS services:** PCES/AEGIS (C) · PGS/NOMOS (C) · Provenance/MOIRAI (P) · TCS/MIMIR (C) · RQS/HERMES (C)



### Supporting Service Dependencies

Inference gateway (routing). MIMIR retrieval infrastructure (corpus selection).


**Data produced:** `{mode: training|rag|company, changed_at, corpus_version}`

**Data consumed:** PCES privilege scope. PGS mode-restriction policy. MIMIR corpus inventory.



---


## 3. System Prompt Editor

**Category:** Session Configuration



### Purpose

Live editable system prompt with save and load, accessible from the Config sidebar. Supports named presets. Changes take effect on the next inference call. A hash of the active prompt is included in the session manifest.



### Mechanism Targets

- Transparency and user agency: reveals the instruction layer that shapes model behavior.
- Externalising the prompt as an explicit modifiable parameter prevents silent drift in model behavior across sessions.


### Runtime Service Dependencies

- Inference gateway: injects system prompt into context for every request.
- Session context store: stores active prompt and tracks changes within the session.


### THEMIS Integration

PGS/NOMOS is the authoritative policy layer. The system prompt cannot override a PGS guardrail. System prompt changes are logged to Provenance/MOIRAI as PolicyEvaluationEvents.


**Active THEMIS services:** PGS/NOMOS (B) · Provenance/MOIRAI (P)



### Supporting Service Dependencies

Inference gateway (prompt injection at request time).


**Data produced:** `{hash, content, version, changed_at, source: custom|preset}`

**Data consumed:** PGS global policy rules.



---


## 4. Model Parameter Configuration

**Category:** Session Configuration



### Purpose

Five configurable sliders covering temperature, top-P, top-K, presence penalty, and max tokens. Three named presets: Precise, Balanced, and Exploratory. Parameters recorded in the session manifest.



### Mechanism Targets

- Expert control over output distribution: low temperature for high-precision work; high temperature for exploratory generation.
- Reproducibility: identical parameter configurations required to reconstruct the conditions that produced a prior response.


### Runtime Service Dependencies

- Inference gateway: parameters injected at request time alongside system prompt.


### THEMIS Integration

Parameters are logged in Provenance/MOIRAI as part of every session record. TCS/MIMIR uses parameter configuration as session context in calibration computation.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (C)



### Supporting Service Dependencies

Inference gateway only.


**Data produced:** `{temperature, top_p, top_k, presence_penalty, max_tokens, preset_used}`

**Data consumed:** None



---


## 5. Prompt Repository

**Category:** Session Configuration



### Purpose

A library of curated analytical prompt templates accessible from the Config sidebar. Templates cover supply chain risk, actor intent analysis, counter-analysis, and calibration checks. Template usage is logged per exchange.



### Mechanism Targets

- Reduces query-level variance: standardised prompts enable meaningful comparison of model performance across sessions.
- Templates structured to elicit analyst framing before model output.


### Runtime Service Dependencies

- Session context store: logs which template was used per exchange.


### THEMIS Integration

PGS/NOMOS maintains a policy-approved prompt library. TCS/MIMIR stratifies calibration analysis by template ID.


**Active THEMIS services:** PGS/NOMOS (C) · TCS/MIMIR (P)



### Supporting Service Dependencies

None direct.


**Data produced:** `{template_id, template_name, exchange_id, customised:bool}`

**Data consumed:** PGS approved interaction class definitions.



---


## 6. Session History

**Category:** Session Scaffolding



### Purpose

A browsable list of prior sessions with metadata including topic, exchange count, claim count, and date. Loading a prior session injects its context into the current conversation and activates disagreement detection for overlapping queries.



### Mechanism Targets

- Continuity of analysis: analysts can build on prior work without re-establishing context.
- Loading a prior session directly activates disagreement detection as a downstream effect.


### Runtime Service Dependencies

- Session context store: retrieves prior session state.
- Behavioral telemetry: logs session load events.


### THEMIS Integration

Prior sessions retrieved from Provenance/MOIRAI records. TCS/MIMIR surfaces prior RAI scores alongside loaded sessions. FGTS/ALETHEIA surfaces prior correction history.


**Active THEMIS services:** Provenance/MOIRAI (C) · TCS/MIMIR (C) · FGTS/ALETHEIA (C)



### Supporting Service Dependencies

Provenance/MOIRAI session records. TCS calibration history store.


**Data produced:** `{loaded_session_id, loaded_at, context_injected:bool}`

**Data consumed:** Provenance/MOIRAI session records. TCS RAI scores per prior session. FGTS correction history.



---


## 7. Input Type Classification

**Category:** Pre-Response Interventions



### Purpose

Auto-classification of each user query into one of four types: Factual, Analytical, Predictive, or Opinion. The badge appears on the user message immediately after submission, before the response renders.



### Mechanism Targets

- Sets epistemic expectations before the response is read. A Predictive claim warrants far higher uncertainty than a Factual claim matched to a retrieved document.
- Seeing the query classified prompts deliberate evaluation rather than passive acceptance.


### Runtime Service Dependencies

- Query classifier: lightweight NLP or embedding-based classification, runs post-submit before inference gateway call.


### THEMIS Integration

PGS/NOMOS maps input types to interaction class definitions. Provenance/MOIRAI logs the classified interaction type as part of the turn record. TCS/MIMIR uses input type as a calibration variable. ERAS/LOGOS captures reasoning at different depths depending on input type.


**Active THEMIS services:** PGS/NOMOS (C) · Provenance/MOIRAI (P) · TCS/MIMIR (P) · ERAS/LOGOS (C)



### Supporting Service Dependencies

Query classifier (lightweight NLP or embedding-based, pre-inference).


**Data produced:** `{exchange_id, query_type: factual|analytical|predictive|opinion, confidence}`

**Data consumed:** PGS interaction class definitions. User query text.



---


## 8. Intention Gate

**Category:** Pre-Response Interventions



### Purpose

A required two-field form that prevents the model response from rendering until the analyst submits: what they already believe about the query topic, and what evidence or argument would change their mind. A character-count minimum is enforced on both fields.



### Mechanism Targets

- Prospective memory (Brandimonte et al.): setting a verification intention before reading creates a cognitive hook that survives engagement with fluent model output.
- Anchoring countermeasure: the analyst commits their prior belief before model output sets the reference point.
- System 2 activation: articulating a prior and a falsification condition forces deliberate engagement before the fluency signal arrives.


### Runtime Service Dependencies

- Session context store: stores prior text and changer text keyed to the exchange ID.
- Behavioral telemetry: logs completion quality including word count, time-to-complete, and whether both fields were substantively populated.


### THEMIS Integration

Intention gate completion quality is consumed by TCS/MIMIR as a behavioral RAI input. Prior belief text is consumed by FGTS/ALETHEIA as a signal for whether analyst corrections correlate with articulated prior beliefs.


**Active THEMIS services:** TCS/MIMIR (P) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

None at runtime.


**Data produced:** `{exchange_id, prior_text, changer_text, word_counts, completion_time_ms}`

**Data consumed:** None



---


## 9. Claim Confidence Badges

**Category:** Response Interventions



### Purpose

Inline badges on each discrete claim: HIGH CONF in teal, MED CONF in amber, UNVERIFIED in red. Generated from TCS calibrated scores, not model self-report. Clicking a badge adds the associated claim to the Verification Queue.



### Mechanism Targets

- Von Restorff Effect: isolated, visually distinct elements receive more attention and are better recalled.
- Breaks the fluency-credibility coupling: LLM output fluency acts as a proxy for credibility. Visual differentiation disrupts the automatic well-written-equals-accurate inference.
- Automation bias countermeasure: calibrated uncertainty signals prevent passive acceptance of authoritative-seeming output.


### Runtime Service Dependencies

- Inference gateway: structured claim extraction requested alongside response generation.
- Claim extractor: identifies discrete claim spans.
- TCS/MIMIR: provides calibrated confidence scores per (claim type, interaction class, model version).
- RQS/HERMES: retrieval match scores inform confidence for grounded claims.


### THEMIS Integration

TCS/MIMIR is the primary runtime consumer: serves calibrated scores based on historical accuracy, not model self-reported confidence. Without TCS, badges display model self-report — systematically overconfident in domains where the model has memorised plausible-sounding but inaccurate content. FGTS/ALETHEIA is the calibration source: verification outcomes flow back through FGTS to update TCS confidence models.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (C) · FGTS/ALETHEIA (C) · RQS/HERMES (C) · ERAS/LOGOS (C)



### Supporting Service Dependencies

Inference gateway. Claim extractor. RQS/HERMES retrieval match scores.


**Data produced:** `{claim_id, span_text, conf_level, interaction_class, claim_type, exchange_id}`

**Data consumed:** TCS/MIMIR calibrated scores. RQS/HERMES retrieval match scores.



---


## 10. Source Type Badges

**Category:** Response Interventions



### Purpose

Inline provenance type indicators adjacent to each claim badge. Eight types: GRND (retrieved source document), PARAM (training weights, no retrieval), SYNTH (inferred from partial sources), TRANSCRIPT (AI-generated transcript of video/audio — derivative artifact with its own ASR accuracy profile), VIDEO (direct AI analysis of video), AUDIO (direct AI analysis of audio), IMAGE (direct AI analysis of image), OCR (OCR-extracted text from scanned document, shown with confidence score e.g. OCR 87%). Each type triggers a different verification action set.



### Mechanism Targets

- Tesler's Law: externalises the epistemic origin of each claim across eight distinct source types.
- Type-appropriate verification behavior: GRND prompts source-document checking; PARAM prompts independent lookup; SYNTH prompts scepticism about specific figures; TRANSCRIPT prompts checking against the source recording; media types prompt review of the relevant artifact.
- TRANSCRIPT is the most important new type epistemically — an AI transcript is a derivative artifact with its own error rate, not equivalent to a retrieved document.
- OCR confidence surfacing: the OCR badge includes the underlying confidence score so the analyst sees extraction quality before verifying.


### Runtime Service Dependencies

- Claim extractor: classifies each claim across all eight source types.
- RQS/HERMES: provides retrieval trace with chunk match scores for text-based types.
- MAS/EIDOLON: provides media authenticity risk score and modality classification for media-derived claims.
- Provenance/MOIRAI (write): claim source type and provenance chain stored at generation time.


### THEMIS Integration

Provenance/MOIRAI stores source type for every claim at generation time — cannot be reliably recovered later if not captured at generation. TCS/MIMIR calibrates by modality: media-derived claims (VIDEO, AUDIO, IMAGE, TRANSCRIPT, OCR) require separate calibration from text-derived claims. FGTS/ALETHEIA routes OCR errors, transcript inaccuracies, and image authenticity flags to different remediation paths.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · TVS/KAIROS (C) · RQS/HERMES (C)



### Supporting Service Dependencies

RQS/HERMES (retrieval trace for text types). Claim extractor (source type classification). MAS/EIDOLON (media authenticity risk score).


**Data produced:** `{claim_id, src_type: grnd|param|synth|transcript|video|audio|image|ocr, source_doc_ids[], match_scores[], synthesis_flag, ocr_confidence_score?, mas_authenticity_score?, modality?}`

**Data consumed:** RQS/HERMES retrieval trace. TVS/KAIROS validity scores for grounded and media sources. MAS/EIDOLON authenticity risk scores.



---


## 11. Counter-Position Panel

**Category:** Response Interventions



### Purpose

A collapsed panel at the bottom of each response containing a structurally generated adversarial challenge to the response claims, framing, and unstated assumptions. Toggled by the analyst. Framed as a challenge, not a disclaimer.



### Mechanism Targets

- Confirmation bias countermeasure: presents disconfirming signals before the analyst finalises their interpretation.
- Dual process activation: the counter-position requires active reading and evaluation rather than passive consumption.
- Agreement rate buffer: challenges analyst framing even when the primary response aligns with it.


### Runtime Service Dependencies

- Inference gateway: secondary adversarial generation call after primary response completes.
- Adversarial generator: structured second prompt producing the strongest plausible challenge.


### THEMIS Integration

PGS/NOMOS governs the adversarial generation call as a distinct interaction class. TCS/MIMIR logs counter-position engagement as a behavioral RAI input. ERAS/LOGOS treats counter-positions as reasoning audit artifacts.


**Active THEMIS services:** PGS/NOMOS (C) · TCS/MIMIR (P) · ERAS/LOGOS (P)



### Supporting Service Dependencies

Inference gateway. Adversarial generator (secondary inference call).


**Data produced:** `{exchange_id, counter_text, opened:bool, time_before_open_ms}`

**Data consumed:** PGS interaction class rules for adversarial generation.



---


## 12. Agreement Rate Tracker

**Category:** Session Monitoring



### Purpose

A persistent header indicator showing the session agreement-to-challenge ratio. Turns amber when two or more consecutive responses align with analyst framing without a challenge. Updates in real time after each response.



### Mechanism Targets

- Social accountability: making the agreement pattern visible changes behavior.
- Sycophancy detection: surfaces the model tendency to agree as a session-level metric.
- Vigilance maintenance: streak warnings fire precisely where criterion shift is most likely.


### Runtime Service Dependencies

- Agreement classifier: classifies each response as aligned or challenging relative to analyst framing.
- Session context store: accumulates agreement and challenge counts within the session.


### THEMIS Integration

Session agreement rate fed to TCS/MIMIR at session close. FGTS/ALETHEIA uses agreement pattern data to identify whether correction frequency correlates with agreement rate.


**Active THEMIS services:** TCS/MIMIR (P) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

Agreement classifier (post-response, per exchange).


**Data produced:** `{exchange_id, aligned:bool, streak_count, session_rate}`

**Data consumed:** Response content. Analyst query framing.



---


## 13. Verification Queue

**Category:** Verification Infrastructure



### Purpose

Right-panel list of claims flagged for analyst verification. Claims auto-populate from response confidence badges. Each item is expandable to show full source provenance before the analyst takes an action. State tracks from pending through confirmed, misrepresents, independent, unverifiable, disputed, and dismissed.



### Mechanism Targets

- Zeigarnik Effect: open verification tasks persist cognitively. The queue creates explicit incomplete tasks rather than implicit responsibility.
- Makes verification a concrete action rather than an abstract obligation.
- Separates reading from verification: the analyst reads the response, then reviews each claim with provenance visible.


### Runtime Service Dependencies

- Session context store: holds queue state within the session per claim ID.
- Verification store: persists outcomes across sessions.
- RQS/HERMES: source document passages retrieved and displayed per claim in the expanded view.


### THEMIS Integration

FGTS/ALETHEIA is the primary THEMIS consumer of Verification Queue outcomes. Every resolved action is a correction event — ground truth flowing into the feedback corpus. The consequence-of-error weight from Session Intent modulates FGTS outcome weighting. TCS/MIMIR receives verification completion rate as a direct RAI input.


**Active THEMIS services:** PCES/AEGIS (C) · Provenance/MOIRAI (P) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · RQS/HERMES (C)



### Supporting Service Dependencies

RQS/HERMES (source document retrieval for provenance panels). Verification store (outcome persistence).


**Data produced:** `{claim_id, action, timestamp, analyst_id, session_id, consequence_weight}`

**Data consumed:** PCES privilege scope. RQS/HERMES source documents. Provenance/MOIRAI claim records.



---


## 14. Context-Sensitive Verification Actions

**Category:** Verification Infrastructure



### Purpose

The verification action set varies by claim source type across all eight badge types. Text: Grounded (Confirmed against source / Misrepresents source / Dismiss), Parametric (Verified independently / Could not verify / Incorrect), Synthesis (Figure confirmed / Not in sources / Dismiss). Media: Transcript (Verified against recording / Transcript may be inaccurate / Dismiss), Video (Verified against clip / Clip unavailable / Characterization incorrect / Dismiss), Audio (Verified against recording / Recording unavailable / Characterization incorrect / Dismiss), Image (Verified against image / Image authenticity flagged / Characterization incorrect / Dismiss), OCR (Confirmed against original document / OCR error — text misread / Dismiss).



### Mechanism Targets

- Action-specific semantics: Confirmed against source (GRND) is a qualitatively stronger epistemic act than Verified independently (PARAM), and the interface reflects this.
- Derivative artifact distinction: Transcript actions distinguish between verifying the claim against the recording versus flagging the transcript itself as inaccurate — two different failure modes.
- OCR error as a distinct failure mode: routes differently in FGTS than model mischaracterisation.


### Runtime Service Dependencies

- Verification store: receives action type with claim source type, modality, and media-specific context.


### THEMIS Integration

Each action type across all eight source types carries different weight and routing in FGTS/ALETHEIA. OCR error signals text extraction failure rather than model reasoning failure. Transcript may be inaccurate signals ASR error. Image authenticity flagged routes to MAS/EIDOLON as a confirmed authenticity concern. TVS/KAIROS receives validity signals when GRND, VIDEO, AUDIO, or IMAGE claims are disputed on the basis of source supersession.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · TVS/KAIROS (C)



### Supporting Service Dependencies

None direct. This is analyst judgment capture. MAS/EIDOLON authenticity score surfaced in IMAGE provenance panel.


**Data produced:** `{claim_id, src_type, modality?, action_type, semantic_weight, mas_authenticity_confirmed?, ocr_error_confirmed?}`

**Data consumed:** Claim source type and modality from Provenance/MOIRAI. MAS/EIDOLON authenticity risk score for IMAGE claims. OCR confidence score for OCR claims.



---


## 15. Reformulation Gate

**Category:** Comprehension Verification



### Purpose

Triggered after responses that challenge analyst framing or introduce complex distinctions. Requires the analyst to paraphrase the model core claim in their own words before proceeding. A side-by-side comparison of the analyst paraphrase and the model key claim is displayed after submission.



### Mechanism Targets

- Semantic processing depth (Craik and Lockhart, 1972): writing a paraphrase forces semantic engagement that surface reading does not.
- Anchoring countermeasure: the reformulation must be formed independently before the model comparison is shown.
- The effort of reformulation cannot be shortcut by fluency recognition.


### Runtime Service Dependencies

- Session context store: stores reformulation prompt, analyst answer, and model reference answer.
- Semantic similarity service: computes embedding cosine similarity between analyst paraphrase and model key claim.


### THEMIS Integration

Reformulation quality scores consumed by TCS/MIMIR as a comprehension signal. FGTS/ALETHEIA uses domain-specific reformulation patterns to identify comprehension gaps that correlate with later correction events. ERAS/LOGOS consumes reformulation data as a signal of whether the analyst engaged with the reasoning before verifying.


**Active THEMIS services:** TCS/MIMIR (P) · FGTS/ALETHEIA (P) · ERAS/LOGOS (C)



### Supporting Service Dependencies

Semantic similarity service (embedding comparison).


**Data produced:** `{exchange_id, analyst_text, model_text, similarity_score, domain}`

**Data consumed:** Model key claim text. Semantic similarity service.



---


## 16. Episode Boundary

**Category:** Session Scaffolding



### Purpose

A session checkpoint firing every three exchanges. Closes the current episode and requires the analyst to commit to one specific claim they will verify independently before acting on the session analysis. Proceeding to the next episode requires a non-empty commitment. Commitment text is logged and surfaced in the next session opening.



### Mechanism Targets

- Vigilance decrement countermeasure: fires at the interval where criterion shift is most likely.
- Zeigarnik Effect: the episode close commitment creates an explicit open task that persists after the session ends.
- Session framing reset: each boundary resets the internal reference point.


### Runtime Service Dependencies

- Session context store: manages episode state, exchange count, trigger logic, and commitment text.
- Behavioral telemetry: logs commitment text and episode close timestamp.


### THEMIS Integration

Episode close commitment text logged to behavioral telemetry. TCS/MIMIR computes cross-episode follow-through rate as a direct RAI input. FGTS/ALETHEIA uses commitment text and follow-through correlation as a signal quality indicator.


**Active THEMIS services:** TCS/MIMIR (P) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

None direct.


**Data produced:** `{episode_num, commitment_text, closed_at, exchange_count, session_id}`

**Data consumed:** Prior episode commitment from behavioral telemetry.



---


## 17. Per-Message Feedback

**Category:** Behavioral Telemetry



### Purpose

Response-level positive and negative feedback via thumbs up and thumbs down controls below each AI response. Toggle state. Granular feedback at the response level, distinct from claim-level verification queue actions.



### Mechanism Targets

- Granular signal capture: session-level satisfaction obscures response-level variation.
- The act of providing explicit feedback activates an evaluative stance.


### Runtime Service Dependencies

- Behavioral telemetry: response feedback keyed to (exchange_id, analyst_id, session_id).


### THEMIS Integration

Feedback valence aggregated by FGTS/ALETHEIA by interaction class, model version, and parameter configuration as a secondary correction signal. TCS/MIMIR uses feedback pattern distributions as an analyst engagement metric.


**Active THEMIS services:** TCS/MIMIR (P) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

None.


**Data produced:** `{exchange_id, valence: up|down, changed:bool, timestamp}`

**Data consumed:** None



---


## 18. Per-Message Annotation

**Category:** Behavioral Telemetry



### Purpose

Inline analyst notes attached to specific AI responses, opened via the Annotate button. Annotations displayed as visually distinct blocks within the response. Persisted in session state and included in session exports.



### Mechanism Targets

- Externalises analyst judgment: creates a retrievable record of analyst reasoning at the moment of reading, structurally separate from model output.
- Prevents retrospective attribution error: analysts who annotate in real time produce more accurate records of their actual reasoning.


### Runtime Service Dependencies

- Session context store: annotation text stored keyed to (exchange_id, annotation_id).
- Annotation store: persists across sessions.


### THEMIS Integration

Annotations logged to Provenance/MOIRAI as analyst-generated enrichments to the turn record. TCS/MIMIR uses annotation frequency as an analyst engagement signal. FGTS/ALETHEIA analyses annotation content for uncertainty markers. ERAS/LOGOS consumes annotations as evidence that the analyst engaged with the reasoning.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · ERAS/LOGOS (C)



### Supporting Service Dependencies

None at runtime.


**Data produced:** `{exchange_id, annotation_id, text, timestamp, analyst_id}`

**Data consumed:** None



---


## 19. Conversation Branching

**Category:** Session Control



### Purpose

Fork the conversation at any exchange to explore an alternative analytical path without affecting the current thread. Branched conversations maintain independent state. Branch point and parent session ID are recorded.



### Mechanism Targets

- Enables controlled comparison: same prior context, different framing, produces different model behavior that can be directly compared.
- Supports adversarial self-analysis: analysts can test whether their framing drove the model conclusions.


### Runtime Service Dependencies

- Session context store: manages branch state including parent session ID and branch point exchange ID.
- Inference gateway: supports context injection from branched history.


### THEMIS Integration

Provenance/MOIRAI records branch lineage — the complete parent-to-branch relationship is part of the provenance graph. FGTS/ALETHEIA uses branch divergence as a framing sensitivity signal.


**Active THEMIS services:** Provenance/MOIRAI (P) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

Inference gateway (branched context injection).


**Data produced:** `{branch_id, parent_session_id, branch_point_exchange_id, created_at}`

**Data consumed:** Parent session context from Provenance/MOIRAI up to branch point.



---


## 20. Context Window Inspector

**Category:** Transparency



### Purpose

A modal inspector showing the full context received by the model for a specific exchange: system prompt with token count, retrieved chunks labeled by source document with match score and token count, conversation history by exchange, and the current user message. Each chunk is independently expandable.



### Mechanism Targets

- Radical transparency: closes the most significant epistemic gap in current AI interfaces.
- Enables diagnostic reasoning: when a response changes unexpectedly, the inspector reveals whether retrieval changed, conversation history grew, or system prompt was modified.
- Prerequisite for provenance literacy.


### Runtime Service Dependencies

- Inference gateway: captures full context snapshot at request time.
- Session context store: stores context snapshot per exchange for later inspection.


### THEMIS Integration

Context Window Inspector makes Provenance/MOIRAI navigable by the analyst. RQS/HERMES retrieval match scores and miss flags are visible per chunk. ERAS/LOGOS reasoning captures are visible for the response. Context inspector usage logged to behavioral telemetry as an engagement signal.


**Active THEMIS services:** Provenance/MOIRAI (C) · RQS/HERMES (C) · ERAS/LOGOS (C)



### Supporting Service Dependencies

Inference gateway (context capture). RQS/HERMES (chunk metadata).


**Data produced:** `{exchange_id, system_prompt_tokens, retrieved_chunks[], history_tokens, user_message_tokens, total_tokens}`

**Data consumed:** Provenance/MOIRAI chunk records. RQS/HERMES chunk metadata and match scores. ERAS/LOGOS reasoning index.



---


## 21. Disagreement Detection

**Category:** Session Monitoring



### Purpose

When a query resembles one from a loaded prior session, the current response is compared against the prior response for the same query. If material divergence is detected, an amber banner surfaces a side-by-side comparison. Dismissible but logged.



### Mechanism Targets

- Makes response instability visible: in standard interfaces, model response variance across sessions is completely invisible.
- Prevents tacit reliance on consistency: analysts sometimes implicitly assume that because the model said X before, X is stable.


### Runtime Service Dependencies

- Semantic similarity service: compares current response claims against stored prior session claims.
- Session context store: stores prior session claims when a session is loaded.


### THEMIS Integration

TCS/MIMIR receives response divergence events as session-level calibration instability signals. FGTS/ALETHEIA uses divergence events as correction signals when the analyst resolves the disagreement by verifying one version against a source. TVS/KAIROS provides context when divergence results from source supersession between sessions.


**Active THEMIS services:** Provenance/MOIRAI (C) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · TVS/KAIROS (C)



### Supporting Service Dependencies

Semantic similarity service. Session context store. Prior session records from Provenance/MOIRAI.


**Data produced:** `{exchange_id, prior_session_id, similarity_score, divergence_type, dismissed:bool}`

**Data consumed:** Provenance/MOIRAI prior session claim records. TVS/KAIROS validity change events.



---


## 22. Session Synthesis Workspace

**Category:** Analyst Scaffolding



### Purpose

A dedicated right-panel workspace for the analyst's own independent synthesis. Three fields: conclusions I am confident in, assumptions I am relying on, and claims I still need to verify. Structurally and visually distinct from all model output. Exportable as a session header attached to the full provenance record.



### Mechanism Targets

- Structural boundary between model-generated and analyst-generated content.
- Peak-end rule intervention: if the analyst completes a synthesis before session close, that cognitive work becomes the memorable peak.
- Session closure: completing the synthesis fields creates closure on open analytical questions.


### Runtime Service Dependencies

- Session context store: synthesis text fields persisted per session.


### THEMIS Integration

Synthesis content is exported as the header of the Provenance/MOIRAI session record. TCS/MIMIR computes longitudinal correlation between synthesis conclusions and verified claims as a long-term RAI signal. ERAS/LOGOS consumes synthesis content as a reasoning completeness signal.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (P) · ERAS/LOGOS (C)



### Supporting Service Dependencies

None.


**Data produced:** `{session_id, conclusions, assumptions, to_verify, completed_at}`

**Data consumed:** None



---


## 23. Export with Provenance Gate

**Category:** Output Control



### Purpose

Export is flagged when unverified claims above a configurable confidence threshold remain in the queue. A warning count appears above the export button. Proceeding requires explicit confirmation. The exported artifact includes full provenance: model version, parameters, system prompt hash, chat mode, corpus version, session intent, all claims with verification states, retrieval trace, and a THEMIS status summary.



### Mechanism Targets

- Zeigarnik Effect enforced structurally: the analyst cannot close the session without explicitly acknowledging open verification items.
- Accountability at the point of consequence: export is when analysis leaves the analyst's hands.
- Externalising epistemic complexity: unverified claims are made visible in the export artifact.


### Runtime Service Dependencies

- Verification store: counts unresolved claims by confidence level.
- Provenance/MOIRAI: assembles full provenance record for export.
- Session manifest: provides parameter and configuration snapshot.


### THEMIS Integration

PCES/AEGIS governs what can be exported. PGS/NOMOS determines export permissions by interaction class. Provenance/MOIRAI export triggers creation of the canonical provenance record. TCS/MIMIR contributes RAI snapshot at export time. FGTS/ALETHEIA contributes the verification summary. ERAS/LOGOS contributes the reasoning audit record.


**Active THEMIS services:** PCES/AEGIS (C) · PGS/NOMOS (C) · Provenance/MOIRAI (P) · TCS/MIMIR (C) · FGTS/ALETHEIA (C) · ERAS/LOGOS (C)



### Supporting Service Dependencies

Verification store. Provenance/MOIRAI. Session manifest.


**Data produced:** `Full session record with provenance, verification states, TCS RAI snapshot, ERAS reasoning record, PGS policy log, PCES privilege classification.`

**Data consumed:** PCES privilege scope. PGS export policy. TCS RAI state. FGTS verification summary. ERAS reasoning index.



---


## 24. Session Manifest

**Category:** Transparency



### Purpose

A modal view of the complete session configuration record: model ID, session ID, knowledge mode, RAG corpus version, temperature, top-P and top-K, system prompt hash, session intent fields, and live verification statistics. Accessible at any point during the session.



### Mechanism Targets

- Reproducibility: the manifest contains every parameter needed to reconstruct the conditions that produced any response.
- Audit trail: a complete record of configuration, intent, and verification state.
- Calibration awareness: showing analysts their live verification statistics mid-session provides real-time feedback.


### Runtime Service Dependencies

- Session context store: all manifest fields drawn from live session state.


### THEMIS Integration

The Session Manifest IS the Provenance/MOIRAI record for this session. PCES/AEGIS privilege classification and matter scope are manifest fields. PGS/NOMOS policy version and active rule set are manifest fields. TCS/MIMIR contributes live RAI score and calibration flags.


**Active THEMIS services:** PCES/AEGIS (C) · PGS/NOMOS (C) · Provenance/MOIRAI (B) · TCS/MIMIR (C)



### Supporting Service Dependencies

None. Manifest assembles from session context store and THEMIS service APIs.


**Data produced:** `Session manifest record: all configuration fields plus live verification statistics plus THEMIS service status.`

**Data consumed:** PCES privilege scope. PGS policy version. TCS live RAI score. Provenance/MOIRAI session record.



---


## 25. Compliance Gaming Detection

**Category:** Session Monitoring



### Purpose

A gaming probability score (0.0 to 1.0) computed by TCS/MIMIR from four behavioral signals: semantic similarity of intention gate entries to prior sessions, time-on-provenance before verification action, batch dismissal pattern, and episode commitment semantic drift. The score appears in the session manifest as "Verification engagement quality" and is visible to analysts and supervisors. High gaming scores reduce the weight TCS/MIMIR assigns to verification outcomes from that session in FGTS calibration updates.



### Mechanism Targets

- Metacognitive awareness: showing analysts their own gaming pattern score activates self-regulatory behavior (Bandura).
- Evaluation apprehension (Lerner and Tetlock): supervisory visibility changes behavior even without direct observation.
- Minimum engagement floors: structural nudges that make the fast path require at least nominal engagement without blocking the action outright. The verify button is grayed for five seconds after a provenance panel is first opened.


### Runtime Service Dependencies

- Session context store: holds engagement timing data and provenance panel open/close events per exchange.
- Behavioral telemetry: logs all engagement signals.
- Semantic similarity service: computes cosine similarity of intention gate entries and episode commitments against prior five sessions.
- TCS/MIMIR analytics function: computes gaming probability score from four behavioral signals.


### THEMIS Integration

TCS/MIMIR is the primary owner: gaming probability score computation is a TCS analytics function extension. TCS computes the score from four behavioral signal streams, logs it per session, incorporates it as a calibration weight modifier, and exposes it via the TCS calibration dashboard API. Provenance/MOIRAI logs the gaming score in the session manifest record. FGTS/ALETHEIA receives verification outcomes annotated with gaming-adjusted confidence weight.


**Active THEMIS services:** Provenance/MOIRAI (P) · TCS/MIMIR (B) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

Semantic similarity service (intention gate and episode commitment comparison). Behavioral telemetry (engagement signals).


**Data produced:** `{session_id, gaming_probability_score, signal_breakdown: {gate_similarity, panel_time_score, batch_dismissal_flag, commitment_drift}, computed_at}`

**Data consumed:** Prior session intention gate text and episode commitments from Provenance/MOIRAI. Behavioral telemetry engagement signals.



---


## 26. Pressure-Aware Session Mode

**Category:** Session Scaffolding



### Purpose

A session mode declaration added to Session Intent: Standard (default), Elevated (time-sensitive), or Deadline-Critical (filing, hearing, or client delivery within the session window). Deadline-Critical mode: intention gate compresses to a single combined field; reformulation gate suspended during session and converted to a post-session requirement; episode boundary fires every two exchanges; export gate hardened — any unverified HIGH-confidence claim blocks export outright and requires supervisor acknowledgment to proceed. A persistent "deadline mode" banner shows in the interface header.



### Mechanism Targets

- Stakes recalibration: consequence salience must remain active throughout the session, not only at session open.
- Friction calibration: compressing gates under time pressure is counterintuitive but research-supported — a compressed but mandatory single-field version produces lower-quality prior elicitation but higher compliance.
- Asymmetric hardening: easier in, harder out. The export gate in Deadline-Critical mode is the critical asymmetry — entry friction reduces while exit enforcement increases.


### Runtime Service Dependencies

- Session context store: stores declared pressure level and activates corresponding intervention profile.
- PGS/NOMOS policy engine: governs deadline-critical export gate hardening rules including supervisor acknowledgment requirement.
- ERAS/LOGOS: provides claim density score per response to determine the post-session reformulation target.


### THEMIS Integration

PGS/NOMOS governs the Deadline-Critical export gate hardening: unverified HIGH-confidence claims block export outright and require supervisor acknowledgment — this is a PGS policy expression, not an ATHENA-layer rule. Provenance/MOIRAI logs pressure mode as a session attribute enabling TCS/MIMIR to stratify calibration separately by pressure mode. FGTS/ALETHEIA receives verification outcomes annotated with pressure mode for stratified ground truth analysis. ERAS/LOGOS contributes claim density scoring for post-session reformulation target selection.


**Active THEMIS services:** PGS/NOMOS (P) · Provenance/MOIRAI (P) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · ERAS/LOGOS (C)



### Supporting Service Dependencies

Session context store (pressure profile activation). PGS policy engine (export gate rules). ERAS/LOGOS (claim density per response).


**Data produced:** `{session_id, pressure_level: standard|elevated|deadline_critical, deadline, deliverable_type, profile_activated_at, post_session_reformulation_required:bool}`

**Data consumed:** ERAS/LOGOS claim density scores. PGS deadline-critical policy rules.



---


## 27. Multi-Analyst Session Framework

**Category:** Session Scaffolding



### Purpose

Three workflow modes for multi-analyst legal work. Supervisory review mode: a senior attorney accesses a prior session in read-annotate mode, sees all AI-originated content with original confidence badges and verification states, can add annotations and override verification states (logged with full attribution), counter-position panel is automatically expanded for all responses. Matter handoff mode: the receiving analyst inherits a structured handoff brief — session intent, unverified claims with provenance, prior episode commitments and follow-through, verified vs disputed claim summary — and must explicitly acknowledge each section before beginning. Collaborative session mode: two analysts active simultaneously with per-analyst attribution for all queries and actions, shared session intent, independent synthesis workspaces, explicit surfacing of verification conflicts between analysts.



### Mechanism Targets

- Attribution accuracy: per-analyst attribution for all actions is the foundational requirement for multi-analyst provenance.
- Handoff acknowledgment: the receiving analyst must explicitly acknowledge each section of the handoff brief, activating prospective memory for the prior analyst's unresolved items.
- Collaborative disagreement surfacing: when two analysts in the same session take conflicting verification actions on the same claim, that conflict is explicit rather than silently overwritten.


### Runtime Service Dependencies

- Session context store: manages multi-analyst session state, per-analyst attribution, shared session intent, and collaborative disagreement events.
- DPS/CODEX: generates handoff briefs from prior session Provenance/MOIRAI records, stores supervisory review records, manages multi-analyst document provenance attribution.
- Provenance/MOIRAI: receives per-analyst attributed events for all three modes.
- PCES/AEGIS: validates analyst authorization for supervisory review access.


### THEMIS Integration

PCES/AEGIS governs supervisory review authorization — which analysts have matter-level supervisory access is a privilege concern. Provenance/MOIRAI receives new event types: per-analyst attributed turn records, supervisory review events with state overrides, and handoff brief generation records. The provenance graph schema must support multiple analyst nodes per session. TCS/MIMIR calibrates each analyst independently even within shared sessions. FGTS/ALETHEIA receives dual-analyst verification outcomes as high-value ground truth signals. Supervisory override of a verification state is the highest-quality ground truth signal in the system.


**Active THEMIS services:** PCES/AEGIS (C) · Provenance/MOIRAI (P) · TCS/MIMIR (C) · FGTS/ALETHEIA (P)



### Supporting Service Dependencies

DPS/CODEX (handoff brief generation, supervisory review record management). PCES/AEGIS (supervisory authorization check).


**Data produced:** `Supervisory review record and handoff brief with full per-analyst attribution.`

**Data consumed:** PCES supervisory authorization grants. Provenance/MOIRAI prior session records. TCS/MIMIR per-analyst calibration profiles.



---


## 28. Cross-Analyst Disagreement Detection

**Category:** Session Monitoring



### Purpose

When a new query is submitted, the semantic similarity service compares it against prior queries from other analysts on the same matter. If a similar prior query exists from a different analyst and the current response materially diverges from the prior response, a cross-analyst disagreement event is created. The matter coordinator sees the full event in a matter-level dashboard. Each analyst receives only a narrow notification ("a related query on this matter has produced a different characterization of [claim topic]") without seeing the other analyst's session, preventing anchoring.



### Mechanism Targets

- Attribution-blind notification: neither analyst is shown the other's full session. Showing one analyst the other's response would anchor their reasoning to the prior framing.
- Matter-level visibility: the supervising attorney sees the full cross-analyst divergence event.
- Calibration instability signal: cross-analyst divergence on a verified claim indicates the model is producing inconsistent outputs for the same claim type on the same matter.


### Runtime Service Dependencies

- Semantic similarity service: compares each new query against prior queries from other analysts on the same matter.
- Provenance/MOIRAI: provides prior analyst queries and claims for same-matter cross-session comparison.
- Session context store: tracks matter-level analyst query fingerprints for real-time comparison on each new submission.


### THEMIS Integration

Provenance/MOIRAI provides the cross-session query capability — prior analyst queries and responses on the same matter must be retrievable at submission time, requiring a matter-level index on MOIRAI records. TCS/MIMIR receives cross-analyst divergence events as matter-level calibration instability signals. FGTS/ALETHEIA receives cross-analyst verification conflicts as high-value ground truth signals. TVS/KAIROS provides validity context when divergence may be explained by source supersession.


**Active THEMIS services:** Provenance/MOIRAI (C) · TCS/MIMIR (P) · FGTS/ALETHEIA (P) · TVS/KAIROS (C)



### Supporting Service Dependencies

Semantic similarity service (cross-analyst query comparison). Provenance/MOIRAI matter-level cross-session query index.


**Data produced:** `{matter_id, analyst_a_session_id, analyst_b_session_id, query_similarity_score, diverging_claims[], divergence_type, created_at}`

**Data consumed:** Provenance/MOIRAI matter-level query and claim records across analysts. TVS/KAIROS source validity state at time of each session.



---


# THEMIS Integration Matrix

C = Consumer (ATHENA intervention reads from this THEMIS service). P = Producer (writes to). B = Both. Dash = No direct integration.


| Intervention | PCES/<br>AEGIS | PGS/<br>NOMOS | Provenance/<br>MOIRAI | TCS/<br>MIMIR | FGTS/<br>ALETHEIA | TVS/<br>KAIROS | RQS/<br>HERMES | KCS/<br>ARGUS | ERAS/<br>LOGOS |
|---|---|---|---|---|---|---|---|---|---|
| 1. Session Intent Declaration | P | P | - | P | - | - | - | - | - |
| 2. Chat Mode / Knowledge Source Configuration | C | C | P | C | - | - | C | - | - |
| 3. System Prompt Editor | - | B | P | - | - | - | - | - | - |
| 4. Model Parameter Configuration | - | - | P | C | - | - | - | - | - |
| 5. Prompt Repository | - | C | - | P | - | - | - | - | - |
| 6. Session History | - | - | C | C | C | - | - | - | - |
| 7. Input Type Classification | - | C | P | P | - | - | - | - | C |
| 8. Intention Gate | - | - | - | P | P | - | - | - | - |
| 9. Claim Confidence Badges | - | - | P | C | C | - | C | - | C |
| 10. Source Type Badges | - | - | P | P | P | C | C | - | - |
| 11. Counter-Position Panel | - | C | - | P | - | - | - | - | P |
| 12. Agreement Rate Tracker | - | - | - | P | P | - | - | - | - |
| 13. Verification Queue | C | - | P | P | P | - | C | - | - |
| 14. Context-Sensitive Verification Actions | - | - | P | P | P | C | - | - | - |
| 15. Reformulation Gate | - | - | - | P | P | - | - | - | C |
| 16. Episode Boundary | - | - | - | P | P | - | - | - | - |
| 17. Per-Message Feedback | - | - | - | P | P | - | - | - | - |
| 18. Per-Message Annotation | - | - | P | P | P | - | - | - | C |
| 19. Conversation Branching | - | - | P | - | P | - | - | - | - |
| 20. Context Window Inspector | - | - | C | - | - | - | C | - | C |
| 21. Disagreement Detection | - | - | C | P | P | C | - | - | - |
| 22. Session Synthesis Workspace | - | - | P | P | - | - | - | - | C |
| 23. Export with Provenance Gate | C | C | P | C | C | - | - | - | C |
| 24. Session Manifest | C | C | B | C | - | - | - | - | - |
| 25. Compliance Gaming Detection | - | - | P | B | P | - | - | - | - |
| 26. Pressure-Aware Session Mode | - | P | P | P | P | - | - | - | C |
| 27. Multi-Analyst Session Framework | C | - | P | C | P | - | - | - | - |
| 28. Cross-Analyst Disagreement Detection | - | - | C | P | P | C | - | - | - |


---


# Primary Data Flows


## The FGTS Calibration Loop

TCS/MIMIR serves calibrated confidence scores to Claim Confidence Badges. The analyst reviews claims with provenance expanded, and takes a typed verification action. That action flows to FGTS/ALETHEIA as a correction event. FGTS routes it to TCS to update confidence models and to RQS to flag retrieval misses. Updated TCS scores are served on the next request. This loop only closes if analysts take explicit verification actions.


## The TVS Currency Chain

KCS/ARGUS monitors external sources and sends active invalidation events to TVS/KAIROS. TVS propagates validity score changes through the Provenance/MOIRAI DAG to all claims whose ancestry includes the invalidated source — now including the FGTS ground truth corpus via TVS scope extension. Source Type Badges for Grounded claims surface the current TVS validity score.


## The ERAS Reasoning Audit Chain

ERAS/LOGOS captures reasoning at the turn level from every governed interaction. Claim Confidence Badges surface ERAS confidence signals for individual claims. Context Window Inspector displays the ERAS reasoning capture for the exchange. The Export Gate includes the ERAS reasoning record in every exported artifact, enabling professional responsibility demonstration that AI reasoning was complete, supported, and reviewed.


---

*End of Document — ATHENA Intervention Catalog v3.2*
