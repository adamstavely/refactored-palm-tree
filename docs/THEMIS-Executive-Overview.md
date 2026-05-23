# THEMIS — Executive Overview
## Trusted Human-AI Enablement for Mission Intelligence and Safety
### Updated Platform Overview · v3.0 · 42 Services · Full Implementation Roadmap

---

## The Strategic Premise

Every intelligence community organisation deploying AI-assisted analysis faces the same ungoverned condition: analysts interact with AI that produces plausible, confident outputs without any mechanism to verify sources, calibrate confidence, surface uncertainty types, or produce auditable records of the analytical process. Oversight bodies cannot review what they cannot see. Decision-makers cannot calibrate confidence in assessments they cannot trace.

This is not an AI quality problem. The AI is producing what it is designed to produce. This is a governance gap: the absence of infrastructure that makes AI assistance in intelligence analysis accountable, auditable, and safe to rely on.

**THEMIS closes the governance gap.**

It does not restrict what analysts can do with AI. It makes what analysts do accountable — to the analyst's own calibration, to the oversight body's requirements, and to the decision-maker's need to know how much to trust what they are reading.

---

## What THEMIS Is

THEMIS is a governance, provenance, and accountability platform deployed between the analyst and the AI. Its ATHENA interface is what the analyst sees and uses. Its 42 backend services govern every aspect of that interaction.

THEMIS answers three questions that currently have no systematic answer in AI-assisted intelligence analysis:

**Where did this AI output come from?** (The Origin axis)
Every AI response is tagged with source type indicators — GRND (retrieved from accessible sources), PARAM (from model training, no specific retrieved source), or SYNTH (analytical synthesis). Every cited source is verified against the accessible corpus. Every reasoning chain is captured and made queryable. Every document produced with AI assistance carries an auditable provenance record.

**Is the intelligence still valid?** (The Currency axis)
Sources are continuously monitored for validity. When a source expires or is superseded, the update propagates to all assessments that relied on it. Collection gaps are characterised precisely enough to become collection requirements. The model's own knowledge currency — how stale its parametric training is in a specific domain — is tracked and surfaced.

**Is analyst reliance calibrated?** (The Trust axis)
Per-analyst, per-domain, per-claim-type calibration posteriors track whether each analyst's reliance on AI is consistent with the AI's actual accuracy. Gaming of the verification system is detected. Uncertainty is characterised not as a single number but as a structured profile: is this uncertainty aleatory (inherent — the adversary may not have decided), epistemic (a collection gap that collection can fill), or model (this claim type is at the boundary of AI reliability)?

These three axes compound over time. The calibration system improves as more analysts verify more claims. The provenance system accumulates analytical history that the intelligence layer draws on. The collection gap system feeds requirements that return new intelligence to the corpus. Every session makes the next session more reliable.

---

## The 42-Service Architecture

The THEMIS platform comprises 42 approved services deployed across three implementation horizons.

### Horizon 1 — Governance Foundation (Months 1–17, 23 platform services)

The platform governance layer. Deploys in five build phases. Every analytical session passes through these services before, during, and after interaction with the AI.

**Safety Gates** guard session access and policy compliance. PCES/AEGIS enforces clearance and compartment controls — no session proceeds without a verified privilege grant. PGS/NOMOS enforces analytic standards compliance, screens for adversarial inputs, and classifies every interaction. MGS/TERMINUS is the controlled boundary for all external MCP tool access — no AI agent reaches an external tool without passing through the gateway.

**Core Infrastructure** is the backbone. MOIRAI is the cryptographic provenance chain: every analytically significant event in every session is signed, hashed, and chained in a tamper-evident record that can prove — not assert — what happened and when. TCS/MIMIR maintains the calibration system: per-analyst, per-domain accuracy posteriors updated from verified corrections and real-world outcomes. DPS/CODEX tracks document lifecycle from AI-assisted session through dissemination.

**Quality Layer** is the analytical quality enforcement surface. Thirteen services verify sources, screen for adversarial manipulation, assess media authenticity, track source validity, monitor retrieval quality, detect model drift, audit reasoning chains, and accumulate the ground truth corpus that calibrates the system. Critically, FGS/PLUTUS implements the platform's tokenomics model — tracking inference consumption, attributing costs, and governing the organisation's AI inference economics. HADES is the restricted adversarial intelligence repository where all adversarial detections, fabrication events, and compromised reasoning records are preserved for the Research & Red Team's analysis and the platform's continuous security improvement.

