# 32. Enterprise Controls & Policy Enforcement

> **Implementation Status (2026-03-27):** Core controls are active. Authority guard, card lifecycle validation, loop detection, path traversal protection, and command blocking are all built and enforced at runtime. Production auth, enterprise SSO/RBAC, and full audit tooling are Phase 3.

## 32.1 Immutable Rules

| Rule | Enforcement Mechanism | Status |
|------|----------------------|--------|
| No agent edits outside its authority | `authority-guard.ts` validates every write operation | ✅ Built |
| No state change without validation | `card-lifecycle.ts` checks transition legality | ✅ Built |
| No merge without review | State transition UNDER_REVIEW → DONE blocked (must go through TESTING) | ✅ Built |
| No release without DoD gate | `quality-gates.ts` blocks release state | ✅ Built |
| No bypassing state engine | All changes flow through orchestration engine | ✅ Built |
| No path traversal in file writes | `workspace.ts` blocks `../` escapes | ✅ Built |
| No dangerous shell commands | `tool-executor.ts` blocks rm -rf, sudo, etc. | ✅ Built |
| Dev agents require VS Code connection | Heartbeat gate via Redis TTL (30s) | ✅ Built |
| No loop/runaway agents | `loop-detector.ts` fuzzy + repetition + reask detectors | ✅ Built |
| No API keys in plaintext | AES-256-GCM encryption for LLM keys | ✅ Built |

## 32.2 Policy Enforcement Points

```
User Request
  → Next.js API Gateway (NextAuth session validation)
    → Orchestration Engine (MessageRouter — persona routing)
      → Authority Guard (agent write permission check)
        → Card Lifecycle (transition legality)
          → Decision Controller (approval required?)
            → Quality Gates (DoD met?)
              → Agent Runtime (execute within authority bounds)
                → LLM Gateway (resolve admin config → provider adapter)
                  → Tool Executor (sandboxed — path traversal + command blocking)
                    → Workspace (/workspaces/{projectId}/)
```

---

# 33. Roadmap & Phased Expansion Strategy

*Last updated: 2026-03-27*

## Phase 1 — Foundation (MVP)

| Deliverable | Status |
|-------------|--------|
| Next.js 16 platform with PostgreSQL/Prisma + Docker | ✅ Complete |
| All 12 platform pages wired to real database | ✅ Complete |
| 4-step guided project creation wizard | ✅ Complete |
| Post-signup onboarding wizard (3 steps) | ✅ Complete |
| Drag-and-drop kanban board with milestones view | ✅ Complete |
| Command palette (cmdk) | ✅ Complete |
| Client-centric UX language across all pages | ✅ Complete |
| Platform Settings drawer (budget, approvals, notifications) | ✅ Complete |
| Decisions UI with "Your AI Team Recommends" banner | ✅ Complete |
| Agents page with human-readable role descriptions | ✅ Complete |
| Admin LLM provider configuration | ✅ Complete |
| Full Prisma schema (24+ models) with seed data | ✅ Complete |

## Phase 2 — Full SDLC (AI Engine + Code Delivery)

| Deliverable | Status |
|-------------|--------|
| All 23 agents with system prompts + authority levels + tool restrictions | ✅ Complete |
| Orchestration engine: MessageRouter, ContextBuilder, AgentExecutor, LLM Gateway | ✅ Complete |
| LLM providers: OpenAI, Anthropic, Ollama (raw fetch, no SDK) | ✅ Complete |
| AES-256-GCM encryption for LLM API keys at rest | ✅ Complete |
| SSE streaming chat (`/api/projects/[id]/chat/stream`) | ✅ Complete |
| `useAgentStream` hook with real-time token rendering | ✅ Complete |
| Tool system: write_file, edit_file, read_file, list_directory, glob, grep | ✅ Complete |
| Tool system: run_command, git_commit, create_card, update_card, create_document | ✅ Complete |
| Tool system: update_document, approve_document, create_decision, remember | ✅ Complete |
| Tool system: consult_agent, ask_user, task_progress, search_artifacts | ✅ Complete |
| Tool system: run_tests, run_build, validate_code, review_changes, run_analysis | ✅ Complete |
| Real file I/O to `/workspaces/{projectId}/` on disk | ✅ Complete |
| Auto git-commit on every file write by agents | ✅ Complete |
| Artifact PostgreSQL persistence with versioning | ✅ Complete |
| VS Code extension v0.1.0 compiled + packaged (.vsix) | ✅ Complete |
| VS Code ↔ web app SSE artifact stream (`/api/projects/[id]/artifacts/stream`) | ✅ Complete |
| VS Code heartbeat gate (`/api/projects/[id]/vscode-ping`) via Redis (30s TTL) | ✅ Complete |
| Dev agents (TL, JD, SD, QA, DO, SEC) blocked until VS Code connected | ✅ Complete |
| Card lifecycle state machine with validation (`card-lifecycle.ts`) | ✅ Complete |
| Loop detection: fuzzy repetition + question re-ask detectors | ✅ Complete |
| Authority guard: per-agent write permission enforcement | ✅ Complete |
| Quality gates enforcer (`quality-gates.ts`) | ✅ Complete |
| Path traversal protection + dangerous command blocking | ✅ Complete |
| Wireframe generation (22 wireframe files in sample project) | ✅ Complete |
| Event-bus and event-handlers in orchestration engine | ✅ Complete |
| Telemetry module and OrchestrationRun table | ✅ Complete |
| BYOM LLM support (OpenAI / Anthropic / Ollama, admin-configured) | ✅ Complete |
| Full quality gate enforcement at card + document level | ✅ Complete |
| Agent task queue infrastructure (BullMQ + Redis installed) | 🔄 Infrastructure ready; parallel multi-agent activation in progress |
| GitHub/GitLab PR creation | 🔄 Tool stub defined; integration not yet active |
| Web search tool | 🔄 Tool stub defined; implementation not yet active |

## Phase 3 — Enterprise

| Deliverable | Status |
|-------------|--------|
| Production authentication (replace NextAuth dev auto-login) | 📋 Planned |
| Stripe billing / subscription management | 📋 Planned (schema with stripeCustomerId/stripeSubscriptionId exists) |
| WebSocket for real-time agent status on dashboard | 📋 Planned (currently SSE only) |
| Multi-tenant SaaS deployment | 📋 Planned |
| Enterprise SSO / RBAC | 📋 Planned |
| Advanced observability dashboards (token cost, performance) | 📋 Planned |
| Cost governance & budget controls (per-project/per-agent caps) | 📋 Planned |
| Deterministic replay engine (events stored; replay not yet implemented) | 📋 Planned |
| Disaster recovery automation | 📋 Planned |
| Cloud deployment pipeline (trigger_deploy stub → real pipeline) | 📋 Planned |
| GitHub / GitLab PR creation integration | 📋 Planned |
| Parallel multi-agent task queue (BullMQ activation) | 📋 Planned |
| Integration Setup Center UI for user-managed third-party credentials | 📋 Planned |
| OS Keychain / Vault for enterprise secret management | 📋 Planned |

## Phase 4 — Scale

| Deliverable | Status |
|-------------|--------|
| Horizontal scaling (distributed agents via BullMQ workers) | 📋 Planned |
| Marketplace (agent plugins, templates) | 📋 Planned |
| Custom agent authoring | 📋 Planned |
| Advanced analytics & AI insights | 📋 Planned |
| White-label / OEM packaging | 📋 Planned |
