# Integration: Model Providers

**Status:** Planned — Phase 1
**Direction:** Upstream (AI compute)
**Owner:** Platform Engineering
**Phase:** Phase 1 (API gateway routing); Phase 6 (self-hosted fine-tuned models)

---

## Purpose

THEMIS is model-provider agnostic. The API gateway abstracts the specific model endpoint from all upstream services. THEMIS services interact with the gateway, not with model providers directly. This abstraction enables provider switching, multi-provider routing, and regional endpoint selection without changes to THEMIS service code.

## Providers

### Anthropic (Claude)
- **Role:** Primary provider for long-context reasoning tasks: evidence analysis, complex legal research synthesis, ERAS chain-of-thought capture.
- **Regional endpoints:** EU and US endpoints available for data residency compliance.
- **Models pinned:** claude-sonnet-4 (primary); claude-opus-4 (ERAS reasoning capture for high-stakes interactions)
- **ZDR:** Zero Data Retention agreement required before any client matter content is processed.

### Azure OpenAI
- **Role:** Enterprise deployment for Azure regional infrastructure matching specific data residency requirements. Primary path for Microsoft 365 integration scenarios.
- **Regional endpoints:** EU (Sweden Central, France Central), US (East US 2)
- **ZDR:** Microsoft Enterprise Data Protection agreement covers Azure OpenAI; verify API settings disable logging.

### Self-Hosted / Fine-Tuned Models (Phase 6+)
- **Role:** Firm-specific model weights trained on the FGTS ground truth corpus. Deployed as self-hosted endpoints within the regional data plane cluster — no data leaves firm infrastructure.
- **Governance:** Every fine-tuned model version requires AI Governance Committee approval before production deployment. Training corpus version, quality benchmarks, and approval chain stored in model registry.

### Embedding Models
- **Role:** Separate from generation models. Used by RQS vector store and ERAS reasoning index.
- **Version management:** Embedding model version stored per chunk. Model version changes require reindexing and AI Governance Committee approval (impacts retrieval quality across the entire corpus).

---

## Model Provider Governance

| Concern | Governance Mechanism |
|---|---|
| Model version pinning | Production interactions use pinned model versions. Changes are governance events: tested, approved, and deployed deliberately — not auto-updated. |
| Training data contamination | No client matter content may be used for provider model training. ZDR agreements required. Explicit data processing agreements prohibiting training data use. |
| Provider outage handling | Fallback provider configuration in API gateway. Fallback must satisfy same data residency requirements as primary. |
| Cost visibility | FGS rate cards maintained per provider per model. Pricing changes are versioned configuration updates. Historical cost records not retroactively repriced. |
| Regional routing | Gateway routes model API calls to provider endpoints within the same regional boundary as the matter data. Cross-region model calls blocked at gateway. |

---

## Rate Limits

Anthropic and Azure OpenAI both enforce token-per-minute (TPM) and request-per-minute (RPM) limits at the API key level. The API gateway is responsible for:
- Monitoring TPM/RPM consumption in real-time
- Routing overflow to the fallback provider before hard rate limits are hit
- Alerting the FGS anomaly detection system when consumption approaches limits (may indicate runaway process)

---

## Failure Handling

If the primary provider is unavailable:
1. Gateway detects failure (timeout or 5xx response)
2. Request routes to fallback provider (same regional boundary)
3. Fallback provider is subject to the same privilege and policy enforcement
4. Alert raised to platform engineering within 60 seconds
5. TurnRecord logs the fallback provider used

If both primary and fallback are unavailable, AI generation is blocked and the analyst receives a clear unavailability message. The interaction is not silently degraded.

---

## Contacts

- Anthropic enterprise support: *(account contact)*
- Azure OpenAI support: *(Azure support portal)*
- Internal: Platform Engineering team
