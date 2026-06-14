# DPS — Document Provenance Service
### CODEX · *"Latin for a bound manuscript — the physical embodiment of recorded knowledge, as opposed to the scroll; that which holds the record in fixed, citable form"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `DPS` |
| **Epithet** | `CODEX` |
| **Full name** | Document Provenance Service |
| **Namespace** | `themis-core` |
| **Layer** | Core Infrastructure |
| **Build phase** | Phase 3–4 (Weeks 9–28) |
| **Build priority** | 14 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — tracks the provenance of analytical documents from creation through dissemination |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**DPS/CODEX answers: Where did this finished analytical document come from — which sessions, turns, claims, and sources contributed to it, and how was AI used in its production?**

### 1.2 Why This Service Exists

MOIRAI tracks the provenance of AI interactions at the session and turn level. What happens after those interactions produce a finished intelligence product is currently outside the provenance chain. An analyst who uses five ATHENA sessions across a week to draft a finished assessment has a MOIRAI record for each session — but no record linking those sessions to the finished document, no record of what AI-assisted claims were incorporated into the final text, and no mechanism to produce the AI usage disclosure that oversight bodies and dissemination policies require.

DPS/CODEX closes the gap between the session-level provenance record and the document-level accountability record. It tracks the lineage of analytical documents from draft through revision to final dissemination — linking each document to the sessions and claims that contributed to it, generating AI usage disclosures that comply with emerging IC AI disclosure requirements, and producing the document-level provenance certificate that PCS/IRIS embeds in consumer packages.

### 1.3 Why This Service Is Fourteenth

DPS requires MOIRAI's session-level events to exist before it can link documents to contributing sessions. It is Phase 3-4 because documents are produced from sessions, and sessions begin in Phase 1-2. DPS's deployment must precede any requirement for AI usage disclosure on finished products — which means it should be operational before analysts begin producing finished intelligence products using ATHENA.

### 1.4 Design Principles

- **Document provenance is the session provenance extended forward.** DPS does not duplicate MOIRAI's session record. It creates a mapping layer between sessions and documents, extending the provenance chain from session-level events to document-level accountability.
- **AI usage disclosure is generated from the document record, not from analyst declaration.** The AI usage disclosure that DPS generates reflects what actually happened in the contributing sessions — not what the analyst reports. The disclosure is derived from MOIRAI events, making it auditable and tamper-evident.
- **Document versioning is first-class.** Analytical documents go through drafts. DPS tracks each version as a distinct provenance record — which sessions contributed to which version, and which sessions produced the final version. Revisions are not overwritten; they are appended.
- **Dissemination is an event in the provenance chain.** When a document is disseminated, DPS records this as a signed MOIRAI event. The dissemination event is the trigger for PCS/IRIS to generate the consumer package.

### 1.5 Explicit Out of Scope

- **Document content storage.** DPS tracks document metadata and provenance. It does not store the document content. The document management system (external to THEMIS) holds the content; DPS holds the provenance record.
- **Access control on finished documents.** Document access control is managed by the dissemination system. DPS records what was disseminated; it does not enforce who can access it.
- **Editorial review.** DPS records the analytical process; it does not evaluate the analytical quality of the finished product.

---

## 2. Core Responsibilities

### 2.1 Primary Function

DPS/CODEX tracks the lifecycle of analytical documents — from draft creation through revision to final dissemination — maintaining a provenance record that links each document version to the ATHENA sessions, turns, and claims that contributed to it. It generates AI usage disclosure reports for finished intelligence products that reflect the actual AI assistance used (derived from MOIRAI events, not analyst declaration) and publishes dissemination events to the MOIRAI provenance chain.

### 2.2 Secondary Functions

- Session-to-document linkage: recording which ATHENA sessions contributed to which document versions
- Claim-level contribution tracking: tracking which AI-assisted claims from ATHENA sessions were incorporated into the finished document
- AI usage disclosure generation: producing structured AI usage disclosures in configurable formats for dissemination packages and bar association / oversight body reporting
- Revision history: maintaining a full version history of the document's evolution with contributing sessions per version
- Dissemination event recording: generating signed MOIRAI events when documents are disseminated
- PCS/IRIS integration: providing document provenance records for consumer package generation

### 2.3 What This Service Does Not Decide

DPS records what AI was used in document production. Whether a document meets disclosure requirements, whether additional human review is required before dissemination, and whether AI usage in this document is appropriate under current policy are human decisions owned by the analyst, supervisor, and dissemination authority. DPS provides the record; humans decide the adequacy.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
DocumentRecord:
  document_id:           uuid
  external_doc_id:       str              # ID in the external document management system
  document_type:         FINISHED_INTEL | DRAFT | WORKING_PAPER | ASSESSMENT | BRIEFING
  classification:        str
  compartments:          [str]
  title_hash:            str              # SHA-256 of title (not stored plain)
  created_at:            datetime
  created_by:            str              # analyst ID hash
  status:                DRAFT | REVIEW | FINAL | DISSEMINATED | SUPERSEDED

