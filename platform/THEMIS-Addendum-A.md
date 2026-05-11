# THEMIS
## Trusted Human-AI Enablement for Matter Intelligence and Safety

# Addendum A — Five New Services + Three Architectural Extensions

| Document type | Platform Architecture Addendum |
|---|---|
| Status | Draft — Internal Review |
| Extends | THEMIS Platform Architecture and Service Design · 9 services · April 25, 2026 |
| New services | CVS/VERITAS, MDS/KRONOS, IAS/SCUDO, MAS/EIDOLON, DPS/CODEX |
| Extensions | Provenance/MOIRAI (cryptographic attestation) · FGTS/ALETHEIA (corpus quality) · TVS/KAIROS (ground truth validity) |


---


# Addendum Overview

This addendum extends the THEMIS Platform Architecture in two ways: five new services addressing gaps in citation verification, model drift, adversarial input, media authenticity, and document provenance; and three architectural extensions to existing services addressing gaps identified through red team analysis.

The five new services are CVS/VERITAS (citation verification), MDS/KRONOS (model drift), IAS/SCUDO (adversarial input screening), MAS/EIDOLON (media authenticity), and DPS/CODEX (document provenance). The three architectural extensions are: cryptographic event signing applied to Provenance/MOIRAI and all event-writing services; correction confidence weighting and human review queue in FGTS/ALETHEIA with a corresponding calibration weight function in TCS/MIMIR; and TVS/KAIROS scope extension to cover the FGTS ground truth corpus.


| Code | Full Name | Phase | Primary Concern |
|---|---|---|---|
| CVS / VERITAS | Citation Verification Service | 3-4 | Proactive generation-time citation resolution. Eliminates the fabricated-citation failure mode at the source. |
| MDS / KRONOS | Model Drift Service | 3-4 | Vendor model version change detection, active matter flagging, calibration re-validation, and matter-level version pinning. |
| IAS / SCUDO | Input Adversarial Screening Service | 5-6 | Ingestion-point scanning for prompt injection and adversarial instruction patterns before retrieved chunks reach the context window. |
| MAS / EIDOLON | Media Authenticity Service | 5-6 | Authenticity risk scoring for video, audio, image, and scanned document evidence. Deepfake detection, manipulation analysis, and OCR confidence scoring. |
| DPS / CODEX | Document Provenance Service | 3-4 | Document-layer chain of custody for high-volume document generation. Buffers paragraph-level provenance events to MOIRAI. Integrates with the custom web authoring solution. |
| TVS/KAIROS (ext.) | Temporal Validity — scope extended | 7-8 | Ground truth corpus validity tracking incorporated into existing TVS service. |
| Provenance/MOIRAI (ext.) | Cryptographic attestation | 3-4 | Event signing via Vault, hash chaining, and RFC 3161 external timestamping. No new service. |
| FGTS/ALETHEIA (ext.) | Corpus quality weighting | 5-6 | Correction confidence weighting function, human review queue, corpus versioning. No new service. |


---


# Section 01 — CVS / VERITAS


> **CVS** — Citation Verification Service
> *Epithet: VERITAS — Latin for truth — the standard against which legal citation is measured*
> Phase: 3-4


## Purpose

CVS is the only THEMIS service that addresses AI failure at the generation layer rather than after the fact. Its single function is to resolve every legal citation produced by the model against authoritative legal databases before the response is displayed to the analyst. A citation that does not resolve to a real case, statute, or regulation is a fabrication — and in legal work, fabricated citations have produced bar discipline, sanctions, and client harm at peer firms.

Not Found is a hard interrupt. The response is held and the analyst is shown a blocking panel identifying the unresolvable citation before the response body renders. This is not a badge. It cannot be dismissed without explicit analyst acknowledgment.



## Design Principles

