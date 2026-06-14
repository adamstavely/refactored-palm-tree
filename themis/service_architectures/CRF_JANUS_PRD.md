# CRF — Cross-Requirement Fusion Service
### JANUS · *"Roman god of beginnings, transitions, and duality — depicted with two faces looking in opposite directions simultaneously; he who sees both the question that was asked and the larger question behind it"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `CRF` |
| **Epithet** | `JANUS` |
| **Full name** | Cross-Requirement Fusion Service |
| **Namespace** | `themis-warning` |
| **Layer** | Intelligence Layer — Warning |
| **Build phase** | Year 3 |
| **Build priority** | 13 of 15 intelligence layer services |
| **Owner team** | Intelligence Layer Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — identifies when separate requirements address the same underlying intelligence problem |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**CRF/JANUS answers: Are there separate analytical requirements currently active in THEMIS that are unknowingly addressing different facets of the same underlying intelligence problem — and what do the cross-requirement patterns reveal that neither requirement would surface individually?**

### 1.2 Why This Service Exists

Intelligence requirements are generated independently by different customers, tasked to different collection systems, and analysed by different analytical teams. The left hand may not know what the right hand is asking. Two separate requirements might be approaching the same adversary programme from different analytical angles — one asking about technical capability, another asking about organisational indicators — without either knowing the other exists or that their combined picture is more revealing than either view alone.

JANUS identifies these connections. When two requirements share sufficient entity, domain, or analytical overlap, it examines whether their combined intelligence picture reveals something that neither would surface alone — a contradiction that needs resolution, a complementary coverage situation where one requirement fills the gap the other has, or a synthesis opportunity where the combination yields a stronger analytical conclusion.

This is also the service that surfaces analytical contradictions across requirements. If Requirement A's ATHENA sessions have established X with high confidence and Requirement B's ATHENA sessions have established the contradiction of X with medium confidence, neither team may know about the other's conclusion. JANUS surfaces the contradiction so it can be resolved rather than allowed to persist in separate analytical silos.

### 1.3 Design Principles

- **Cross-requirement visibility is itself a sensitive capability.** Knowing that Requirement A and Requirement B are related implies knowledge of both requirements. JANUS's cross-requirement analysis is subject to strict compartment controls — analysts working on Requirement A do not automatically see the details of Requirement B, even when JANUS identifies a connection.
- **Contradiction identification is more valuable than synthesis identification.** Finding two requirements that would reinforce each other is valuable. Finding two requirements whose conclusions contradict each other is urgent. JANUS prioritises contradiction signals in its output.
- **Connection does not imply combination.** JANUS identifies that two requirements may be productively connected and notifies the appropriate supervisors. The decision to combine, share, or synthesise the analytical work is a human management decision, not an automated system action.

---

## 2. Core Responsibilities

### 2.1 Primary Function

CRF/JANUS maintains profiles of all active and recently completed THEMIS analytical requirements, computes pairwise overlap scores across domain, entity, claim type, and collection method dimensions, identifies CONTRADICTION, COMPLEMENTARY_COVERAGE, and SYNTHESIS_OPPORTUNITY relationships between requirement pairs, and notifies supervisors of high-priority connections requiring management attention.

### 2.2 Secondary Functions

- Analytical contradiction tracking: monitoring when requirements produce conflicting conclusions that need resolution
- Coverage complement mapping: identifying when one requirement has collection coverage that fills another requirement's gaps
- Synthesis opportunity identification: recognising when two requirements together would produce a more complete analytical picture
- Cross-requirement entity map: building a map of which entities appear across which requirements (compartment-controlled)
- Supervisor notification routing: routing connection alerts to supervisors with appropriate access to both requirements

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
ActiveRequirementProfile:
  profile_id:              uuid
  requirement_id:          str
  domain:                  str
  entity_ids:              [uuid]           # OGS entities — compartment-scoped
  claim_types:             [str]
  collection_methods_used: [str]
  analytical_positions:    [str]            # current established positions (hashed)
  classification:          str
  compartment:             str
  session_count:           int
  last_active:             datetime

