# KCS — Knowledge Currency Service
### ARGUS · *"Greek for Argus Panoptes — the hundred-eyed giant who sees all, the all-watching sentinel"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `KCS` |
| **Epithet** | `ARGUS` |
| **Full name** | Knowledge Currency Service |
| **Namespace** | `themis-quality` |
| **Layer** | Knowledge Layer |
| **Build phase** | Phase 7–8 (Weeks 47–66) |
| **Build priority** | 13 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Currency — monitors external indicators that supersede corpus sources and model parametric knowledge |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**KCS/ARGUS answers: Has the world changed in ways that supersede what is in our corpus — and does the model's parametric knowledge still reflect the current state of the domains it is being asked about?**

### 1.2 Why This Service Exists

TVS/KAIROS tracks the validity of individual sources as time passes, applying decay models and processing invalidation events it receives. What TVS does not do is watch for the events that should trigger those invalidation signals. Something must generate `KCS_SOURCE_SUPERSEDED` events — something must be watching.

KCS/ARGUS is the watcher. It monitors external indicator feeds, new collection ingestion, and watchlist conditions to detect when the external world has changed in ways that render corpus sources stale. It also monitors a distinct but related currency problem: model parametric knowledge. PARAM claims rely on what the model learned during training. A model trained with data through a certain date may have accurate parametric knowledge in slow-moving domains but systematically stale parametric knowledge in fast-moving ones. KCS provides the model knowledge currency signal that UCS/TYCHE uses to calibrate epistemic dominance for PARAM claims by domain.

Without KCS, TVS/KAIROS processes invalidation events it receives but cannot detect invalidation events it has not been told about. KCS is the detection layer. TVS is the propagation layer. Both are required for the Currency axis to function.

### 1.3 Why This Service Is Thirteenth

KCS is Phase 7–8 because it requires a mature corpus (significant sources to monitor), a mature TVS/KAIROS (to propagate the supersession events KCS generates), and a mature MOIRAI provenance graph (to determine blast radius). It is also placed late because the external indicator monitoring infrastructure — watchlist management, event feed integration — requires operational standing that earlier platform phases don't need. KCS also depends on ERAS/LOGOS reasoning records to identify which domains are most dependent on current parametric knowledge, which requires ERAS to have accumulated data.

### 1.4 Design Principles

- **KCS detects; TVS propagates.** KCS generates supersession signals. TVS/KAIROS receives them and propagates validity updates through the provenance graph. The two services are coupled but distinct. KCS does not update validity records directly.
- **Watchlists are policy artefacts, not technical configurations.** What external events trigger source supersession is an analytical judgment. Watchlist entries are maintained by the analytic standards authority, not the platform team.
- **Model knowledge currency is domain-specific.** A model's parametric knowledge has different effective currency windows in different domains. Fast-moving geopolitical domains have effectively expired parametric knowledge within weeks of the training cut-off. Stable technical or historical domains may retain accurate parametric knowledge for years. KCS tracks this at the domain level.
- **Coverage maps are the positive complement to ARGUS-LACUNA's gap maps.** KCS maintains a map of what collection coverage exists. ARGUS-LACUNA (CGS) uses the inverse of this to surface collection gaps. The two services are designed to work together as the visibility and gap-detection system for the Currency axis.

### 1.5 Explicit Out of Scope

- **Executing collection.** KCS identifies when new collection has superseded corpus sources and when coverage gaps exist. It does not task collection — that is TIS/DIKE via the gap-to-requirement pipeline.
- **Invalidating sources directly.** KCS generates supersession signals that TVS processes. KCS does not modify validity records.
- **Monitoring sources for accuracy.** KCS monitors for external events that make sources stale. Whether sources were accurate when ingested is CVS/VERITAS's concern.

---

## 2. Core Responsibilities

### 2.1 Primary Function

KCS/ARGUS maintains a watchlist of external indicators and continuously monitors for events that would supersede corpus intelligence sources — generating `KCS_SOURCE_SUPERSEDED` events for TVS/KAIROS when supersession conditions are met. It also tracks the effective currency window of the model's parametric knowledge by domain, providing a `ModelKnowledgeCurrencyRecord` that UCS/TYCHE consumes when characterising epistemic uncertainty for PARAM claims. Additionally, it maintains the corpus coverage map that ARGUS-LACUNA (CGS) uses as its positive-coverage baseline.

### 2.2 Secondary Functions