- Runs at generation time, after PGS/NOMOS output screening and before the response is displayed to the analyst.
- Pattern-matches legal citation formats: case citations, statute references, and regulation identifiers.
- Resolves each match against Westlaw and Lexis APIs within the privilege and matter scope established by PCES/AEGIS.
- Returns Verified, Unresolved, or Not Found per citation. Not Found is a hard interrupt.
- Verified and Unresolved citations surface as inline indicators adjacent to the citation text.


## Service Specification

| Attribute | Detail |
|---|---|
| Namespace | themis-quality |
| Trigger | Fires on every model response before rendering. Mode configurable per PGS interaction class: mandatory-blocking for evidence_analysis and legal_drafting; advisory for research classes. |
| Latency target | P95 < 2 seconds for responses containing up to 10 citations. Not Found cases are synchronous and blocking. |
| External dependencies | Westlaw API, Lexis API. Credentials managed through Vault. Only citation strings transmitted, not client matter content. |
| Storage | Citation resolution cache (Redis, TTL: 24h Verified, 4h Unresolved, 1h Not Found). |
| Fallback | If both APIs are unavailable, CVS surfaces an advisory rather than blocking. Interaction logged with verification_status: service_unavailable. |


## Integration

| Service | Relationship | Detail |
|---|---|---|
| PCES / AEGIS | Constraint | Matter scope constrains which citation databases CVS may query. |
| PGS / NOMOS | Consumer / Policy | Interaction class determines CVS mode: mandatory-blocking or advisory. |
| Provenance / MOIRAI | Producer | CVS publishes CitationVerificationEvent for every citation checked. |
| ERAS / LOGOS | Feeds into | Not Found and Unresolved citations routed to ERAS as unsupported claim signals. |
| FGTS / ALETHEIA | Feeds into | Not Found citations confirmed as fabrications logged to FGTS as high-confidence correction events. |
| TCS / MIMIR | Feeds into | CVS Not Found rate per interaction class is a calibration signal. |


---


# Section 02 — MDS / KRONOS


> **MDS** — Model Drift Service
> *Epithet: KRONOS — Greek god of time — who governs change across time*
> Phase: 3-4


## Purpose

Legal matters run for months. A matter opened in January and resolved in December will span multiple AI vendor model updates. TCS/MIMIR calibration baselines established at matter open may be invalid by matter close. MDS monitors vendor API model version identifiers continuously, fires an event when a version change is detected, identifies all active matters where the prior version was used, and notifies TCS/MIMIR that calibration baselines for those matters require re-evaluation. Where pinning is configured for a matter, MDS enforces version stability across sessions until the matter is closed or the pin is explicitly released.



## Design Principles

- Polls vendor API version endpoints on a configurable interval (default: every 4 hours).
- On detection of a version change, immediately fires a ModelVersionChangeEvent to Provenance/MOIRAI before any other action.
- Identifies all active matter sessions using the prior model version from Provenance/MOIRAI session records.
- Sends re-validation requests to TCS/MIMIR for affected matters. Re-validation does not reset calibration baselines — it flags them as potentially stale pending review.
- Supports matter-level model version pinning. A pinned matter will not route inference calls to an updated model version without explicit release.
- Version pin release requires AI Governance Committee acknowledgment for active matters.


## Service Specification

| Attribute | Detail |
|---|---|
| Namespace | themis-intelligence (alongside KCS/ARGUS) |
| Polling interval | Configurable per vendor. Default: every 4 hours. Version change detection latency target: < 8 hours. |
| Storage | Version history registry (PostgreSQL, immutable append-only): {vendor, model_id, version_string, detected_at, active_matters_affected[]}. |
| Governance surface | AI Governance Committee dashboard shows current model versions per vendor, pending re-validation matters, active version pins, and historical version change log. |


## Integration

| Service | Relationship | Detail |
|---|---|---|
| Provenance / MOIRAI | Depends on / Feeds into | MDS reads active session records. Writes ModelVersionChangeEvent as an immutable audit record. |
| TCS / MIMIR | Feeds into | MDS sends re-validation requests. TCS flags calibration baselines as potentially stale. |
| Inference Gateway | Feeds into | MDS enforces version pinning at the inference gateway. Pinned matters are intercepted before routing. |


