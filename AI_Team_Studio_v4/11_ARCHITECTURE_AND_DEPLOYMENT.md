# 24. Enterprise System Architecture

> **Implementation Status (2026-03-27):** The system is built as a Next.js 16 monolith with PostgreSQL, not a FastAPI/Node.js split. The orchestration engine, agent runtime, LLM gateway, and tool runner are all within the Next.js API layer. The VS Code extension handles file delivery to the developer's local workspace. Multi-tenant SaaS mode and full cloud deployment are Phase 3 targets.

## 24.1 Actual Technology Stack (Built)

| Layer | Technology | Purpose | Status |
|-------|-----------|---------|--------|
| **Frontend** | Next.js 16 (App Router, React 19, Turbopack) | State-driven UI with role-based views | ✅ Built |
| **Backend API** | Next.js API routes (App Router) | REST routes, SSE streaming, webhooks | ✅ Built |
| **Orchestration Engine** | Custom TypeScript (in-process) | MessageRouter, ContextBuilder, AgentExecutor, LLM Gateway | ✅ Built |
| **Agent Runtime** | 23 TypeScript agent definitions | System prompts, authority, tool restrictions | ✅ Built |
| **LLM Gateway** | Raw fetch adapters (no SDK) | OpenAI / Anthropic / Ollama routing | ✅ Built |
| **Tool Runner** | Node.js fs + child_process in `/workspaces/` | File I/O, git operations, shell commands | ✅ Built (path traversal + command blocking) |
| **Storage** | PostgreSQL (Docker, port 14000) via Prisma 7.x | All structured state — 24+ models | ✅ Built |
| **Code Workspace** | Disk: `/workspaces/{projectId}/` | Agent-generated source files | ✅ Built |
| **Git** | Local git in workspace dir (auto-commit) | Source code version control | ✅ Built |
| **Secrets** | AES-256-GCM encryption in PostgreSQL | LLM API keys at rest | ✅ Built |
| **Auth** | NextAuth.js v5 (dev mode auto-login) | Session management | 🔄 Dev mode — production auth Phase 3 |
| **Real-time** | Server-Sent Events (SSE) | Chat streaming + artifact delivery | ✅ Built |
| **Queue** | BullMQ + Redis (installed) | Agent task queue infrastructure | 🔄 Installed; parallel multi-agent queue not yet active |
| **VS Code Extension** | TypeScript + webpack v0.1.0 | File delivery + heartbeat gate | ✅ Built + packaged as .vsix |
| **Styling** | Tailwind CSS 4 | Dark theme + amber accent | ✅ Built |
| **State** | Zustand | Client-side state management | ✅ Built |
| **UI Components** | Radix UI, @dnd-kit, cmdk, framer-motion, Recharts | Rich interactive UI | ✅ Built |