- New collection currency processing: when new collection is ingested that covers the same topic as an existing corpus source, assessing whether the new collection supersedes the prior source
- Watchlist management: CRUD operations on the WatchlistEntry catalog (owned by analytic standards authority)
- Knowledge cut-off monitoring: tracking model training cut-off dates by domain and computing effective parametric knowledge windows
- Coverage map maintenance: domain × collection_method coverage density map for ARGUS-LACUNA
- Pre-scheduled blast radius computation: running MOIRAI blast radius queries on a schedule for high-priority sources, so that on-demand blast radius is faster when needed

### 2.3 What This Service Does Not Decide

KCS detects potential supersession and generates signals. Whether a detected supersession event is analytically significant, whether an analyst should update their assessment in light of a supersession, and whether a source should be fully invalidated or merely flagged are analytical and operational decisions. KCS generates the signal; humans and TVS/KAIROS determine consequences.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
WatchlistEntry:
  entry_id:              uuid
  domain:                str              # analytical domain monitored
  indicator_type:        EVENT | COLLECTION | ASSESSMENT | EXTERNAL_FEED
  indicator_description: str
  monitored_source_ids:  [uuid]           # corpus sources this entry monitors
  supersession_condition:str              # condition that triggers supersession
  confidence_threshold:  float
  maintained_by:         str              # analytic standards authority reference
  version:               str
  effective_from:        datetime
  active:                bool

SupersessionSignal:
  signal_id:             uuid
  watchlist_entry_id:    uuid
  source_id:             uuid             # corpus source being superseded
  superseded_by_id:      uuid | null      # new source that supersedes it, if applicable
  signal_type:           NEW_COLLECTION | EXTERNAL_EVENT | ASSESSMENT_REVERSAL | MANUAL
  confidence:            HIGH | MEDIUM | LOW
  basis:                 str              # plain language: what triggered this
  processed_by_tvs:      bool
  tvs_event_id:          uuid | null
  generated_at:          datetime

ModelKnowledgeCurrencyRecord:
  record_id:             uuid
  model_version:         str
  domain:                str
  training_cutoff_date:  datetime         # when model training data for this domain ends
  effective_currency_days: int            # how many days the parametric knowledge remains useful
  currency_status:       CURRENT | DEGRADED | STALE | UNRELIABLE
  degradation_curve:     FAST | MEDIUM | SLOW  # how quickly parametric knowledge expires
  last_assessed:         datetime
  assessed_by:           str              # analytic standards authority or Research & Red Team

CoverageMapEntry:
  entry_id:              uuid
  domain:                str
  collection_method:     SIGINT | HUMINT | GEOINT | OSINT | TECHINT | MASINT | ALLSOURCE
  coverage_density:      HIGH | MEDIUM | LOW | SPARSE | NONE
  last_collection_date:  datetime | null
  source_count:          int
  validity_weighted_count: float          # sources weighted by TVS validity score
  last_updated:          datetime
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | WatchlistEntry, SupersessionSignal, ModelKnowledgeCurrencyRecord, CoverageMapEntry | Indefinite |
| Coverage map cache | Redis | CoverageMapEntry (queried frequently by ARGUS-LACUNA) | 1h TTL + invalidation |
| Event store | MOIRAI | Signed currency and supersession events | Indefinite |
| External event buffer | Kafka | External feed events awaiting processing | 24h |

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| WatchlistEntry | Classification of the monitored domain | Compartment-gated; watchlist content is controlled |
| SupersessionSignal | Inherits monitored source classification | Compartment-gated |
| ModelKnowledgeCurrencyRecord | Controlled Unclassified | Platform-wide; accessible to UCS/TYCHE |
| CoverageMapEntry | Inherits domain classification | Compartment-gated; aggregate accessible to ARGUS-LACUNA |

### 3.4 Retention and Purge Policy

WatchlistEntry records are retained indefinitely (version-controlled). SupersessionSignal records are retained indefinitely — the history of what triggered source supersession is required for retrospective analysis. ModelKnowledgeCurrencyRecord retained per model version indefinitely. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
GET /currency/{source_id}
  Auth:     TVS service account | any session token (PCES-scoped)
  Response: {
    source_id:             uuid,
    watchlist_active:      bool,       # is this source actively monitored?
    supersession_signals:  [SupersessionSignal],
    overall_status:        CURRENT | SIGNAL_PENDING | SUPERSEDED
  }
  SLA: p99 < 200ms