---


# Section 03 — IAS / SCUDO


> **IAS** — Input Adversarial Screening Service
> *Epithet: SCUDO — Italian for shield — the defensive layer at the ingestion boundary*
> Phase: 5-6


## Purpose

When an analyst reviews an opposing party's filing, an exhibit, or a deposition transcript through ATHENA, that document's text enters the model's context window via the retrieval pipeline. If that text contains instruction-format strings, meta-prompt patterns, or known injection signatures — whether placed deliberately or appearing incidentally — those strings may manipulate the model's behavior in ways that PCES and PGS are not designed to detect. IAS operates at the retrieval ingestion point, before chunks reach RQS/HERMES and the context window.



## Design Principles

- Sits at the retrieval ingestion point, before artifacts enter the retrieval candidate pool.
- Scans for instruction-format text patterns: imperative verb constructions directed at AI systems, role-reassignment language, system prompt override attempts, and jailbreak signature strings.
- Produces a continuous risk score per chunk (0.0 to 1.0). Thresholds configurable by PGS/NOMOS policy: Advisory (0.3-0.6), Flagged (0.6-0.85), Quarantined (>0.85).
- Quarantined chunks are excluded from the retrieval candidate pool and cannot reach the context window until released by an authorized analyst.
- All injection events are logged to Provenance/MOIRAI as InjectionScreeningEvents. Source artifacts remain unchanged.
- Injection signature taxonomy is maintained by the platform engineering team with input from the Red Team Evaluator. Updates require AI Governance Committee approval.


## Service Specification

| Attribute | Detail |
|---|---|
| Namespace | themis-quality |
| Detection methods | Signature matching against known injection taxonomy. Embedding similarity against injection pattern corpus. Heuristic scoring on imperative-directive linguistic structures. All three run per chunk. |
| Risk scoring | Continuous 0.0 to 1.0 scale. Advisory (0.3-0.6), Flagged (0.6-0.85), Quarantined (>0.85). Practice-group-specific overrides available. |
| False positive handling | Flagged chunks are visible to the analyst with matched pattern description. Analysts can release flagged chunks with an override reason. Override events are logged and reported to AI Governance Committee. |


---


# Section 04 — MAS / EIDOLON


> **MAS** — Media Authenticity Service
> *Epithet: EIDOLON — Greek for phantom or apparition — the image that may deceive*
> Phase: 5-6


## Purpose

Legal work increasingly involves AI-assisted analysis of video depositions, audio recordings, image evidence, and scanned documents. No service in the original THEMIS architecture assesses the authenticity of these media artifacts. MAS sits at media ingestion and produces an authenticity risk signal per artifact before it enters the THEMIS evidence corpus.



## Failure Modes by Modality

| Modality | Primary Failure Modes | MAS Detection Approach |
|---|---|---|
| Video | Deepfake generation. Temporal manipulation (frame removal, splice). Metadata timestamp and GPS spoofing. AI transcription errors propagated as ground truth. | Ensemble classifier: facial inconsistency scoring, temporal artifact detection, compression pattern analysis. Metadata authenticity check. Transcription flagged as derivative artifact. |
| Audio | Voice cloning and synthesis. Speaker misidentification. Spliced recordings. | Voice synthesis classifier. Speaker verification against reference embedding. Acoustic continuity analysis. Spectral anomaly detection. |
| Images | AI-generated or manipulated images. EXIF metadata spoofing. Steganographic injection. Document image forgery. | GAN artifact detection. JPEG compression inconsistency analysis. EXIF metadata authenticity scoring. Steganographic scan. |
| Scanned documents | OCR errors on low-quality scans propagated as ground truth. Handwriting or degraded text misread by AI. | Per-region OCR confidence scoring. Low-confidence regions flagged inline — AI claims derived from flagged regions carry a lower confidence ceiling. |


## Key Design Principles

