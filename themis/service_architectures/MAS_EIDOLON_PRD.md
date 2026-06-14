# MAS — Media Authenticity Service
### EIDOLON · *"Greek for 'phantom' or 'spectre' — the double, the image that is not what it appears to be"*
*THEMIS Platform · Service PRD · v1.0*

---

## Service Header

| Field | Value |
|---|---|
| **Service code** | `MAS` |
| **Epithet** | `EIDOLON` |
| **Full name** | Media Authenticity Service |
| **Namespace** | `themis-quality` |
| **Layer** | Quality Layer |
| **Build phase** | Phase 5–6 (Weeks 29–46) |
| **Build priority** | 11 of 23 platform services |
| **Owner team** | THEMIS Platform Team |
| **Status** | Design |
| **Document version** | 1.0 |
| **Last updated** | 2026-05 |
| **Accountability axis** | Origin — assesses the authenticity of media intelligence |

---

## 1. Design Philosophy

### 1.1 The Question This Service Answers

**MAS/EIDOLON answers: Does this media artifact show signs of synthesis, manipulation, or inauthenticity that should limit the confidence placed in intelligence derived from it?**

### 1.2 Why This Service Exists

Synthetic media — deepfake video, AI-generated audio, manipulated imagery, adversarially altered documents — are deployed as intelligence deception tools. An adversary who can introduce convincing synthetic media into an analyst's workflow can fabricate evidence for any assertion.

AI-assisted analysis compounds this risk. The AI will analyse a deepfake video with the same analytical confidence it would apply to authentic footage. The fluency of the AI's output provides no signal that the source media is inauthentic. Without a dedicated authenticity assessment layer, AI-assisted analysis is as susceptible to synthetic media as unaided analysis — and potentially more so, because the AI's confident analytical output may discourage the analyst from examining the underlying media critically.

### 1.3 Why This Service Is Eleventh

MAS requires MOIRAI for provenance records and IAS for coordinated adversarial detection (a synthetic media artifact that also contains embedded adversarial instructions is a dual threat requiring both services). Phase 5-6 allows the platform to acquire and configure media authenticity detection models before they are needed in analytical sessions.

### 1.4 Design Principles

- **The confidence ceiling is hard, not advisory.** When MAS assesses a media artifact as HIGH risk for synthesis or manipulation, the confidence ceiling on any AI claim derived from that artifact is binding. Analysts cannot override it without supervisor review.
- **MAS operates in an active adversarial arms race.** Detection models will lag generation capability. The operational half-life of any specific detection model is limited. The Research & Red Team must run continuous evaluation against emerging synthesis techniques. MAS's confidence assessments must communicate this inherent uncertainty.
- **Absence of detected manipulation is not confirmation of authenticity.** MAS cannot prove a media artifact is authentic — it can only characterise the degree to which it shows signs of manipulation. "No manipulation detected" means the artifact does not match known manipulation signatures; it does not mean the artifact is authentic. This distinction is critical for analyst training.
- **Metadata integrity is a first-order signal, not a secondary check.** Camera and device metadata — when present and consistent — provides strong authenticity evidence. Missing or inconsistent metadata is itself a high-risk indicator. MAS treats metadata analysis as a primary signal layer, not a supplementary check.

### 1.5 Explicit Out of Scope

- **Determining whether the subject matter of media is real.** MAS assesses whether the media artifact itself is authentic; it does not determine whether what the media depicts actually occurred.
- **Content moderation.** MAS is an authenticity service; it does not screen for prohibited content.
- **Transcription.** MAS provides ASR confidence scoring for audio and video; transcription itself is handled by the corpus ingestion pipeline.

---

## 2. Core Responsibilities

### 2.1 Primary Function

MAS/EIDOLON analyses media artifacts ingested into or referenced in analytical sessions — video, audio, images, and scanned documents — to produce a structured authenticity assessment that characterises synthesis risk, manipulation indicators, metadata integrity, and the confidence ceiling that should be applied to any analytical conclusions derived from the artifact.

### 2.2 Secondary Functions

- ASR confidence scoring: for audio and video artifacts, providing a confidence score for the AI's speech-to-text transcription that ATHENA displays on TRANSCRIPT badges
- OCR confidence scoring: for scanned document artifacts, providing a confidence score for the AI's optical character recognition that ATHENA displays on OCR badges
- Metadata provenance reporting: extracting and validating camera, device, and chain-of-custody metadata for IOB and forensic use
- IAS/SCUDO integration: flagging media artifacts with embedded adversarial content (steganographic injection, embedded instructions in image/audio channels)
- Temporal media analysis: for video, assessing frame-level consistency and temporal authenticity

