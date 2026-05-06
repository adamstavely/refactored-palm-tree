# THEMIS

**Trusted Human-AI Enablement for Matter Intelligence and Safety**

THEMIS is the governance, accountability, and quality infrastructure for AI-assisted legal work. It is not an AI product — it is the platform that makes AI use safe, traceable, defensible, and over time, genuinely intelligent about the firm's own practice.

---

## What THEMIS Does

| Layer | Services | What It Ensures |
|---|---|---|
| **Safety Gates** | PCES / AEGIS · PGS / NOMOS | Every AI interaction on client data is privilege-enforced and policy-governed |
| **Core Infrastructure** | MOIRAI · MIMIR | Every AI output has a complete chain of custody; analyst reliance is measured |
| **Quality Loops** | PLUTUS · ALETHEIA · KAIROS | AI spend is governed; corrections improve the platform; evidence stays current |
| **Operational Intelligence** | HERMES · ARGUS | Retrieval is observable; the external legal world is monitored |
| **Strategic Capability** | LOGOS · HADES | AI reasoning is auditable; governance guarantees are continuously tested |
| **Intelligence Layer** | SCRIBE · STOA · ORACLE · MIRROR · MNEMOSYNE · PYTHIA | The platform learns from the firm's own matters and anticipates what comes next |

---

## Start Here — By Audience

### I'm an engineer joining the platform team
1. [Platform Architecture](platform/THEMIS_Platform_Architecture.md) — the full system design
2. [Data Models & Schemas](schemas/DATA_MODELS.md) — every schema in the platform
3. [Glossary](GLOSSARY.md) — vocabulary reference before reading anything else
4. The service you're working on in [services/](services/)
5. [ADR Index](adr/README.md) — decisions already made and why

### I'm an attorney or analyst using THEMIS-governed tools
1. [What THEMIS Means for Your Work](onboarding/attorneys.md) *(coming soon)*
2. [Glossary](GLOSSARY.md) — plain-language definitions
3. Understanding [score bands and validity warnings](services/TVS_Architecture.md#document-editor-integration)
4. Understanding the [Hold Queue](services/PGS_Architecture.md#hitl-workflow-orchestration)

### I'm on the AI Governance Committee
1. [Platform Architecture — Executive Summary](platform/THEMIS_Platform_Architecture.md#00--executive-summary)
2. [Business Case](platform/THEMIS_Platform_Architecture.md#01--business-case)
3. [Governance Model](platform/THEMIS_Platform_Architecture.md#08--governance-model)
4. [HADES Failure Catalog](services/HADES_Architecture.md#the-failure-catalog) — what gets reviewed quarterly
5. [CHANGELOG](CHANGELOG.md) — what has changed in the platform

### I'm evaluating the intelligence layer (Year 2+)
1. [Intelligence Layer Proposal](platform/THEMIS_Intelligence_Layer_Proposal.md)
2. Individual service docs: [SCRIBE](services/SCRIBE_Architecture.md) · [STOA](services/STOA_Architecture.md) · [ORACLE](services/ORACLE_Architecture.md) · [MIRROR](services/MIRROR_Architecture.md) · [MNEMOSYNE](services/MNEMOSYNE_Architecture.md) · [PYTHIA](services/PYTHIA_Architecture.md)

---

## Repository Structure

```
/
├── README.md                          ← you are here
├── GLOSSARY.md                        ← vocabulary reference
├── CHANGELOG.md                       ← platform change log
│
├── platform/                          ← platform-level documents
│   ├── THEMIS_Platform_Architecture.md
│   ├── THEMIS_Intelligence_Layer_Proposal.md
│   └── THEMIS_Roadmap.md
│
├── services/                          ← per-service architecture docs
│   ├── PCES_Architecture.md           (AEGIS — Safety Gate 1)
│   ├── PGS_Architecture.md            (NOMOS — Safety Gate 2)
│   ├── MOIRAI_Architecture.md         (Provenance Service)
│   ├── TCS_Architecture.md            (MIMIR — Trust Calibration)
│   ├── FGS_Architecture.md            (PLUTUS — Financial Governance)
│   ├── FGTS_Architecture.md           (ALETHEIA — Feedback & Ground Truth)
│   ├── TVS_Architecture.md            (KAIROS — Temporal Validity)
│   ├── RQS_Architecture.md            (HERMES — Retrieval Quality)
│   ├── KCS_Architecture.md            (ARGUS — Knowledge Currency)
│   ├── ERAS_Architecture.md           (LOGOS — Reasoning Audit)
│   ├── HADES_Architecture.md          (Adversarial Evaluation)
│   ├── SCRIBE_Architecture.md         (Intelligence Layer)
│   ├── STOA_Architecture.md           (Intelligence Layer)
│   ├── ORACLE_Architecture.md         (Intelligence Layer)
│   ├── MIRROR_Architecture.md         (Intelligence Layer)
│   ├── MNEMOSYNE_Architecture.md      (Intelligence Layer)
│   └── PYTHIA_Architecture.md         (Intelligence Layer)
│
├── adr/                               ← Architecture Decision Records
│   └── README.md                      ← ADR index and template
│
├── integrations/                      ← external system integration docs
│   ├── README.md
│   ├── pacer.md
│   ├── westlaw-lexis.md
│   ├── imanage.md
│   ├── aderant-elite.md
│   ├── microsoft365.md
│   └── model-providers.md
│
└── schemas/                           ← data model and schema reference
    └── DATA_MODELS.md
```

---

## Platform at a Glance

```
SAFETY GATES (Phase 1-2 · Weeks 1-8)
  PCES/AEGIS ──── NOMOS/PGS
        │
        ▼
CORE INFRASTRUCTURE (Phase 3-4 · Weeks 9-28)
  MOIRAI ──── MIMIR
        │
        ▼
QUALITY LOOPS (Phase 5-6 · Weeks 29-46)
  PLUTUS ──── ALETHEIA ──── KAIROS
        │
        ▼
OPERATIONAL INTELLIGENCE (Phase 7-8 · Weeks 47-66)
  HERMES ──── ARGUS ──── LOGOS ──── HADES
        │
        ▼
INTELLIGENCE LAYER (Year 2-3)
  SCRIBE · STOA · ORACLE · MIRROR · MNEMOSYNE · PYTHIA
```

---

## Key Principles

- **Governance before capability.** Safety gates deploy before anything that touches client data.
- **Provenance as infrastructure.** MOIRAI is the substrate. Content without provenance does not exist in THEMIS.
- **Policy as configuration.** Firm AI policy lives in the PGS rule engine, not in documentation that relies on human compliance.
- **Immutability of record.** Audit trails are append-only. Corrections produce new records, never overwrites.
- **Value compounds with maturity.** Each service is more valuable with the others operational. The platform is a flywheel.

---

## Current Status

See [CHANGELOG.md](CHANGELOG.md) for the latest platform changes.

See [platform/THEMIS_Roadmap.md](platform/THEMIS_Roadmap.md) for the 24-sprint implementation roadmap (MVP → Full Operating Capability).

---

*THEMIS — Trusted Human-AI Enablement for Matter Intelligence and Safety*