- Runs at media ingestion, before AI analysis of media content begins. Non-blocking for text processing.
- Transcripts of video and audio recordings are tagged as TRANSCRIPT derivative artifacts with their own ASR accuracy metadata, separate from the source recording.
- MAS does not modify source artifacts. All screening is non-destructive.
- Authenticity risk scores propagate into ATHENA Source Type Badges. A GRND claim sourced from a high-risk-authenticity video shows a combined signal: grounded but authenticity-flagged.
- MAS is an architectural sibling to IAS/SCUDO — both are ingestion-point screening services.


## Service Specification

| Attribute | Detail |
|---|---|
| Namespace | themis-quality |
| Detection model registry | Versioned ensemble of classifiers per media type. Registry is Git-managed. AI Governance Committee approval required for classifier threshold changes. |
| Risk scoring | Continuous 0.0 to 1.0 per artifact per modality. Thresholds configurable by PGS policy per media type and matter class. |
| Storage | Authenticity event log (PostgreSQL, append-only). Risk score cache (Redis, TTL 24h). Quarantine registry (Redis). Transcript accuracy registry (PostgreSQL). |


---


# Section 05 — DPS / CODEX


> **DPS** — Document Provenance Service
> *Epithet: CODEX — Latin for book — Justinian's Codex was the foundational compilation of Roman law*
> Phase: 3-4


## Purpose

The THEMIS provenance chain covers session-level and claim-level records through Provenance/MOIRAI. Without a dedicated document layer, that chain severs the moment AI-generated content moves from an ATHENA session into a document production workflow. For firms generating documents at high volume, routing every paragraph-level provenance event directly through Provenance/MOIRAI creates write contention that degrades the core provenance graph for all other services.

DPS is the document-layer service that solves both problems simultaneously. It integrates directly with the firm's custom web authoring solution. The authoring solution writes document events to DPS APIs and queries DPS for provenance state. DPS handles all THEMIS service interactions on behalf of the authoring solution.



## Document Lifecycle Events

| Event | Trigger | Detail |
|---|---|---|
| AIContentInsertedEvent | ATHENA-sourced content placed in a document | Full provenance payload associated with content at paragraph level. |
| ContentEditedEvent | Attorney modifies AI-originated content | Edit depth score (embedding distance from AI original to human revision) computed and stored. |
| DocumentOpenedEvent | Document opened in authoring solution | TVS/KAIROS currency check triggered for all sources referenced in document provenance map. |
| DocumentFinalizedEvent | Document filed or submitted | Complete provenance snapshot: all AI-originated paragraphs, claim-level provenance, verification states, TVS validity at finalization, TCS/MIMIR RAI snapshot. |


## Service Specification

| Attribute | Detail |
|---|---|
| Namespace | themis-core alongside Provenance/MOIRAI and TCS/MIMIR |
| Document registry | PostgreSQL — document records keyed on document ID with paragraph-level provenance maps. |
| Write buffer | Kafka — absorbs high-frequency ContentEditedEvents before bulk-flushing to Provenance/MOIRAI. |
| Provenance panel cache | Redis — real-time document provenance state per document ID for authoring solution provenance panel queries. P95 < 100ms. |
| Critical timing | DPS document-level event schemas must be designed in Phase 3-4 alongside Provenance/MOIRAI. Retrofitting is not viable — the first document events are cold start by definition. |


---


# Section 06 — TVS / KAIROS Extended Scope

**Extension type:** TVS scope change. No new service required.



## Rationale

FGTS/ALETHEIA accumulates a corpus of analyst corrections and ground truth signals that TCS/MIMIR uses to improve calibration over time. That corpus is itself subject to temporal decay. Legal ground truth changes when statutes are amended, courts overturn precedent, or regulatory positions shift. TVS/KAIROS already owns decay functions, active invalidation, and DAG propagation for active matter evidence. The concern for the FGTS corpus is structurally identical.



## What Changes in TVS

