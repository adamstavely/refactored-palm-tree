# PCES — Privilege & Consent Enforcement Service
### AEGIS · *"Shield — the protective layer"*
*Part of the THEMIS Platform · Safety Gate Layer · Build Priority: 1*

---

## Design Philosophy

Most access control systems answer: **can this user see this document?** The Privilege and Consent Enforcement Service (PCES) answers a harder question: **can this specific chunk of content legally enter an AI context window for this specific query in this specific matter?**

A paralegal with full access to a case file may not be permitted to include privileged communications in a RAG prompt. An attorney on Matter A may not have an AI system surface content from Matter B even if they have read access to both. PCES enforces privilege, work product protection, confidentiality obligations, and conflict-of-interest boundaries at the retrieval layer — before content reaches the model.

> **Why This Must Be First:** Every other THEMIS service assumes AI is operating on appropriate data. Provenance logs the lineage of a privilege violation. TVS scores the validity of improperly accessed evidence. Trust Calibration measures reliance on contaminated output. PCES is the gate. Without it, the platform is building on a broken foundation.

| System | Question Answered | Granularity | Layer |
|---|---|---|---|
| Access Control | Can this user see this document? | Identity-based, coarse-grained, static | IAM / RBAC |
| **PCES** | Can this chunk enter an AI context window? | Content-based, fine-grained, context-dependent | Runtime retrieval filter |

---

## Architecture Overview

```
Retrieval Layer (RAG)
    │  candidate chunks returned from vector search
    ▼
┌─────────────────────────────────────────────────────┐
│         Privilege & Consent Enforcement Service     │
│                                                     │
│  ┌──────────────┐  ┌───────────────┐  ┌─────────┐  │
│  │  Privilege   │  │  Conflict-of  │  │ Consent │  │
│  │  Classifier  │  │  Interest     │  │ Tracker │  │
│  │              │  │  Detector     │  │         │  │
│  └──────┬───────┘  └───────┬───────┘  └────┬────┘  │
│         └──────────────────┼────────────────┘       │
│                            ▼                        │
│                 ┌─────────────────────┐             │
│                 │  Enforcement Engine  │             │
│                 │  PASS / REDACT / BLOCK│            │
│                 └──────────┬────────────┘            │
│                            │                        │
│                 ┌──────────▼────────────┐           │
│                 │  Privilege Audit Log   │           │
│                 └───────────────────────┘           │
└─────────────────────────────────────────────────────┘
    │  filtered + annotated chunks
    ▼
Model Context Window
```

---

## Privilege Classification Model

Every chunk in the corpus carries a privilege classification assigned at ingest:

```yaml
PrivilegeRecord:
  chunk_id:           sha256            # FK → Provenance Service
  privilege_type:     attorney_client | work_product | common_interest
                      | third_party_confidential | no_privilege
  matter_id:          uuid
  protecting_party:   entity_id
  created_by:         user_id
  designated_at:      ISO8601
  waived:             bool
  waiver_event_id:    uuid | null
  propagated_from:    chunk_id | null   # if derived from a privileged source
```

**Privilege propagation:** If a chunk carries attorney_client privilege, any AI output derived from it inherits that designation through the MOIRAI provenance DAG.

**Cross-matter boundary detection:** The most dangerous failure mode is matter bleed — RAG surfacing content from Matter B into a prompt scoped to Matter A. PCES enforces matter boundaries by checking every candidate chunk's `matter_id` against the active query's matter scope. Cross-matter retrieval is blocked by default.

---

## Conflict-of-Interest Detection

```
CoI Check (per retrieval request):
  1. Extract entities referenced in query and candidate chunks
  2. Look up each entity in the Conflict Graph
  3. For each entity pair (query_entity, chunk_entity):
       if relationship == ADVERSE and both are current clients:
           → BLOCK chunk; raise CoIAlert
       if relationship == FORMER_ADVERSE within lookback_window:
           → FLAG chunk; require supervising attorney confirmation
  4. Log all CoI checks regardless of outcome
```

| Relationship Type | Enforcement Behavior |
|---|---|
| CURRENT_CLIENT | Active matter representation — highest protection |
| FORMER_CLIENT | Past representation — lookback window applies (default 3 years) |
| ADVERSE | Entity is adverse to a current client — cross-matter retrieval blocked |
| RELATED_PARTY | Entity has material relationship to current client — flagged for review |
| WITNESS | Deponent or prospective witness — context-sensitive restrictions |