### 2.3 What This Service Does Not Decide

MAS characterises risk and produces a confidence ceiling. Whether an analyst may proceed with an artifact that has a HIGH synthesis risk is a supervisor decision. Whether a media artifact with a MEDIUM risk should be used in a finished intelligence product is an analytical judgment. MAS provides the signal; humans decide the consequences.

---

## 3. Data Architecture

### 3.1 Primary Data Models

```yaml
AuthenticityAssessment:
  assessment_id:         uuid
  session_id:            uuid | null      # null for corpus ingestion assessments
  source_id:             uuid
  media_type:            VIDEO | AUDIO | IMAGE | SCANNED_DOCUMENT
  risk_level:            LOW | MEDIUM | HIGH | INCONCLUSIVE
  confidence_ceiling:    float            # maximum confidence score for derived claims
  signals:               [AuthenticitySignal]
  metadata_integrity:    MetadataIntegrity
  asr_confidence:        float | null     # for VIDEO and AUDIO only
  ocr_confidence:        float | null     # for SCANNED_DOCUMENT only
  detection_model_version: str
  assessment_timestamp:  datetime
  inconclusive_reason:   str | null       # why assessment is INCONCLUSIVE if applicable

AuthenticitySignal:
  signal_id:             uuid
  assessment_id:         uuid
  signal_type:           DEEPFAKE_FACIAL | GAN_ARTIFACT | AUDIO_SYNTHESIS | TEMPORAL_INCONSISTENCY | METADATA_ANOMALY | STEGANOGRAPHIC | COMPRESSION_ARTIFACT | LIGHTING_INCONSISTENCY
  confidence:            float
  description:           str              # plain language
  location_reference:    str | null       # timestamp (video/audio) or region (image)
  severity:              HIGH | MEDIUM | LOW

MetadataIntegrity:
  metadata_present:      bool
  metadata_consistent:   bool | null      # null if metadata not present
  device_signature:      str | null       # device type from metadata, if present
  capture_timestamp:     datetime | null
  geolocation:           str | null       # if present
  chain_of_custody:      [str]            # custody declarations in metadata
  anomalies:             [str]            # list of metadata inconsistencies detected

MediaProcessingRecord:
  record_id:             uuid
  source_id:             uuid
  media_type:            str
  file_hash:             str              # SHA-256 of original media file
  file_size_bytes:       int
  duration_seconds:      float | null     # for VIDEO and AUDIO
  processed_at:          datetime
  processing_model_version: str
```

### 3.2 Storage Architecture

| Store | Technology | Purpose | Retention |
|---|---|---|---|
| Primary store | PostgreSQL | AuthenticityAssessment, AuthenticitySignal, MetadataIntegrity, MediaProcessingRecord | Session + 7 years |
| Event store | MOIRAI | Signed authenticity events | Indefinite |
| Detection model store | Object storage | Versioned detection models (not media files) | Per model version |

*Note: Raw media files are NOT stored by MAS. MAS processes media and stores only the assessment and metadata. Storage of raw media files is a corpus infrastructure responsibility.*

### 3.3 Data Classification

| Data type | Classification floor | Compartment handling |
|---|---|---|
| AuthenticityAssessment | Inherits media source classification | Compartment-gated |
| MetadataIntegrity | Inherits source classification | Compartment-gated; forensic details restricted |
| Detection model version | Controlled Unclassified | Platform team access |

### 3.4 Retention and Purge Policy

AuthenticityAssessment and signal records retained for session lifetime plus seven years. MediaProcessingRecord (hashes and metadata only) retained indefinitely for forensic use. MOIRAI events retained indefinitely.

---

## 4. API Contract

### 4.1 Endpoints