- TVS receives a new event subscription: FGTS corpus items register with TVS in the same way active matter chunks do.
- When a KCS/ARGUS invalidation event arrives for a source underlying one or more FGTS corpus items, TVS propagates a CorpusItemValidityDecayEvent to FGTS/ALETHEIA.
- FGTS marks affected corpus items as potentially stale and routes them to a human review queue.
- Point-in-time reconstruction extended to the corpus: TCS can query what the corpus validity state was at any historical moment.
- Corpus item decay profiles are configurable separately from evidence decay profiles.


---


# Section 07 — Provenance Chain Cryptographic Attestation

**Extension type:** Provenance/MOIRAI write-path extension + Vault key management. No new service.



## The Gap

Provenance/MOIRAI is append-only, which prevents modification of events after the fact. Append-only is necessary but not sufficient. Any authorized service can write valid-looking events to an append-only log. In a legal context where THEMIS provenance records may be submitted to courts or bar associations as evidence of governed AI use, a chain that any authorized party can write to is not a tamper-evident audit trail. It is a trusted log.



## Three Mechanisms


### Event Signing via Vault

Each THEMIS service has an asymmetric key pair managed by HashiCorp Vault (already in themis-infra). Services never hold private keys directly — they call Vault's signing API and receive a signature. The signature is attached to the event before MOIRAI submission. MOIRAI's write API validates the signature on receipt. Invalid or missing signatures cause rejection, not logging.

Key rotation does not invalidate prior events because the signing key ID is stored with each event and the public key registry retains all historical public keys after rotation.



### Hash Chaining

Every event includes a hash of the prior event in the chain. Any tampering with a prior event breaks the hash of every subsequent event. An attacker cannot insert, modify, or delete any event in the chain without the break propagating forward and becoming detectable.



### External Timestamping (RFC 3161)

Every 24 hours, MOIRAI submits a digest of the current chain state to an external RFC 3161 compliant timestamp authority and stores the response as an anchor event. An external party can verify that the chain existed in a specific state at a specific time without access to THEMIS systems. A compromised MOIRAI could forge new events appended after a TSA anchor but could not alter events prior to the anchor without breaking the hash chain.



## Event Schema Extension

| Field | Purpose |
|---|---|
| signature | Cryptographic signature over the canonical event payload. Produced by the originating service via Vault signing API. |
| signing_key_id | Identifies which key pair signed this event. Used to select the correct public key from the registry for verification. |
| prev_event_hash | SHA-256 of the prior event in the service's event stream. Creates the hash chain. MOIRAI validates on write. |
| tsa_token | RFC 3161 timestamp authority token. Present on TSA anchor events (generated every 24 hours). |


## Services Required to Sign

All nine original THEMIS services plus all five new services: CVS/VERITAS, MDS/KRONOS, IAS/SCUDO, MAS/EIDOLON, and DPS/CODEX. The inference gateway and claim extractor also sign their respective events. No event reaches MOIRAI unsigned.



## Verification API Extension

| Mode | Purpose |
|---|---|
| Single event verification | Verify one event's signature, payload integrity, and position in the hash chain. |
| Range verification | Verify a sequence of events by walking the hash chain between two event IDs. |
| Full chain audit | Verify every event in a session or matter scope. Produces a signed audit certificate suitable for court or bar association submission. |
| Key registry query | Returns the public key for any signing_key_id, including rotated keys no longer in active use. |

> **Phase constraint:** Cryptographic attestation must be designed and implemented in Phase 3-4 alongside the Provenance/MOIRAI core build. An unsigned event cannot be retroactively signed without breaking the hash chain. There is no viable retrofit path.



---


# Section 08 — FGTS Corpus Quality: Correction Confidence Weighting

**Extension type:** FGTS/ALETHEIA + TCS/MIMIR extensions. No new service.



## The Gap

The FGTS ground truth corpus accepts all analyst verification actions as ground truth. Gaming detection (Intervention 25) addresses analysts who satisfy the interface without genuine engagement. It does not address the deeper problem: a sincere, genuinely engaged analyst with a systematically incorrect view of a specific regulatory domain will produce high-engagement, non-gaming corrections that corrupt TCS/MIMIR calibration for that domain across the entire analyst population.