---

## Consent & Confidentiality Tracking

```yaml
ConsentRecord:
  chunk_id:           sha256
  obligation_type:    nda | protective_order | settlement | consent_decree
  obligated_to:       entity_id
  scope:              str
  expires_at:         ISO8601 | null
  permits_ai_use:     bool
  permits_ai_use_for: [purpose, ...]
  source_document_id: chunk_id
```

> **Default position:** If `permits_ai_use` is not explicitly true, the chunk is flagged for attorney review before entering an AI context. Many confidentiality agreements predate AI and do not address model processing.

---

## Runtime Enforcement Pipeline

```
For each candidate chunk from retrieval:

  1. CHECK matter scope
       chunk.matter_id ∈ query.authorized_matters?
       NO  → BLOCK

  2. CHECK privilege
       chunk.privilege_type == no_privilege?  → PASS to step 3
       chunk.waived == true?                  → PASS to step 3
       requestor has privilege_access grant?  → PASS to step 3
       ELSE → BLOCK; log PrivilegeFilterEvent

  3. CHECK consent obligations
       chunk.permits_ai_use == true?           → PASS to step 4
       chunk.obligation_type == null?           → PASS to step 4
       ELSE → FLAG; require attorney confirmation or BLOCK

  4. CHECK conflict of interest
       Run CoI graph check
       ADVERSE relationship found → BLOCK; raise CoIAlert
       FORMER_ADVERSE within window → FLAG

  5. PASS — chunk enters context window
       Annotate with privilege_record_id for provenance logging
```

**Redaction vs. Blocking:** For partially privileged chunks, the PCES supports targeted redaction — privileged spans are masked before the chunk enters the context window. The redaction event is logged with span coordinates matching MOIRAI spatial metadata.

---

## Events Emitted

```yaml
PrivilegeAnnotated:        # at ingest
  chunk_id, privilege_type, matter_id, designated_at

PrivilegeFilterEvent:      # at every enforcement decision
  chunk_id, turn_id, reason: BLOCKED|REDACTED|FLAGGED
  requestor, matter_context, timestamp

CoIAlertRaised:            # when adverse party detected
  chunk_id, conflicting_entity_ids, matter_id, turn_id
```

---

## Attorney-Facing Surfaces

- **Privilege Log Export** — Machine-readable and formatted for discovery production. Generated directly from PrivilegeFilterEvent log without manual compilation.
- **CoI Dashboard** — Real-time view of active conflict flags, pending resolutions, entity graph visualization.
- **Access Denied UX** — Analysts see a clear explanation without the content of the blocked chunk being revealed.

---

## Integration Points

| Service | Role |
|---|---|
| PROV / MOIRAI | Writes PrivilegeAnnotated events; reads provenance DAG for privilege propagation |
| PGS / NOMOS | PCES runs after PGS in the evaluation pipeline; PGS governs interactions, PCES governs data |
| FGS / PLUTUS | PCES hard stops feed into the HITL Hold Queue via budget ceiling integration |
| TVS / KAIROS | Privilege-annotated chunks carry validity context from TVS |

---

## Implementation Roadmap

### Phase 1 — Core Enforcement (Weeks 1–6)
- Privilege classification schema; PrivilegeRecord attached to every ingested chunk
- Matter scope enforcement: cross-matter retrieval blocked by default
- Basic privilege filter: attorney_client and work_product blocking
- PrivilegeFilterEvent log wired to Provenance Service
- Privilege log export: machine-readable and formatted for discovery production

### Phase 2 — CoI & Consent (Weeks 7–14)
- Conflict Graph deployment: entity nodes, relationship types, temporal bounds
- CoI check integrated into pre-prompt filter pipeline
- CoI dashboard: active alerts, resolution workflow
- ConsentRecord schema; `permits_ai_use` field populated at intake
- Third-party confidentiality flagging with attorney confirmation gate

### Phase 3 — Propagation & Redaction (Weeks 15–20)
- Privilege propagation through provenance DAG: AI outputs inherit ancestor privilege
- Targeted redaction: partial-chunk privilege masking with span logging
- Waiver event handling: recorded waiver clears privilege for downstream retrieval
- Cross-matter authorization: attorney-granted exceptions with full audit trail

---

**Depends on:** None
**Feeds into:** PROV / MOIRAI (privilege annotations), PGS / NOMOS (pipeline integration), all services (safe operating envelope)
