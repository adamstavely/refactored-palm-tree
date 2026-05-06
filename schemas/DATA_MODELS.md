# THEMIS Data Models

Single authoritative reference for every schema in the THEMIS platform. Organized by service and data store. All inter-service contracts (Kafka event schemas) are in the Events section.

When a schema changes, this document is updated in the same PR. Schema version history is tracked in [CHANGELOG.md](../CHANGELOG.md).

---

## Table of Contents

1. [Provenance Service / MOIRAI (Neo4j)](#provenance-service--moirai)
2. [Trust Calibration Service / MIMIR (PostgreSQL)](#trust-calibration-service--mimir)
3. [Financial Governance Service / PLUTUS (PostgreSQL)](#financial-governance-service--plutus)
4. [Feedback & Ground Truth Service / ALETHEIA (PostgreSQL)](#feedback--ground-truth-service--aletheia)
5. [Temporal Validity Service / KAIROS (PostgreSQL + Redis)](#temporal-validity-service--kairos)
6. [Retrieval Quality Service / HERMES (PostgreSQL)](#retrieval-quality-service--hermes)
7. [Knowledge Currency Service / ARGUS (PostgreSQL)](#knowledge-currency-service--argus)
8. [Privilege & Consent Enforcement Service / AEGIS (PostgreSQL)](#privilege--consent-enforcement-service--aegis)
9. [Policy & Guardrails Service / NOMOS (PostgreSQL)](#policy--guardrails-service--nomos)
10. [Reasoning Audit Service / LOGOS (Elasticsearch)](#reasoning-audit-service--logos)
11. [Kafka Event Schemas](#kafka-event-schemas)
12. [Cross-Service Reference Types](#cross-service-reference-types)

---

## Provenance Service / MOIRAI

**Store:** Neo4j graph database
**Source of truth:** Append-only Kafka event log (Neo4j is a derived view; can be rebuilt from events)

### Node Types

#### EvidenceSource
```
Node: EvidenceSource
Properties:
  source_id          uuid          PK
  dms_document_id    str           FK → iManage/NetDocuments
  matter_id          uuid          FK → matter management system
  data_jurisdiction  str           eu | uk | us | apac
  ingested_at        datetime
  ingested_by        uuid          pseudonymised user_id (IAM)
  source_type        str           document | video | email_thread | chat | image
  title              str
  file_hash          sha256
```

#### Chunk
```
Node: Chunk
Properties:
  chunk_id           sha256        PK — content-addressed identity
  lsh_fingerprint    str           MinHash (k=5, n=128)
  source_id          uuid          FK → EvidenceSource
  matter_id          uuid
  data_jurisdiction  str
  modality           str           document | video | email | chat | image
  privilege_type     str           attorney_client | work_product | common_interest
                                   | third_party_confidential | no_privilege
  watermark_key_id   uuid | null   FK → watermark key in Vault (if watermarked)

  # Document modality fields
  page_number        int | null
  bounding_box       jsonb | null  {x1, y1, x2, y2}
  reading_order_seq  int | null
  section_label      str | null
  content_type       str | null    body | heading | footnote | caption | table_cell

  # Video/audio modality fields
  timecode_start     float | null
  timecode_end       float | null
  speaker_id         str | null

  # Email/chat modality fields
  thread_id          str | null
  message_position   int | null
  sender_id          str | null    pseudonymised
  message_timestamp  datetime | null
```

#### TurnRecord
```
Node: TurnRecord
Properties:
  turn_id                uuid       PK
  session_id             uuid       FK → AISession
  turn_index             int
  role                   str        user | assistant | tool
  timestamp              datetime
  matter_id              uuid
  data_jurisdiction      str

  # Prompt lineage links
  analyst_input_id       uuid | null   FK → AnalystInput
  prompt_assembly_id     uuid | null   FK → PromptAssembly
  system_prompt_id       uuid | null   FK → SystemPromptRecord
  prompt_template_id     uuid | null   FK → PromptTemplate

  # Context
  retrieval_chunk_ids    uuid[]
  prior_turn_ids         uuid[]
  input_token_count      int
  output_token_count     int
  output_chunk_id        uuid | null   FK → OutputChunk

  # Provenance status
  status                 str        complete | incomplete | hold_pending | hold_resolved
  hold_record_id         uuid | null
  branch_parent          uuid | null
  sub_steps              uuid[]

  # Watermarking
  watermark_key_id       uuid | null
```

#### PromptTemplate
```
Node: PromptTemplate
Properties:
  template_id        uuid       PK
  version            str        semver
  interaction_class  str
  base_text          str
  assembly_recipe    jsonb      {dynamic_components: [...]}
  approved_by        uuid       user_id (AI Governance Committee member)
  approved_at        datetime
  pgs_rule_ids       uuid[]
  effective_from     datetime
  effective_until    datetime | null
  deprecated         bool
```

#### AnalystInput
```
Node: AnalystInput
Properties:
  input_id           uuid       PK
  turn_id            uuid       FK → TurnRecord
  raw_text           str        nullified after matter_close + 30 days
  input_type         str        text | text_with_attachment | highlighted_passage
  attachments        uuid[]     FK → Chunk (attached evidence)
  interface_state    jsonb      {open_document_tabs, foregrounded_doc, selected_text}
  pgs_classification str
  pgs_evaluation_id  uuid
  matter_id          uuid
  data_jurisdiction  str
```

#### PromptAssembly
```
Node: PromptAssembly
Properties:
  assembly_id               uuid      PK
  turn_id                   uuid      FK → TurnRecord
  system_prompt_id          uuid      FK → SystemPromptRecord
  analyst_input_id          uuid      FK → AnalystInput
  retrieval_chunk_ids       uuid[]    FK → Chunk[]
  assembled_messages_hash   sha256
  token_count               int
  assembly_timestamp        datetime
  pgs_rule_summary          jsonb
  privilege_filter_applied  bool
```

#### OutputChunk
```
Node: OutputChunk
Properties:
  chunk_id           sha256     PK — content-addressed
  turn_id            uuid       FK → TurnRecord
  lsh_fingerprint    str
  watermark_key_id   uuid | null
  matter_id          uuid
  data_jurisdiction  str
```

#### LegalDocument
```
Node: LegalDocument
Properties:
  doc_id                uuid      PK
  matter_id             uuid
  title                 str
  version               int
  finalized_at          datetime
  finalized_by          uuid      pseudonymised user_id
  certification_hash    sha256
  provenance_snapshot   sha256    hash of graph state at certification
  data_jurisdiction     str
```

#### RetrievalTrajectory (Q-RAG extension)
```
Node: RetrievalTrajectory
Properties:
  trajectory_id      uuid      PK
  turn_id            uuid      FK → TurnRecord
  total_steps        int
  terminal_chunks    uuid[]    FK → Chunk[] — final set passed to model
  steps              jsonb     [{step_index, query_embedding_hash, retrieved_chunks,
                                 step_score, informed_by_step}]
```

#### NullificationRecord
```
Node: NullificationRecord
Properties:
  nullification_id      uuid      PK
  node_id               uuid      FK → nullified node
  node_type             str
  nullified_fields      str[]
  nullification_basis   str       gdpr_erasure | matter_close | retention_expiry | court_order
  nullified_at          datetime
  authorised_by         uuid
  downstream_notified   bool
```

### Edge Types

| Edge Label | From | To | Properties |
|---|---|---|---|
| INGEST | EvidenceSource | Chunk | `ingested_at`, `chunk_sequence` |
| RETRIEVED | TurnRecord | Chunk | `retrieval_rank`, `similarity_score`, `validity_at_retrieval` |
| CONSUMED | TurnRecord | OutputChunk | `prior context — assistant message` |
| PRODUCED | TurnRecord | OutputChunk | `generation_timestamp` |
| CITED_IN | OutputChunk/Chunk | DocumentSection | `paste_timestamp`, `detection_method`, `confidence` |
| COMPILED_INTO | DocumentSection | LegalDocument | `section_order` |
| GOVERNED_BY | TurnRecord | PromptTemplate | `version_at_use` |
| ASSEMBLED_FROM | PromptAssembly | PromptTemplate | — |
| ASSEMBLED_FROM | PromptAssembly | AnalystInput | — |
| INJECTED_WITH | PromptAssembly | Chunk | `injection_order`, `token_count` |
| EXPRESSED_AS | AnalystInput | TurnRecord | — |
| ACTIVE_AT | PromptTemplate | TurnRecord | `effective_from` |
| BUILT_FROM | TurnRecord | PromptAssembly | — |
| TRAJECTORY_FOR | RetrievalTrajectory | TurnRecord | — |
| FOLLOWS | TurnRecord | TurnRecord | `session_id`, `turn_index` — intra-session DAG |
| BRANCHES_FROM | TurnRecord | TurnRecord | `branch_timestamp` |
| NULLIFIED | NullificationRecord | (any node) | `nullified_at`, `basis` |

---

## Trust Calibration Service / MIMIR

**Store:** PostgreSQL

```sql
-- Per-analyst calibration record
CREATE TABLE analyst_calibration (
  analyst_id          uuid         NOT NULL,
  interaction_class   varchar(64)  NOT NULL,
  matter_type         varchar(64)  NOT NULL,
  rai_score           numeric(5,4) NOT NULL,
  calibration_error   numeric(5,4) NOT NULL,
  calibration_score   numeric(5,4) NOT NULL,
  status              varchar(32)  NOT NULL CHECK (status IN
                      ('calibrated','over_relying','under_relying')),
  intervention_routed bool         NOT NULL DEFAULT false,
  last_updated        timestamptz  NOT NULL,
  PRIMARY KEY (analyst_id, interaction_class, matter_type)
);

-- Score history (append-only)
CREATE TABLE calibration_history (
  id                  bigserial    PRIMARY KEY,
  analyst_id          uuid         NOT NULL,
  interaction_class   varchar(64)  NOT NULL,
  matter_type         varchar(64)  NOT NULL,
  calibration_score   numeric(5,4) NOT NULL,
  rai_score           numeric(5,4) NOT NULL,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- AI system Elo ratings
CREATE TABLE ai_system_elo (
  system_id           uuid         NOT NULL,
  interaction_class   varchar(64)  NOT NULL,
  matter_type         varchar(64)  NOT NULL,
  current_rating      numeric(8,2) NOT NULL DEFAULT 1200,
  total_interactions  int          NOT NULL DEFAULT 0,
  correction_rate     numeric(5,4),
  last_updated        timestamptz  NOT NULL,
  PRIMARY KEY (system_id, interaction_class, matter_type)
);

-- Elo rating history (append-only)
CREATE TABLE elo_history (
  id                  bigserial    PRIMARY KEY,
  system_id           uuid         NOT NULL,
  interaction_class   varchar(64)  NOT NULL,
  matter_type         varchar(64)  NOT NULL,
  rating              numeric(8,2) NOT NULL,
  trigger_turn_id     uuid,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);
```

---

## Financial Governance Service / PLUTUS

**Store:** PostgreSQL

```sql
-- Per-turn cost record (immutable after write)
CREATE TABLE cost_record (
  turn_id             uuid         PRIMARY KEY,
  session_id          uuid         NOT NULL,
  matter_id           uuid         NOT NULL,
  client_id           uuid         NOT NULL,
  analyst_id          uuid         NOT NULL,
  model               varchar(128) NOT NULL,
  input_tokens        int          NOT NULL,
  output_tokens       int          NOT NULL,
  rate_card_version   varchar(32)  NOT NULL,
  input_cost          numeric(12,6) NOT NULL,
  output_cost         numeric(12,6) NOT NULL,
  total_cost          numeric(12,6) NOT NULL,
  interaction_class   varchar(64),
  billing_model       varchar(32)  NOT NULL CHECK (billing_model IN
                      ('passthrough','markup','absorbed')),
  markup_multiplier   numeric(6,4) NOT NULL DEFAULT 1.0,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Rate card (versioned; never deleted)
CREATE TABLE rate_card (
  version             varchar(32)  NOT NULL,
  model               varchar(128) NOT NULL,
  interaction_class   varchar(64),
  input_per_mtok      numeric(10,6) NOT NULL,
  output_per_mtok     numeric(10,6) NOT NULL,
  effective_from      timestamptz  NOT NULL,
  effective_until     timestamptz,
  PRIMARY KEY (version, model, COALESCE(interaction_class,'*'))
);

-- Matter budget
CREATE TABLE matter_budget (
  matter_id           uuid         PRIMARY KEY,
  budget_total        numeric(12,2) NOT NULL,
  soft_ceiling_pct    numeric(5,4) NOT NULL DEFAULT 0.80,
  hard_ceiling_pct    numeric(5,4) NOT NULL DEFAULT 1.00,
  current_spend       numeric(12,2) NOT NULL DEFAULT 0,
  status              varchar(32)  NOT NULL DEFAULT 'normal' CHECK (status IN
                      ('normal','soft_warning','hard_ceiling','authorized_over')),
  billing_model       varchar(32)  NOT NULL,
  updated_at          timestamptz  NOT NULL
);

-- Budget authorisation events (append-only)
CREATE TABLE budget_authorisation (
  event_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  matter_id           uuid         NOT NULL,
  authorised_by       uuid         NOT NULL,
  co_authorised_by    uuid         NOT NULL,
  previous_ceiling    numeric(12,2) NOT NULL,
  new_ceiling         numeric(12,2) NOT NULL,
  justification       text         NOT NULL,
  created_at          timestamptz  NOT NULL DEFAULT now()
);

-- Anomaly events
CREATE TABLE anomaly_event (
  event_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  matter_id           uuid         NOT NULL,
  turn_id             uuid,
  anomaly_type        varchar(64)  NOT NULL,
  expected_value      numeric(12,6),
  actual_value        numeric(12,6),
  z_score             numeric(8,4),
  alerted_at          timestamptz  NOT NULL DEFAULT now(),
  resolved_at         timestamptz,
  resolution_note     text
);
```

---

## Feedback & Ground Truth Service / ALETHEIA

**Store:** PostgreSQL

```sql
-- Correction events (immutable after write)
CREATE TABLE correction_event (
  correction_id       uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  turn_id             uuid         NOT NULL,
  chunk_id            char(64)     NOT NULL,  -- sha256
  analyst_id          uuid         NOT NULL,
  correction_type     varchar(32)  NOT NULL CHECK (correction_type IN
                      ('reject','edit','partial_edit','flag','approve_override')),
  original_content    text         NOT NULL,
  corrected_content   text,
  edit_distance       numeric(5,4),
  reason_code         varchar(64)  NOT NULL CHECK (reason_code IN
                      ('factual_error','outdated','irrelevant','style',
                       'privilege_concern','citation_error','other')),
  reason_note         text,
  retrieval_chunk_ids uuid[]       NOT NULL DEFAULT '{}',
  matter_type         varchar(64),
  interaction_class   varchar(64),
  matter_id           uuid         NOT NULL,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Ground truth corpus
CREATE TABLE corpus_entry (
  entry_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  correction_id       uuid         NOT NULL REFERENCES correction_event,
  matter_id           uuid         NOT NULL,
  consent_level       varchar(32)  NOT NULL CHECK (consent_level IN
                      ('none','anonymised','full')),
  consent_id          uuid,
  input_context       text         NOT NULL,
  ai_output           text         NOT NULL,
  ground_truth        text         NOT NULL,
  correction_type     varchar(32)  NOT NULL,
  quality_score       numeric(5,4) NOT NULL,
  labels              text[]       NOT NULL DEFAULT '{}',
  usable_for_ft       bool         NOT NULL DEFAULT false,
  usable_for_ft_full  bool         NOT NULL DEFAULT false,
  version             int          NOT NULL DEFAULT 1,
  created_at          timestamptz  NOT NULL DEFAULT now(),
  updated_at          timestamptz  NOT NULL DEFAULT now()
);

-- Consent records
CREATE TABLE consent_record (
  consent_id          uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  matter_id           uuid         NOT NULL,
  client_id           uuid         NOT NULL,
  ai_assisted_work    bool         NOT NULL DEFAULT true,
  ai_training_anon    bool         NOT NULL DEFAULT false,
  ai_training_full    bool         NOT NULL DEFAULT false,
  captured_at         timestamptz  NOT NULL,
  captured_by         uuid         NOT NULL,
  consent_doc_ref     text         NOT NULL,
  jurisdiction        varchar(32)  NOT NULL,
  review_due          timestamptz  NOT NULL,
  revoked             bool         NOT NULL DEFAULT false,
  revocation_event_id uuid
);

-- Consent revocation events (append-only)
CREATE TABLE revocation_event (
  event_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  consent_id          uuid         NOT NULL,
  matter_id           uuid         NOT NULL,
  client_id           uuid         NOT NULL,
  revoked_by          uuid         NOT NULL,
  affected_entries    int          NOT NULL,
  future_runs_blocked bool         NOT NULL DEFAULT true,
  revoked_at          timestamptz  NOT NULL DEFAULT now()
);

-- Analyst signal records
CREATE TABLE analyst_signal (
  analyst_id          uuid         PRIMARY KEY,
  correction_count    int          NOT NULL DEFAULT 0,
  correction_accuracy numeric(5,4),
  domain_expertise    jsonb,       -- {matter_type: accuracy_score}
  catch_rate          numeric(5,4),
  signal_weight       numeric(5,4) NOT NULL DEFAULT 1.0,
  updated_at          timestamptz  NOT NULL DEFAULT now()
);
```

---

## Temporal Validity Service / KAIROS

**Store:** PostgreSQL (validity records + history) + Redis (current score cache)

```sql
-- Validity records
CREATE TABLE validity_record (
  chunk_id            char(64)     PRIMARY KEY,  -- sha256
  decay_score         numeric(5,4) NOT NULL,
  contested           bool         NOT NULL DEFAULT false,
  invalidated         bool         NOT NULL DEFAULT false,
  invalidated_by      char(64),                  -- FK → chunk_id
  composite_score     numeric(5,4) NOT NULL,
  decay_profile       varchar(64)  NOT NULL,
  data_jurisdiction   varchar(16)  NOT NULL,
  ingested_at         timestamptz  NOT NULL,
  last_scored_at      timestamptz  NOT NULL,

  -- AI output fields (null for non-AI chunks)
  model_cutoff        timestamptz,
  retrieval_freshness numeric(5,4)
);

-- Score history (append-only — never updated, only inserted)
CREATE TABLE validity_score_history (
  id                  bigserial    PRIMARY KEY,
  chunk_id            char(64)     NOT NULL,
  composite_score     numeric(5,4) NOT NULL,
  decay_score         numeric(5,4) NOT NULL,
  trigger_type        varchar(32)  NOT NULL CHECK (trigger_type IN
                      ('scheduled_decay','active_invalidation','conflict_resolution',
                       'kcs_invalidation','propagation')),
  trigger_id          uuid,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Conflict events
CREATE TABLE conflict_event (
  conflict_id         uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  chunk_a             char(64)     NOT NULL,
  chunk_b             char(64)     NOT NULL,
  conflict_type       varchar(32)  NOT NULL CHECK (conflict_type IN
                      ('direct_contradiction','supersession','partial_overlap')),
  detection_method    varchar(32)  NOT NULL CHECK (detection_method IN
                      ('semantic','structural','metadata')),
  contradiction_score numeric(5,4) NOT NULL,
  matter_id           uuid         NOT NULL,
  status              varchar(32)  NOT NULL DEFAULT 'open' CHECK (status IN
                      ('open','resolved','dismissed')),
  detected_at         timestamptz  NOT NULL DEFAULT now(),
  resolved_at         timestamptz,
  resolver_id         uuid,
  decision            varchar(32)  CHECK (decision IN
                      ('a_supersedes_b','b_supersedes_a','both_valid','both_discarded')),
  rationale           text
);

-- Decay profiles (configuration)
CREATE TABLE decay_profile (
  profile_name        varchar(64)  PRIMARY KEY,
  function_type       varchar(32)  NOT NULL CHECK (function_type IN
                      ('step','exponential','linear','aggressive_exponential','model_vintage')),
  half_life_days      int,
  floor_score         numeric(5,4) NOT NULL DEFAULT 0.05,
  default_score       numeric(5,4) NOT NULL DEFAULT 1.0,
  on_event_score      numeric(5,4),  -- for step functions
  compound_penalty    bool         NOT NULL DEFAULT false,
  updated_at          timestamptz  NOT NULL DEFAULT now()
);
```

**Redis cache schema:**
```
KEY:   validity:{chunk_id}
VALUE: {score: float, contested: bool, invalidated: bool, cached_at: ISO8601}
TTL:   300 seconds
```

---

## Retrieval Quality Service / HERMES

**Store:** PostgreSQL

```sql
-- Per-query retrieval quality record
CREATE TABLE retrieval_quality_record (
  record_id           uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  query_turn_id       uuid         NOT NULL,
  matter_id           uuid         NOT NULL,
  retrieved_chunks    char(64)[]   NOT NULL,  -- sha256 array
  confirmed_relevant  char(64)[]   NOT NULL DEFAULT '{}',
  confirmed_missed    char(64)[]   NOT NULL DEFAULT '{}',
  precision           numeric(5,4),
  recall_estimate     numeric(5,4),
  mean_validity_score numeric(5,4),
  stale_chunk_count   int          NOT NULL DEFAULT 0,
  query_class         varchar(64),
  matter_type         varchar(64),
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Retrieval miss signals (from FGTS)
CREATE TABLE retrieval_miss_signal (
  signal_id           uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  correction_id       uuid         NOT NULL,
  query_turn_id       uuid         NOT NULL,
  missing_chunk_desc  text         NOT NULL,
  miss_pattern        varchar(64)  CHECK (miss_pattern IN
                      ('category_miss','recency_miss','granularity_miss',
                       'validity_blind','cross_matter_ghost')),
  matter_type         varchar(64),
  interaction_class   varchar(64),
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Precision baselines (rolling, per query class + matter type)
CREATE TABLE precision_baseline (
  query_class         varchar(64)  NOT NULL,
  matter_type         varchar(64)  NOT NULL,
  baseline_precision  numeric(5,4) NOT NULL,
  sample_size         int          NOT NULL,
  computed_at         timestamptz  NOT NULL,
  PRIMARY KEY (query_class, matter_type)
);
```

---

## Knowledge Currency Service / ARGUS

**Store:** PostgreSQL

```sql
-- External source registry
CREATE TABLE source_record (
  source_id           uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  name                varchar(256) NOT NULL,
  source_type         varchar(64)  NOT NULL CHECK (source_type IN
                      ('court_docket','regulatory_feed','legal_db',
                       'news','scholarly','agency_guidance')),
  jurisdiction        text[]       NOT NULL,
  practice_areas      text[]       NOT NULL DEFAULT '{}',
  polling_interval    interval     NOT NULL,
  push_endpoint       text,
  reliability_score   numeric(5,4) NOT NULL DEFAULT 0.75,
  active              bool         NOT NULL DEFAULT true,
  last_checked        timestamptz,
  created_at          timestamptz  NOT NULL DEFAULT now()
);

-- External development events
CREATE TABLE kcs_event (
  event_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  source_id           uuid         NOT NULL REFERENCES source_record,
  development_url     text         NOT NULL,
  development_type    varchar(64)  NOT NULL CHECK (development_type IN
                      ('ruling','amendment','repeal','new_filing',
                       'guidance','contradictory_testimony')),
  affected_chunks     jsonb        NOT NULL,  -- [{chunk_id, impact_score, impact_type}]
  confidence          numeric(5,4) NOT NULL,
  detected_at         timestamptz  NOT NULL DEFAULT now(),
  status              varchar(32)  NOT NULL DEFAULT 'pending_review' CHECK (status IN
                      ('pending_review','forwarded_to_tvs','dismissed')),
  human_reviewed      bool         NOT NULL DEFAULT false,
  reviewer_id         uuid,
  reviewed_at         timestamptz
);

-- Matter watch lists
CREATE TABLE watch_list_entry (
  entry_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  matter_id           uuid         NOT NULL,
  entity_type         varchar(64)  NOT NULL CHECK (entity_type IN
                      ('case_citation','party','statute','regulation','witness')),
  entity_value        text         NOT NULL,
  alert_threshold     numeric(5,4) NOT NULL DEFAULT 0.50,
  notify              uuid[]       NOT NULL DEFAULT '{}',
  created_by          uuid         NOT NULL,
  active              bool         NOT NULL DEFAULT true,
  created_at          timestamptz  NOT NULL DEFAULT now()
);
```

---

## Privilege & Consent Enforcement Service / AEGIS

**Store:** PostgreSQL

```sql
-- Privilege records (per chunk)
CREATE TABLE privilege_record (
  chunk_id            char(64)     PRIMARY KEY,
  privilege_type      varchar(64)  NOT NULL CHECK (privilege_type IN
                      ('attorney_client','work_product','common_interest',
                       'third_party_confidential','no_privilege')),
  matter_id           uuid         NOT NULL,
  protecting_party    uuid,
  designated_by       uuid         NOT NULL,
  designated_at       timestamptz  NOT NULL,
  waived              bool         NOT NULL DEFAULT false,
  waiver_event_id     uuid,
  propagated_from     char(64)     -- FK → parent chunk if derived
);

-- Privilege filter events (append-only audit log)
CREATE TABLE privilege_filter_event (
  event_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  chunk_id            char(64)     NOT NULL,
  turn_id             uuid,
  decision            varchar(16)  NOT NULL CHECK (decision IN
                      ('PASS','BLOCKED','REDACTED','FLAGGED')),
  reason              text,
  requestor           uuid         NOT NULL,
  matter_context      uuid,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Conflict graph (entities and relationships)
CREATE TABLE conflict_entity (
  entity_id           uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_type         varchar(32)  NOT NULL,
  entity_name         text         NOT NULL,
  matter_ids          uuid[]       NOT NULL DEFAULT '{}'
);

CREATE TABLE conflict_relationship (
  relationship_id     uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_a            uuid         NOT NULL REFERENCES conflict_entity,
  entity_b            uuid         NOT NULL REFERENCES conflict_entity,
  relationship_type   varchar(32)  NOT NULL CHECK (relationship_type IN
                      ('CURRENT_CLIENT','FORMER_CLIENT','ADVERSE',
                       'RELATED_PARTY','WITNESS')),
  effective_from      timestamptz  NOT NULL,
  effective_until     timestamptz,
  matter_id           uuid
);

-- Consent records
CREATE TABLE ai_consent_record (
  consent_id          uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  matter_id           uuid         NOT NULL,
  obligation_type     varchar(64)  NOT NULL,
  obligated_to        uuid,
  scope               text,
  expires_at          timestamptz,
  permits_ai_use      bool         NOT NULL DEFAULT false,
  source_chunk_id     char(64)
);
```

---

## Policy & Guardrails Service / NOMOS

**Store:** PostgreSQL

```sql
-- Policy rules
CREATE TABLE policy_rule (
  rule_id             uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  name                varchar(256) NOT NULL,
  version             varchar(32)  NOT NULL,
  priority            int          NOT NULL,
  scope               varchar(32)  NOT NULL CHECK (scope IN
                      ('global','practice_group','matter_type','jurisdiction')),
  scope_value         text,
  trigger_type        varchar(64)  NOT NULL,
  condition_cel       text         NOT NULL,
  action              varchar(32)  NOT NULL CHECK (action IN
                      ('BLOCK','WARN','TRANSFORM','REQUIRE_APPROVAL')),
  action_params       jsonb,
  effective_from      timestamptz  NOT NULL,
  effective_until     timestamptz,
  authored_by         uuid         NOT NULL,
  approved_by         uuid         NOT NULL,
  rationale           text         NOT NULL,
  hades_scenario_id   uuid,   -- co-authored HADES probe
  active              bool         NOT NULL DEFAULT true
);

-- Policy evaluation events (append-only)
CREATE TABLE policy_evaluation_event (
  event_id            uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  turn_id             uuid         NOT NULL,
  interaction_class   varchar(64),
  rules_evaluated     uuid[]       NOT NULL DEFAULT '{}',
  rules_triggered     uuid[]       NOT NULL DEFAULT '{}',
  outcome             varchar(32)  NOT NULL CHECK (outcome IN
                      ('PASS','WARN','BLOCK','REQUIRE_APPROVAL')),
  pii_detected        bool         NOT NULL DEFAULT false,
  matter_id           uuid,
  recorded_at         timestamptz  NOT NULL DEFAULT now()
);

-- Hold records
CREATE TABLE hold_record (
  hold_id             uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  turn_id             uuid         NOT NULL,
  session_id          uuid         NOT NULL,
  analyst_id          uuid         NOT NULL,
  trigger_rule_id     uuid         NOT NULL REFERENCES policy_rule,
  trigger_category    varchar(32)  NOT NULL CHECK (trigger_category IN
                      ('privilege','policy','pii','budget','coi_flag')),
  interaction_class   varchar(64),
  matter_id           uuid         NOT NULL,
  prompt_assembly_id  uuid,
  created_at          timestamptz  NOT NULL DEFAULT now(),
  sla_deadline        timestamptz  NOT NULL,
  assigned_to         uuid,
  status              varchar(32)  NOT NULL DEFAULT 'open' CHECK (status IN
                      ('open','approved','modified','rejected','escalated')),
  escalated_at        timestamptz,
  escalated_to        uuid
);

-- Hold resolution events (append-only)
CREATE TABLE hold_resolution (
  resolution_id       uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
  hold_id             uuid         NOT NULL REFERENCES hold_record,
  turn_id             uuid         NOT NULL,
  reviewer_id         uuid         NOT NULL,
  resolved_at         timestamptz  NOT NULL DEFAULT now(),
  decision            varchar(16)  NOT NULL CHECK (decision IN
                      ('approved','modified','rejected')),
  rationale           text         NOT NULL CHECK (length(rationale) >= 20),
  modified_prompt_id  uuid,
  escalated_from      uuid,
  provenance_event_id uuid         NOT NULL
);
```

---

## Reasoning Audit Service / LOGOS

**Store:** Elasticsearch

```json
// Index: themis-reasoning-captures
{
  "mappings": {
    "properties": {
      "capture_id":          { "type": "keyword" },
      "turn_id":             { "type": "keyword" },
      "matter_id":           { "type": "keyword" },
      "interaction_class":   { "type": "keyword" },
      "matter_type":         { "type": "keyword" },
      "chain_of_thought":    { "type": "text", "analyzer": "english" },
      "alternatives":        { "type": "text" },
      "evidence_citations":  { "type": "keyword" },
      "unsupported_claim_count": { "type": "integer" },
      "completeness_score":  { "type": "float" },
      "is_adversarial":      { "type": "boolean" },
      "recorded_at":         { "type": "date" },

      "claims": {
        "type": "nested",
        "properties": {
          "claim_id":           { "type": "keyword" },
          "text":               { "type": "text", "analyzer": "english" },
          "claim_type":         { "type": "keyword" },
          "supporting_chunks":  { "type": "keyword" },
          "confidence":         { "type": "keyword" },
          "validity_at_capture":{ "type": "float" },
          "unsupported":        { "type": "boolean" }
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

## Kafka Event Schemas

All inter-service events are published to Kafka topics. Events are JSON. Schema versions follow the service version.

### Core Events

```json
// Topic: themis.provenance.turn-initiated
{
  "event_id": "uuid",
  "event_type": "TurnInitiated",
  "turn_id": "uuid",
  "session_id": "uuid",
  "matter_id": "uuid",
  "analyst_id": "uuid",          // pseudonymised
  "interaction_class": "string",
  "retrieval_chunk_ids": ["sha256"],
  "prompt_assembly_id": "uuid",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.provenance.turn-completed
{
  "event_id": "uuid",
  "event_type": "TurnCompleted",
  "turn_id": "uuid",
  "output_chunk_id": "sha256",
  "input_token_count": 1200,
  "output_token_count": 340,
  "model": "claude-sonnet-4",
  "model_cutoff": "ISO8601 | null",
  "retrieval_freshness": 0.82,
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.pgs.policy-evaluation
{
  "event_id": "uuid",
  "event_type": "PolicyEvaluationEvent",
  "turn_id": "uuid",
  "outcome": "PASS | WARN | BLOCK | REQUIRE_APPROVAL",
  "rules_triggered": ["uuid"],
  "interaction_class": "string",
  "pii_detected": false,
  "matter_id": "uuid",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.pgs.hold-created
{
  "event_id": "uuid",
  "event_type": "HoldCreated",
  "hold_id": "uuid",
  "turn_id": "uuid",
  "trigger_category": "privilege | policy | pii | budget | coi_flag",
  "matter_id": "uuid",
  "sla_deadline": "ISO8601",
  "assigned_to": "uuid | null",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.pgs.hold-resolved
{
  "event_id": "uuid",
  "event_type": "HoldResolved",
  "resolution_id": "uuid",
  "hold_id": "uuid",
  "turn_id": "uuid",
  "decision": "approved | modified | rejected",
  "reviewer_id": "uuid",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.fgts.correction
{
  "event_id": "uuid",
  "event_type": "CorrectionEvent",
  "correction_id": "uuid",
  "turn_id": "uuid",
  "chunk_id": "sha256",
  "analyst_id": "uuid",
  "correction_type": "reject | edit | partial_edit | flag | approve_override",
  "edit_distance": 0.34,
  "reason_code": "factual_error | outdated | ...",
  "matter_type": "string",
  "interaction_class": "string",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.fgts.ai-performance-update  (FGTS → TCS)
{
  "event_id": "uuid",
  "event_type": "AIPerformanceUpdate",
  "turn_id": "uuid",
  "interaction_class": "string",
  "matter_type": "string",
  "outcome": "corrected | accepted",
  "error_category": "string",
  "severity": 0.45,
  "analyst_id": "uuid",
  "analyst_weight": 0.87,
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.fgts.retrieval-miss  (FGTS → RQS)
{
  "event_id": "uuid",
  "event_type": "RetrievalMissSignal",
  "correction_id": "uuid",
  "query_turn_id": "uuid",
  "missing_chunk_desc": "string",
  "miss_pattern": "category_miss | recency_miss | ...",
  "matter_type": "string",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.tvs.validity-updated
{
  "event_id": "uuid",
  "event_type": "ValidityUpdated",
  "chunk_id": "sha256",
  "new_score": 0.61,
  "previous_score": 0.79,
  "trigger_type": "active_invalidation | scheduled_decay | ...",
  "trigger_id": "uuid | null",
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.kcs.invalidation-request  (KCS → TVS)
{
  "event_id": "uuid",
  "event_type": "ActiveInvalidationRequest",
  "kcs_event_id": "uuid",
  "affected_chunks": [
    { "chunk_id": "sha256", "impact_score": 0.92, "impact_type": "supersession" }
  ],
  "development_url": "string",
  "development_type": "ruling | amendment | repeal | ...",
  "confidence": 0.94,
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}

// Topic: themis.fgs.budget-warning
{
  "event_id": "uuid",
  "event_type": "BudgetSoftWarning | BudgetHardCeiling",
  "matter_id": "uuid",
  "current_spend": 8400.00,
  "budget_total": 10000.00,
  "utilisation_pct": 0.84,
  "timestamp": "ISO8601",
  "schema_version": "1.0"
}
```

---

## Cross-Service Reference Types

Standard types used across multiple services.

### Identifiers

| Field | Type | Notes |
|---|---|---|
| `matter_id` | uuid | Authoritative source: matter management system |
| `client_id` | uuid | Authoritative source: matter management system |
| `analyst_id` | uuid | Pseudonymised; mapping to real identity lives in IAM only |
| `chunk_id` | sha256 (64 hex chars) | Content-addressed; same content = same ID |
| `turn_id` | uuid | Generated by API gateway sidecar at TurnInitiated |
| `session_id` | uuid | Generated at session start; groups turns |

### Enumerations

**data_jurisdiction:** `eu | uk | us | apac | global`
**interaction_class:** `research | document_drafting | evidence_analysis | legal_advice | third_party_facing | system_probe | administrative`
**privilege_type:** `attorney_client | work_product | common_interest | third_party_confidential | no_privilege`
**correction_type:** `reject | edit | partial_edit | flag | approve_override`
**consent_level:** `none | anonymised | full`

### Nullification Marker Format

When content fields are nullified, replaced with:
```
[NULLIFIED — basis: {nullification_basis} — {nullified_at}]
```

Example: `[NULLIFIED — basis: matter_close — 2026-07-15T00:00:00Z]`