```
POST /assess
  Auth:     corpus ingestion service account | ATHENA service account
  Request:  {
    session_id:          uuid | null,
    source_id:           uuid,
    media_type:          str,
    media_hash:          str,             # SHA-256 of the media file
    media_url:           str              # internal corpus URL (not external)
  }
  Response: {
    assessment_id:       uuid,
    risk_level:          str,
    confidence_ceiling:  float,
    signals:             [AuthenticitySignal],
    asr_confidence:      float | null,
    ocr_confidence:      float | null,
    metadata_integrity:  MetadataIntegrity
  }
  SLA: p99 < 5000ms (media processing is compute-intensive)

GET /assessment/{assessment_id}
  Auth:     session token | supervisor token | IOB token
  Response: AuthenticityAssessment with full signal detail

GET /assessment/by-source/{source_id}
  Auth:     session token | supervisor token
  Response: AuthenticityAssessment (most recent for this source)

GET /session/{session_id}/media-report
  Auth:     session token | supervisor token
  Response: {
    media_artifacts:     int,
    high_risk_count:     int,
    ceilings_applied:    int,
    assessments:         [{ source_id, risk_level, confidence_ceiling }]
  }

GET /audit/authenticity-summary?from={date}&to={date}
  Auth:     Research & Red Team | IOB
  Response: {
    period:              { from, to },
    total_assessments:   int,
    by_risk_level:       { LOW: int, MEDIUM: int, HIGH: int, INCONCLUSIVE: int },
    by_media_type:       [{ type, count, high_risk_rate }],
    detection_model_version: str
  }

GET /health
  Response: {
    status, dependencies: { moirai, pces },
    detection_model_version: str,
    model_age_days:      int,
    assessments_today:   int,
    high_risk_rate_24h:  float,
    last_event_hash:     str
  }
```

### 4.2 MOIRAI Event Schema

```yaml
EventType:          MAS_ASSESSMENT_COMPLETE
service_id:         "MAS"
session_id:         uuid | null
classification:     str
event_payload:
  assessment_id:          uuid
  source_id:              uuid
  media_type:             str
  risk_level:             str
  confidence_ceiling:     float
  signal_count:           int
  high_severity_signals:  int
  metadata_present:       bool
  metadata_consistent:    bool | null
  detection_model_version:str
prev_event_hash:    str
signature:          str
timestamp:          datetime

EventType:          MAS_HIGH_RISK_DETECTED
event_payload:
  assessment_id:          uuid
  source_id:              uuid
  dominant_signal_type:   str
  confidence_ceiling:     float
```

| Event type | Trigger | Downstream consumers |
|---|---|---|
| `MAS_ASSESSMENT_COMPLETE` | Every media assessment | MOIRAI, ATHENA (badge update with risk level and ceiling), TCS/MIMIR |
| `MAS_HIGH_RISK_DETECTED` | Risk level = HIGH | MOIRAI, IAS/SCUDO (coordinated threat check), supervisor notification |

### 4.3 Consumed Events

| Source service | Event type | Action taken |
|---|---|---|
| IAS/SCUDO | `IAS_ADVERSARIAL_SESSION_FLAGGED` | Triggers elevated scrutiny on all media in that session |
| Corpus ingestion | New media artifact ingested | Triggers `POST /assess` for authenticity baseline |

---

## 5. Integration Map

### 5.1 Depends On

| Service | Epithet | Role | Call type | Failure behavior |
|---|---|---|---|---|
| MOIRAI | Provenance | Signed authenticity events | Async event | Events buffered; assessment still proceeds |
| PCES/AEGIS | Classification Enforcement | Media source compartment validation | Sync | Assessment proceeds with cached session scope |

### 5.2 Feeds Into

| Service | Epithet | What MAS provides | How |
|---|---|---|---|
| ATHENA | Interface | Risk level badge; confidence ceiling on derived claims; ASR/OCR confidence | API |
| IAS/SCUDO | Adversarial Screening | HIGH risk media alerts for coordinated threat detection | `MAS_HIGH_RISK_DETECTED` event |
| TCS/MIMIR | Trust Calibration | Confidence ceiling applied to calibration weight for sessions using high-risk media | MOIRAI event |
| ERAS/LOGOS | Reasoning Audit | Authenticity assessment for media-derived claims | API query |
| IOB Reporting | Oversight | Media authenticity summary | Audit endpoint |

### 5.3 PCES/AEGIS Integration

- **Enforcement point:** Media source compartment validated before assessment is returned to analyst-facing endpoints.
- **Failure behavior:** Assessment proceeds for already-ingested corpus media; analyst endpoint validation falls back to cached session context.

---

## 6. Non-Functional Requirements

### 6.1 Latency

| Operation | p50 target | p95 target | p99 target |
|---|---|---|---|
| Image assessment | 500ms | 2000ms | 5000ms |
| Audio assessment (per minute of audio) | 2000ms | 5000ms | 10000ms |
| Video assessment (per minute of video) | 5000ms | 15000ms | 30000ms |
| Scanned document assessment | 200ms | 1000ms | 3000ms |

Media processing is compute-intensive. Assessments run asynchronously at corpus ingestion time; ATHENA queries the stored assessment. If an assessment is not yet complete when a session begins, ATHENA shows `ASSESSMENT_PENDING` and checks periodically.

### 6.2 Throughput

| Metric | Target |
|---|---|
| Image assessments/hour | 200 |
| Audio assessments/hour | 50 |
| Video assessments/hour | 20 |
| Scanned document assessments/hour | 300 |

