# HADES — Continuous Adversarial Evaluation Service
### *"The Underworld — probing what lies beneath"*
*Part of the THEMIS Platform · Adversarial Evaluation · Build Phase: 5 (Library: Phase 1)*

---

## Design Philosophy

Every other THEMIS service governs, measures, or improves the platform's normal operating behavior. HADES does something categorically different: **it deliberately tries to break the platform and records what happens.**

> **The Core Principle:** A governance platform that has never been adversarially tested is not a governed platform — it is a platform with untested assumptions. HADES converts those assumptions into verified properties. Every THEMIS service that makes a security or correctness claim must have a corresponding HADES probe that continuously verifies that claim holds under pressure.

> **Design-Now, Build-Later:** Every PGS rule written in Phase 1 must have a corresponding HADES probe scenario authored at the same time. The probe does not run until Phase 5 — but when HADES launches, it has full coverage of every rule in production since day one.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│          HADES — Continuous Adversarial Evaluation Service           │
│                                                                      │
│  ┌────────────────────────┐   ┌────────────────────────────────────┐ │
│  │ Adversarial Prompt     │   │ Probe Execution Engine             │ │
│  │ Library                │   │ - scheduled continuous runs        │ │
│  │ - versioned scenarios  │──▶│ - triggered on rule/model change   │ │
│  │ - 5 surface coverage   │   │ - coverage tracking                │ │
│  │ - governance-approved  │   └────────────────┬───────────────────┘ │
│  └────────────────────────┘                    │                     │
│                                                ▼                    │
│                            ┌────────────────────────────────────┐    │
│                            │  Synthetic Evaluation Environment  │    │
│                            │  (zero real client data)           │    │
│                            └────────────────┬───────────────────┘    │
│                                             │                        │
│                            ┌────────────────▼───────────────────┐    │
│                            │  Finding Classifier & Router       │    │
│                            └──────┬──────────────┬──────────────┘    │
│                                   │              │                   │
│                       ┌───────────▼──┐   ┌───────▼────────────────┐  │
│                       │ Failure      │   │ Remediation Router /   │  │
│                       │ Catalog      │   │ Alert Engine           │  │
│                       └──────────────┘   └────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

HADES runs exclusively against the Synthetic Evaluation Environment. It reads service configurations from production (read-only) but **never touches production data or production provenance records.**

---

## The Five Adversarial Surfaces

### Surface 1 — Safety Gate Probing (PGS + PCES)
- Prompt reformulation attacks: rephrasing blocked interaction types to evade classification
- Privilege filter evasion: paraphrasing privileged content to bypass PCES
- Matter boundary probing: cross-matter semantic similarity attacks
- System probe patterns: attempts to extract model configuration or PGS rule logic
- Jailbreak pattern library: continuously updated from KCS external threat intelligence
- Compound multi-turn attacks: adversarial intent distributed across multiple turns

### Surface 2 — Retrieval Poisoning Detection (RQS + PCES)
- Keyword stuffing injection: artificial retrieval ranking without genuine relevance
- Authority spoofing: synthetic chunks resembling authoritative sources with incorrect assertions
- Embedding inversion attacks: high cosine similarity to queries with contradictory substance
- Chunk boundary exploitation: adversarial content straddling split points
- Staleness amplification: high-TVS-score content with outdated legal positions
- Suppression attacks: volume content pushing legitimate evidence below retrieval thresholds

### Surface 3 — Provenance Integrity Testing (MOIRAI)
- Chunk ID collision attempts: manipulated hashes testing content-addressed identity
- Turn record replay attacks: stale turn records with modified timestamps
- PromptAssembly injection: fabricated assembly records with false system prompt versions
- Fingerprint defeat: systematic character manipulation against all three ZWC/LSH/watermark signals
- Lineage severing: output chunks claiming no ancestor relationships
- Certification hash manipulation: document modification post-certification

### Surface 4 — Model Consistency & Drift Testing (API Gateway)
- Output consistency probing: identical prompts run repeatedly; variance above threshold = finding
- Model version regression: behavioral delta between old and new model versions
- Context window boundary testing: quality degradation near context limit
- System prompt sensitivity: output variation from minor instruction changes
- Cross-provider consistency: equivalent prompts across model providers

### Surface 5 — Calibration Boundary Stress Testing (TCS)
- Elo rating manipulation: synthetic correction patterns to inflate/deflate AI ratings
- RAI boundary cases: edge cases where formula produces counterintuitive results
- Calibration drift under distribution shift: interaction class distribution changes
- Correction-free analyst simulation: testing over-reliance detection
- Adversarial correction patterns: systematic under-reliance patterns
- Cohort signal manipulation: small adversarial analyst groups skewing cohort signals

