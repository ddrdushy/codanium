# AI Team Studio — Gaps & Phase Breakdown

> Last updated: 2026-03-27
> Status snapshot based on full codebase analysis.

---

## Current Build Status Summary

| Phase | Status | Coverage |
|-------|--------|----------|
| Phase 1 — Foundation | ✅ Complete | Next.js 16 + PostgreSQL/Prisma, 12 platform pages, kanban, wizard, onboarding |
| Phase 2 — Full SDLC | ✅ Mostly Complete | 23 agents, orchestration engine, SSE streaming, VS Code extension, real file I/O, git auto-commit |
| Phase 3 — Enterprise | 🔄 In Progress | Auth/billing/WebSocket/GitHub PR/deploy all pending |
| Phase 4 — Scale | 📋 Planned | Multi-tenant SaaS, marketplace, white-label |

---

## Phase 3 — Enterprise (In Progress)

### 3.1 High Priority Gaps

| # | Gap | Current State | What's Needed |
|---|-----|---------------|---------------|
| P3-01 | **Production Authentication** | NextAuth dev mode — auto-login, no real sessions | NextAuth production config, credential providers, session security, user registration flow |
| P3-02 | **WebSocket Real-Time Dashboard** | Agent status uses SSE only, no live dashboard updates | Replace SSE polling with WebSocket (e.g. Pusher, Ably, or native ws) for real-time agent progress on dashboard |
| P3-03 | **GitHub / GitLab PR Creation** | `create_pr` tool stub — returns placeholder message | Connect GitHub API (Octokit) to actually push branches and open PRs from agent output |
| P3-04 | **Cloud Deployment Pipeline** | `trigger_deploy` tool stub — does nothing | Connect to Vercel/Railway/Fly.io API or Docker registry push to execute real deployments |

### 3.2 Medium Priority Gaps

| # | Gap | Current State | What's Needed |
|---|-----|---------------|---------------|
| P3-05 | **Stripe Billing & Subscriptions** | Completely unbuilt | Stripe Checkout, subscription plans, usage metering, webhook handler, billing portal |
| P3-06 | **Agent Task Queue (Parallel)** | Tasks run sequentially | BullMQ (scaffolded) wired to task processor — parallel multi-agent workflows |
| P3-07 | **Web Search Tool** | `web_search` returns stub | Integrate Tavily / Serper / Brave Search API — agents can research the web |
| P3-08 | **Bidirectional File Sync** | VS Code → server: one-way only | Allow VS Code to push local edits back to server artifacts (for human-in-the-loop corrections) |

### 3.3 Enterprise Gaps (Not Started)

| # | Gap | Current State | What's Needed |
|---|-----|---------------|---------------|
| P3-09 | **Enterprise SSO / RBAC** | Admin/user roles only | OAuth2 / SAML provider integration, fine-grained role-based access control |
| P3-10 | **Deterministic Replay Engine** | Events stored in DB, replay not built | Replay engine that reconstructs full project state from Event log |
| P3-11 | **Disaster Recovery** | No backup/restore pipeline | Automated DB snapshots, workspace backup, point-in-time recovery |
| P3-12 | **Advanced Observability Dashboards** | No dashboards | LLM cost per project/agent, error rates, agent performance metrics, SLO tracking |
| P3-13 | **Budget Enforcement** | Cost Governor agent defined but not wired | Hard spending limits per project/user, auto-pause on budget breach |

---

## Phase 4 — Scale (Planned)

| # | Deliverable | Detail |
|---|-------------|--------|
| P4-01 | **Multi-Tenant SaaS Deployment** | Tenant isolation, subdomain routing, per-tenant DB or schema |
| P4-02 | **Horizontal Scaling** | Distributed agent workers, queue-based load distribution |
| P4-03 | **Marketplace / Plugin System** | Agent plugin authoring, template sharing, community marketplace |
| P4-04 | **Custom Agent Authoring** | Users can create, configure, and publish their own agents |
| P4-05 | **Advanced Analytics & AI Insights** | Cross-project analytics, delivery velocity, agent effectiveness scores |
| P4-06 | **White-Label / OEM Packaging** | Rebrandable platform for enterprise resellers |

---

## Recommended Build Order

### Sprint 1 — Unlock Real Users
```
P3-01  Production Authentication
P3-05  Stripe Billing
```
Rationale: Nothing else matters until real users can sign up and pay.

### Sprint 2 — Complete the Dev→Deploy Chain
```
P3-03  GitHub PR Creation
P3-04  Cloud Deployment Pipeline
P3-06  Agent Task Queue (Parallel)
```
Rationale: Completes the full end-to-end delivery promise of the platform.

### Sprint 3 — UX & Intelligence
```
P3-02  WebSocket Real-Time Dashboard
P3-07  Web Search Tool
P3-08  Bidirectional File Sync
```
Rationale: Makes the platform feel alive and agents more capable.

### Sprint 4 — Enterprise Readiness
```
P3-09  Enterprise SSO / RBAC
P3-10  Deterministic Replay Engine
P3-12  Advanced Observability Dashboards
P3-13  Budget Enforcement
```

### Sprint 5 — Scale
```
P4-01  Multi-Tenant SaaS
P4-02  Horizontal Scaling
P4-03  Marketplace
```

---

## What's Already Built (Phase 1 & 2 — Complete)

### Core Platform
- Next.js 16 (Turbopack, React 19, App Router)
- PostgreSQL via Docker (port 14000), Prisma 7.x ORM, 24+ models
- All 12 platform pages wired to real DB
- 4-step project creation wizard
- 3-step onboarding wizard
- Drag-and-drop kanban board with milestones view
- Command palette (cmdk)
- Admin settings with LLM provider management

### AI Orchestration Engine
- MessageRouter — intent classification → agent selection
- ContextBuilder — project state injection into system prompts
- AgentExecutor — LLM call with tool use
- LLM Gateway — admin-configured provider resolution (OpenAI / Anthropic / Ollama)
- SSE streaming chat (`/api/projects/[id]/chat/stream`)
- Loop detection (fuzzy repetition + question re-ask detectors)
- Card lifecycle state machine with transition validation

### 23 Agents
- All defined with system prompts, authority levels, tool restrictions
- Dev agents (TL, JD, SD, QA, DO, SEC) gate-blocked until VS Code connected

### Tool System (13 tools)
- `write_file`, `edit_file`, `read_file`, `list_directory`, `glob`, `grep`
- `run_command`, `git_commit`
- `create_card`, `update_card_status`, `create_decision`, `create_document`
- `search_artifacts`, `remember`
- Path traversal protection + dangerous command blocking

### Real Code Generation
- Agents write actual files to `/workspaces/{projectId}/` on disk
- Artifact records persisted to PostgreSQL with versioning + agent attribution
- Auto-git-commit on every write with `feat: add {path} (by {agent})`
- 18+ real project workspaces generated

### VS Code Extension (v0.1.0)
- SSE artifact stream (`/api/projects/[id]/artifacts/stream`)
- Real-time file delivery to developer's local workspace
- Redis heartbeat gate — dev agents blocked until extension pings (15s interval, 30s TTL)
- File tracker (read-only enforcement on AI-managed files)
- Session restore on reopen
- `.vsix` package ready to install

### Security
- AES-256-GCM encryption for API keys at rest
- Path traversal blocked in workspace manager
- Dangerous command blocklist (rm -rf /, sudo, chmod 777, etc.)
- Agent authority filtering — only authorised agents can call each tool

---

*Document maintained in `AI_Team_Studio_v4/15_GAPS_AND_PHASE_BREAKDOWN.md`*
