# THEMIS Data Models

Single authoritative reference for every schema in the THEMIS platform. Organised by service and data store. All inter-service event contracts are in the Events section.

When a schema changes, this document is updated in the same PR. Schema version history is tracked in CHANGELOG.md.

> **v2.0 — IC Intelligence Platform.** This document supersedes the prior law firm schema. All references to `matter_id`, `client_id`, `privilege_type`, `data_jurisdiction`, `consent_record`, `hold_record`, and Elo ratings have been replaced with IC intelligence platform equivalents. See CHANGELOG for full diff.

---

## Table of Contents

**Platform Layer**
1. [MOIRAI — Provenance Service (Neo4j + PostgreSQL + Kafka)](#1-moirai--provenance-service)
2. [PCES / AEGIS — Classification Enforcement Service (PostgreSQL)](#2-pces--aegis--classification-enforcement-service)
3. [PGS / NOMOS — Policy & Guardrails Service (PostgreSQL)](#3-pgs--nomos--policy--guardrails-service)
4. [FGS / PLUTUS — Financial Governance Service (PostgreSQL + Redis)](#4-fgs--plutus--financial-governance-service)
5. [TCS / MIMIR — Trust Calibration Service (PostgreSQL)](#5-tcs--mimir--trust-calibration-service)
6. [FGTS / ALETHEIA — Feedback & Ground Truth Service (PostgreSQL)](#6-fgts--aletheia--feedback--ground-truth-service)
7. [TVS / KAIROS — Temporal Validity Service (PostgreSQL + Redis)](#7-tvs--kairos--temporal-validity-service)
8. [RQS / HERMES — Retrieval Quality Service (PostgreSQL)](#8-rqs--hermes--retrieval-quality-service)
9. [KCS / ARGUS — Knowledge Currency Service (PostgreSQL)](#9-kcs--argus--knowledge-currency-service)
10. [CVS / VERITAS — Source Corroboration Service (PostgreSQL)](#10-cvs--veritas--source-corroboration-service)
11. [IAS / SCUDO — Adversarial Screening Service (PostgreSQL)](#11-ias--scudo--adversarial-screening-service)
12. [MAS / EIDOLON — Media Authenticity Service (PostgreSQL)](#12-mas--eidolon--media-authenticity-service)
13. [MDS / KRONOS — Model Drift Service (PostgreSQL)](#13-mds--kronos--model-drift-service)
14. [SCBS / SENTINEL-CAP — Session Capability Bounding Service (PostgreSQL + Redis)](#14-scbs--sentinel-cap--session-capability-bounding-service)
15. [CBS / BROKER — Credential Broker Service (PostgreSQL + Redis + Vault)](#15-cbs--broker--credential-broker-service)
16. [RSS / ROLLBACK — State Snapshot Service (PostgreSQL + Redis + Object Storage)](#16-rss--rollback--state-snapshot-service)
17. [UCS / TYCHE — Uncertainty Characterisation Service (PostgreSQL)](#17-ucs--tyche--uncertainty-characterisation-service)
18. [DPS / CODEX — Document Provenance Service (PostgreSQL)](#18-dps--codex--document-provenance-service)
19. [OFS / NEMESIS — Outcome Feedback Service (PostgreSQL)](#19-ofs--nemesis--outcome-feedback-service)
20. [ERAS / LOGOS — Reasoning Audit Service (Elasticsearch)](#20-eras--logos--reasoning-audit-service)
21. [HADES — Adversarial Intelligence Repository (PostgreSQL + Elasticsearch air-gapped)](#21-hades--adversarial-intelligence-repository)

**Event Contracts**

22. [Kafka Event Schemas](#22-kafka-event-schemas)
23. [Cross-Service Reference Types](#23-cross-service-reference-types)

---

## 1. MOIRAI — Provenance Service

**Stores:** Neo4j (provenance graph) · PostgreSQL (append-only event ledger) · Kafka (event bus)
**Source of truth:** The PostgreSQL append-only event ledger. Neo4j is a derived, queryable view rebuilt from the ledger. The Kafka stream is the real-time feed.

Every event in MOIRAI is cryptographically signed and hash-chained. No event can be added, deleted, or modified without breaking the chain.

### 1.1 Neo4j Node Types

#### Session
```
Node: Session
Properties:
  session_id             uuid          PK — issued by PCES/AEGIS
  analyst_id_hash        str           SHA-256 of analyst IAM identity
  classification         str           classification ceiling for this session
  compartments           str[]         compartments accessible in this session
  session_type           str           ANALYTICAL | RESEARCH | AGENT | STOA
  requirement_ref        str | null    external requirement identifier
  pces_grant_ref         uuid          FK → PCES CompartmentDecision
  started_at             datetime
  ended_at               datetime | null
  status                 str           ACTIVE | COMPLETE | SUSPENDED | TERMINATED
```

#### CorpusSource
```
Node: CorpusSource
Properties:
  source_id              uuid          PK
  source_type            str           SIGINT | HUMINT | GEOINT | OSINT | TECHINT |
                                       IMINT | MASINT | FININT | CYBER | OPEN_SOURCE
  classification         str           classification of this source
  compartment            str | null    compartment if applicable
  collection_date        datetime | null
  ingested_at            datetime
  ingested_by            str           analyst_id_hash
  title                  str
  file_hash              str           SHA-256
  capability_zone        str           green | amber | red | unassessed
  tvs_validity_score     float | null  current TVS/KAIROS score
  kcs_superseded         bool          false unless KCS has flagged supersession
```

#### Chunk
```
Node: Chunk
Properties:
  chunk_id               str           SHA-256 — content-addressed identity
  lsh_fingerprint        str           MinHash (k=5, n=128)
  source_id              uuid          FK → CorpusSource
  classification         str
  compartment            str | null
  modality               str           document | video | audio | email | chat | image
  capability_zone        str           green | amber | red | unassessed

  # Document modality fields
  page_number            int | null
  bounding_box           jsonb | null  {x1, y1, x2, y2}
  reading_order_seq      int | null
  section_label          str | null
  content_type           str | null    body | heading | footnote | caption | table_cell

  # Video/audio modality fields
  timecode_start         float | null
  timecode_end           float | null
  speaker_id             str | null    pseudonymised

  # Email/chat modality fields
  thread_id              str | null
  message_position       int | null
  sender_id              str | null    pseudonymised
  message_timestamp      datetime | null
```

#### TurnRecord
```
Node: TurnRecord
Properties:
  turn_id                uuid          PK
  session_id             uuid          FK → Session
  turn_index             int
  role                   str           user | assistant | tool
  timestamp              datetime
  classification         str           inherits from session
  requirement_ref        str | null

  # Prompt lineage
  prompt_version_id      uuid | null   FK → PRS PromptVersion
  prompt_version_hash    str | null    SHA-256 — MOIRAI-attested
  skill_id               uuid | null   FK → SKS Skill (if skill-invoked)

  # Context
  retrieval_chunk_ids    str[]         SHA-256 array
  prior_turn_ids         uuid[]
  input_token_count      int
  output_token_count     int
  output_chunk_id        str | null    SHA-256 → OutputChunk

  # Provenance chain
  prev_event_hash        str           SHA-256 of prior MOIRAI event
  signature              str           HMAC-SHA256 service signature
```

#### Claim
```
Node: Claim
Properties:
  claim_id               uuid          PK
  turn_id                uuid          FK → TurnRecord
  text_hash              str           SHA-256 of claim text
  claim_type             str           GRND | PARAM | SYNTH
  supporting_chunk_ids   str[]         SHA-256 array
  unsupported            bool
  cvs_verified           bool | null
  cvs_verification_id    uuid | null   FK → CVS VerificationRecord
  eras_capture_id        uuid          FK → ERAS ReasoningCapture
  ucs_uncertainty_type   str | null    aleatory | epistemic | model | uncharacterised
  analyst_resolved       bool          false until analyst answers UCS questions
```

#### OutputChunk
```
Node: OutputChunk
Properties:
  chunk_id               str           SHA-256 — content-addressed
  turn_id                uuid          FK → TurnRecord
  lsh_fingerprint        str
  classification         str
  watermark_key_id       uuid | null
```

#### AnalyticalProduct
```
Node: AnalyticalProduct
Properties:
  product_id             uuid          PK
  title                  str
  version                int
  classification         str
  compartment            str | null
  requirement_ref        str | null
  ai_assisted            bool
  finalized_at           datetime
  finalized_by           str           analyst_id_hash
  certification_hash     str           SHA-256
  provenance_snapshot    str           SHA-256 of graph state at certification
  pcs_package_id         uuid | null   FK → PCS ConsumerPackage
  dps_record_id          uuid          FK → DPS/CODEX DocumentRecord
```

#### RetrievalTrajectory
```
Node: RetrievalTrajectory
Properties:
  trajectory_id          uuid          PK
  turn_id                uuid          FK → TurnRecord
  total_steps            int
  terminal_chunks        str[]         SHA-256 array — final context window chunks
  rqs_quality_id         uuid | null   FK → RQS RetrievalQualityRecord
  steps                  jsonb         [{step_index, query_embedding_hash,
                                         retrieved_chunks, step_score,
                                         tvs_weighted_score}]
```

#### SourceInvalidationRecord
```
Node: SourceInvalidationRecord
Properties:
  invalidation_id        uuid          PK
  source_id              uuid          FK → CorpusSource
  invalidation_basis     str           kcs_supersession | tvs_decay | analyst_correction |
                                       supervisor_authority | iob_order
  invalidated_at         datetime
  authorised_by          str           analyst_id_hash | service_id
  downstream_claims      int           count of Claims that cited this source
  blast_radius_notified  bool
```

### 1.2 Neo4j Edge Types

| Edge Label | From | To | Properties |
|---|---|---|---|
| `GRANTS` | Session | Chunk | `compartment`, `granted_at` |
| `INGEST` | CorpusSource | Chunk | `ingested_at`, `chunk_sequence` |
| `RETRIEVED` | TurnRecord | Chunk | `retrieval_rank`, `similarity_score`, `tvs_score_at_retrieval` |
| `PRODUCED` | TurnRecord | OutputChunk | `generation_timestamp` |
| `PRODUCED_CLAIM` | TurnRecord | Claim | `claim_index` |
| `SUPPORTED_BY` | Claim | Chunk | `cited_rank`, `cvs_status` |
| `COMPILED_INTO` | Claim | AnalyticalProduct | `section_ref` |
| `USED_PROMPT` | TurnRecord | PromptVersionRef | `version_hash` |
| `TRAJECTORY_FOR` | RetrievalTrajectory | TurnRecord | — |
| `FOLLOWS` | TurnRecord | TurnRecord | `session_id`, `turn_index` |
| `WITHIN` | TurnRecord | Session | — |
| `INVALIDATED_BY` | CorpusSource | SourceInvalidationRecord | `invalidated_at` |
| `ATTESTS` | ProvenanceEvent | (any node) | `prev_event_hash`, `signature`, `tsa_anchor_ref` |

### 1.3 PostgreSQL — Append-Only Event Ledger

```sql
-- Every MOIRAI event (append-only; no UPDATE or DELETE ever)
CREATE TABLE moirai_event (
  event_id           uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type         varchar(128)  NOT NULL,
  service_id         varchar(64)   NOT NULL,
  session_id         uuid,
  classification     varchar(64)   NOT NULL,
  event_payload      jsonb         NOT NULL,
  prev_event_hash    char(64)      NOT NULL,   -- SHA-256 of prior event
  event_hash         char(64)      NOT NULL UNIQUE,
  signature          text          NOT NULL,   -- HMAC-SHA256 per-service key
  tsa_anchor_ref     text,                     -- RFC 3161 timestamp authority ref
                                               -- populated on 24h anchor events
  recorded_at        timestamptz   NOT NULL DEFAULT now()
);

CREATE INDEX idx_moirai_session   ON moirai_event (session_id);
CREATE INDEX idx_moirai_service   ON moirai_event (service_id);
CREATE INDEX idx_moirai_recorded  ON moirai_event (recorded_at);
-- No other indexes — this table is append-only; reads are for audit traversal
```

---

## 2. PCES / AEGIS — Classification Enforcement Service

**Store:** PostgreSQL

```sql
-- Session privilege grant (one per analyst session)
CREATE TABLE session_privilege (
  grant_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL UNIQUE,
  analyst_id_hash        char(64)      NOT NULL,
  clearance_level        varchar(64)   NOT NULL,
  compartments           text[]        NOT NULL DEFAULT '{}',
  session_type           varchar(32)   NOT NULL CHECK (session_type IN
                         ('ANALYTICAL','RESEARCH','AGENT','STOA')),
  requirement_ref        text,
  pressure_mode          varchar(32)   NOT NULL DEFAULT 'NONE' CHECK (pressure_mode IN
                         ('NONE','ADVISORY','ACTIVE','CRITICAL')),
  granted_at             timestamptz   NOT NULL DEFAULT now(),
  expires_at             timestamptz   NOT NULL,
  revoked                bool          NOT NULL DEFAULT false,
  revoked_at             timestamptz,
  revocation_reason      text,
  moirai_event_id        uuid          NOT NULL   -- FK → moirai_event
);

-- Per-request compartment decision (append-only audit log)
CREATE TABLE compartment_decision (
  decision_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL REFERENCES session_privilege (session_id),
  turn_id                uuid,
  resource_classification varchar(64)  NOT NULL,
  resource_compartment   varchar(64),
  decision               varchar(16)   NOT NULL CHECK (decision IN
                         ('PERMIT','DENY','DOWNGRADE','ESCALATE')),
  reason                 text,
  evaluated_at           timestamptz   NOT NULL DEFAULT now()
);

-- Conflict of interest record (IC CoI: analyst connected to entities in requirement)
CREATE TABLE conflict_of_interest (
  coi_id                 uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  analyst_id_hash        char(64)      NOT NULL,
  session_id             uuid          NOT NULL,
  entity_ref             text          NOT NULL,   -- OGS entity_id or free text
  coi_type               varchar(64)   NOT NULL CHECK (coi_type IN
                         ('PERSONAL_CONNECTION','PRIOR_REPORTING','FINANCIAL_INTEREST',
                          'FAMILY_CONNECTION','PRIOR_INVOLVEMENT')),
  severity               varchar(16)   NOT NULL CHECK (severity IN
                         ('INFORMATIONAL','ADVISORY','BLOCKING')),
  detected_at            timestamptz   NOT NULL DEFAULT now(),
  reviewed_by            char(64),
  reviewed_at            timestamptz,
  resolution             varchar(32)   CHECK (resolution IN
                         ('PERMITTED','RECUSED','SUPERVISED','BLOCKED'))
);

-- Pressure mode assessment log
CREATE TABLE pressure_assessment (
  assessment_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  analyst_id_hash        char(64)      NOT NULL,
  pressure_indicators    text[]        NOT NULL,
  pressure_mode          varchar(32)   NOT NULL,
  trigger_service        varchar(32),
  assessed_at            timestamptz   NOT NULL DEFAULT now()
);
```

---

## 3. PGS / NOMOS — Policy & Guardrails Service

**Store:** PostgreSQL

```sql
-- Policy versions (IOB-approved; immutable once active)
CREATE TABLE policy_version (
  version_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  version_string         varchar(32)   NOT NULL UNIQUE,
  iob_approval_ref       text          NOT NULL,
  effective_from         timestamptz   NOT NULL,
  effective_until        timestamptz,
  authored_by            char(64)      NOT NULL,
  change_summary         text          NOT NULL,
  active                 bool          NOT NULL DEFAULT true
);

-- Policy rules (grouped under a policy version)
CREATE TABLE policy_rule (
  rule_id                uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  policy_version_id      uuid          NOT NULL REFERENCES policy_version,
  name                   varchar(256)  NOT NULL,
  priority               int           NOT NULL,
  scope                  varchar(32)   NOT NULL CHECK (scope IN
                         ('global','interaction_class','domain','classification_level')),
  scope_value            text,
  trigger_type           varchar(64)   NOT NULL CHECK (trigger_type IN
                         ('input_screen','output_screen','interaction_classify',
                          'standards_compliance','adversarial_detect','classification_boundary')),
  condition_cel          text          NOT NULL,
  action                 varchar(32)   NOT NULL CHECK (action IN
                         ('BLOCK','WARN','FLAG','REQUIRE_REVIEW','CLASSIFY')),
  action_params          jsonb,
  rationale              text          NOT NULL,
  active                 bool          NOT NULL DEFAULT true
);

-- Per-request screening record (append-only)
CREATE TABLE screening_record (
  record_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  turn_id                uuid          NOT NULL,
  session_id             uuid          NOT NULL,
  screen_type            varchar(32)   NOT NULL CHECK (screen_type IN
                         ('INPUT','OUTPUT','RETRIEVAL')),
  interaction_class      varchar(64),
  rules_evaluated        uuid[]        NOT NULL DEFAULT '{}',
  rules_triggered        uuid[]        NOT NULL DEFAULT '{}',
  outcome                varchar(32)   NOT NULL CHECK (outcome IN
                         ('PASS','WARN','BLOCK','FLAG')),
  icd_203_compliant      bool          NOT NULL DEFAULT true,
  classification_flag    bool          NOT NULL DEFAULT false,
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Interaction classification record
CREATE TABLE interaction_classification (
  classification_id      uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  turn_id                uuid          NOT NULL,
  session_id             uuid          NOT NULL,
  interaction_class      varchar(64)   NOT NULL,
  fgs_cost_weight        numeric(6,4)  NOT NULL,
  classified_at          timestamptz   NOT NULL DEFAULT now()
);

-- Screening thresholds (configurable per interaction class)
CREATE TABLE screening_threshold (
  threshold_id           uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  interaction_class      varchar(64)   NOT NULL,
  completeness_min       numeric(5,4)  NOT NULL DEFAULT 0.70,
  unsupported_claim_max  int           NOT NULL DEFAULT 3,
  adversarial_score_max  numeric(5,4)  NOT NULL DEFAULT 0.30,
  policy_version_id      uuid          NOT NULL REFERENCES policy_version,
  UNIQUE (interaction_class, policy_version_id)
);
```

---

## 4. FGS / PLUTUS — Financial Governance Service

**Store:** PostgreSQL (transactions + accounts) · Redis (live balance cache)

```sql
-- Token accounts (TEAM, REQUIREMENT, DIVISION, RESERVE, ANALYST)
CREATE TABLE token_account (
  account_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  account_type           varchar(32)   NOT NULL CHECK (account_type IN
                         ('TEAM','REQUIREMENT','DIVISION','RESERVE','ANALYST')),
  account_name           varchar(256)  NOT NULL,
  period_string          varchar(16)   NOT NULL,   -- e.g. "2026-Q2"
  period_allocation      bigint        NOT NULL,   -- tokens allocated this period
  consumed               bigint        NOT NULL DEFAULT 0,
  reserve_draws          bigint        NOT NULL DEFAULT 0,
  status                 varchar(32)   NOT NULL DEFAULT 'ACTIVE' CHECK (status IN
                         ('ACTIVE','APPROACHING','EXHAUSTED','SUSPENDED')),
  managed_by             char(64)      NOT NULL,   -- supervisor analyst_id_hash
  created_at             timestamptz   NOT NULL DEFAULT now(),
  updated_at             timestamptz   NOT NULL DEFAULT now()
);

-- Per-turn token transaction (immutable after write)
CREATE TABLE token_transaction (
  transaction_id         uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id             uuid          NOT NULL REFERENCES token_account,
  session_id             uuid          NOT NULL,
  turn_id                uuid,
  analyst_id_hash        char(64)      NOT NULL,
  interaction_class      varchar(64)   NOT NULL,
  model_version          varchar(128)  NOT NULL,
  tokens_consumed        int           NOT NULL,
  tokens_estimated       int           NOT NULL,   -- pre-call estimate from SCBS
  transaction_type       varchar(32)   NOT NULL CHECK (transaction_type IN
                         ('SESSION','AGENT_TASK','STOA','BACKGROUND_SERVICE')),
  requirement_ref        text,
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Allocation periods (append-only)
CREATE TABLE allocation_period (
  period_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  period_string          varchar(16)   NOT NULL UNIQUE,
  start_date             date          NOT NULL,
  end_date               date          NOT NULL,
  total_allocated        bigint        NOT NULL,
  reserve_pool           bigint        NOT NULL,
  tokenomics_model_id    uuid          NOT NULL,
  approved_by            char(64)      NOT NULL,   -- IOB approval reference
  opened_at              timestamptz   NOT NULL DEFAULT now()
);

-- Reserve access requests
CREATE TABLE reserve_request (
  request_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  requesting_account_id  uuid          NOT NULL REFERENCES token_account,
  analyst_id_hash        char(64)      NOT NULL,
  tokens_requested       bigint        NOT NULL,
  justification          text          NOT NULL,
  approved_by            char(64),
  status                 varchar(16)   NOT NULL DEFAULT 'PENDING' CHECK (status IN
                         ('PENDING','APPROVED','DENIED')),
  tokens_granted         bigint,
  requested_at           timestamptz   NOT NULL DEFAULT now(),
  resolved_at            timestamptz
);

-- Consumption anomaly events
CREATE TABLE consumption_anomaly (
  anomaly_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id             uuid          NOT NULL REFERENCES token_account,
  session_id             uuid,
  anomaly_type           varchar(64)   NOT NULL CHECK (anomaly_type IN
                         ('EXCESS_CONSUMPTION','UNUSUAL_PATTERN',
                          'RESERVE_OVERUSE','ZERO_CONSUMPTION')),
  severity               varchar(16)   NOT NULL CHECK (severity IN
                         ('HIGH','MEDIUM','LOW')),
  description            text          NOT NULL,
  tokens_involved        bigint        NOT NULL,
  detected_at            timestamptz   NOT NULL DEFAULT now(),
  reviewed               bool          NOT NULL DEFAULT false,
  reviewed_at            timestamptz
);

-- Tokenomics model versions (IOB-approved)
CREATE TABLE tokenomics_model (
  model_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  version                varchar(32)   NOT NULL UNIQUE,
  allocation_basis       text          NOT NULL,
  reserve_fraction       numeric(5,4)  NOT NULL,
  escalation_threshold   numeric(5,4)  NOT NULL DEFAULT 0.80,
  interaction_class_weights jsonb      NOT NULL,  -- {class: relative_weight}
  approved_by            char(64)      NOT NULL,  -- IOB reference
  effective_from         timestamptz   NOT NULL
);
```

**Redis balance cache:**
```
KEY:   fgs:balance:{account_id}:{period_string}
VALUE: {allocated: int, consumed: int, remaining: int, status: str, cached_at: ISO8601}
TTL:   300 seconds
```

---

## 5. TCS / MIMIR — Trust Calibration Service

**Store:** PostgreSQL

```sql
-- Bayesian calibration posterior per analyst × domain × claim_type
CREATE TABLE calibration_posterior (
  posterior_id           uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  analyst_id_hash        char(64)      NOT NULL,
  domain                 varchar(64)   NOT NULL,
  claim_type             varchar(64)   NOT NULL,   -- GRND | PARAM | SYNTH
  interaction_class      varchar(64)   NOT NULL,
  alpha                  numeric(10,4) NOT NULL DEFAULT 1.0,  -- Beta dist prior
  beta                   numeric(10,4) NOT NULL DEFAULT 1.0,
  mean                   numeric(5,4)  GENERATED ALWAYS AS
                         (alpha / (alpha + beta)) STORED,
  calibration_state      varchar(32)   NOT NULL DEFAULT 'PRIOR_ONLY' CHECK
                         (calibration_state IN
                         ('PRIOR_ONLY','PROVISIONAL','CALIBRATED',
                          'FLAGGED','SUSPENDED')),
  observation_count      int           NOT NULL DEFAULT 0,
  last_updated           timestamptz   NOT NULL DEFAULT now(),
  PRIMARY KEY (analyst_id_hash, domain, claim_type, interaction_class)
    ON CONFLICT DO NOTHING
);

-- Posterior update history (append-only)
CREATE TABLE posterior_history (
  id                     bigserial     PRIMARY KEY,
  analyst_id_hash        char(64)      NOT NULL,
  domain                 varchar(64)   NOT NULL,
  claim_type             varchar(64)   NOT NULL,
  alpha_before           numeric(10,4) NOT NULL,
  beta_before            numeric(10,4) NOT NULL,
  alpha_after            numeric(10,4) NOT NULL,
  beta_after             numeric(10,4) NOT NULL,
  trigger_correction_id  uuid,         -- FK → FGTS correction_event
  trigger_outcome_id     uuid,         -- FK → OFS outcome_record
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Gaming probability score per session
CREATE TABLE gaming_probability_score (
  score_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  analyst_id_hash        char(64)      NOT NULL,
  gaming_probability     numeric(5,4)  NOT NULL,
  contributing_signals   jsonb         NOT NULL,  -- {signal_type: weight}
  state_before           varchar(32)   NOT NULL,
  state_after            varchar(32)   NOT NULL,
  scored_at              timestamptz   NOT NULL DEFAULT now()
);

-- Model-level confidence signal (aggregate, for ATHENA display)
CREATE TABLE model_confidence_signal (
  signal_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  turn_id                uuid          NOT NULL,
  analyst_id_hash        char(64)      NOT NULL,
  domain                 varchar(64)   NOT NULL,
  interaction_class      varchar(64)   NOT NULL,
  calibration_state      varchar(32)   NOT NULL,
  posterior_mean         numeric(5,4),
  gaming_adjustment      numeric(5,4)  NOT NULL DEFAULT 0.0,
  cps_zone               varchar(16),  -- green | amber | red | unassessed (proposed)
  signal_emitted_at      timestamptz   NOT NULL DEFAULT now()
);
```

---

## 6. FGTS / ALETHEIA — Feedback & Ground Truth Service

**Store:** PostgreSQL

```sql
-- Correction events (immutable after write)
CREATE TABLE correction_event (
  correction_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  turn_id                uuid          NOT NULL,
  claim_id               uuid,                     -- FK → MOIRAI Claim
  chunk_id               char(64)      NOT NULL,   -- SHA-256
  analyst_id_hash        char(64)      NOT NULL,
  correction_action      varchar(64)   NOT NULL CHECK (correction_action IN
                         ('CONFIRMED','INDEPENDENTLY_VERIFIED',
                          'MISREPRESENTS_SOURCE','FACTUALLY_INCORRECT',
                          'OUTDATED','OUT_OF_SCOPE','UNSUPPORTED',
                          'PARTIALLY_CORRECT','CANNOT_VERIFY')),
  original_content_hash  char(64)      NOT NULL,   -- SHA-256; not stored plain
  corrected_content_hash char(64),                 -- SHA-256 if correction provided
  interaction_class      varchar(64),
  domain                 varchar(64),
  session_id             uuid          NOT NULL,
  requirement_ref        text,
  source_type            varchar(64),              -- SIGINT | HUMINT | GEOINT etc.
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Five-factor weighting per correction
CREATE TABLE weighting_factors (
  weight_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  correction_id          uuid          NOT NULL UNIQUE REFERENCES correction_event,
  analyst_domain_weight  numeric(5,4)  NOT NULL,  -- analyst expertise in this domain
  source_type_weight     numeric(5,4)  NOT NULL,  -- source type reliability
  claim_type_weight      numeric(5,4)  NOT NULL,  -- claim type difficulty
  interaction_class_weight numeric(5,4) NOT NULL, -- interaction class complexity
  gaming_probability_adj numeric(5,4)  NOT NULL,  -- gaming probability penalty
  composite_weight       numeric(5,4)  NOT NULL,  -- product of all factors
  computed_at            timestamptz   NOT NULL DEFAULT now()
);

-- Ground truth corpus entries
CREATE TABLE corpus_entry (
  entry_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  correction_id          uuid          NOT NULL REFERENCES correction_event,
  session_id             uuid          NOT NULL,
  domain                 varchar(64)   NOT NULL,
  interaction_class      varchar(64)   NOT NULL,
  input_context_hash     char(64)      NOT NULL,   -- SHA-256; not stored plain
  ai_output_hash         char(64)      NOT NULL,
  correction_action      varchar(64)   NOT NULL,
  composite_weight       numeric(5,4)  NOT NULL,
  quality_score          numeric(5,4)  NOT NULL,
  labels                 text[]        NOT NULL DEFAULT '{}',
  corpus_version_id      uuid,
  created_at             timestamptz   NOT NULL DEFAULT now()
);

-- Corpus versions (append-only; each correction produces a new version)
CREATE TABLE corpus_version (
  version_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  version_number         int           NOT NULL,
  entry_count            int           NOT NULL,
  weighted_entry_count   numeric(12,4) NOT NULL,
  domain_distribution    jsonb         NOT NULL,
  moirai_event_id        uuid          NOT NULL,
  created_at             timestamptz   NOT NULL DEFAULT now()
);

-- Analyst correction profile (aggregate signal for TCS)
CREATE TABLE analyst_signal (
  analyst_id_hash        char(64)      PRIMARY KEY,
  correction_count       int           NOT NULL DEFAULT 0,
  confirmed_rate         numeric(5,4),             -- fraction CONFIRMED
  domain_expertise       jsonb,                    -- {domain: weight}
  interaction_class_dist jsonb,                    -- {class: count}
  gaming_risk            numeric(5,4)  NOT NULL DEFAULT 0.0,
  signal_weight          numeric(5,4)  NOT NULL DEFAULT 1.0,
  updated_at             timestamptz   NOT NULL DEFAULT now()
);
```

---

## 7. TVS / KAIROS — Temporal Validity Service

**Store:** PostgreSQL (validity records + history) · Redis (current score cache)

```sql
-- Validity records (one per corpus chunk)
CREATE TABLE validity_record (
  chunk_id               char(64)      PRIMARY KEY,  -- SHA-256
  decay_score            numeric(5,4)  NOT NULL,
  contested              bool          NOT NULL DEFAULT false,
  invalidated            bool          NOT NULL DEFAULT false,
  invalidated_by_chunk   char(64),                   -- superseding chunk SHA-256
  composite_score        numeric(5,4)  NOT NULL,
  decay_profile          varchar(64)   NOT NULL,
  classification         varchar(64)   NOT NULL,
  source_type            varchar(64)   NOT NULL,      -- IC collection method
  ingested_at            timestamptz   NOT NULL,
  last_scored_at         timestamptz   NOT NULL,
  model_knowledge_cutoff timestamptz,               -- for PARAM-type chunks
  retrieval_freshness    numeric(5,4)
);

-- Score history (append-only)
CREATE TABLE validity_score_history (
  id                     bigserial     PRIMARY KEY,
  chunk_id               char(64)      NOT NULL,
  composite_score        numeric(5,4)  NOT NULL,
  decay_score            numeric(5,4)  NOT NULL,
  trigger_type           varchar(64)   NOT NULL CHECK (trigger_type IN
                         ('scheduled_decay','kcs_supersession','analyst_correction',
                          'scribe_diff_detected','moirai_blast_radius',
                          'conflict_resolution','propagation')),
  trigger_id             uuid,
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Conflict events between chunks
CREATE TABLE conflict_event (
  conflict_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  chunk_a                char(64)      NOT NULL,
  chunk_b                char(64)      NOT NULL,
  conflict_type          varchar(64)   NOT NULL CHECK (conflict_type IN
                         ('direct_contradiction','supersession','partial_overlap',
                          'temporal_inconsistency')),
  detection_method       varchar(32)   NOT NULL CHECK (detection_method IN
                         ('semantic','structural','kcs_event','scribe_diff')),
  contradiction_score    numeric(5,4)  NOT NULL,
  classification         varchar(64)   NOT NULL,
  status                 varchar(32)   NOT NULL DEFAULT 'open' CHECK (status IN
                         ('open','resolved','dismissed')),
  detected_at            timestamptz   NOT NULL DEFAULT now(),
  resolved_at            timestamptz,
  resolver_id_hash       char(64),
  decision               varchar(32)   CHECK (decision IN
                         ('a_supersedes_b','b_supersedes_a','both_valid','both_discarded')),
  rationale              text
);

-- Decay profiles (domain-configurable)
CREATE TABLE decay_profile (
  profile_name           varchar(64)   PRIMARY KEY,
  source_type            varchar(64),               -- null = applies to all
  function_type          varchar(32)   NOT NULL CHECK (function_type IN
                         ('step','exponential','linear','aggressive_exponential',
                          'model_vintage','collection_dependent')),
  half_life_days         int,
  floor_score            numeric(5,4)  NOT NULL DEFAULT 0.05,
  default_score          numeric(5,4)  NOT NULL DEFAULT 1.0,
  on_event_score         numeric(5,4),
  updated_at             timestamptz   NOT NULL DEFAULT now()
);
```

**Redis cache:**
```
KEY:   tvs:validity:{chunk_id}
VALUE: {score: float, contested: bool, invalidated: bool, cached_at: ISO8601}
TTL:   300 seconds
```

---

## 8. RQS / HERMES — Retrieval Quality Service

**Store:** PostgreSQL

```sql
-- Per-query retrieval quality record
CREATE TABLE retrieval_quality_record (
  record_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  query_turn_id          uuid          NOT NULL,
  session_id             uuid          NOT NULL,
  domain                 varchar(64),
  requirement_ref        text,
  retrieved_chunks       char(64)[]    NOT NULL,   -- SHA-256 array
  confirmed_relevant     char(64)[]    NOT NULL DEFAULT '{}',
  confirmed_missed       char(64)[]    NOT NULL DEFAULT '{}',
  precision_score        numeric(5,4),
  recall_estimate        numeric(5,4),
  mean_validity_score    numeric(5,4),
  stale_chunk_count      int           NOT NULL DEFAULT 0,
  coverage_score         numeric(5,4),              -- domain coverage 0.0–1.0
  interaction_class      varchar(64),
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Retrieval miss signals (from FGTS analyst corrections)
CREATE TABLE retrieval_miss_signal (
  signal_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  correction_id          uuid          NOT NULL,
  query_turn_id          uuid          NOT NULL,
  session_id             uuid          NOT NULL,
  domain                 varchar(64),
  source_type_missing    varchar(64),              -- which IC collection type was absent
  miss_pattern           varchar(64)   CHECK (miss_pattern IN
                         ('domain_gap','recency_miss','collection_type_gap',
                          'validity_blind','classification_ceiling_gap',
                          'entity_gap')),
  interaction_class      varchar(64),
  gap_signal_forwarded   bool          NOT NULL DEFAULT false,  -- forwarded to TIS/DIKE?
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);

-- Coverage baselines (rolling, per domain + interaction class)
CREATE TABLE coverage_baseline (
  domain                 varchar(64)   NOT NULL,
  interaction_class      varchar(64)   NOT NULL,
  baseline_coverage      numeric(5,4)  NOT NULL,
  baseline_precision     numeric(5,4)  NOT NULL,
  sample_size            int           NOT NULL,
  computed_at            timestamptz   NOT NULL,
  PRIMARY KEY (domain, interaction_class)
);
```

---

## 9. KCS / ARGUS — Knowledge Currency Service

**Store:** PostgreSQL

```sql
-- External source registry (IC collection feeds and databases)
CREATE TABLE source_record (
  source_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  name                   varchar(256)  NOT NULL,
  source_type            varchar(64)   NOT NULL CHECK (source_type IN
                         ('SIGINT_FEED','HUMINT_REPORT_SYSTEM',
                          'GEOINT_TASKING','OSINT_MONITOR',
                          'TECHINT_DATABASE','FININT_FEED',
                          'CYBER_INTELLIGENCE','ALL_SOURCE_DB')),
  domains                text[]        NOT NULL DEFAULT '{}',
  entity_types_covered   text[]        NOT NULL DEFAULT '{}',
  polling_interval       interval      NOT NULL,
  push_endpoint          text,
  reliability_score      numeric(5,4)  NOT NULL DEFAULT 0.75,
  classification         varchar(64)   NOT NULL,
  active                 bool          NOT NULL DEFAULT true,
  last_checked           timestamptz,
  created_at             timestamptz   NOT NULL DEFAULT now()
);

-- External development events (new intelligence affecting corpus)
CREATE TABLE kcs_event (
  event_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  source_id              uuid          NOT NULL REFERENCES source_record,
  development_type       varchar(64)   NOT NULL CHECK (development_type IN
                         ('new_collection','source_superseded','entity_update',
                          'capability_change','programme_development',
                          'contradictory_intelligence','correction_issued')),
  affected_chunks        jsonb         NOT NULL,  -- [{chunk_id, impact_score, impact_type}]
  confidence             numeric(5,4)  NOT NULL,
  classification         varchar(64)   NOT NULL,
  detected_at            timestamptz   NOT NULL DEFAULT now(),
  status                 varchar(32)   NOT NULL DEFAULT 'pending_review' CHECK (status IN
                         ('pending_review','forwarded_to_tvs','dismissed')),
  human_reviewed         bool          NOT NULL DEFAULT false,
  reviewer_id_hash       char(64),
  reviewed_at            timestamptz
);

-- Watchlist entries (entities and domains under active monitoring)
CREATE TABLE watch_list_entry (
  entry_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid,                     -- originating session if from ATHENA
  entity_type            varchar(64)   NOT NULL CHECK (entity_type IN
                         ('PERSON','ORGANIZATION','FACILITY',
                          'CAPABILITY','PROGRAMME','VESSEL',
                          'TECHNOLOGY','EVENT','DOMAIN')),
  entity_ref             text          NOT NULL,   -- OGS entity_id or free text
  alert_threshold        numeric(5,4)  NOT NULL DEFAULT 0.50,
  notify_analyst_hashes  char(64)[]   NOT NULL DEFAULT '{}',
  classification         varchar(64)   NOT NULL,
  created_by             char(64)      NOT NULL,
  active                 bool          NOT NULL DEFAULT true,
  created_at             timestamptz   NOT NULL DEFAULT now()
);

-- Supersession signals (forwarded to TVS/KAIROS)
CREATE TABLE supersession_signal (
  signal_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  kcs_event_id           uuid          NOT NULL REFERENCES kcs_event,
  superseded_chunk_id    char(64)      NOT NULL,   -- SHA-256
  superseding_chunk_id   char(64),                 -- SHA-256 of replacement; null if no replacement
  impact_score           numeric(5,4)  NOT NULL,
  forwarded_to_tvs       bool          NOT NULL DEFAULT false,
  forwarded_at           timestamptz
);

-- Coverage map (what collection methods cover what domains)
CREATE TABLE coverage_map_entry (
  entry_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  domain                 varchar(64)   NOT NULL,
  source_type            varchar(64)   NOT NULL,
  coverage_score         numeric(5,4)  NOT NULL,  -- 0.0 = no coverage, 1.0 = full
  chunk_count            int           NOT NULL,
  validity_weighted_count numeric(12,4) NOT NULL,
  last_computed          timestamptz   NOT NULL,
  UNIQUE (domain, source_type)
);

-- Model knowledge currency (how stale the model is in a domain)
CREATE TABLE model_knowledge_currency (
  currency_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  model_version          varchar(128)  NOT NULL,
  domain                 varchar(64)   NOT NULL,
  knowledge_cutoff       timestamptz   NOT NULL,
  days_stale             int           NOT NULL,
  currency_score         numeric(5,4)  NOT NULL,  -- 1.0 = current; decays over time
  assessed_at            timestamptz   NOT NULL DEFAULT now(),
  UNIQUE (model_version, domain)
);
```

---

## 10. CVS / VERITAS — Source Corroboration Service

**Store:** PostgreSQL

```sql
-- Per-claim source verification record
CREATE TABLE verification_record (
  verification_id        uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  claim_id               uuid          NOT NULL,   -- FK → MOIRAI Claim
  turn_id                uuid          NOT NULL,
  session_id             uuid          NOT NULL,
  chunk_id               char(64),                 -- SHA-256 — the cited source
  verification_status    varchar(32)   NOT NULL CHECK (verification_status IN
                         ('VERIFIED_ACCURATE','VERIFIED_INACCURATE',
                          'MISREPRESENTS_SOURCE','SOURCE_NOT_FOUND',
                          'CANNOT_VERIFY','SYNTHETIC_LIKELY')),
  source_badge           varchar(8)    NOT NULL CHECK (source_badge IN
                         ('GRND','PARAM','SYNTH')),
  corpus_lookup_performed bool         NOT NULL DEFAULT true,
  retrieval_match_score  numeric(5,4),
  verified_at            timestamptz   NOT NULL DEFAULT now()
);

-- Fabrication / corpus poisoning events
CREATE TABLE fabrication_event (
  event_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  verification_id        uuid          NOT NULL REFERENCES verification_record,
  session_id             uuid          NOT NULL,
  fabrication_type       varchar(64)   NOT NULL CHECK (fabrication_type IN
                         ('HALLUCINATED_CITATION','MISATTRIBUTED_CONTENT',
                          'SYNTHETIC_SOURCE','CORPUS_POISONING_INDICATOR',
                          'ENTITY_CONFUSION')),
  severity               varchar(16)   NOT NULL CHECK (severity IN
                         ('CRITICAL','HIGH','MEDIUM','LOW')),
  hades_forwarded        bool          NOT NULL DEFAULT false,
  detected_at            timestamptz   NOT NULL DEFAULT now()
);
```

---

## 11. IAS / SCUDO — Adversarial Screening Service

**Store:** PostgreSQL

```sql
-- Input and retrieval screening records
CREATE TABLE screening_event (
  event_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  turn_id                uuid          NOT NULL,
  screen_target          varchar(32)   NOT NULL CHECK (screen_target IN
                         ('ANALYST_INPUT','RETRIEVED_CHUNK','MCP_RESPONSE')),
  target_hash            char(64)      NOT NULL,   -- SHA-256 of screened content
  outcome                varchar(16)   NOT NULL CHECK (outcome IN
                         ('PASS','BLOCK','FLAG')),
  threat_patterns_matched text[]       NOT NULL DEFAULT '{}',
  adversarial_score      numeric(5,4)  NOT NULL,
  threat_category        varchar(64),              -- PROMPT_INJECTION | INDIRECT_INJECTION |
                                                   -- CORPUS_POISONING | JAILBREAK | etc.
  hades_forwarded        bool          NOT NULL DEFAULT false,
  screened_at            timestamptz   NOT NULL DEFAULT now()
);

-- Adversarial threat catalog entries (maintained by Research & Red Team)
CREATE TABLE threat_catalog_entry (
  entry_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  threat_name            varchar(256)  NOT NULL,
  threat_category        varchar(64)   NOT NULL,
  pattern_hash           char(64)      NOT NULL UNIQUE,  -- SHA-256 of pattern
  detection_method       varchar(32)   NOT NULL CHECK (detection_method IN
                         ('REGEX','EMBEDDING','CLASSIFIER','HEURISTIC')),
  severity               varchar(16)   NOT NULL CHECK (severity IN
                         ('CRITICAL','HIGH','MEDIUM','LOW')),
  ic_specific            bool          NOT NULL DEFAULT true,
  source                 varchar(64)   NOT NULL CHECK (source IN
                         ('OWASP_LLM','IC_SPECIFIC','HADES_DERIVED',
                          'RESEARCH_RED_TEAM')),
  hades_event_ref        uuid,                          -- HADES AdversarialEvent if derived
  added_at               timestamptz   NOT NULL DEFAULT now(),
  active                 bool          NOT NULL DEFAULT true
);
```

---

## 12. MAS / EIDOLON — Media Authenticity Service

**Store:** PostgreSQL

```sql
-- Media authenticity assessment
CREATE TABLE authenticity_assessment (
  assessment_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  chunk_id               char(64)      NOT NULL,   -- SHA-256 — assessed chunk
  session_id             uuid          NOT NULL,
  turn_id                uuid          NOT NULL,
  media_type             varchar(32)   NOT NULL CHECK (media_type IN
                         ('IMAGE','VIDEO','AUDIO','DOCUMENT')),
  risk_level             varchar(16)   NOT NULL CHECK (risk_level IN
                         ('LOW','MEDIUM','HIGH','CRITICAL')),
  synthesis_probability  numeric(5,4)  NOT NULL,
  manipulation_indicators text[]       NOT NULL DEFAULT '{}',
  detection_methods      text[]        NOT NULL DEFAULT '{}',
  analyst_badge_shown    bool          NOT NULL DEFAULT false,
  hades_forwarded        bool          NOT NULL DEFAULT false,
  assessed_at            timestamptz   NOT NULL DEFAULT now()
);
```

---

## 13. MDS / KRONOS — Model Drift Service

**Store:** PostgreSQL

```sql
-- Model version registry
CREATE TABLE model_version_record (
  version_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  model_string           varchar(128)  NOT NULL UNIQUE,
  provider               varchar(64)   NOT NULL,
  training_cutoff        timestamptz,
  registered_at          timestamptz   NOT NULL DEFAULT now(),
  registered_by          char(64)      NOT NULL,
  active                 bool          NOT NULL DEFAULT true,
  deprecated_at          timestamptz,
  successor_version      varchar(128)
);

-- Drift assessment records
CREATE TABLE drift_assessment (
  assessment_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  model_version          varchar(128)  NOT NULL,
  domain                 varchar(64)   NOT NULL,
  interaction_class      varchar(64)   NOT NULL,
  drift_score            numeric(5,4)  NOT NULL,
  drift_detected         bool          NOT NULL DEFAULT false,
  calibration_stale      bool          NOT NULL DEFAULT false,
  prs_re_evaluation_needed bool        NOT NULL DEFAULT false,
  assessed_at            timestamptz   NOT NULL DEFAULT now()
);

-- MCP server version tracking
CREATE TABLE mcp_server_version (
  version_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  server_name            varchar(256)  NOT NULL,
  version_string         varchar(64)   NOT NULL,
  detected_at            timestamptz   NOT NULL DEFAULT now(),
  prior_version          varchar(64),
  behaviour_change_detected bool       NOT NULL DEFAULT false
);
```

---

## 14. SCBS / SENTINEL-CAP — Session Capability Bounding Service

**Store:** PostgreSQL · Redis (live ledger)

```sql
-- Capability envelopes (one per agent session)
CREATE TABLE capability_envelope (
  envelope_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_session_id       uuid          NOT NULL UNIQUE,
  analyst_session_id     uuid          NOT NULL,   -- parent PCES session
  declared_intent        text          NOT NULL,
  max_spend_tokens       bigint        NOT NULL,
  max_resource_scope     text[]        NOT NULL DEFAULT '{}',
  environment_designation varchar(16)  NOT NULL CHECK (environment_designation IN
                         ('SANDBOX','STAGING','PRODUCTION')),
  ttl_seconds            int           NOT NULL,
  created_at             timestamptz   NOT NULL DEFAULT now(),
  expires_at             timestamptz   NOT NULL,
  status                 varchar(16)   NOT NULL DEFAULT 'ACTIVE' CHECK (status IN
                         ('ACTIVE','SUSPENDED','TERMINATED','EXPIRED'))
);

-- Live spend ledger (also mirrored in Redis for hot path)
CREATE TABLE spend_ledger (
  ledger_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  envelope_id            uuid          NOT NULL UNIQUE REFERENCES capability_envelope,
  total_budget           bigint        NOT NULL,
  total_spent            bigint        NOT NULL DEFAULT 0,
  spend_rate_current     numeric(10,4),
  spend_rate_baseline    numeric(10,4),
  anomaly_flag           bool          NOT NULL DEFAULT false,
  updated_at             timestamptz   NOT NULL DEFAULT now()
);

-- Individual spend entries (append-only)
CREATE TABLE spend_entry (
  entry_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  ledger_id              uuid          NOT NULL REFERENCES spend_ledger,
  action_type            varchar(32)   NOT NULL CHECK (action_type IN
                         ('INFERENCE','RETRIEVAL','WRITE','READ','MCP_CALL')),
  resource               text          NOT NULL,
  tokens_consumed        bigint        NOT NULL,
  estimated_tokens       bigint        NOT NULL,
  action_id              uuid          NOT NULL,
  timestamp              timestamptz   NOT NULL DEFAULT now()
);

-- Escalation events
CREATE TABLE scbs_escalation_event (
  escalation_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  envelope_id            uuid          NOT NULL REFERENCES capability_envelope,
  trigger                varchar(32)   NOT NULL CHECK (trigger IN
                         ('SPEND_EXCEEDED','ANOMALOUS_RATE',
                          'TTL_EXCEEDED','SCOPE_VIOLATION','MANUAL')),
  trigger_detail         text          NOT NULL,
  escalated_to           char(64),
  resolution             varchar(16)   CHECK (resolution IN
                         ('APPROVED','TERMINATED','PENDING')),
  resolution_rationale   text,
  escalated_at           timestamptz   NOT NULL DEFAULT now(),
  resolved_at            timestamptz
);
```

**Redis live ledger:**
```
KEY:   scbs:ledger:{envelope_id}
VALUE: {budget: int, spent: int, remaining: int, anomaly: bool, updated_at: ISO8601}
TTL:   envelope TTL + 60 seconds
```

---

## 15. CBS / BROKER — Credential Broker Service

**Store:** PostgreSQL · Redis (active handle cache) · Vault (underlying credentials — never in PostgreSQL)

```sql
-- Capability handles (issued to agent sessions)
CREATE TABLE capability_handle (
  handle_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  handle_ref             char(64)      NOT NULL UNIQUE,  -- opaque token given to agent
  envelope_id            uuid          NOT NULL,          -- FK → SCBS capability_envelope
  service_name           varchar(128)  NOT NULL,
  surface_version        varchar(32)   NOT NULL,
  operations_allowed     text[]        NOT NULL,
  credential_vault_path  text          NOT NULL,          -- Vault path; NOT the credential
  issued_at              timestamptz   NOT NULL DEFAULT now(),
  expires_at             timestamptz   NOT NULL,
  revoked                bool          NOT NULL DEFAULT false,
  revoked_at             timestamptz,
  revocation_reason      text,
  call_count             int           NOT NULL DEFAULT 0,
  last_used              timestamptz
);

-- Capability surfaces (API owner attested; ARB approved)
CREATE TABLE capability_surface (
  surface_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  service_name           varchar(128)  NOT NULL,
  version                varchar(32)   NOT NULL,
  owner_team             varchar(128)  NOT NULL,
  owner_attestation      text          NOT NULL,
  arb_approval_ref       text,
  effective_from         timestamptz   NOT NULL,
  active                 bool          NOT NULL DEFAULT true,
  UNIQUE (service_name, version)
);

-- Operation definitions (part of a capability surface)
CREATE TABLE operation_definition (
  operation_id           uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  surface_id             uuid          NOT NULL REFERENCES capability_surface,
  name                   varchar(128)  NOT NULL,
  description            text          NOT NULL,
  idempotent             bool          NOT NULL DEFAULT true,
  requires_context       text[]        NOT NULL DEFAULT '{}',
  max_frequency_per_min  int
);

-- Proxied call records (append-only audit log)
CREATE TABLE call_record (
  call_id                uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  handle_id              uuid          NOT NULL REFERENCES capability_handle,
  envelope_id            uuid          NOT NULL,
  service_name           varchar(128)  NOT NULL,
  operation              varchar(128)  NOT NULL,
  request_metadata_hash  char(64)      NOT NULL,   -- SHA-256; content not stored
  response_status        int           NOT NULL,
  duration_ms            int           NOT NULL,
  timestamp              timestamptz   NOT NULL DEFAULT now()
);
```

---

## 16. RSS / ROLLBACK — State Snapshot Service

**Store:** PostgreSQL (snapshot index + rollback records) · Redis (HOT tier) · Object storage (COLD tier)

```sql
-- State snapshot metadata
CREATE TABLE state_snapshot (
  snapshot_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  envelope_id            uuid          NOT NULL,
  action_id              uuid          NOT NULL,
  action_type            varchar(32)   NOT NULL CHECK (action_type IN
                         ('WRITE','DELETE','MODIFY','CREATE')),
  action_description     text          NOT NULL,
  target_resource        text          NOT NULL,
  state_hash             char(64)      NOT NULL,   -- SHA-256 of pre-action state
  storage_tier           varchar(8)    NOT NULL CHECK (storage_tier IN
                         ('HOT','WARM','COLD')),
  storage_ref            text          NOT NULL,   -- path in storage tier
  created_at             timestamptz   NOT NULL DEFAULT now(),
  expires_at             timestamptz   NOT NULL,
  rolled_back            bool          NOT NULL DEFAULT false,
  rolled_back_at         timestamptz
);

-- Rollback records (append-only audit)
CREATE TABLE rollback_record (
  rollback_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  snapshot_id            uuid          NOT NULL REFERENCES state_snapshot,
  rollback_type          varchar(32)   NOT NULL CHECK (rollback_type IN
                         ('FULL_SESSION','ACTION_BOUNDARY')),
  initiated_by           char(64)      NOT NULL,   -- analyst_id_hash
  actions_reversed       uuid[]        NOT NULL DEFAULT '{}',
  conflict_detected      bool          NOT NULL DEFAULT false,
  conflict_description   text,
  status                 varchar(16)   NOT NULL CHECK (status IN
                         ('SUCCESS','PARTIAL','FAILED','CONFLICT_PENDING')),
  started_at             timestamptz   NOT NULL DEFAULT now(),
  completed_at           timestamptz
);
```

---

## 17. UCS / TYCHE — Uncertainty Characterisation Service

**Store:** PostgreSQL

```sql
-- Uncertainty profile per analytical claim
CREATE TABLE uncertainty_profile (
  profile_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id             uuid          NOT NULL,
  turn_id                uuid          NOT NULL,
  claim_id               uuid          NOT NULL,   -- FK → MOIRAI Claim
  primary_uncertainty_type varchar(32) NOT NULL CHECK (primary_uncertainty_type IN
                         ('aleatory','epistemic','model','uncharacterised')),
  aleatory_dominance     numeric(5,4)  NOT NULL DEFAULT 0.0,
  epistemic_dominance    numeric(5,4)  NOT NULL DEFAULT 0.0,
  model_dominance        numeric(5,4)  NOT NULL DEFAULT 0.0,
  temporal_horizon_flag  bool          NOT NULL DEFAULT false,
  collection_gap_signal  numeric(5,4)  NOT NULL DEFAULT 0.0,  -- from CGS/ARGUS-LACUNA
  cps_zone               varchar(16),   -- from CPS/APORIA if available
  tcs_confidence_signal  numeric(5,4),
  analyst_resolution_pending bool      NOT NULL DEFAULT true,
  profiled_at            timestamptz   NOT NULL DEFAULT now()
);

-- Analyst resolution of UCS three questions
CREATE TABLE analyst_resolution (
  resolution_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  profile_id             uuid          NOT NULL REFERENCES uncertainty_profile,
  session_id             uuid          NOT NULL,
  analyst_id_hash        char(64)      NOT NULL,
  question_1_answer      text,         -- Uncertainty type confirmation
  question_2_answer      text,         -- Reducibility assessment
  question_3_answer      text,         -- Collection gap characterisation
  taxonomy_override      varchar(32),  -- null if analyst accepted taxonomy
  override_rationale     text,
  resolved_at            timestamptz   NOT NULL DEFAULT now()
);
```

---

## 18. DPS / CODEX — Document Provenance Service

**Store:** PostgreSQL

```sql
-- Analytical product document records
CREATE TABLE document_record (
  document_id            uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  title                  text          NOT NULL,
  classification         varchar(64)   NOT NULL,
  compartment            varchar(64),
  requirement_ref        text,
  ai_assisted            bool          NOT NULL DEFAULT true,
  session_ids            uuid[]        NOT NULL DEFAULT '{}',
  version                int           NOT NULL DEFAULT 1,
  status                 varchar(32)   NOT NULL DEFAULT 'DRAFT' CHECK (status IN
                         ('DRAFT','REVIEW','FINALIZED','DISSEMINATED','SUPERSEDED','RECALLED')),
  created_at             timestamptz   NOT NULL DEFAULT now(),
  finalized_at           timestamptz,
  finalized_by           char(64)
);

-- AI usage disclosure (appended at dissemination)
CREATE TABLE ai_usage_disclosure (
  disclosure_id          uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  document_id            uuid          NOT NULL REFERENCES document_record,
  session_count          int           NOT NULL,
  claim_count            int           NOT NULL,
  grnd_claim_count       int           NOT NULL,
  param_claim_count      int           NOT NULL,
  synth_claim_count      int           NOT NULL,
  verification_rate      numeric(5,4)  NOT NULL,
  unverified_count       int           NOT NULL,
  model_versions_used    text[]        NOT NULL DEFAULT '{}',
  moirai_chain_hash      char(64)      NOT NULL,
  generated_at           timestamptz   NOT NULL DEFAULT now()
);

-- Dissemination records
CREATE TABLE dissemination_record (
  dissemination_id       uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  document_id            uuid          NOT NULL REFERENCES document_record,
  version                int           NOT NULL,
  channel                varchar(64)   NOT NULL,
  recipient_classification varchar(64) NOT NULL,
  pcs_package_id         uuid,         -- FK → PCS ConsumerPackage if generated
  disseminated_at        timestamptz   NOT NULL DEFAULT now(),
  disseminated_by        char(64)      NOT NULL
);
```

---

## 19. OFS / NEMESIS — Outcome Feedback Service

**Store:** PostgreSQL

```sql
-- Outcome events (when real-world intelligence confirms or disconfirms an assessment)
CREATE TABLE outcome_record (
  outcome_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  document_id            uuid          NOT NULL,   -- FK → DPS document_record
  session_ids            uuid[]        NOT NULL DEFAULT '{}',
  requirement_ref        text,
  domain                 varchar(64)   NOT NULL,
  claim_types            text[]        NOT NULL DEFAULT '{}',
  outcome_classification varchar(32)   NOT NULL CHECK (outcome_classification IN
                         ('CONFIRMED','DISCONFIRMED','PARTIALLY_CONFIRMED',
                          'AMBIGUOUS','UNOBSERVABLE')),
  outcome_confidence     varchar(16)   NOT NULL CHECK (outcome_confidence IN
                         ('HIGH','MEDIUM','LOW')),
  outcome_date           timestamptz,
  outcome_evidence_ref   text,
  classified_by          char(64)      NOT NULL,   -- analyst or supervisor
  supervisor_confirmed   bool          NOT NULL DEFAULT false,
  confirmed_at           timestamptz,
  tcs_update_triggered   bool          NOT NULL DEFAULT false,
  fgts_correction_ref    uuid,
  recorded_at            timestamptz   NOT NULL DEFAULT now()
);
```

---

## 20. ERAS / LOGOS — Reasoning Audit Service

**Store:** Elasticsearch

```json
// Index: themis.eras.reasoning-captures
{
  "mappings": {
    "properties": {
      "capture_id":           { "type": "keyword" },
      "turn_id":              { "type": "keyword" },
      "session_id":           { "type": "keyword" },
      "classification":       { "type": "keyword" },
      "compartment":          { "type": "keyword" },
      "interaction_class":    { "type": "keyword" },
      "domain":               { "type": "keyword" },
      "requirement_ref":      { "type": "keyword" },
      "model_version":        { "type": "keyword" },
      "chain_of_thought":     { "type": "text", "analyzer": "english" },
      "alternatives":         { "type": "text" },
      "evidence_citations":   { "type": "keyword" },
      "unsupported_claim_count": { "type": "integer" },
      "completeness_score":   { "type": "float" },
      "adversarial_session":  { "type": "boolean" },
      "hades_forwarded":      { "type": "boolean" },
      "prev_event_hash":      { "type": "keyword" },
      "signature":            { "type": "keyword" },
      "recorded_at":          { "type": "date" },

      "claims": {
        "type": "nested",
        "properties": {
          "claim_id":               { "type": "keyword" },
          "text_hash":              { "type": "keyword" },
          "claim_type":             { "type": "keyword" },
          "source_badge":           { "type": "keyword" },
          "supporting_chunks":      { "type": "keyword" },
          "unsupported":            { "type": "boolean" },
          "cvs_status":             { "type": "keyword" },
          "tvs_score_at_capture":   { "type": "float" },
          "ucs_uncertainty_type":   { "type": "keyword" }
        }
      },

      "confidence_signals": {
        "type": "nested",
        "properties": {
          "level":    { "type": "keyword" },
          "span":     { "type": "text" },
          "span_pos": { "type": "integer_range" }
        }
      }
    }
  }
}
```

---

## 21. HADES — Adversarial Intelligence Repository

**Stores:** PostgreSQL (metadata + technique entries) · Elasticsearch air-gapped indices (adversarial content — access restricted to Research & Red Team and IOB)

> **Access:** Research & Red Team and IOB only. No service account access. No analyst access. Air-gapped Elasticsearch indices are network-isolated from the main THEMIS infrastructure.

```sql
-- Adversarial event metadata (no content stored plain)
CREATE TABLE adversarial_event (
  event_id               uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  source_service         varchar(32)   NOT NULL CHECK (source_service IN
                         ('IAS_SCUDO','ERAS_LOGOS','CVS_VERITAS',
                          'MAS_EIDOLON','ADS_CASSANDRA','WSF_LACHESIS')),
  source_event_id        uuid          NOT NULL,
  session_id             uuid,
  event_type             varchar(64)   NOT NULL CHECK (event_type IN
                         ('PROMPT_INJECTION_BLOCKED',
                          'PROMPT_INJECTION_BYPASSED',
                          'CORPUS_POISONING_DETECTED',
                          'INDIRECT_INJECTION_BLOCKED',
                          'ADVERSARIAL_REASONING_RECORD',
                          'REASONING_MANIPULATION_DETECTED',
                          'REASONING_MANIPULATION_BYPASSED',
                          'FABRICATED_CITATION_DETECTED',
                          'CORPUS_POISONING_INDICATOR',
                          'SYNTHETIC_MEDIA_HIGH_RISK',
                          'SYNTHETIC_MEDIA_CONFIRMED',
                          'ANOMALY_ADVERSARIAL_CONFIRMED',
                          'ADVERSARIAL_SIGNAL_FUSION')),
  technique_category     varchar(128),
  severity               varchar(16)   NOT NULL CHECK (severity IN
                         ('CRITICAL','HIGH','MEDIUM','LOW')),
  outcome                varchar(32)   NOT NULL CHECK (outcome IN
                         ('BLOCKED','BYPASSED','PARTIALLY_BYPASSED','UNKNOWN')),
  content_hash           char(64)      NOT NULL,   -- SHA-256; content in air-gapped ES
  classification         varchar(64)   NOT NULL,
  analysis_notes         text,
  ingested_at            timestamptz   NOT NULL DEFAULT now()
);

-- Adversarial reasoning records (crown jewel — AI chain-of-thought during adversarial session)
CREATE TABLE adversarial_reasoning_record (
  record_id              uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  event_id               uuid          NOT NULL REFERENCES adversarial_event,
  eras_capture_id        uuid          NOT NULL,
  adversarial_content_type varchar(64) NOT NULL,
  detected               bool          NOT NULL,
  detection_service      varchar(32),
  chain_of_thought_hash  char(64)      NOT NULL,   -- SHA-256; in air-gapped ES
  manipulation_mechanism text,                     -- R&RT annotation
  model_version          varchar(128)  NOT NULL
);

-- Adversarial technique taxonomy
CREATE TABLE technique_entry (
  technique_id           uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  technique_name         varchar(256)  NOT NULL,
  technique_category     varchar(64)   NOT NULL,
  first_observed         timestamptz   NOT NULL,
  last_observed          timestamptz   NOT NULL,
  event_count            int           NOT NULL DEFAULT 1,
  bypass_rate            numeric(5,4)  NOT NULL DEFAULT 0.0,
  description            text          NOT NULL,
  ias_catalog_ref        uuid,         -- FK → IAS threat_catalog_entry if published
  UNIQUE (technique_name)
);

-- Model failure modes derived from adversarial analysis
CREATE TABLE failure_mode (
  failure_id             uuid          PRIMARY KEY DEFAULT gen_random_uuid(),
  model_version          varchar(128)  NOT NULL,
  domain                 varchar(64),
  claim_type             varchar(64),
  failure_description    text          NOT NULL,
  evidence_event_count   int           NOT NULL DEFAULT 0,
  first_documented       timestamptz   NOT NULL DEFAULT now(),
  cps_zone_impact        text,
  remediation_status     varchar(32)   NOT NULL DEFAULT 'OPEN' CHECK (remediation_status IN
                         ('OPEN','ADDRESSED','ACCEPTED_RISK','MONITORING'))
);
```

**Air-gapped Elasticsearch indices (Research & Red Team terminal access only):**
```json
// Index: hades.adversarial-content  (air-gapped; no external access)
{
  "mappings": {
    "properties": {
      "event_id":            { "type": "keyword" },
      "event_type":          { "type": "keyword" },
      "technique_category":  { "type": "keyword" },
      "outcome":             { "type": "keyword" },
      "content_text":        { "type": "text", "analyzer": "english" },
      "source_service":      { "type": "keyword" },
      "session_id":          { "type": "keyword" },
      "model_version":       { "type": "keyword" },
      "classification":      { "type": "keyword" },
      "ingested_at":         { "type": "date" }
    }
  }
}

// Index: hades.reasoning-records  (air-gapped; separately classified)
{
  "mappings": {
    "properties": {
      "record_id":                 { "type": "keyword" },
      "event_id":                  { "type": "keyword" },
      "model_version":             { "type": "keyword" },
      "detected":                  { "type": "boolean" },
      "adversarial_content_type":  { "type": "keyword" },
      "chain_of_thought":          { "type": "text", "analyzer": "english" },
      "manipulation_mechanism":    { "type": "text" },
      "session_id":                { "type": "keyword" },
      "ingested_at":               { "type": "date" }
    }
  }
}
```

---

## 22. Kafka Event Schemas

All inter-service events are published to Kafka. Topics follow the pattern `themis.events.{namespace}.{event-type}`. Every event carries a MOIRAI-signed provenance envelope.

### 22.1 MOIRAI Provenance Envelope (present on every event)

```json
{
  "event_id":         "uuid",
  "service_id":       "PCES | PGS | FGS | TCS | ...",
  "session_id":       "uuid | null",
  "classification":   "string",
  "prev_event_hash":  "sha256",
  "signature":        "hmac-sha256",
  "timestamp":        "ISO8601",
  "schema_version":   "2.0"
}
```

*All events below carry this envelope. Fields shown are the event-specific payload.*

---

### 22.2 Session and Classification Events

```json
// themis.events.gates.session-granted
{
  "event_type":         "PCES_SESSION_GRANTED",
  "session_id":         "uuid",
  "analyst_id_hash":    "sha256",
  "session_type":       "ANALYTICAL | RESEARCH | AGENT | STOA",
  "classification":     "string",
  "compartments":       ["string"],
  "requirement_ref":    "string | null",
  "expires_at":         "ISO8601"
}

// themis.events.gates.session-revoked
{
  "event_type":         "PCES_SESSION_REVOKED",
  "session_id":         "uuid",
  "reason":             "string"
}

// themis.events.gates.compartment-denied
{
  "event_type":         "PCES_COMPARTMENT_DENIED",
  "session_id":         "uuid",
  "resource_classification": "string",
  "decision":           "DENY | DOWNGRADE"
}
```

### 22.3 Policy and Screening Events

```json
// themis.events.gates.screening-complete
{
  "event_type":         "PGS_SCREENING_COMPLETE",
  "turn_id":            "uuid",
  "session_id":         "uuid",
  "screen_type":        "INPUT | OUTPUT | RETRIEVAL",
  "outcome":            "PASS | WARN | BLOCK | FLAG",
  "interaction_class":  "string",
  "rules_triggered":    ["uuid"],
  "icd_203_compliant":  true
}

// themis.events.gates.output-blocked
{
  "event_type":         "PGS_OUTPUT_BLOCKED",
  "turn_id":            "uuid",
  "session_id":         "uuid",
  "rule_ids":           ["uuid"],
  "reason":             "string"
}
```

### 22.4 Provenance and Turn Events

```json
// themis.events.core.turn-initiated
{
  "event_type":           "TURN_INITIATED",
  "turn_id":              "uuid",
  "session_id":           "uuid",
  "analyst_id_hash":      "sha256",
  "interaction_class":    "string",
  "requirement_ref":      "string | null",
  "retrieval_chunk_ids":  ["sha256"],
  "prompt_version_hash":  "sha256 | null"
}

// themis.events.core.turn-completed
{
  "event_type":           "TURN_COMPLETED",
  "turn_id":              "uuid",
  "output_chunk_id":      "sha256",
  "input_token_count":    1200,
  "output_token_count":   340,
  "model_version":        "string",
  "model_knowledge_cutoff": "ISO8601 | null",
  "retrieval_freshness":  0.82,
  "claim_count":          4,
  "unsupported_count":    1
}

// themis.events.core.claim-extracted
{
  "event_type":           "ERAS_CLAIM_EXTRACTED",
  "capture_id":           "uuid",
  "turn_id":              "uuid",
  "session_id":           "uuid",
  "claim_count":          4,
  "unsupported_count":    1,
  "completeness_score":   0.78,
  "adversarial_session":  false
}
```

### 22.5 Financial Governance Events

```json
// themis.events.quality.token-transaction
{
  "event_type":           "FGS_TOKEN_TRANSACTION",
  "transaction_id":       "uuid",
  "account_id":           "uuid",
  "session_id":           "uuid",
  "tokens_consumed":      1450,
  "interaction_class":    "string",
  "utilisation_pct":      0.67
}

// themis.events.quality.account-approaching
{
  "event_type":           "FGS_ACCOUNT_APPROACHING_LIMIT",
  "account_id":           "uuid",
  "account_name":         "string",
  "utilisation_pct":      0.82,
  "remaining_tokens":     41000,
  "period_string":        "2026-Q2"
}

// themis.events.quality.reserve-approved
{
  "event_type":           "FGS_RESERVE_APPROVED",
  "request_id":           "uuid",
  "account_id":           "uuid",
  "tokens_granted":       50000
}
```

### 22.6 Feedback and Correction Events

```json
// themis.events.quality.correction
{
  "event_type":           "FGTS_CORRECTION",
  "correction_id":        "uuid",
  "turn_id":              "uuid",
  "chunk_id":             "sha256",
  "analyst_id_hash":      "sha256",
  "correction_action":    "CONFIRMED | INDEPENDENTLY_VERIFIED | MISREPRESENTS_SOURCE | ...",
  "interaction_class":    "string",
  "domain":               "string",
  "composite_weight":     0.84
}

// themis.events.quality.corpus-versioned
{
  "event_type":           "FGTS_CORPUS_VERSIONED",
  "corpus_version_id":    "uuid",
  "version_number":       42,
  "entry_count":          8734,
  "weighted_entry_count": 6201.4
}

// themis.events.quality.performance-update  (FGTS → TCS)
{
  "event_type":           "FGTS_AI_PERFORMANCE_UPDATE",
  "turn_id":              "uuid",
  "interaction_class":    "string",
  "domain":               "string",
  "correction_action":    "string",
  "composite_weight":     0.84,
  "analyst_id_hash":      "sha256"
}

// themis.events.quality.retrieval-miss  (FGTS → RQS)
{
  "event_type":           "FGTS_RETRIEVAL_MISS",
  "correction_id":        "uuid",
  "query_turn_id":        "uuid",
  "session_id":           "uuid",
  "domain":               "string",
  "source_type_missing":  "HUMINT | SIGINT | ...",
  "miss_pattern":         "domain_gap | collection_type_gap | ..."
}
```

### 22.7 Validity and Currency Events

```json
// themis.events.quality.validity-updated
{
  "event_type":           "TVS_VALIDITY_UPDATED",
  "chunk_id":             "sha256",
  "new_score":            0.61,
  "previous_score":       0.79,
  "trigger_type":         "kcs_supersession | scheduled_decay | ...",
  "trigger_id":           "uuid | null"
}

// themis.events.quality.source-invalidated
{
  "event_type":           "TVS_SOURCE_INVALIDATED",
  "source_id":            "uuid",
  "chunk_ids_affected":   ["sha256"],
  "blast_radius_count":   14,
  "invalidation_basis":   "string"
}

// themis.events.quality.kcs-supersession  (KCS → TVS)
{
  "event_type":           "KCS_SUPERSESSION_REQUEST",
  "kcs_event_id":         "uuid",
  "affected_chunks":      [
    {"chunk_id": "sha256", "impact_score": 0.92, "impact_type": "supersession"}
  ],
  "development_type":     "new_collection | source_superseded | ...",
  "confidence":           0.94
}
```

### 22.8 Adversarial and Quality Events

```json
// themis.events.quality.ias-blocked
{
  "event_type":           "IAS_INPUT_BLOCKED",
  "session_id":           "uuid",
  "turn_id":              "uuid",
  "screen_target":        "ANALYST_INPUT | RETRIEVED_CHUNK | MCP_RESPONSE",
  "threat_category":      "PROMPT_INJECTION | INDIRECT_INJECTION | ...",
  "adversarial_score":    0.94
}

// themis.events.quality.cvs-fabrication
{
  "event_type":           "CVS_FABRICATION_DETECTED",
  "session_id":           "uuid",
  "claim_id":             "uuid",
  "fabrication_type":     "HALLUCINATED_CITATION | MISATTRIBUTED_CONTENT | ...",
  "severity":             "CRITICAL | HIGH | MEDIUM | LOW"
}

// themis.events.quality.model-version-changed
{
  "event_type":           "MDS_MODEL_VERSION_CHANGED",
  "prior_version":        "string",
  "new_version":          "string",
  "drift_detected":       true,
  "calibration_stale_count": 847,
  "prs_re_evaluation_needed": true
}
```

### 22.9 Agent-Native Events

```json
// themis.events.agent.session-created
{
  "event_type":           "SCBS_SESSION_CREATED",
  "envelope_id":          "uuid",
  "analyst_session_id":   "uuid",
  "declared_intent":      "string",
  "max_spend_tokens":     50000,
  "environment":          "SANDBOX | STAGING | PRODUCTION"
}

// themis.events.agent.spend-exceeded
{
  "event_type":           "SCBS_SPEND_EXCEEDED",
  "envelope_id":          "uuid",
  "limit_type":           "SPEND | SCOPE | TTL",
  "value_at_trigger":     50100,
  "max_allowed":          50000
}

// themis.events.agent.session-terminated
{
  "event_type":           "SCBS_SESSION_TERMINATED",
  "envelope_id":          "uuid",
  "termination_reason":   "string",
  "total_spend":          47832,
  "write_actions":        3,
  "escalations":          0
}

// themis.events.agent.handle-issued
{
  "event_type":           "CBS_HANDLE_ISSUED",
  "handle_id":            "uuid",
  "envelope_id":          "uuid",
  "service_name":         "string",
  "operations_granted":   ["string"],
  "expires_at":           "ISO8601"
}

// themis.events.agent.scope-violation
{
  "event_type":           "CBS_SCOPE_VIOLATION",
  "handle_id":            "uuid",
  "attempted_operation":  "string",
  "allowed_operations":   ["string"]
}

// themis.events.agent.snapshot-created
{
  "event_type":           "RSS_SNAPSHOT_CREATED",
  "snapshot_id":          "uuid",
  "action_id":            "uuid",
  "action_type":          "WRITE | DELETE | MODIFY | CREATE",
  "state_hash":           "sha256",
  "expires_at":           "ISO8601"
}
```

### 22.10 Uncertainty and Outcome Events

```json
// themis.events.core.uncertainty-profiled
{
  "event_type":             "UCS_PROFILE_CREATED",
  "profile_id":             "uuid",
  "session_id":             "uuid",
  "claim_id":               "uuid",
  "primary_uncertainty_type": "aleatory | epistemic | model | uncharacterised",
  "analyst_resolution_pending": true
}

// themis.events.core.outcome-classified
{
  "event_type":             "OFS_OUTCOME_CLASSIFIED",
  "outcome_id":             "uuid",
  "document_id":            "uuid",
  "requirement_ref":        "string | null",
  "domain":                 "string",
  "outcome_classification": "CONFIRMED | DISCONFIRMED | PARTIALLY_CONFIRMED | AMBIGUOUS",
  "outcome_confidence":     "HIGH | MEDIUM | LOW"
}
```

---

## 23. Cross-Service Reference Types

Standard types and identifiers used consistently across all THEMIS services.

### 23.1 Core Identifiers

| Field | Type | Authoritative source | Notes |
|---|---|---|---|
| `session_id` | uuid | PCES/AEGIS | Issued at session grant; scopes all activity in that session |
| `analyst_id_hash` | char(64) SHA-256 | PCES/AEGIS | One-way hash of IAM identity; mapping lives in PCES only |
| `chunk_id` | char(64) SHA-256 | Content-addressed | Same content = same ID across all services |
| `turn_id` | uuid | API gateway | Generated at TurnInitiated; links all per-turn records |
| `claim_id` | uuid | ERAS/LOGOS | Extracted from TurnRecord by ERAS |
| `envelope_id` | uuid | SCBS | Scopes all agent session capability records |
| `requirement_ref` | str | External requirements system | Opaque reference; THEMIS does not own the requirement lifecycle |
| `moirai_event_id` | uuid | MOIRAI | Links any service record to the MOIRAI chain entry |

### 23.2 Enumerations

**source_type (IC collection disciplines):**
```
SIGINT | HUMINT | GEOINT | OSINT | TECHINT | IMINT | MASINT | FININT | CYBER | OPEN_SOURCE
```

**interaction_class (IC analytical):**
```
capability_assessment | intent_analysis | order_of_battle | technical_assessment |
strategic_warning | collection_gap | evidence_synthesis | requirement_decomposition |
single_source_query | multi_source_analysis | administrative
```

**claim_type (source basis):**
```
GRND    — retrieved from and supported by accessible corpus source
PARAM   — from model parametric knowledge; no specific retrieved source
SYNTH   — analytical synthesis combining GRND and PARAM elements
```

**correction_action:**
```
CONFIRMED | INDEPENDENTLY_VERIFIED | MISREPRESENTS_SOURCE |
FACTUALLY_INCORRECT | OUTDATED | OUT_OF_SCOPE | UNSUPPORTED |
PARTIALLY_CORRECT | CANNOT_VERIFY
```

**calibration_state:**
```
PRIOR_ONLY    — fewer than 30 weighted corrections in this domain; no meaningful posterior
PROVISIONAL   — 30–99 corrections; posterior forming but statistically weak
CALIBRATED    — ≥ 100 corrections; posterior reliable for confidence signalling
FLAGGED       — gaming probability elevated; posterior under review
SUSPENDED     — calibration suspended pending supervisor review
```

**capability_zone (from CPS/APORIA — proposed):**
```
green       — model reliably capable on this claim type in this domain
amber       — model demonstrates meaningful variance; additional scrutiny warranted
red         — model demonstrates systematic error; hard confidence ceiling applied
unassessed  — no evaluation benchmark completed for this combination
```

**uncertainty_type (UCS/TYCHE):**
```
aleatory    — inherent uncertainty; the adversary or system may not have decided
epistemic   — collection gap; better intelligence could reduce this uncertainty
model       — this claim type is at the boundary of AI reliable capability
uncharacterised — analyst has not yet resolved the three UCS questions
```

**adversarial_event_type (HADES taxonomy):**
```
PROMPT_INJECTION_BLOCKED | PROMPT_INJECTION_BYPASSED |
CORPUS_POISONING_DETECTED | INDIRECT_INJECTION_BLOCKED |
ADVERSARIAL_REASONING_RECORD | REASONING_MANIPULATION_DETECTED |
REASONING_MANIPULATION_BYPASSED | FABRICATED_CITATION_DETECTED |
SYNTHETIC_MEDIA_HIGH_RISK | SYNTHETIC_MEDIA_CONFIRMED |
ANOMALY_ADVERSARIAL_CONFIRMED | ADVERSARIAL_SIGNAL_FUSION
```

**environment_designation (SCBS):**
```
SANDBOX | STAGING | PRODUCTION
```

**session_type (PCES):**
```
ANALYTICAL | RESEARCH | AGENT | STOA
```

### 23.3 Removed Identifiers (Law Firm Context — Not Used)

The following identifiers from the prior schema do not exist in the IC intelligence platform:

| Removed | Replaced by | Notes |
|---|---|---|
| `matter_id` | `session_id` + `requirement_ref` | No matter-management system |
| `client_id` | — | No external client relationship |
| `data_jurisdiction` | `classification` + `compartment` | IC classification system replaces jurisdictional data governance |
| `privilege_type` | — | No attorney-client privilege in IC context |
| `consent_record` | — | No client AI training consent |
| `consent_level` | — | Removed entirely |
| `billing_model` (passthrough/markup/absorbed) | `tokenomics_model` | Internal token economics; no client billing |
| `rate_card` (dollar amounts) | `tokenomics_model.interaction_class_weights` | Token weights, not dollar rates |
| `matter_budget` | `token_account` | Token allocations, not matter budgets |
| `hold_record` / `hold_resolution` | — | Legal review hold system removed entirely |
| `elo_rating` / `elo_history` | `calibration_posterior` (Bayesian) | Elo replaced with Beta-distribution Bayesian model |

### 23.4 MOIRAI Chain Integrity Rules

All events emitted to MOIRAI must satisfy:
1. `prev_event_hash` must equal the SHA-256 of the immediately preceding MOIRAI event
2. `signature` must be a valid HMAC-SHA256 computed with the emitting service's Vault-stored signing key
3. `event_hash` (stored in `moirai_event`) must equal SHA-256 of the full event payload including the signing fields
4. Events older than 24 hours must have a `tsa_anchor_ref` from an RFC 3161 timestamp authority

Chain integrity is verified by the MOIRAI integrity monitor on a continuous basis. Any break in the chain is a P0 incident.

---

*THEMIS Data Models v2.0 · IC Intelligence Platform · 21 services documented*
*Supersedes v1.0 (law firm schema)*
*Maintained by: THEMIS Platform Team · Changes require PRD update + this document update in same PR*