**Agent-Native Infrastructure** makes agentic AI use safe. SCBS/SENTINEL-CAP enforces session capability envelopes — bounded spend, bounded scope, defined time-to-live. CBS/BROKER ensures AI agents never hold raw credentials — they hold time-limited, operation-scoped handles that are revoked immediately on session close. RSS/ROLLBACK guarantees that no write action in an agentic session is irreversible within a 72-hour window.

**Interaction Layer** governs the analytical methodology encoded in prompts and skills. PRS/PROMETHEUS provides a versioned, tested, approved prompt repository — every prompt used in ATHENA has a MOIRAI-attested version hash. SKS/DAEDALUS provides the skill registry with graduated sharing from personal to platform tier, quality-rated by real-world outcome data.

### Horizon 2 — Intelligence Layer (Year 2, 13 services)

The platform compounds in intelligence value. Five namespaces extend the governance foundation into active analytical intelligence.

**themis-knowledge** provides the semantic foundation. OGS/YGGDRASIL maintains the canonical entity model that resolves cross-INT naming inconsistencies. MOS/SAGA assembles memory context from prior analytical sessions so each session builds on what came before. SCRIBE detects analytically significant changes between document versions. MNEMOSYNE extracts and surfaces institutional knowledge — what the analytical community has learned from corrections, outcomes, and verified assessments — as a queryable knowledge graph.

**themis-research** provides active research intelligence. STOA orchestrates multi-step research with analyst-approved decomposition and documented methodology trails. MIRROR identifies prior requirements with structural similarities to the current work. PYTHIA monitors active sessions and proactively surfaces relevant signals — prior requirement outcomes, document updates, collection gaps — before the analyst knows to look for them.

**Intelligence Cycle Completion** (Addendum F) closes the loops the governance foundation leaves open. UCS/TYCHE characterises the nature of uncertainty rather than reducing it to a single score. OFS/NEMESIS captures real-world outcomes and feeds them back to calibration — making the system accountable to what actually happened. PCS/IRIS translates platform governance metadata into consumer packages for policymakers. TIS/DIKE routes collection gap signals into the existing requirements workflow.

### Horizon 3 — Warning Layer (Year 3–4, 6 services)

The platform gains strategic warning capability. Six services in the `themis-warning` namespace monitor for the signals that precede significant intelligence developments.

TRS/CHRONOS models trajectories and generates scenario spaces with observable precursor indicators. ADS/CASSANDRA maintains behavioural baselines for entities and detects significant deviations. CGS/ARGUS-LACUNA characterises collection gaps precisely enough to drive collection requirements. WSF/LACHESIS fuses individually weak signals into analytically significant patterns. CRF/JANUS identifies when separate analytical requirements are unknowingly addressing facets of the same intelligence problem. SWS/SENTINEL synthesises all warning signals into strategic warning advisories for senior analytical leadership.

---

## The Tokenomics Model

At the scale of an intelligence analytical community, AI inference is a significant and variable operational cost that requires active governance. FGS/PLUTUS implements the organisation's **tokenomics model** — the economic framework that governs how inference capacity is allocated, tracked, and attributed.

Every team receives a periodic token allocation based on their expected analytical workload. Every session turn is attributed to the appropriate team account in real time. When teams approach their allocation, ATHENA surfaces a utilisation indicator. Reserve access for high-priority work is available with supervisor approval and IOB notification.

The tokenomics model enables budget forecasting, workload monitoring, and operational planning. It also produces the organisation's first empirical data on the true cost of AI-assisted analysis at scale — data that is essential for sustaining the platform's funding model and demonstrating its return on investment.

---

## The Adversarial Dimension

Intelligence organisations are targeted. Adversaries aware that the IC uses AI-assisted analysis are designing operations specifically against AI analytical workflows — adversarial corpus poisoning, prompt injection in documents likely to be retrieved, deepfake media intended to fabricate evidence, and influence operations designed to anchor AI outputs toward predetermined conclusions.

THEMIS addresses this across multiple services: IAS/SCUDO screens inputs and retrieved content; MAS/EIDOLON assesses media authenticity; CVS/VERITAS verifies that sources exist and say what the AI claims; ADS/CASSANDRA detects behavioural anomalies that may indicate adversarially placed activity; LACHESIS fuses weak signals that individually fall below detection thresholds.

HADES aggregates every adversarial event across all services into a restricted-access repository for the Research & Red Team. The adversarial reasoning record — the AI's full chain-of-thought during a session where adversarial content reached the context window — is the highest-value analytical artefact for understanding how adversarial techniques work against this specific platform. HADES makes this record permanent, structured, and queryable.