POST /supersession/signal
  Auth:     corpus ingestion service account | KCS internal
  Request:  {
    watchlist_entry_id:    uuid | null,
    source_id:             uuid,
    superseded_by_id:      uuid | null,
    signal_type:           str,
    confidence:            str,
    basis:                 str
  }
  Response: {
    signal_id:             uuid,
    tvs_notified:          bool        # whether TVS was immediately notified
  }

GET /model-currency/{domain}
  Auth:     UCS service account | any service account
  Params:   model_version: str
  Response: ModelKnowledgeCurrencyRecord
  SLA: p99 < 100ms (cached)

GET /coverage-map/{domain}
  Auth:     ARGUS-LACUNA service account | IOB token
  Response: [CoverageMapEntry]         # all collection methods for this domain
  SLA: p99 < 100ms (cached)

GET /coverage-map/summary
  Auth:     ARGUS-LACUNA service account | supervisor token | IOB token
  Response: {
    domains:               int,
    high_coverage_count:   int,
    sparse_count:          int,
    none_count:            int
  }

# Watchlist management — owned by analytic standards authority
POST /watchlist
  Auth:     analytic standards authority token
  Request:  WatchlistEntry (without entry_id)
  Response: { entry_id: uuid }

GET /watchlist/{entry_id}
  Auth:     supervisor token | IOB token
  Response: WatchlistEntry

PATCH /watchlist/{entry_id}
  Auth:     analytic standards authority token
  Request:  { active: bool, indicator_description: str | null }
  Response: { entry_id: uuid, updated: bool }

