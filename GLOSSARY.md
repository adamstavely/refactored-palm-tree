# THEMIS Glossary

Vocabulary reference for the THEMIS platform. Terms are organized alphabetically within categories. Where a term has a specific technical meaning that differs from common usage, that is called out explicitly.

---

## Platform & Services

**THEMIS** — Trusted Human-AI Enablement for Matter Intelligence and Safety. The overall platform name. In Greek mythology, Themis is the goddess of law, justice, and the right ordering of things.

**AEGIS** — The epithet for PCES (Privilege & Consent Enforcement Service). Greek for "shield."

**ALETHEIA** — The epithet for FGTS (Feedback & Ground Truth Service). Greek for "truth" or "disclosure."

**ARGUS** — The epithet for KCS (Knowledge Currency Service). In mythology, Argus was the all-seeing giant with a hundred eyes.

**ERAS** — Explainability & Reasoning Audit Service. The service that captures and indexes model reasoning at the claim level. Epithet: LOGOS.

**FGTS** — Feedback & Ground Truth Service. Captures analyst corrections to AI outputs and builds the firm's proprietary training corpus. Epithet: ALETHEIA.

**FGS** — Financial Governance Service. Tracks AI token costs at the matter and client level, enforces budgets, and detects anomalous spend. Epithet: PLUTUS.

**HADES** — Continuous Adversarial Evaluation Service. The service that deliberately probes THEMIS to verify governance guarantees hold under pressure.

**HERMES** — The epithet for RQS (Retrieval Quality Service). In mythology, Hermes was the messenger who finds and delivers.

**KAIROS** — The epithet for TVS (Temporal Validity Service). Greek for "the right moment" or "the appointed time."

**KCS** — Knowledge Currency Service. Monitors external legal sources (court dockets, regulatory feeds, legal databases) and generates active invalidation events when developments affect the internal corpus. Epithet: ARGUS.

**LOGOS** — The epithet for ERAS. Greek for "reason" or "word."

**MIMIR** — The epithet for TCS (Trust Calibration Service). In Norse mythology, Mimir is the god of wisdom whose well grants knowledge.

**MIRROR** — Similar Matter Intelligence. Intelligence layer service that identifies the most similar historical matters to any active matter across six dimensions.

**MNEMOSYNE** — Institutional Memory Service. Intelligence layer service that captures, structures, and makes queryable the firm's tacit institutional knowledge. In mythology, Mnemosyne is the goddess of memory and mother of the Muses.

**MOIRAI** — The AI Content Provenance Service. In mythology, the Moirai (Fates) tracked the thread of every life — origin, path, and end. Epithet for the Provenance Service.

**NOMOS** — The epithet for PGS (Policy & Guardrails Service). Greek for "law as convention."

**ORACLE** — Matter Outcome Intelligence. Intelligence layer service that surfaces predictive intelligence from the firm's historical matter outcomes.

**PCES** — Privilege & Consent Enforcement Service. Governs what content can enter an AI context window at the chunk level. Epithet: AEGIS.

**PGS** — Policy & Guardrails Service. Governs what kinds of AI interactions are permitted, classifies interaction types, detects PII, evaluates policy rules, and screens outputs. Epithet: NOMOS.

**PLUTUS** — The epithet for FGS. In mythology, Plutus is the god of wealth.

**PYTHIA** — Predictive Research Intelligence. Intelligence layer service that anticipates what a matter team needs before they ask. In mythology, the Pythia was the Oracle at Delphi.

**RQS** — Retrieval Quality Service. Measures whether the right content is being retrieved in the RAG layer. Epithet: HERMES.

**SCRIBE** — Document Version & Semantic Diff Intelligence. Intelligence layer service that tracks substantive changes across document versions with AI attribution.

**STOA** — Legal Research Orchestration. Intelligence layer service that structures multi-step legal research as documented workflows. In antiquity, the Stoa was a covered walkway where philosophers gathered to reason together.

**TCS** — Trust Calibration Service. Measures whether analysts are relying on AI appropriately relative to its actual performance. Epithet: MIMIR.

**THEMIS Ethics** — (planned) Dedicated service for tracking compliance with bar association AI disclosure requirements across all practice jurisdictions.

**TVS** — Temporal Validity Service. Re-evaluates evidence currency over time using decay functions and active invalidation events. Epithet: KAIROS.

---

## Architecture & Infrastructure

**ADR** — Architecture Decision Record. A short document capturing a significant design decision: what was decided, what alternatives were considered, and why. See [adr/](adr/).

**Append-only event log** — The Provenance Service's source of truth. Events are never modified; corrections produce new events. Enables full replay and reconstruction of the provenance graph from first principles.