RequirementConnection:
  connection_id:           uuid
  requirement_a_id:        str
  requirement_b_id:        str
  connection_type:         CONTRADICTION | COMPLEMENTARY_COVERAGE | SYNTHESIS_OPPORTUNITY | ENTITY_OVERLAP
  strength:                float
  description:             str
  compartment_a:           str
  compartment_b:           str
  requires_clearance:      str              # minimum clearance to view connection detail
  supervisor_notified:     bool
  supervisor_id_hash:      str | null
  resolved:                bool
  resolution:              str | null
  detected_at:             datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Requirement profiles | PostgreSQL | ActiveRequirementProfile | Per requirement lifecycle + 2 years |
| Connections | PostgreSQL | RequirementConnection | Indefinite |
| Event store | MOIRAI | Signed connection events | Indefinite |

---

## 4. API Contract

### 4.1 Endpoints

```
GET /connections/active
  Auth:     supervisor token (compartment-scoped to requirements they have access to)
  Params:   type: str, strength_min: float
  Response: [RequirementConnection (detail redacted to compartment access)]

GET /connections/{connection_id}
  Auth:     supervisor token with access to both compartments
  Response: RequirementConnection with full description

POST /connections/{connection_id}/resolve
  Auth:     supervisor token
  Request:  { resolution: str }
  Response: { connection_id: uuid, resolved: bool }

GET /requirements/{requirement_id}/connections
  Auth:     supervisor token with compartment access
  Response: [RequirementConnection]

GET /health
  Response: {
    status, dependencies: { moirai, ogs, mirror, pces },
    active_requirements:   int,
    active_connections:    int,
    unresolved_contradictions:int,
    last_event_hash:       str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          CRF_CONNECTION_DETECTED
service_id:         "CRF"
session_id:         null
classification:     str
event_payload:
  connection_id:          uuid
  connection_type:        str
  strength:               float
  requires_clearance:     str
  supervisor_notified:    bool
prev_event_hash:    str
signature:          str
timestamp:          datetime
```

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MIRROR | Requirement Similarity | Requirement profiles for overlap computation | API | Connection detection degraded |
| OGS/YGGDRASIL | Ontology | Entity overlap across requirements | Sync query | Entity overlap dimension unavailable |
| PCES/AEGIS | Classification | Compartment enforcement on cross-requirement access | Sync | Cross-compartment connections not surfaced |

### 5.2 Feeds Into

| Service | Epithet | What CRF provides | How |
|---|---|---|---|
| SENTINEL | Strategic Warning | Cross-requirement contradiction and synthesis signals | Event |
| Supervisors | Management | Connection notifications requiring management decision | Notification |

---

## 6. Known Design Risks

- **Cross-requirement analysis creates a classification aggregation problem.** Knowing that Requirements A and B are related in a specific way may reveal more than either requirement individually. JANUS connections must carry the higher of the two requirements' classifications. Access control on connection detail requires clearance for both requirements. The REQUIRES_CLEARANCE field implements this; the notification routing must validate before delivering.

---

## 7. Implementation Roadmap

### Phase 1 — Requirement Profiling and Overlap Detection (Year 3, Weeks 1–12)

- ActiveRequirementProfile schema from MIRROR profiles
- Pairwise overlap computation (domain, entity, claim type)
- CONTRADICTION and COMPLEMENTARY_COVERAGE detection
- Compartment-controlled supervisor notification
- MOIRAI events

**Phase gate criterion:** Overlap correctly computed for requirement pairs with known relationships in test data. Compartment controls prevent cross-compartment connection detail access without appropriate clearance. Supervisor notification delivered correctly.

### Phase 2 — Synthesis Opportunity, SENTINEL Integration, and Resolution Tracking (Year 3, Weeks 13–24)

- SYNTHESIS_OPPORTUNITY detection
- SENTINEL connection feed
- Resolution tracking and lifecycle management

**Phase gate criterion:** Synthesis opportunities identified on test requirement sets. Resolution workflow logs to MOIRAI. ARB sign-off.

---

## 8. Policy Dependencies

No GC items gate JANUS. Cross-requirement visibility policy is an analytic standards authority operational policy — which supervisors may see which cross-requirement connections is a management policy, not a GC policy decision.

---

## 9. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | Intelligence Layer Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Year 3 gate review.*
