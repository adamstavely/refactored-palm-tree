# MOIRAI — AI Content Provenance Service
### *"The Fates — who tracks origins and lineage"*
*Part of the THEMIS Platform · Core Infrastructure · Build Priority: 3*

---

## Design Philosophy

MOIRAI is a **content chain of custody system**, not an AI usage tracker. AI usage tracking is one edge type in the provenance graph — not the system's primary purpose. Every piece of content that passes through the THEMIS platform — from evidence ingestion through AI interaction to final work product — has a verifiable, queryable lineage record.

The system is built on five architectural commitments: content-addressed identity (SHA-256 + LSH), append-only immutable event log, turn-level DAG (not flat session manifests), prompt lineage as first-class citizens, and a privilege boundary that separates auditable chain of custody from protected attorney work product.

---

## Architecture Overview

```
Evidence / Source Material
    │
    │ ingest + chunk + fingerprint
    ▼
[ Layer 1: Ingestion ]  ──────────────────────── Chunk Registry
    │                                                   │
    │ retrieved into AI context                         │
    ▼                                                   │
[ Layer 2: AI Interaction ]  ─── Turn DAG ──────── Provenance Graph
    │                                                   │
    │ output copy-pasted / transported                  │
    ▼                                                   │
[ Layer 3: Transport ]  ──── fingerprint signals ───────┘
    │
    │ assembled into work product
    ▼
[ Layer 4: Document Assembly ]  ─── certification hash
    │
    │ audit / review / discovery
    ▼
[ Layer 5: Verification & Audit ]
```

---

## Layer 1 — Ingestion & Spatial Provenance

```yaml
chunk_spatial_record:
  chunk_id:          sha256(normalized_content)
  lsh_fingerprint:   minhash(content, k=5, n=128)
  source_doc_id:     uuid
  modality:          document | video | email | chat | image

  # Document modality
  page_number:       int
  bounding_box:      {x1, y1, x2, y2}
  reading_order_seq: int
  section_label:     str
  content_type:      body | heading | footnote | caption | table_cell

  # Video/audio modality
  timecode_start:    float
  timecode_end:      float
  speaker_id:        str

  # Email/chat modality
  thread_id:         str
  message_position:  int
  sender_id:         str
  timestamp:         ISO8601
```

---

## Layer 2 — Turn-Level Interaction DAG

A flat session manifest fails for multi-turn conversations because:
- Retrieval context changes turn by turn
- Prior AI outputs become inputs (conversation history in context window)
- Branching/edit-and-regenerate creates parallel paths
- Agentic turns have internal sub-steps

```yaml
TurnRecord:
  turn_id:              uuid
  session_id:           uuid
  turn_index:           int
  timestamp:            ISO8601
  role:                 user | assistant | tool

  # Prompt lineage links
  analyst_input_id:     uuid         # FK → AnalystInput node
  prompt_assembly_id:   uuid         # FK → PromptAssembly node
  system_prompt_id:     uuid         # FK → SystemPromptRecord
  prompt_template_id:   uuid         # FK → PromptTemplate node

  # Context and retrieval
  retrieval_chunk_ids:  [uuid, ...]
  prior_turn_ids:       [uuid, ...]  # intra-session DAG
  context_window_snapshot: [messages]
  input_token_count:    int          # truncation detection signal

  # Output
  output_chunk_id:      uuid
  output_lsh:           hash
  output_token_count:   int

  # Agentic
  sub_steps:            [uuid, ...]
  branch_parent:        uuid | null
```

---

## Layer 3 — Prompt Lineage (First-Class Provenance)

### Three New Graph Nodes

**PromptTemplate** — Versioned, governance-approved system instruction set.
```yaml
  template_id:       uuid
  version:           semver
  interaction_class: str
  base_text:         str
  assembly_recipe:   { dynamic_components: [...] }
  approved_by:       user_id
  pgs_rule_ids:      [uuid, ...]
  effective_from:    ISO8601
```

**AnalystInput** — Raw, unaugmented user query before any system transformation.
```yaml
  input_id:          uuid
  raw_text:          str
  input_type:        text | text_with_attachment | highlighted_passage
  attachments:       [chunk_id, ...]
  interface_state:   { open_document_tabs, foregrounded_doc, selected_text }
  pgs_classification: str
  pgs_evaluation_id: uuid
```

**PromptAssembly** — The exact payload sent to the model.
```yaml
  assembly_id:          uuid
  system_prompt_id:     uuid
  analyst_input_id:     uuid
  retrieval_chunk_ids:  [uuid, ...]
  assembled_messages_hash: sha256
  token_count:          int
  assembly_timestamp:   ISO8601
```

### Failure Attribution Model

| Root Cause | Graph Signal | Remediation |
|---|---|---|
| Model failure | PromptAssembly.retrieval_quality HIGH + SystemPrompt CURRENT | Model version review; provider escalation |
| Retrieval failure | PromptAssembly.retrieval_chunk_ids have low RQS precision/TVS scores | RQS RetrievalMissSignal; retrieval model review |
| Prompt instruction failure | SystemPromptRecord version STALE or pgs_rule_ids MISMATCH | Prompt template review; AI Governance Committee |
| Analyst intent ambiguity | AnalystInput.interface_state EMPTY + raw_text AMBIGUOUS | Analyst training; interface design improvement |

---

## Layer 4 — Transport & Fingerprinting

Three-signal strategy:

| Signal | Mechanism | Reliability | Scope |
|---|---|---|---|
| Clipboard interception | Secondary MIME type `text/provenance+json` on copy | High | Within-platform only |
| ZWC encoding | Invisible Unicode characters encoding chunk_id | Low-Medium | External transit (fragile) |
| LSH registry detection | Fuzzy match against chunk registry | High | Universal fallback |