**Content-addressed identity** — A method of identifying content by its cryptographic hash (SHA-256) rather than by name or location. Two chunks with the same hash are the same content, regardless of where they appear.

**DAG** — Directed Acyclic Graph. The data structure the Provenance Service uses to represent content lineage. Directed means edges have direction (A → B means A influenced B). Acyclic means no cycles — content cannot be its own ancestor.

**Data plane** — In the regional topology, the cluster where matter data actually lives. Distinct from the control plane (which handles routing and policy but contains no client data).

**Kubernetes namespace** — An isolation boundary within a Kubernetes cluster. THEMIS services are organized into namespaces (`themis-gates`, `themis-core`, etc.) to control network access and resource allocation between service groups.

**LSH** — Locality-Sensitive Hash. A fuzzy fingerprinting technique (MinHash or SimHash) that enables similarity matching — two chunks that are slightly different will have similar LSH values, making it possible to identify near-duplicate or lightly-edited content.

**Neo4j** — The graph database used by the Provenance Service (MOIRAI) to store the content lineage graph. Chosen for its native support for relationship traversal and DAG queries.

**Sidecar buffer** — A companion container deployed alongside the API gateway that accepts provenance events and forwards them to the Provenance Service asynchronously. Provides local disk buffering if the Provenance Service is unavailable.

**SEE** — Synthetic Evaluation Environment. The isolated Kubernetes namespace where HADES runs adversarial probes. Contains no real client data; mirrors production service topology.

**ZDR** — Zero Data Retention. An agreement with a model provider that API call content is not retained or used for training. Required for all providers that process client matter content.

---

## Provenance & Lineage

**AnalystInput** — A provenance graph node capturing the raw, unaugmented user query before any system transformation — exactly what the analyst typed, including interface state (open document tabs, selected text, foregrounded document).

**Certification Hash** — A cryptographic hash of a finalized document's content combined with a snapshot of its provenance graph state. If either the document or its provenance record is modified post-certification, the hash breaks.

**Chain of custody** — The complete, unbroken record of where content came from and every transformation it underwent from source evidence to final work product. The Provenance Service is a content chain of custody system.

**Chunk** — The atomic unit of content in THEMIS. A discrete segment of a source document, video, or communication, identified by its SHA-256 content hash and LSH fingerprint, with spatial metadata capturing its location within its source.

**Compound provenance** — When a document paragraph draws from multiple sources — part source material, part AI synthesis, part original writing. The provenance graph represents this as a DAG where the paragraph node has multiple parent nodes.

**CITED_IN edge** — A provenance graph edge written when content is pasted or cited into a document section. Created by the document editor's paste-time provenance capture.

**Genealogical distance** — The number of transformation hops between a piece of content and its primary source material. Direct source material has distance 0. An AI summary of source material has distance 1. An AI summary of that summary has distance 2.

**Intra-session DAG** — The provenance graph structure capturing which prior turn outputs were in the context window when subsequent turns were generated. Turn 5's output is conditioned on Turns 1–4 — the intra-session DAG makes this explicit.

**Nullification** — The retention-compliant alternative to node deletion in the provenance graph. Content fields (raw text, document content) are replaced with a nullification marker while structural fields (node IDs, edges, timestamps) are retained. Preserves graph integrity while satisfying deletion obligations.

**PromptAssembly** — A provenance graph node capturing the exact payload sent to the model: the assembled system prompt, the augmented user prompt (with RAG chunks injected), and all metadata about how the assembly was constructed. The definitive record of what the model actually received.

**PromptTemplate** — A provenance graph node storing the versioned system instruction set. Governance Committee-approved; version-controlled alongside PGS rules.

**Provenance Summary Report** — A human-readable breakdown generated on document save showing the composition of a work product: percentage that is original attorney writing, direct source material, AI-generated, AI-derived, and unknown origin.

**Spatial metadata** — The location of a chunk within its source document. For text documents: page number, bounding box, reading-order sequence. For video: timecode range, speaker ID. For email/chat: thread ID, message position.

**Turn record** — The provenance record for a single AI model interaction. Captures which prior turns were in the context window, which retrieval chunks were used, what prompt assembly was sent, and what output was produced.

**Weak ancestry** — A flag on provenance graph edges where the context window was truncated during generation. The model may not have actually seen the nominally-present ancestor turn. Provenance paths through weak_ancestry edges carry reduced confidence.

---

## Validity & Currency

**Active invalidation** — A discrete event where new evidence explicitly contradicts or supersedes an older one. Distinct from passive decay. Has a timestamp, an actor, and creates a hard relationship in the validity graph. Sources: conflict resolution, KCS invalidation events.