## Correction Confidence Weighting

The correction confidence weight for any verification action is a function of five factors. TCS/MIMIR computes this weight at the time each correction event arrives from FGTS and applies it to calibration model updates.


| Factor | Source | Effect on Weight |
|---|---|---|
| Analyst domain RAI | TCS/MIMIR | Domain-specific RAI, not overall RAI. An analyst with a strong overall score but no history in securities litigation is not a high-quality ground truth source for securities litigation claims. |
| Session gaming score | TCS/MIMIR (Intervention 25) | Gaming probability score is inverted as a weight multiplier. High gaming score reduces the weight of all corrections from that session. |
| Session pressure mode | Session context store (Intervention 26) | Deadline-Critical mode applies a weight multiplier below one. Time-pressured verification is inherently less reliable. |
| Supervisory confirmation | DPS/CODEX (Intervention 27) | A supervisory review that confirms or overrides a verification state applies a substantial weight multiplier. Supervisory override propagates retroactively to adjust the weight of the original correction. |
| Peer agreement | Cross-Analyst Disagreement Detection (Intervention 28) | A second analyst independently making the same verification action multiplies both corrections' weights. Disagreement sends both corrections to the human review queue. |


## Human Review Queue

Corrections whose computed weight falls below a minimum threshold trigger the human review queue rather than being incorporated directly. Three additional triggers: the correction contradicts a prior high-confidence correction on the same claim type and domain; the claim involved a HIGH confidence badge being marked as Incorrect; or the correcting analyst has fewer than the minimum domain-specific verification history required.

Queue reviewers see the claim, source documents, the analyst's correction, their domain calibration history, and any prior corrections on the same claim type. Reviewers accept, reject, or escalate. Accepted corrections enter the corpus at their computed weight. Rejected corrections are logged to Provenance/MOIRAI but not incorporated. Escalated corrections go to a senior practice group attorney.



## Corpus Versioning and Rollback

FGTS/ALETHEIA maintains versioned snapshots of the corpus at configurable intervals — weekly during active build phases, monthly once stable. If a batch of corrections from a miscalibrated analyst requires removal, the corpus can be rolled back to the pre-contamination snapshot and recomputed forward excluding those corrections. This is tractable because FGTS uses weighted accumulation rather than gradient descent.

Corpus version snapshots are signed events in Provenance/MOIRAI, bringing them within the cryptographic attestation chain defined in Section 07.



---


# Section 09 — Updated Service Dependency Matrix

New and modified rows. Original nine-service matrix unchanged for all existing service relationships.


| Service | Depends On | Feeds Into |
|---|---|---|
| CVS / VERITAS | PGS/NOMOS (interaction class), PCES/AEGIS (matter scope), Provenance/MOIRAI (turn record) | ERAS/LOGOS (unsupported claim signal), FGTS/ALETHEIA (fabrication detection), Provenance/MOIRAI (CitationVerificationEvents — signed) |
| MDS / KRONOS | Vendor API endpoints (version polling), Provenance/MOIRAI (active session records) | Provenance/MOIRAI (ModelVersionChangeEvents — signed), TCS/MIMIR (calibration re-validation trigger), AI Governance Committee |
| IAS / SCUDO | RQS/HERMES (pre-ingestion position), PGS/NOMOS (quarantine policy) | RQS/HERMES (sanitised chunks), PGS/NOMOS (injection event log), Provenance/MOIRAI (adversarial input events — signed) |
| MAS / EIDOLON | Provenance/MOIRAI (artifact records), PGS/NOMOS (disposition thresholds), TVS/KAIROS (invalidation trigger) | Provenance/MOIRAI (MediaAuthenticityEvents — signed), TCS/MIMIR (modality calibration), FGTS/ALETHEIA (authenticity ground truth), DPS/CODEX, ERAS/LOGOS |
| DPS / CODEX | PCES/AEGIS (matter scope), TVS/KAIROS (currency queries), Provenance/MOIRAI (document event target) | Provenance/MOIRAI (document lifecycle events — signed and batched), FGTS/ALETHEIA (edit depth signals), TCS/MIMIR (RAI snapshot at finalization) |
| TVS/KAIROS (extended) | Provenance/MOIRAI, KCS/ARGUS — unchanged | FGTS/ALETHEIA ground truth corpus (new), RQS/HERMES, KCS/ARGUS — existing feeds unchanged |
| Provenance/MOIRAI (crypto ext.) | Vault (signing key validation), all event-writing services, RFC 3161 TSA | All consumers of MOIRAI events (now signed and hash-chained). Verification API extended for external audit use. |
| FGTS/ALETHEIA (quality ext.) | TCS/MIMIR (correction weight computation), DPS/CODEX (supervisory override signals), Intervention 28 (peer agreement signals) | TCS/MIMIR (weighted corrections), Provenance/MOIRAI (corpus snapshot events — signed, review queue decisions) |