DocumentVersion:
  version_id:            uuid
  document_id:           uuid
  version_number:        int
  content_hash:          str              # SHA-256 of document content at this version
  contributing_sessions: [uuid]           # FK → MOIRAI session records
  contributing_turns:    [uuid]           # FK → MOIRAI turn records
  contributing_claims:   [ClaimContribution]
  created_at:            datetime
  created_by:            str              # analyst ID hash
  version_notes:         str | null

ClaimContribution:
  contribution_id:       uuid
  version_id:            uuid
  claim_id:              uuid             # FK → MOIRAI claim record
  claim_text_hash:       str
  claim_type:            str
  source_type:           str              # GRND | PARAM | SYNTH | etc.
  incorporated:          bool             # was this claim incorporated into the document text?
  verified:              bool             # was this claim verified before incorporation?
  verification_action:   str | null       # what verification action was taken

DisseminationRecord:
  dissemination_id:      uuid
  document_id:           uuid
  version_id:            uuid
  disseminated_at:       datetime
  disseminated_by:       str
  dissemination_channel: str              # channel / system / distribution list (hashed)
  ai_usage_disclosure_id:uuid             # FK → AiUsageDisclosure
  consumer_package_id:   uuid | null      # FK → PCS/IRIS consumer package

AiUsageDisclosure:
  disclosure_id:         uuid
  document_id:           uuid
  version_id:            uuid
  generated_at:          datetime
  disclosure_format:     str              # format specifier (jurisdiction/policy)
  ai_assisted:           bool
  session_count:         int              # number of contributing ATHENA sessions
  claim_count:           int              # AI-assisted claims in document
  unverified_claim_count:int
  param_claim_count:     int              # claims with no retrieved source
  verification_rate:     float            # fraction of AI claims that were verified
  model_versions_used:   [str]
  disclosure_text:       str              # generated plain-language disclosure
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | DocumentRecord, DocumentVersion, ClaimContribution, DisseminationRecord, AiUsageDisclosure | Session + 7 years |
| Event store | MOIRAI | Signed document and dissemination events | Indefinite |
| Disclosure cache | Redis | Generated disclosures (heavy computation; cacheable) | 24h TTL |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| DocumentRecord | Document classification | Compartment-gated per document |
| ClaimContribution | Inherits document classification | Session-compartmented |
| AiUsageDisclosure | Configurable (may be lower than document) | Per dissemination policy |
| DisseminationRecord | Document classification | IOB access |

### 3.4 Retention and Purge Policy

DocumentRecord and all linked records retained for document lifecycle plus seven years minimum. DisseminationRecord retained indefinitely — dissemination history is a permanent accountability record. AiUsageDisclosure retained indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /documents
  Auth:     ATHENA service account | analyst session token
  Request:  {
    external_doc_id:     str,
    document_type:       str,
    classification:      str,
    compartments:        [str],
    contributing_session_id: uuid
  }
  Response: { document_id: uuid, version_id: uuid }

POST /documents/{document_id}/versions
  Auth:     ATHENA service account | analyst session token
  Request:  {
    content_hash:        str,
    contributing_sessions:[uuid],
    contributing_turns:  [uuid],
    version_notes:       str | null
  }
  Response: { version_id: uuid, version_number: int }

POST /documents/{document_id}/versions/{version_id}/claims
  Auth:     ATHENA service account
  Request:  {
    claim_id:            uuid,
    incorporated:        bool,
    verified:            bool,
    verification_action: str | null
  }
  Response: { contribution_id: uuid }

POST /documents/{document_id}/disseminate
  Auth:     supervisor token | dissemination authority token
  Request:  {
    version_id:          uuid,
    disseminated_by:     str,
    dissemination_channel_hash: str,
    disclosure_format:   str
  }
  Response: {
    dissemination_id:    uuid,
    disclosure:          AiUsageDisclosure,
    moirai_event_id:     uuid
  }
  SLA: p99 < 2000ms (disclosure generation is non-trivial)

GET /documents/{document_id}/provenance
  Auth:     session token | supervisor token | IOB token
  Response: {
    document:            DocumentRecord,
    versions:            [DocumentVersion],
    disseminations:      [DisseminationRecord]
  }

GET /documents/{document_id}/disclosure/{version_id}
  Auth:     supervisor token | dissemination authority | IOB token
  Response: AiUsageDisclosure

GET /documents/{document_id}/disclosure/{version_id}?format={format}
  Auth:     same as above
  Response: { disclosure_text: str, format: str }