**Composite validity score** — The weighted combination of decay_score, contested flag, and (for AI outputs) compound AI staleness. Formula: if invalidated → 0.0; if contested → decay_score × 0.5; else → decay_score. AI outputs apply additional model_cutoff and retrieval_freshness penalties.

**Compound AI staleness** — Three independent temporal bounds that multiply for AI-generated content: (1) model training cutoff age, (2) retrieval context freshness at generation time (frozen), (3) output age. Each bound is tracked and displayed separately.

**ConflictEvent** — A TVS record representing a detected contradiction between two chunks. States: open → resolved (with documented attorney decision) | dismissed (false positive).

**Decay profile** — The content-type-specific function governing how a chunk's validity score decreases over time. Types include: step function (statutes, case law), exponential (expert reports), linear (financial statements), aggressive exponential (news/media), model vintage (AI outputs).

**Knowledge decay** — TVS-style validity scoring applied by MNEMOSYNE to institutional knowledge nodes. Knowledge has a half-life — a judge's 2018 ruling pattern may not reflect current behavior. Fresh signals carry full weight; aging signals decay exponentially.

**Passive decay** — The continuous reduction in a source's validity score purely as a function of elapsed time and domain volatility. No triggering event required. Distinguished from active invalidation, which is event-driven.

**Point-in-time reconstruction** — The ability to reconstruct the validity state of any chunk or document at any historical moment, using the append-only score_history log. Critical for litigation: demonstrating what was known and how reliable it was at the time of a filing.

**Retrieval freshness** — The minimum TVS validity score across all RAG chunks at the moment of AI output generation. Frozen at generation time — it records what was true when the synthesis occurred, not what is true now.

**Score band** — A color-coded validity indicator displayed in the document editor margin: green (> 0.85), blue (0.60–0.85), amber (0.40–0.60), red (< 0.40), purple (contested).

---

## Trust & Calibration

**Calibration error** — The distance between an analyst's reliance behavior and the AI system's actual accuracy for that interaction class. Computed as: `1 - |RAI - Elo_normalized|`. A score of 1.0 is perfect calibration; 0.0 is maximally miscalibrated.

**Elo rating** — An AI system performance rating adapted from chess. Updated after each interaction based on whether the analyst corrected the output (loss) or accepted it without subsequent revision (win). Tracked per interaction class × matter type.

**Over-reliance** — An analyst accepting AI outputs at a rate significantly higher than the AI's actual accuracy for that interaction class. Flagged when RAI score > 0.80.

**RAI** — Reliance-Accuracy Index. A normalized measure of analyst reliance relative to AI accuracy. RAI = 0.0 means the analyst never relies on AI; RAI = 1.0 means the analyst always accepts AI output; RAI = 0.5 is balanced.

**Under-reliance** — An analyst correcting AI outputs at a rate significantly higher than the error rate warrants. Flagged when RAI score < 0.20. May indicate systematic distrust, skill gap in evaluating AI output, or domain-specific AI failure that warrants investigation.

---

## Privilege & Governance

**Attorney-client privilege** — A legal protection for confidential communications between an attorney and client made for the purpose of legal advice. PCES enforces this at the chunk level — privileged content cannot enter an AI context window without explicit authorization.

**CoI** — Conflict of Interest. A situation where the firm's representation of one client may be adverse to another. PCES maintains a Conflict Graph and blocks retrieval of content that could expose adverse party information.

**Hard stop** — A THEMIS enforcement action that cannot be overridden by any individual approval. Examples: active adverse-party CoI on a current client, sealed matter boundary violation. Requires formal out-of-band process involving ethics counsel.

**HITL** — Human-in-the-Loop. The general category of workflows where a human must review and approve an AI interaction before it proceeds. In THEMIS, specifically refers to the Hold Queue workflow managed by PGS.

**Hold Queue** — The set of AI interactions suspended pending attorney or senior analyst review. Each hold has an assigned reviewer, an SLA deadline, and a resolution path (approve, modify, or reject). Managed by PGS / NOMOS.

**Matter bleed** — The failure mode where RAG retrieval surfaces content from Matter A in a prompt scoped to Matter B. PCES prevents this through matter-scoped retrieval enforcement.

**Privilege propagation** — The mechanism by which attorney-client privilege flows through the provenance DAG. If a chunk carries privilege, any AI output derived from it inherits that designation.

**Supervised gate** — A THEMIS enforcement action (REQUIRE_APPROVAL) where AI generation is held pending human review, but the reviewer can approve, modify, or reject the interaction. Distinguished from a hard stop, which has no approval path.