### 6.3 Availability

| Metric | Target |
|---|---|
| Uptime | 99.0% — MAS unavailability means media source badges show ASSESSMENT_PENDING |
| MOIRAI event durability | 99.999% |
| RTO | 30 minutes (assessment backlog processed on recovery) |
| RPO | 1 hour |

### 6.4 Graceful Degradation

| Dependency unavailable | Service behavior | Analyst-facing impact |
|---|---|---|
| MOIRAI | Events buffered; assessment still computed | No analyst impact; provenance gap logged |
| Compute infrastructure | Assessment queue backs up; in-progress assessments preserved | ASSESSMENT_PENDING badge shown; no blocking |

---

## 7. Security Model

### 7.1 Authentication

Corpus ingestion and ATHENA use service accounts for assessment requests. Analyst session token for analyst-facing read endpoints. IOB token for audit. Detection model updates require Research & Red Team approval with MOIRAI event.

### 7.2 Authorization

| Caller type | Access scope | Grant mechanism |
|---|---|---|
| Corpus ingestion service | `POST /assess` | Service account |
| ATHENA | `GET /assessment/by-source/*` | Service account |
| Analyst (own session) | Session media report | Session token |
| Supervisor | Team session media reports | Supervisor token |
| Research & Red Team | Audit; detection model management | Research team token |
| IOB | Full audit | IOB token |

### 7.3 Secrets Handling

| Secret | Vault path | Rotation policy |
|---|---|---|
| MOIRAI signing key | `themis/mas/signing-key` | 90 days |
| PostgreSQL credentials | `themis/mas/db-credentials` | 30 days |
| Detection model registry credentials | `themis/mas/model-registry` | 30 days |

---

## 8. Failure Modes

| Failure mode | Probability | Impact | Detection | Mitigation |
|---|---|---|---|---|
| Deepfake evades detection (generation outpaces detection) | High (expected) | P1 — high-quality synthetic media assessed as LOW risk | Research & Red Team ongoing adversarial testing | Conservative confidence ceilings on MEDIUM risk; IOB report on detection model age |
| False positive (authentic media assessed as HIGH risk) | Medium | P2 — legitimate intelligence downweighted | Analyst complaint rate; supervisor review rate | Supervisor override pathway; MEDIUM risk rather than HIGH for borderline cases |
| Detection model drift (generation distribution shifts) | Medium | P1 — model accuracy degrades silently | Monthly Research & Red Team evaluation | MDS/KRONOS-equivalent model version tracking for detection models; monthly evaluation |

### 8.1 Known Design Risks

- **Deepfake detection is in an active arms race that detection is currently losing.** State-of-the-art deepfake detection models have meaningful false negative rates on current generation techniques, and new generation methods consistently outpace detection. This is a hard operational constraint, not an engineering problem. Mitigation: conservative confidence ceilings (MEDIUM risk sources receive a ceiling, not just HIGH), explicit detection model age surfaced in ATHENA, regular Research & Red Team adversarial evaluation, and analyst training that "no manipulation detected" is not authentication.
- **Audio synthesis detection is less mature than video deepfake detection.** Audio-only synthesis (text-to-speech impersonation) is harder to detect than video deepfakes because audio has fewer forensic signals than video. MAS's audio synthesis signal confidence is lower than its video deepfake confidence. Confidence ceilings for pure AUDIO artifacts should be conservative.
- **The confidence ceiling creates an asymmetric error.** A false positive on MAS (legitimate artifact assessed as HIGH risk) applies a confidence ceiling to legitimate intelligence, potentially suppressing accurate analysis. The cost of missing a HIGH-risk synthetic artifact is higher than the cost of a false positive — but the false positive rate matters for operational confidence in the service. Research & Red Team must track the ratio.

---

## 9. Observability

### 9.1 Key Metrics

| Metric | Type | Alert threshold | Severity |
|---|---|---|---|
| `mas.assessment.latency_p99` | Histogram | `> 30s for video` | P2 |
| `mas.high_risk_rate_24h` | Gauge | `> 10%` | P2 |
| `mas.moirai.emit.failure_rate` | Counter | `> 0.1% over 1m` | P0 |
| `mas.detection_model.age_days` | Gauge | `> 30` | P1 |
| `mas.assessment_queue.depth` | Gauge | `> 100 pending` | P1 |

### 9.2 Health Check

```
GET /health
Response: {
  status, dependencies: { moirai, pces },
  detection_model_version:  str,
  model_age_days:           int,
  assessment_queue_depth:   int,
  high_risk_rate_24h:       float,
  last_event_hash:          str
}
```