GET /health
  Response: {
    status, dependencies: { moirai, tvs, redis, kafka },
    active_watchlist_entries:  int,
    signals_generated_24h:     int,
    coverage_map_domains:      int,
    last_event_hash:           str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          KCS_SOURCE_SUPERSEDED
service_id:         "KCS"
session_id:         null             # corpus-level event, not session-scoped
classification:     str
event_payload:
  source_id:              uuid
  superseded_by_id:       uuid | null
  signal_type:            str
  confidence:             str
  watchlist_entry_id:     uuid | null
  basis:                  str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          KCS_MODEL_CURRENCY_UPDATED
event_payload:
  model_version:          str
  domain:                 str
  currency_status:        str
  effective_currency_days:int

EventType:          KCS_COVERAGE_MAP_UPDATED
event_payload:
  domain:                 str
  collection_method:      str
  coverage_density:       str
  source_count:           int
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `KCS_SOURCE_SUPERSEDED` | Supersession signal processed | TVS/KAIROS (source invalidation), MOIRAI, ATHENA (active session currency alerts) |
| `KCS_MODEL_CURRENCY_UPDATED` | Model knowledge currency reassessed | UCS/TYCHE (epistemic dominance for PARAM claims), ATHENA |
| `KCS_COVERAGE_MAP_UPDATED` | Coverage map entry changes | ARGUS-LACUNA, TIS/DIKE (collection gap signals) |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| Corpus ingestion | New source ingested | Coverage map updated; existing sources checked for supersession |
| MDS/KRONOS | `MDS_MODEL_VERSION_CHANGED` | ModelKnowledgeCurrencyRecord reassessed for new model version |
| OFS/NEMESIS | `OFS_ASSESSMENT_DISCONFIRMED` | Potential supersession signal generated for sources underlying disconfirmed assessment |
| TVS/KAIROS | `TVS_SOURCE_INVALIDATED` | Coverage map updated to remove invalidated source from density counts |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed supersession events; blast radius pre-computation queries | Async event + Sync query | Events buffered; blast radius pre-computation deferred |
| TVS/KAIROS | Temporal Validity | Receives KCS supersession signals for propagation | Async event | Signals queued; TVS processes on recovery |
| PCES/AEGIS | Classification Enforcement | Compartment scope on watchlist and coverage map queries | Sync | Proceeds with cached session scope |

### 5.2 Feeds Into

| Service | Epithet | What KCS provides | How |
|---|---|---|---|
| TVS/KAIROS | Temporal Validity | Supersession signals for source invalidation propagation | `KCS_SOURCE_SUPERSEDED` event |
| UCS/TYCHE | Uncertainty Characterization | Model knowledge currency records for PARAM claim epistemic dominance | API |
| CGS/ARGUS-LACUNA | Collection Gap Service | Coverage map as the positive-coverage baseline | API + `KCS_COVERAGE_MAP_UPDATED` events |
| TIS/DIKE | Tasking Integration | Coverage gaps (derived from coverage map) as collection requirement inputs | Via ARGUS-LACUNA |
| ATHENA | Interface | Active session currency alerts when a monitored source is superseded | `KCS_SOURCE_SUPERSEDED` event |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Watchlist and coverage map queries are compartment-gated. A session cannot access watchlist entries for compartments outside its privilege scope.
- **Compartment inheritance:** SupersessionSignal and WatchlistEntry inherit the classification of the monitored domain.
- **Failure behavior:** PCES unavailable → queries proceed with cached session compartment scope; watchlist and coverage map data served from cache.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Source currency check | 30ms | 100ms | 200ms |
| Model currency lookup (cached) | 5ms | 20ms | 100ms |
| Coverage map query (cached) | 5ms | 20ms | 100ms |
| Supersession signal processing | 500ms | 2000ms | 5000ms |

### 6.2 Throughput

| Metric | Target |
|---|---|
| Supersession signals processed/hour | 50 |
| Coverage map queries/second | 100 |
| Watchlist evaluation cycles/day | 24 (hourly) |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — KCS unavailability means new supersession signals not generated |
| MOIRAI event durability | 99.999% |
| RTO | 30 minutes |
| RPO | 1 hour |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| TVS/KAIROS | Supersession signals queued; TVS processes on recovery | No immediate analyst impact; currency updates delayed |
| MOIRAI | Events buffered; signal processing continues | No analyst impact; provenance gap logged |
| Redis (coverage cache) | Coverage map queries from PostgreSQL (higher latency) | ARGUS-LACUNA response latency increases |
| External event feeds | Watchlist monitoring continues on cached indicators | New external events not detected until feed recovers |

---

## 7. Security Model

### 7.1 Authentication

Corpus ingestion service account for supersession signal submission. Analytic standards authority token for watchlist management. Service accounts for inter-service queries. Session tokens for analyst-facing currency checks.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Corpus ingestion | `POST /supersession/signal` | Service account |
| Analytic standards authority | Watchlist management | Authority token |
| TVS/KAIROS | Currency check | Service account |
| UCS/TYCHE | Model currency records | Service account |
| ARGUS-LACUNA | Coverage map | Service account |
| Analyst session | Source currency (compartment-scoped) | Session token |
| Supervisor / IOB | Watchlist view; coverage summary | Supervisor / IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/kcs/signing-key` | 90 days |
| PostgreSQL credentials | `themis/kcs/db-credentials` | 30 days |
| Redis credentials | `themis/kcs/redis-credentials` | 30 days |
| External feed credentials | `themis/kcs/feed-credentials` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Watchlist entry staleness (condition no longer matches domain dynamics) | Medium | P2 — supersession conditions not triggered when they should be | Analytic standards authority quarterly review | Watchlist entries have mandatory review dates; alert when overdue |
| Coverage map lag (new sources not reflected promptly) | Low | P2 — ARGUS-LACUNA reports gaps that no longer exist | Coverage map update latency monitoring | Event-driven updates on corpus ingestion |
| Model currency window miscalibration | Medium | P2 — PARAM claims characterised with wrong epistemic dominance | UCS override rate in relevant domains | Annual review of ModelKnowledgeCurrencyRecord against OFS/NEMESIS outcomes |

### 8.1 Known Design Risks

- **The watchlist is only as good as the analytic standards authority's maintenance.** Watchlist entries that are outdated, incomplete, or wrong will produce either false supersession signals or missed supersessions. The watchlist is a policy artefact maintained by humans; its quality determines KCS's value. Mitigation: mandatory quarterly review cycle for all watchlist entries; entries with overdue reviews are flagged.
- **Model knowledge currency windows are expert estimates, not empirical measurements.** The `effective_currency_days` values are analyst judgments about how quickly different domains change. These are not derived from outcome data. Mitigation: OFS/NEMESIS outcome data will eventually show which domains had PARAM claims validated despite stale model knowledge (suggesting the currency window should be extended) and which had PARAM claims disconfirmed (suggesting the window should be shortened). Calibration is a Year 3 task.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `kcs.supersession.signals_24h` | Gauge | Spike > 5x baseline | P2 |
| `kcs.watchlist.overdue_review_count` | Gauge | `> 0` | P2 |
| `kcs.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `kcs.coverage_map.update_lag_minutes` | Gauge | `> 60` after new source ingestion | P2 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, tvs, redis, kafka },
  active_watchlist_entries:  int,
  overdue_review_count:      int,
  signals_24h:               int,
  coverage_map_domains:      int,
  last_event_hash:           str
}
```

### 9.3 Log Schema

```json
{
  "timestamp":          "ISO-8601",
  "service":            "KCS/ARGUS",
  "event":              "SUPERSESSION_DETECTED | COVERAGE_UPDATED | MODEL_CURRENCY_UPDATED",
  "source_id":          "uuid | null",
  "domain":             "string",
  "signal_type":        "string",
  "confidence":         "HIGH | MEDIUM | LOW",
  "duration_ms":        0
}
```

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/kcs/signing-key`
- **Chain participation:** Yes
- **What it attests:** Every supersession signal generated by KCS is permanently recorded in the provenance chain. An oversight body can reconstruct exactly what external events triggered source currency updates and when.
- **What it cannot prove:** The watchlist entries that drove supersession detection were themselves accurate. If a watchlist entry contained a wrong supersession condition, the resulting signal is permanently recorded as legitimate.

