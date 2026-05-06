# Integrations

External systems that THEMIS connects to, organized by direction.

**Upstream** — systems that feed data into THEMIS (evidence, matter metadata, identity).
**Downstream** — systems that consume THEMIS output (billing, document editor, client portal).
**Model providers** — AI API endpoints; governed by [model-providers.md](model-providers.md).

| Integration | Direction | File | Status |
|---|---|---|---|
| PACER (court dockets) | Upstream → KCS | [pacer.md](pacer.md) | Planned — Phase 7 |
| Westlaw / Lexis (legal databases) | Upstream → KCS | [westlaw-lexis.md](westlaw-lexis.md) | Planned — Phase 7 |
| iManage / NetDocuments (DMS) | Upstream → MOIRAI | [imanage.md](imanage.md) | Planned — Phase 3 |
| Aderant / Elite (matter management) | Upstream + Downstream | [aderant-elite.md](aderant-elite.md) | Planned — Phase 1 |
| Microsoft 365 (archive + OneDrive) | Upstream → MOIRAI | [microsoft365.md](microsoft365.md) | Planned — Phase 3 |
| Model Providers (Anthropic, Azure OpenAI) | AI compute layer | [model-providers.md](model-providers.md) | Planned — Phase 1 |

## Integration Governance

All integrations that involve matter data require AI Governance Committee approval before activation. Integrations are:
- Reviewed annually for continued necessity and security posture
- Owned by a named integration owner responsible for credential rotation and API version management
- Registered in the integration registry (this directory) as versioned configuration
- Subject to ZDR (Zero Data Retention) agreements where the integration involves model providers

New integrations are proposed by adding a file to this directory and opening a PR. The PR must include: integration owner, data classification, direction, authentication mechanism, and AI Governance Committee approval reference.
