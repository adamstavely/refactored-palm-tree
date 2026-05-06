# CHANGELOG

All significant changes to the THEMIS platform — service behavior, event schemas, governance policies, and API surfaces — are documented here. Changes are listed in reverse chronological order.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

Architecture and documentation complete. Platform in pre-build phase. No production changes to record.

---

## [0.1.0] — 2026-05-05 — Initial Architecture Release

### Added — Platform Architecture
- Full THEMIS platform architecture documented across 14 sections
- Nine core service architecture documents (PCES, PGS, MOIRAI, TCS, FGS, FGTS, TVS, RQS, KCS, ERAS)
- HADES adversarial evaluation service architecture
- Intelligence Layer proposal: SCRIBE, STOA, ORACLE, MIRROR, MNEMOSYNE, PYTHIA
- 24-sprint product roadmap (MVP → Full Operating Capability, 17 months)
- OTA prototype agreement for MOIRAI Provenance Service

### Added — Architectural Decisions
- Data sovereignty regional topology: hub-and-spoke with EU, UK, US, APAC data planes
- Financial Governance Service (FGS / PLUTUS) as Phase 1 deliverable
- Client Transparency Portal and Client Attestation API design
- HITL Hold Queue with supervised gates and hard stops

### Added — Prompt Lineage (MOIRAI extension)
- PromptTemplate node: versioned system instruction set with AI Governance Committee approval record
- AnalystInput node: raw user query with interface state capture
- PromptAssembly node: exact payload sent to model with assembled_messages_hash
- Seven new provenance graph edges: GOVERNED_BY, ASSEMBLED_FROM, INJECTED_WITH, EXPRESSED_AS, ACTIVE_AT, BUILT_FROM, PRODUCED
- Four-way failure attribution model: model vs. retrieval vs. prompt instruction vs. analyst intent

### Added — Data Retention Architecture (MOIRAI extension)
- Nullification policy by node type; retention periods established
- Matter close lifecycle: staged nullification at T+30d, T+90d, T+7yr
- GDPR Article 17 handling: pseudonymisation at IAM boundary; analyst erasure without MOIRAI changes
- Litigation hold interaction: nullification suspended while hold active

### Added — Training Data Consent Architecture (FGTS extension)
- Three-tier consent model: AI_ASSISTED_WORK, AI_TRAINING_ANONYMISED, AI_TRAINING_FULL
- ConsentRecord schema with consent_level tagging on CorpusEntry
- RevocationEvent: immediate corpus tagging update; past runs documented in model registry
- Four fine-tuning governance gates: consent coverage, privilege review, jurisdiction assessment, AI Governance Committee approval

### Added — Repository Structure
- README.md: audience-specific navigation and platform overview
- GLOSSARY.md: full vocabulary reference organized by category
- CHANGELOG.md: this document
- platform/ directory: platform-level documents
- services/ directory: 17 individual service architecture documents
- adr/ directory: Architecture Decision Records index and template
- integrations/ directory: external system integration references
- schemas/ directory: data model and schema reference

---

## Versioning Policy

THEMIS uses semantic versioning at the platform level:

- **MAJOR** version — breaking changes to inter-service event contracts or API surfaces that require coordinated deployment across multiple services
- **MINOR** version — new capabilities added in a backward-compatible manner; new services added; new fields added to existing schemas
- **PATCH** version — bug fixes, documentation corrections, configuration changes that do not affect service behavior

Each service maintains its own version independently. The platform version reflects the minimum compatible service version set.

### Schema Change Policy

Changes to event schemas (Kafka topics) and API contracts follow a deprecation process:
1. New field added with `nullable: true` and documented in schemas/DATA_MODELS.md
2. Consuming services updated to handle both old and new formats (compatibility period: minimum 2 sprint cycles)
3. Old format deprecated with a MINOR version bump and CHANGELOG entry
4. Old format removed with a MAJOR version bump and advance notice of minimum 4 sprint cycles

### Policy Change Policy

Changes to PGS rule definitions that affect analyst-facing behavior are documented here under the relevant service heading with:
- The rule ID and version
- What behavior changed
- Why the change was made
- The effective date

---

*Maintained by the THEMIS Platform Engineering team.*
*Questions: raise an issue or contact the Platform Architect.*