**Work product doctrine** — A legal protection for materials prepared by or for an attorney in anticipation of litigation. Distinct from attorney-client privilege; PCES enforces both as separate privilege types.

---

## Financial Governance

**Anomaly detection** — FGS capability that identifies statistically unusual AI spend patterns: sessions spending 10× the matter average, single queries exceeding weekly matter totals, retrieval loops that appear to iterate without converging.

**Budget envelope** — A matter-level AI spending limit with a soft ceiling (alert at a configurable percentage) and a hard ceiling (blocks further AI interactions until a supervising attorney authorizes an increase).

**Rate card** — A versioned mapping of model × token type → cost per token. Stored as versioned configuration; historical records retain the rate that applied at time of generation. Never retroactively repriced.

**ZDR agreement** — See Zero Data Retention under Architecture & Infrastructure.

---

## Retrieval & RAG

**Embedding drift** — The phenomenon where an embedding model's representation of content shifts over time relative to the original corpus indexing, degrading retrieval quality. RQS monitors for this by tracking similarity scores of known-good query/document pairs.

**Q-RAG** — A multi-step retrieval architecture (Kirchenbauer et al., ICLR 2026) that trains only the retrieval embedder using reinforcement learning while keeping the LLM frozen. Particularly suited to legal research where questions decompose into multiple sub-questions requiring sequential retrieval steps.

**RAG** — Retrieval-Augmented Generation. The pattern of providing a language model with retrieved relevant documents as context rather than relying solely on parametric (trained) knowledge. The foundation of most legal AI research and evidence analysis workflows.

**Retrieval miss** — A failure mode where relevant content exists in the corpus but was not surfaced by the retrieval system. Distinguished from hallucination (where the model fabricates content) — a retrieval miss produces a correct-sounding but incomplete answer. Systematically harder to detect than hallucination.

**Retrieval poisoning** — An adversarial technique where malicious content is injected into the retrieval corpus to manipulate what the model receives as context. Tested by HADES Surface 2 probes.

**Retrieval trajectory** — In Q-RAG, the sequence of embedding steps that led to the final retrieved document set. Each step is informed by the previous retrieval results. Captured in the RetrievalTrajectory provenance node.

**Terminal chunks** — In Q-RAG multi-step retrieval, the final set of documents passed to the model's context window after all retrieval steps complete.

---

## Adversarial Evaluation

**Adversarial library** — The versioned, governance-approved collection of probe scenarios organized by surface, technique, and target service. Grows continuously from threat intelligence, FGTS correction patterns, red team exercises, and published research.

**Failure Catalog** — The append-only institutional record of every adversarial finding HADES has produced: what failed, under what conditions, at what severity, and how it was remediated. Compounds in value over the platform's lifetime.

**Probe** — A single adversarial test scenario. Has a fixed input, expected system behavior, and failure indicator. Versioned and governance-approved before activation. Co-authored with the PGS rule it is designed to test.

**Surface** — One of five categories of adversarial testing: (1) Safety Gate Probing, (2) Retrieval Poisoning Detection, (3) Provenance Integrity Testing, (4) Model Consistency & Drift Testing, (5) Calibration Boundary Stress Testing.

---

## Intelligence Layer

**Complication pattern** — In MIRROR, a category of unexpected difficulty that arose in a similar historical matter at a specific phase (e.g., expert witness challenge, privilege dispute, discovery sanction). Surfaced proactively to the active matter team.

**Knowledge graph** — In MNEMOSYNE, the structured network of institutional knowledge nodes (strategies, opposing party behaviors, judge tendencies, expert performance, AI prompting approaches) and the relationships between them.

**Lifecycle stage** — In PYTHIA, the current phase of a matter's progression: pre-litigation, pleadings, discovery, motion practice, trial preparation, post-trial. Used to model what research and evidence the matter team will likely need next.

**Matter intelligence brief** — A PYTHIA output: a daily or weekly synthesis per active matter covering external developments (KCS), evidence validity status (TVS), outcome trajectory (ORACLE), and proactive research recommendations (STOA).

**Semantic diff** — In SCRIBE, a comparison of document versions that classifies changes by legal meaning impact (substantive, structural, or stylistic) rather than character-level differences. Requires legal domain fine-tuning to distinguish meaning-changing edits from style changes.

**Tacit knowledge** — Institutional knowledge that exists in experienced attorneys' minds but is not written down: which strategies work against which opposing counsel, which arguments play well before which judges, which AI prompting approaches produce reliable analysis for specific matter types. MNEMOSYNE is designed to capture and structure this.
