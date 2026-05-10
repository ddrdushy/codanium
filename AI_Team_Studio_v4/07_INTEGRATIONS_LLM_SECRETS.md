# 16. Integration & Secret Management Framework

> **Implementation Status (2026-03-27):** LLM API keys are stored encrypted (AES-256-GCM) in the PostgreSQL `LLMProviderConfig` table and managed via the Admin Settings panel. Full OS Keychain / Vault integration is planned for Phase 3. The Integration Setup Center UI for user-managed third-party credentials is not yet built.

## 16.1 Integration Setup Center

**Current Implementation:**
- Admin-only LLM provider configuration via Admin Settings page
- API keys encrypted with AES-256-GCM before storage in PostgreSQL
- Supports OpenAI, Anthropic, and Ollama providers
- Resolution priority: Agent-level override → Project-level override → Admin platform default

**Planned (Phase 3):**
- Dedicated UI panel listing required integrations per project
- Step-by-step guided setup for user API keys
- Connectivity testing before marking integration active
- OS Keychain / Vault for user-managed secrets

## 16.2 Secret Management Rules

| Rule | Enforcement | Status |
|------|------------|--------|
| LLM API keys never stored in plaintext | AES-256-GCM encryption in PostgreSQL | ✅ Built |
| Secrets never committed to Git | .gitignore + workspace path restrictions | ✅ Built |
| Only references stored in project files | Config files use encrypted DB references | ✅ Built |
| Secrets injected at runtime | LLM Gateway resolves keys from DB at request time | ✅ Built |
| OS Keychain / Vault support | HashiCorp Vault, AWS Secrets Manager | 📋 Planned (Phase 3) |
| Rotation policies enforced | Expiration tracking + alerts | 📋 Planned |
| Every access audited | Audit log for key access | 📋 Planned |

## 16.3 Supported Secret Stores

| Store | Status |
|-------|--------|
| PostgreSQL (AES-256-GCM encrypted) | ✅ Current implementation |
| Environment variables (local dev fallback) | ✅ Supported via .env.local |
| OS Keychain (macOS/Windows/Linux) | 📋 Planned Phase 3 |
| HashiCorp Vault | 📋 Planned Phase 3 |
| AWS Secrets Manager | 📋 Planned Phase 3 |
| Azure Key Vault | 📋 Planned Phase 3 |
| GCP Secret Manager | 📋 Planned Phase 3 |

---

# 17. LLM & BYOM Architecture

> **Implementation Status (2026-03-27):** LLM providers are fully implemented using raw fetch adapters (no SDK dependencies). Provider configuration is admin-managed and stored encrypted in PostgreSQL. Three providers are supported: OpenAI, Anthropic, and Ollama. The platform uses a single platform-wide provider with optional agent-level and project-level overrides.

## 17.1 Supported Providers

| Provider | Status | Implementation |
|----------|--------|---------------|
| OpenAI (GPT-4o, GPT-4, etc.) | ✅ Built | `openai-adapter.ts` — raw fetch, no SDK |
| Anthropic (Claude 3.x, etc.) | ✅ Built | `anthropic-adapter.ts` — raw fetch, no SDK |
| Ollama (local models) | ✅ Built | `ollama-adapter.ts` — raw fetch, no SDK |
| Azure OpenAI | 📋 Planned | OpenAI-compatible endpoint |
| Groq / Together | 📋 Planned | OpenAI-compatible endpoint |
| Custom self-hosted endpoint | 📋 Planned | vLLM, TGI compatible |

## 17.2 Configuration Levels

| Level | Scope | Status |
|-------|-------|--------|
| **Platform default (Admin)** | All projects | ✅ Built — Admin Settings page configures platform-wide LLM |
| **Project override** | Per-project | ✅ Supported in resolution priority chain |
| **Agent override** | Per-agent | ✅ Supported in resolution priority chain |
| **Session override** | Per-session | 📋 Planned |
| **User-level config** | Per-user | Not planned — admin controls LLM for security/cost reasons |

## 17.3 LLM Gateway Controls

| Control | Description | Status |
|---------|-------------|--------|
| **Model routing** | Route requests to correct provider based on admin config + agent override | ✅ Built — LLM Gateway in orchestration engine |
| **Raw fetch adapters** | No SDK dependencies — direct HTTP to OpenAI/Anthropic/Ollama APIs | ✅ Built |
| **SSE streaming** | Token-by-token streaming via `/api/projects/[id]/chat/stream` | ✅ Built |
| **API key encryption** | AES-256-GCM for keys at rest | ✅ Built — `src/lib/ai/encryption.ts` |
| **Budget caps** | Per-project and per-agent token budgets | 📋 Planned Phase 3 |
| **Token tracking** | Real-time token usage monitoring | 📋 Partial — telemetry schema defined |
| **Policy filters** | Input/output content filtering for compliance | 📋 Planned |
| **Fallback logic** | Automatic failover to backup model on provider error | 📋 Planned |
| **Rate limiting** | Throttle requests to prevent budget overrun | 📋 Planned |