---

## Failure Catalog

```yaml
FailureRecord:
  failure_id:           uuid
  probe_id:             uuid
  surface:              safety_gate | retrieval_poisoning | provenance_integrity
                        | model_consistency | calibration_boundary
  affected_service:     str
  severity:             critical | high | medium | low | informational
  failure_type:         bypass | evasion | manipulation | inconsistency | degradation
  trigger_conditions:   str
  probe_input:          str
  observed_output:      str
  expected_behavior:    str
  first_detected:       ISO8601
  last_confirmed:       ISO8601
  status:               open | remediated | accepted_risk | false_positive
  remediation_event_id: uuid | null
```

### Severity & SLA

| Severity | Definition | SLA |
|---|---|---|
| **CRITICAL** | Safety gate bypass — could expose privileged data | 4 hours to acknowledged remediation plan |
| **HIGH** | Significant evasion degrading a governance guarantee | 24 hours to plan; 5 business days to fix |
| **MEDIUM** | Quality concern but not immediate governance failure | 10 business days to remediation |
| **LOW** | Minor anomalies with limited practical impact | Quarterly governance cycle review |
| **INFORMATIONAL** | Expected behavior confirmed; probe passed | No remediation required |

---

## Adversarial Prompt Library

```yaml
AdversarialScenario:
  scenario_id:        uuid
  version:            semver
  surface:            str
  technique:          str
  target_service:     str
  target_rule_id:     uuid | null
  prompt_template:    str
  expected_behavior:  str
  failure_indicator:  str
  severity_if_fails:  str
  authored_by:        str   # human red team | auto-generated | external research
  source_reference:   str | null
  approved_by:        user_id
  last_run:           ISO8601
  last_result:        pass | fail | error
  consecutive_passes: int
```

**Library governance:** Every scenario requires AI Governance Committee approval. Co-authorship requirement: no PGS rule promoted to production without a committed HADES scenario.

**Expansion triggers:**
- KCS external threat intelligence: new adversarial AI techniques from academic research/CVEs
- FGTS correction patterns: systematic failures converted to stress test scenarios
- Human red team exercises: quarterly formalization of findings
- Post-incident analysis: new scenarios after CRITICAL/HIGH findings

---

## Probe Execution Scheduling

| Run Type | Scope | Cadence | Purpose |
|---|---|---|---|
| Continuous baseline | INFORMATIONAL and LOW scenarios | Every 6 hours | Steady-state coverage signal |
| Daily sweep | All MEDIUM scenarios | Daily at off-peak | Regression detection |
| Weekly deep run | Full library including HIGH and CRITICAL | Weekly | Comprehensive governance report |
| Triggered — rule change | All scenarios targeting changed rule | Within 1 hour | Verify rule holds adversarially |
| Triggered — model version | Full consistency suite (Surface 4) | Within 2 hours | Catch behavioral drift |
| Triggered — FGTS expansion | New scenarios from correction patterns | Within 24 hours | Test reproducibility in SEE |

---

## Re-Test Protocol

A finding is not marked remediated on assertion — only when the specific probe returns PASS:

```
1. Service owner deploys fix to production and SEE
2. HADES re-runs failing probe (same scenario_id, same corpus version)
3. PASS → FailureRecord.status = 'remediated'; immutable record written
4. FAIL → last_confirmed updated; SLA clock resets
5. 3 consecutive PASSes → probe added to weekly regression suite at elevated priority
```

---

## Integration Points

| Service | Role |
|---|---|
| PGS / NOMOS | Primary Surface 1 target; reads production rule config; bypass findings route back as rule update candidates |
| FGTS / ALETHEIA | Bidirectional: FGTS correction patterns expand library; HADES adversarial failures route to FGTS as high-priority corpus entries |
| RQS / HERMES | Surface 2 probes run against RQS in SEE; poisoning findings route as hardening recommendations |
| ERAS / LOGOS | Adversarial reasoning records indexed as distinct ERAS category |
| KCS / ARGUS | External threat intelligence feeds adversarial library expansion |
| TCS / MIMIR | Surface 5 probes target TCS; findings feed calibration model parameter review |

---

**Build placement:** Library and schema design in Phase 1 (co-authored with PGS rules). HADES service built in Phase 5 with full coverage from day one.
**Depends on:** PGS, PCES, PROV, TCS, RQS (must be operational to probe)
**Feeds into:** PGS (rule updates), FGTS (adversarial corpus), RQS (hardening), AI Governance Committee (quarterly catalog review)