GET /audit/ai-usage-summary?from={date}&to={date}
  Auth:     IOB token
  Response: {
    period:              { from, to },
    documents_disseminated: int,
    ai_assisted_count:   int,
    mean_verification_rate: float,
    unverified_claims_in_products: int
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces },
    documents_tracked:   int,
    disclosures_generated_24h: int,
    last_event_hash:     str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          DPS_DOCUMENT_CREATED
service_id:         "DPS"
session_id:         null             # document-level event
classification:     str
event_payload:
  document_id:            uuid
  document_type:          str
  version_id:             uuid
  contributing_sessions:  int
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          DPS_DOCUMENT_DISSEMINATED
event_payload:
  document_id:            uuid
  version_id:             uuid
  session_count:          int
  claim_count:            int
  unverified_claim_count: int
  verification_rate:      float
  disclosure_format:      str

EventType:          DPS_UNVERIFIED_CLAIMS_DISSEMINATED
event_payload:
  document_id:            uuid
  unverified_count:       int
  total_claim_count:      int
  dissemination_id:       uuid
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `DPS_DOCUMENT_CREATED` | New document record created | MOIRAI |
| `DPS_DOCUMENT_DISSEMINATED` | Document disseminated | MOIRAI, PCS/IRIS (consumer package trigger) |
| `DPS_UNVERIFIED_CLAIMS_DISSEMINATED` | Document with unverified AI claims disseminated | MOIRAI, supervisor notification, IOB reporting |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| MOIRAI | `SESSION_TURN_CREATED` | Available for session-to-document linking when analyst associates session with document |
| ERAS/LOGOS | `ERAS_CAPTURE_CREATED` | Reasoning captures linked to document version when available |
| TVS/KAIROS | `TVS_SOURCE_INVALIDATED` | Document versions using that source flagged for review |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Session and turn records linked to documents; dissemination events | Async event + sync query | Events buffered; document creation still proceeds |
| PCES/AEGIS | Classification Enforcement | Document compartment validation on all endpoints | Sync | Proceeds with cached session scope |

### 5.2 Feeds Into

| Service | Epithet | What DPS provides | How |
|---|---|---|---|
| PCS/IRIS | Policymaker Communication | Document provenance for consumer package generation | `DPS_DOCUMENT_DISSEMINATED` event + API |
| IOB Reporting | Oversight | AI usage summary; unverified claim dissemination alerts | Audit endpoints |
| ATHENA | Interface | Document creation/versioning workflow | API |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** All document endpoints validate session token against document classification scope.
- **Compartment inheritance:** DocumentRecord inherits the analyst's session compartments. A document cannot be classified above the analyst's clearance.
- **Failure behavior:** PCES unavailable → document creation and version endpoints unavailable; dissemination blocked without access verification.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Document creation | 100ms | 300ms | 500ms |
| Version creation | 100ms | 300ms | 500ms |
| Disclosure generation | 500ms | 1500ms | 2000ms |
| Dissemination (with disclosure) | 1000ms | 2000ms | 3000ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Document operations/second | 20 |
| Dissemination events/hour | 50 |
| Disclosure generations/hour | 100 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.5% — DPS unavailability prevents document dissemination with AI disclosure |
| MOIRAI event durability | 99.999% |
| RTO | 15 minutes |
| RPO | 5 minutes |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Dissemination events buffered; document operations still proceed | Provenance gap logged; analyst can still create and version documents |
| PCES | Document creation and dissemination blocked | Session must re-authenticate before document operations |

---

## 7. Security Model

### 7.1 Authentication

Analysts use session tokens for document creation and versioning. Dissemination requires supervisor or dissemination authority token. IOB audit endpoints require IOB token.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Analyst (own session) | Document creation; version creation; claim contribution | Session token |
| Supervisor | Dissemination; document review; team provenance | Supervisor token |
| Dissemination authority | `POST /documents/{id}/disseminate` | Authority token |
| PCS/IRIS | Document provenance for consumer package | Service account |
| IOB | Full provenance; AI usage audit | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/dps/signing-key` | 90 days |
| PostgreSQL credentials | `themis/dps/db-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Incomplete session linkage (analyst forgets to associate session with document) | High | P2 — AI usage disclosure understates AI contribution | Analyst training; ATHENA workflow prompt | ATHENA surfaces session-to-document association prompt at session close |
| Unverified claim dissemination without supervisor awareness | Medium | P1 — finished product contains unverified AI claims | `DPS_UNVERIFIED_CLAIMS_DISSEMINATED` event | Supervisor notification on dissemination with unverified claims; threshold-based blocking |

### 8.1 Known Design Risks