---

## 11. Implementation Roadmap

### Phase 1 — Supersession Signal Processing and Coverage Map (Weeks 47–54)

- WatchlistEntry schema and watchlist management endpoints
- SupersessionSignal processing and TVS notification
- `KCS_SOURCE_SUPERSEDED` MOIRAI event emission
- CoverageMapEntry schema and basic coverage map populated from corpus
- Coverage map Redis cache
- `GET /coverage-map` endpoints for ARGUS-LACUNA

**Phase gate criterion:** Supersession signal triggers TVS/KAIROS invalidation within 60 seconds. Coverage map populated for all indexed corpus domains. ARGUS-LACUNA can query coverage map successfully.

### Phase 2 — Model Currency, Watchlist Monitoring, and External Feeds (Weeks 55–66)

- ModelKnowledgeCurrencyRecord schema and `GET /model-currency` endpoint
- UCS/TYCHE integration for PARAM claim epistemic dominance
- Automated watchlist evaluation cycle (hourly)
- External event feed integration (configurable)
- Watchlist review date enforcement and overdue alerts
- New collection ingestion → supersession check pipeline
- Pre-scheduled blast radius computation for high-priority sources

**Phase gate criterion:** Model currency records configurable and queried by UCS/TYCHE. Watchlist evaluation fires supersession signals on test trigger conditions. Overdue review alert fires for test entries. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate KCS deployment. Watchlist content is an analytic standards authority policy responsibility — the technical service is deployable independently of watchlist population.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

In the ATHENA session header, an amber indicator appears when a source currently used in the session has received a supersession signal from KCS: "A source used in this session may have been superseded by new collection. Review currency before finalising any assessment built on this source." On PARAM claim badges, the model knowledge currency level is shown: CURRENT, DEGRADED, or STALE for the relevant domain and model version.

### 13.2 What the Analyst Should Do

Supersession signal active: check whether the superseding source is accessible and whether it materially changes the assessment. If the assessment depends heavily on a superseded source, update the assessment or caveat it explicitly. PARAM badge showing DEGRADED or STALE: treat the claim with explicit epistemic uncertainty. Seek independent collection. Do not rely on the model's parametric knowledge in this domain without independent verification.

### 13.3 What the Signal Does Not Mean

A supersession signal does not mean the original source was wrong — new collection may confirm rather than contradict it. DEGRADED model knowledge currency does not mean the AI cannot produce useful outputs in this domain — it means the analyst should apply additional independent verification. STALE does not mean every PARAM claim in this domain is wrong; it means the analyst cannot rely on parametric knowledge without independent confirmation.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Watchlist evaluation at scale.** As the corpus grows to tens of thousands of sources and the watchlist grows to hundreds of entries, the hourly evaluation cycle will become computationally expensive. Resolution path: watchlist entries are indexed by domain and source type; evaluation uses Elasticsearch filtering rather than full corpus scan.

### 14.2 Operational Assumptions

- **Assumption 1: The analytic standards authority has capacity to maintain the watchlist.** Watchlist management requires ongoing domain expertise and maintenance time. If the authority does not have bandwidth, the watchlist will become stale and KCS's supersession detection will degrade. Resolution path: watchlist maintenance must be included in the analytic standards authority's operational planning before Phase 1 deployment.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 7 gate review.*