---

## Proposed Extension — The Competence Axis

Two additional services are proposed pending ARB approval. Together they form a fourth accountability axis:

**Can the AI reliably perform this specific task?**

CPS/APORIA maintains empirically-derived capability zone profiles (Green/Amber/Red) for each combination of model version, domain, and claim type — derived from Research & Red Team evaluation benchmarks, not from the model's self-report. Red zone assessments carry hard confidence ceilings regardless of TCS calibration.

ODS/LETHE provides real-time out-of-distribution detection — identifying when specific queries or content items fall far enough outside the model's training distribution that its learned patterns may not apply, even within domains where the model has demonstrated general capability.

The ARB should consider both proposals together as the Competence Axis package following Gate-5 completion, when the Research & Red Team has sufficient operational data to design the evaluation methodology.

---

## Implementation Timeline

| Horizon | Services | Duration | Key milestone |
|---|---|---|---|
| H1 — Governance Foundation | 23 platform services | Months 1–17 | Gate-5: IOB formal H1 assessment |
| H2 — Intelligence Layer | 13 intelligence services | Year 2 | Gate-9: Warning governance confirmed |
| H3 — Warning Layer | 6 warning services | Year 3–4 | Full compound intelligence flywheel |
| Proposed | 2 competence axis services | Year 2–3+ | Pending ARB approval |

Nine control gates govern phase transitions. No phase begins without gate confirmation. Gate-5 (H1 complete) is the major transition point: the IOB conducts a formal platform assessment, all GC policy items for H1 are confirmed, and H2 is formally authorised.

---

## Investment Rationale

**The risk THEMIS mitigates is concrete.** An intelligence assessment that an AI fabricated a source for, that an analyst relied on without verification, that reached a policymaker as finished intelligence, that influenced a consequential decision — and that the organisation cannot investigate because no provenance record exists — is not a hypothetical risk. It is the current baseline condition in ungoverned AI assistance.

**The cost of not governing is compounding.** Every session that produces an unverifiable provenance record, every prompt that proliferates without governance, every calibration signal that goes uncaptured, every adversarial technique that goes unrecorded — these are costs that accumulate without THEMIS and are irretrievable once the sessions have passed.

**The intelligence layer value compounds.** The analytical community that deploys THEMIS in Month 1 and accumulates verified corrections, outcome data, and entity graph knowledge through Month 24 enters Year 3 with a substantially more capable analytical system than one that deploys THEMIS in Year 2. The data floor requirements for ORACLE, MNEMOSYNE, and SENTINEL cannot be shortened by budget; they can only be started earlier.

**Tokenomics makes the investment self-governing.** FGS/PLUTUS creates the economic model that makes AI inference costs visible, attributed, and governable. The organisation that cannot answer "how much did our AI assistance cost last quarter and what did we get for it" cannot sustain the investment. Tokenomics makes that answer available from session one.

---

## What Success Looks Like

**Year 1 (H1 complete):** Every analytical session is provenance-tracked. Every cited source has been verified or flagged. Every analyst's AI reliance is calibrated in the domains they work in. Adversarial patterns are accumulating in HADES. The Research & Red Team has updated IAS/SCUDO at least twice from HADES findings. The IOB can answer any question about any session from the MOIRAI chain. The tokenomics model is producing accurate budget forecasts.

**Year 2 (H2 complete):** Analysts begin sessions with meaningful memory context from prior work. STOA multi-step research produces methodology trails that satisfy oversight requirements. Collection gaps are flowing through TIS/DIKE to requirements officers. Consumer packages from PCS/IRIS are reaching policymakers with actionable confidence translations and falsification indicators. PYTHIA is surfacing relevant prior requirement outcomes to analysts before they commit to approaches that have been tried and evaluated.

**Year 3+ (H3 operational):** The warning layer is generating strategic warning advisories with multi-source corroboration. SENTINEL warnings are acknowledged within 4 hours. The analytical community knows, systematically, what collection gaps exist and what would fill them. The platform's institutional knowledge — accumulated from corrections, outcomes, and analytical patterns — makes the analytical community as a whole more accurate than any individual working without it.

This is the THEMIS proposition: AI assistance that compounds in reliability over time, that is accountable to oversight from session one, and that makes the analytical community stronger with every session it processes.

---

*THEMIS Executive Overview v3.0 · 42 approved services · 2 proposed · 9 control gates · 4-year implementation*
*AI Trust Cell — THEMIS Platform Team*