## Updated Namespace Assignments

| Namespace | Services | Note |
|---|---|---|
| themis-gates | PCES/AEGIS, PGS/NOMOS | Unchanged. |
| themis-core | Provenance/MOIRAI, TCS/MIMIR, DPS/CODEX (new) | DPS added. Must be built concurrently with MOIRAI in Phase 3-4. |
| themis-quality | FGTS/ALETHEIA, TVS/KAIROS, RQS/HERMES, CVS/VERITAS (new), IAS/SCUDO (new), MAS/EIDOLON (new) | CVS, IAS, and MAS added. |
| themis-intelligence | KCS/ARGUS, ERAS/LOGOS, MDS/KRONOS (new) | MDS added. All three are external monitoring or reasoning intelligence functions. |
| themis-infra | Kafka, Kong, Vault | Unchanged. MDS and CVS add new external API credential entries to Vault. Vault extended with per-service signing key pairs for cryptographic attestation. |


---


# Section 10 — Updated Implementation Roadmap

| Phase | Weeks | Services | Key Deliverables |
|---|---|---|---|
| 1-2 | 1-8 | PCES, PGS | Safety gates unchanged. No new services in this phase. |
| 3-4 | 9-28 | Provenance, TCS, CVS, MDS, DPS + Crypto | Cryptographic attestation: Vault key pairs per service, event schema extended, MOIRAI write-path validation live, RFC 3161 TSA integration, all services signing events. CVS/VERITAS citation verification live. MDS/KRONOS vendor polling and version pinning. DPS/CODEX document event schema designed with MOIRAI, Kafka buffer and Redis cache deployed. |
| 5-6 | 29-46 | FGTS, TVS, RQS, IAS, MAS + FGTS Quality | FGTS quality extensions: correction confidence weighting live, human review queue deployed, corpus versioning operational, supervisory override retroactive weight adjustment wired. IAS/SCUDO injection taxonomy v1.0 and pipeline integration. MAS/EIDOLON detection model ensemble v1.0 per modality live. |
| 7-8 | 47-66 | KCS, ERAS, TVS extension | KCS/ARGUS and ERAS/LOGOS deployed. TVS/KAIROS scope extended to FGTS ground truth corpus. CorpusItemValidityDecayEvent live. All 13 services operational. Full platform metrics dashboard live. |


## Revised Service Count

|  | Count | Services |
|---|---|---|
| Original architecture | 9 | PCES, PGS, Provenance, TCS, FGTS, TVS, RQS, KCS, ERAS |
| This addendum — new services | +5 | CVS/VERITAS, MDS/KRONOS, IAS/SCUDO, MAS/EIDOLON, DPS/CODEX |
| This addendum — extensions | +0 new services | TVS (ground truth corpus validity), Provenance/MOIRAI (cryptographic attestation), FGTS/ALETHEIA + TCS/MIMIR (corpus quality weighting) |
| Total | 13 | Five new services. Three architectural extensions to existing services. |


---

*End of Addendum A — Five new services · Three architectural extensions · THEMIS total: 13 services*