- **Session-to-document linkage depends on analyst workflow behaviour.** DPS can only link sessions to documents if the analyst explicitly associates them. An analyst who uses ATHENA informally and writes the finished document outside the workflow will have no DPS record. Mitigation: ATHENA prompts session-to-document association at session close; the prompt cannot be dismissed without a response (associate or mark as not contributing to a document).
- **AI usage disclosure accuracy depends on claim contribution tracking.** The disclosure's claim count and verification rate are only accurate if all AI-assisted claims are submitted via `POST /documents/{id}/versions/{id}/claims`. If the analyst incorporates AI claims into the document without registering them with DPS, the disclosure will understate AI contribution. Mitigation: ATHENA workflow integration should auto-populate claim contributions from the session's verified claims.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `dps.dissemination.latency_p99` | Histogram | `> 3000ms for 5m` | P1 |
| `dps.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `dps.unverified_dissemination_rate` | Gauge | `> 10%` of disseminations | P2 |
| `dps.disclosure.generation_error_rate` | Counter | `> 1%` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces },
  documents_tracked:         int,
  disseminations_24h:        int,
  unverified_dissemination_rate_24h: float,
  last_event_hash:           str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":          "ISO-8601",
  "service":            "DPS/CODEX",
  "event":              "DOCUMENT_CREATED | VERSION_CREATED | DOCUMENT_DISSEMINATED | UNVERIFIED_DISSEMINATED",
  "document_id":        "uuid",
  "version_id":         "uuid | null",
  "contributing_sessions": 0,
  "unverified_claims":  0,
  "duration_ms":        0
}
```

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/dps/signing-key`
- **Chain participation:** Yes
- **What it attests:** The `DPS_DOCUMENT_DISSEMINATED` event provides an unalterable record of what AI assistance contributed to a finished product at dissemination time — session count, claim count, verification rate, model versions used. An oversight body can verify the AI usage disclosure matches the MOIRAI-attested dissemination event.

---

## 11. Implementation Roadmap

### Phase 1 — Document Lifecycle Tracking (Weeks 9–16)

- DocumentRecord, DocumentVersion, ClaimContribution schemas
- Document creation, versioning, and claim contribution endpoints
- Session-to-document linkage via contributing_sessions field
- MOIRAI event emission: `DPS_DOCUMENT_CREATED`
- ATHENA session-close prompt for document association

**Phase gate criterion:** Every analytical document created in ATHENA has a DPS record. Session-to-document linkage recorded for at least 90% of contributing sessions in test. MOIRAI events chained correctly.

### Phase 2 — Disclosure, Dissemination, and IOB Interface (Weeks 17–28)

- AiUsageDisclosure generation with configurable format support
- Dissemination endpoint with disclosure generation and MOIRAI event
- `DPS_DOCUMENT_DISSEMINATED` and `DPS_UNVERIFIED_CLAIMS_DISSEMINATED` events
- PCS/IRIS integration for consumer package trigger
- Supervisor notification on unverified claim dissemination
- IOB audit endpoints

**Phase gate criterion:** AI usage disclosure generated correctly from MOIRAI session records. Unverified claim dissemination triggers supervisor notification. PCS/IRIS receives dissemination event and generates consumer package. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate DPS deployment. The AI usage disclosure format configuration (which jurisdiction-specific formats to support) is a policy decision owned by the analytic standards authority that does not gate the technical service.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

At ATHENA session close, a prompt appears: "Did this session contribute to a finished analytical product? [Yes — link to document] [No — session only]." If Yes, the analyst selects or creates the associated document record. During document drafting, ATHENA surfaces the list of AI-assisted claims that have been incorporated into the current version, with their verification status, so the analyst can see the AI contribution in context.

### 13.2 What the Analyst Should Do

Associate every contributing ATHENA session with the document it contributes to. The AI usage disclosure is only accurate if session linkage is complete. Mark each AI claim as incorporated or not incorporated when drafting. A claim marked as incorporated with is_verified=false will be flagged in the disclosure — verify before dissemination or document the reason for unverified use explicitly.

### 13.3 What the Signal Does Not Mean

An AI usage disclosure showing zero unverified claims does not mean all claims in the document are accurate — it means all AI-assisted claims that the analyst incorporated were verified before incorporation. The verification quality still depends on how rigorously the analyst verified each claim.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Disclosure format configuration scope.** AI disclosure requirements are evolving across oversight bodies. The disclosure format must be configurable without engineering changes. Resolution path: disclosure format is driven by a format template registry maintained by the analytic standards authority; new formats require only template additions, not code changes.

### 14.2 Operational Assumptions

- **Assumption 1: The external document management system has an API for document ID registration.** DPS tracks provenance by external document ID. If the document management system does not expose an API, DPS cannot link to it automatically and session-to-document linkage must be manual. Resolution path: confirm document management system API access before Phase 1.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 3 gate review.*