## 24.2 Actual Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  Next.js 16 (Turbopack)                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Frontend (React 19, App Router)                     │   │
│  │  12 platform pages · kanban board · command palette  │   │
│  │  SSE streaming chat · decision UI · agent views      │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  API Routes                                          │   │
│  │  /api/projects/** · /api/admin/** · /api/llm/**      │   │
│  │  /api/projects/[id]/chat/stream (SSE)               │   │
│  │  /api/projects/[id]/artifacts/stream (SSE)          │   │
│  │  /api/projects/[id]/vscode-ping (heartbeat)         │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Orchestration Engine (src/lib/ai/orchestration/)    │   │
│  │  MessageRouter → ContextBuilder → AgentExecutor      │   │
│  │  LLM Gateway → OpenAI/Anthropic/Ollama adapters      │   │
│  │  card-lifecycle · authority-guard · loop-detector    │   │
│  │  quality-gates · event-bus · task-processor          │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  23 Agent Definitions (src/lib/ai/agents/)           │   │
│  │  System prompts · authority bounds · tool filters    │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Tool System (src/lib/ai/tools/)                     │   │
│  │  write_file · edit_file · read_file · list_directory │   │
│  │  glob · grep · run_command · git_commit · create_card│   │
│  │  create_document · update_card · create_decision     │   │
│  │  remember · search_artifacts · consult_agent · etc.  │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────┬──────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
   PostgreSQL (port 14000)  /workspaces/    Redis (heartbeat)
   Prisma 7.x · 24+ models  {projectId}/    BullMQ (planned)
   AES-256-GCM keys         .git + source
```

## 24.3 VS Code Extension Architecture

```
VS Code Extension v0.1.0
├── WorkspaceManager         — SSE artifact stream consumer + file writer
├── FileTracker              — Tracks written files, prevents duplicates
├── ApiClient                — Authenticated HTTP client to web app
├── Heartbeat (setInterval)  — POST /api/projects/{id}/vscode-ping every 25s
│                              Redis TTL 30s — dev agents blocked without this
└── Views
    ├── Projects & Board     — TreeView of projects, cards, decisions
    ├── AI Team              — Agent status view
    └── Chat (webview)       — Embedded chat UI
```

---

# 25. Multi-Tenant SaaS Architecture

> **Implementation Status (2026-03-27):** The data model supports multi-tenancy with Organization and OrgMember models in PostgreSQL, and each project is scoped to an owner. Full tenant isolation, shared services architecture, and SaaS deployment are Phase 3 targets.

## 25.1 Isolation Model

| Layer | Isolation Level | Status |
|-------|----------------|--------|
| **Project state** | Fully isolated — each project has its own cards, decisions, events in PostgreSQL | ✅ Built — projectId foreign key on all records |
| **Source code** | Fully isolated — separate `/workspaces/{projectId}/` directory per project | ✅ Built |
| **LLM configuration** | Per-project model profile overrides | ✅ Built — resolution priority chain |
| **Organization model** | Organization + OrgMember tables | ✅ Schema built |
| **Cross-tenant isolation** | Strict API scoping | 🔄 Partial — tenant boundary enforcement planned Phase 3 |

## 25.2 Shared Services (Target State)

| Service | Shared Across Tenants |
|---------|----------------------|
| LLM Gateway | Yes (with per-project routing rules) |
| Policy/Prompt Engine | Yes (with per-project overrides) |
| Budget Enforcer | Yes (with per-project caps) |
| Observability Stack | Yes (with per-project filtering) |

## 25.3 Tenant Boundary Rules (Target)

- No cross-tenant data access under any circumstance
- Tenant ID is embedded in every record
- API calls are scoped to tenant context
- Admin console shows only tenant-owned resources

---

# 26. Deployment Topologies

## 26.1 Current Mode (Dev / Hybrid)

| Component | Location | Status |
|-----------|----------|--------|
| UI | Local browser (Next.js dev server port 3000) | ✅ Running |
| Backend | Next.js on port 3000 | ✅ Running |
| PostgreSQL | Docker container on port 14000 | ✅ Running |
| Redis | Docker container (for heartbeat + BullMQ) | ✅ Running |
| Workspace | Local filesystem `/workspaces/{projectId}/` | ✅ Running |
| Git | Local git in workspace dir | ✅ Auto-initialized |
| LLM | Admin-configured: OpenAI/Anthropic/Ollama | ✅ Running |
| VS Code Extension | Local VS Code install (v0.1.0 .vsix) | ✅ Packaged |

## 26.2 Local Mode (Planned)

| Component | Location |
|-----------|----------|
| UI | Local browser |
| Control Plane | Local |
| Workspace | Local filesystem |
| Git | Local Git repository |
| LLM | Local model (Ollama) or remote API |
| Tool Runner | Local process sandbox |

**Best for**: Individual developers, offline work, privacy-sensitive projects.

## 26.3 Hybrid Mode (Planned)

| Component | Location |
|-----------|----------|
| UI | Browser |
| Control Plane | Cloud |
| Workspace | Local filesystem (via VS Code extension) |
| Git | GitHub / GitLab |
| LLM | Cloud API |

**Best for**: Small teams, collaborative development with local code.

## 26.4 Full SaaS Mode (Planned — Phase 3)

| Component | Location |
|-----------|----------|
| UI | Web browser |
| Control Plane | Cloud |
| Workspace | Cloud-managed |
| Git | Managed Git service |
| LLM | BYOM Gateway (cloud) |
| Tool Runner | Isolated cloud sandboxes |
| Storage | Multi-tenant cloud store |

**Best for**: Enterprise teams, fully managed experience, compliance requirements.

---

# 27. Scalability & Performance Engineering

## 27.1 Design Principles (Current and Target)

| Principle | Implementation | Status |
|-----------|---------------|--------|
| **Stateless orchestrator** | All state from PostgreSQL — no in-memory state | ✅ Built |
| **Prisma connection pooling** | PostgreSQL connection pool via Prisma | ✅ Built |
| **SSE for streaming** | Token-by-token agent output via SSE | ✅ Built |
| **BullMQ task queue** | Installed for parallel agent execution | 🔄 Installed; activation planned |
| **Horizontal scaling** | Multiple Next.js instances behind load balancer | 📋 Planned Phase 3 |
| **Distributed agent execution** | Agents on separate workers via BullMQ | 📋 Planned Phase 3 |
| **Caching** | Redis for heartbeat; query caching planned | 🔄 Partial |

## 27.2 Performance Targets

| Metric | Target |
|--------|--------|
| State transition latency | < 100ms |
| Agent task assignment | < 500ms |
| Board load time | < 1s for projects with < 1000 cards |
| LLM request routing | < 50ms overhead |
| UI refresh after state change | < 200ms |

---

# 28. Disaster Recovery & Rollback Strategy

> **Implementation Status (2026-03-27):** Git history in the workspace directory provides code rollback capability. PostgreSQL provides durable state with standard backup mechanisms. Automated DR, release rollback packages, and deterministic replay are Phase 3 targets.

## 28.1 Recovery Mechanisms

| Mechanism | RPO | RTO | Status |
|-----------|-----|-----|--------|
| **PostgreSQL backups** | Configurable | < 30 min restore | 🔄 Standard DB backup — automated DR planned |
| **Git history** | Every write_file commit | < 5 min | ✅ Auto git-commit on every file write |
| **Release rollback packages** | Last release | < 10 min | 📋 Planned with deployment pipeline |
| **Deterministic replay** | Full history | Varies | 📋 Events stored; replay engine planned Phase 3 |

## 28.2 Rollback Levels

| Level | What Gets Rolled Back | Status |
|-------|----------------------|--------|
| **Code rollback** | Git revert in workspace dir | ✅ Via git history |
| **State rollback** | PostgreSQL restore | 🔄 Manual DB restore; automated rollback planned |
| **Decision rollback** | Not possible — decisions are immutable (but new reversal decisions can be created) | ✅ Immutable design preserved |
| **Full project rollback** | Code + DB state to named checkpoint | 📋 Planned Phase 3 |