---

## 10. Cryptographic Attestation

- **Vault key path:** `themis/mas/signing-key`
- **Chain participation:** Yes
- **What it attests:** The authenticity assessment for every media artifact used in analytical sessions is permanently recorded — the risk level, the specific signals detected, the detection model version, and the confidence ceiling applied. An oversight body can determine what authenticity assessment was available for any media-derived intelligence claim.
- **What it cannot prove:** A `LOW` risk assessment does not prove the artifact is authentic. The detection model may have missed manipulation that fell outside its training distribution.

---

## 11. Implementation Roadmap

### Phase 1 — Image and Scanned Document Assessment (Weeks 29–36)

- AuthenticityAssessment schema and `POST /assess` endpoint for IMAGE and SCANNED_DOCUMENT
- Deepfake/GAN artifact detection for images (initial detection model v1.0)
- OCR confidence scoring for scanned documents
- Metadata integrity extraction and validation
- Confidence ceiling computation and MOIRAI event emission
- ATHENA media badge integration

**Phase gate criterion:** Image assessments produce meaningfully differentiated risk levels on Research & Red Team test media set (including known synthetic and authentic samples). ATHENA displays MAS badge on all image and scanned document sources.

### Phase 2 — Audio, Video, ASR Confidence, and High-Risk Integration (Weeks 37–46)

- Video deepfake detection: facial synthesis, temporal consistency, frame-level analysis
- Audio synthesis detection: speaker authenticity, prosodic consistency, voice synthesis signals
- ASR confidence scoring for TRANSCRIPT badges
- `MAS_HIGH_RISK_DETECTED` event and IAS/SCUDO integration
- Detection model versioning and Research & Red Team evaluation pipeline
- Full audit endpoints

**Phase gate criterion:** Video deepfake detection operational on Research & Red Team test video set. Audio synthesis detection operational. ASR confidence scores displayed in ATHENA. HIGH risk detection triggers IAS coordinated check. ARB and Cell Lead sign-off.

---

## 12. Policy Dependencies

No GC items gate MAS deployment. The confidence ceiling policy (which risk levels trigger hard ceilings vs. advisory flags) is a platform design decision owned by the AI Trust Cell in consultation with the analytic standards authority.

---

## 13. Training and Analyst Guidance

### 13.1 What the Analyst Sees

Media artifacts in ATHENA show a risk badge: green shield (LOW), amber shield (MEDIUM), red shield (HIGH), grey shield (ASSESSMENT_PENDING), or grey X (INCONCLUSIVE). Expanding shows the specific signals detected and the confidence ceiling in effect. Under HIGH risk, the ATHENA session header shows: "Confidence ceiling applied to claims derived from this source. Independent corroboration required before use in any analytical product."

### 13.2 What the Analyst Should Do

HIGH risk: do not base any analytical conclusion solely on this artifact. Seek corroborating collection through independent channels. If the artifact is essential and the analytical assessment cannot proceed without it, escalate to supervisor — do not proceed under the confidence ceiling without supervisory review. MEDIUM risk: note the risk level in your analytical record. Seek corroboration where possible. LOW risk: treat normally, but note that the assessment does not confirm authenticity — it characterises the absence of detected manipulation signals.

### 13.3 What the Signal Does Not Mean

A LOW risk assessment does not mean the artifact is authentic. It means the detection model did not identify manipulation signals in this artifact. Current detection models do not catch all synthetic media. A HIGH risk assessment does not mean the artifact is synthetic — it means it shows manipulation signals consistent with synthesis or manipulation at a level that warrants restricting confidence in derived claims.

---

## 14. Open Questions and Research Dependencies

### 14.1 Technical Open Questions

- **Q1: Detection model refresh cadence.** How frequently must the detection model be retrained or replaced to maintain operationally useful accuracy against current generation techniques? Resolution path: Research & Red Team to specify minimum model refresh cadence based on generation technique evolution rate. Model age alert at 30 days is a starting point.
- **Q2: Confidence ceiling thresholds.** What confidence ceiling values are appropriate for LOW, MEDIUM, and HIGH risk levels? These are policy-informed engineering choices that require Research & Red Team input and analytic standards authority concurrence. Resolution path: Research & Red Team to specify ceiling values with rationale before Phase 1 deployment.

---

## 15. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05 | THEMIS Platform Team | Initial PRD |

## Appendix D: Red Team Findings
*Pending — Phase 5 gate review. Research & Red Team must evaluate detection models on current synthetic media before Phase 1 deployment.*