**Composite confidence score:**
```
composite = ZWC_match(0.3) + LSH_similarity(0.4) + watermark_p_value(0.3)
# If ZWC absent: redistribute weight to LSH and watermark
requires_human_review: composite_confidence < 0.75
```

---

## Layer 5 — Document Assembly & Verification

**Paste-time provenance capture:** Every paste operation triggers ZWC extraction + LSH lookup. On match, a `CITED_IN` edge is written to the provenance graph.

**Provenance Summary Report:**
```
Original attorney writing:         38%
Source material (direct):          29%
AI-generated (single hop):         21%
AI-derived (2+ hops):               9%
Unknown origin:                     3%

AI Sessions Contributing:          4 sessions, 3 analysts
Models Used:                       [model versions]
Provenance Certification Hash:     sha256(content + graph_snapshot)
```

---

## The Provenance Graph Model

### Node Types

| Node | Description |
|---|---|
| EvidenceSource | Original document, video, email chain entering the system |
| Chunk | Spatial unit of content with content hash, LSH, and spatial coordinates |
| AISession | Conversation session with session manifest metadata |
| Turn | Single exchange. The primary AI interaction provenance unit |
| PromptTemplate | Versioned system instruction set |
| AnalystInput | Raw user query before augmentation |
| PromptAssembly | Exact payload sent to model |
| OutputChunk | Content produced by a specific turn |
| DocumentSection | Section within a work product document |
| LegalDocument | Finalized work product with certification hash |

### Edge Types

| Edge | Relationship | Meaning |
|---|---|---|
| INGEST | EvidenceSource → Chunk | Source decomposed into chunk at ingest |
| RETRIEVED | Turn → Chunk | Chunk was in RAG retrieval context |
| CONSUMED | Turn → OutputChunk (prior) | Prior output was in context window |
| PRODUCED | Turn → OutputChunk | Turn generated this output |
| CITED_IN | OutputChunk/Chunk → DocumentSection | Pasted/cited into document section |
| COMPILED_INTO | DocumentSection → LegalDocument | Section is part of document |
| GOVERNED_BY | Turn → PromptTemplate | Interaction governed by this template version |
| ASSEMBLED_FROM | PromptAssembly → PromptTemplate | Assembly used this system prompt |
| ASSEMBLED_FROM | PromptAssembly → AnalystInput | Assembly incorporated this raw input |
| INJECTED_WITH | PromptAssembly → Chunk | RAG chunk injected into assembled prompt |

---

## Data Retention & Deletion Architecture

### Retention Policy by Node Type

| Node Type | Retention | Category | Deletion Treatment |
|---|---|---|---|
| EvidenceSource / Chunk | Matter + 7 years | Content | Content nullified; structure retained |
| TurnRecord | Matter + 7 years | Structural | Prompt content nullified; IDs and edges retained |
| PromptAssembly | Matter + 7 years | Content | Assembled content nullified; recipe retained |
| AnalystInput | Matter + 3 years | Content — PII risk | Raw text nullified; structure retained |
| OutputChunk | Matter + 7 years | Content | Content nullified; chunk_id retained |
| PromptTemplate | Indefinite | Structural | Never deleted |

### Nullification vs. Deletion

Nullification preserves graph structure while satisfying deletion obligations. Node IDs, edges, and timestamps are retained. Content fields are replaced with a nullification marker including basis and timestamp.

```yaml
NullificationRecord:
  nullification_id:    uuid
  node_id:             uuid
  node_type:           str
  nullified_fields:    [str]
  nullification_basis: gdpr_erasure | matter_close | retention_expiry | court_order
  nullified_at:        ISO8601
  authorised_by:       user_id
  downstream_notified: bool
```

### GDPR Article 17

- **Analyst erasure:** Analyst identity stored as pseudonym. Identity mapping lives in IAM. Erasure satisfies at the IAM boundary — MOIRAI never holds directly identifying data.
- **Client/witness data erasure:** Chunk raw_text fields nullified on valid erasure request. Downstream OutputChunk nodes receive `nullification_notice` flag.
- **Litigation hold:** All nullification suspended while hold is active. GDPR Article 17(3)(e) exception applies for active litigation — assessed by ethics counsel per request.

---

## API Surfaces

```
# Chain of Custody API (non-privileged)
GET  /lineage/{chunk_id}
GET  /session/{session_id}/manifest
GET  /document/{doc_id}/provenance-report
GET  /audit/{case_id}/ai-usage-summary
POST /litigation-hold/{case_id}
POST /detect                           # content fingerprint detection

# Forensic API (privileged — separate auth scope)
GET  /forensic/session/{session_id}/full
GET  /forensic/turn/{turn_id}/prompt-assembly
```

---

## Implementation Roadmap

### Phase 1 — Foundation (Weeks 9–14)
- Chunk ID + LSH at ingestion; spatial metadata for all modalities
- TurnInitiated / TurnCompleted events via sidecar buffer
- Neo4j graph store; append-only event log

### Phase 2 — Interaction DAG (Weeks 15–18)
- Turn-level records; context_window_snapshot capture
- resolve_prior_turns(); truncation detection; branch tracking
- Streaming response accumulator

### Phase 3 — Prompt Lineage (Weeks 19–20)
- PromptTemplate, AnalystInput, PromptAssembly nodes
- Seven new graph edges; failure attribution model
- System prompt version governance integration

### Phase 4 — Transport & Assembly (Weeks 21–28)
- Three-signal fingerprinting; clipboard interception
- Document assembly paste-time provenance; Provenance Summary Report
- Certification Hash; Chain of Custody and Forensic APIs
- Data retention schema; nullification pipeline

---

**Depends on:** PCES / AEGIS, PGS / NOMOS
**Feeds into:** All THEMIS services (read-only graph access)
