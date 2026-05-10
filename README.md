# Codanium — Your Vibe, Multiplied

[![Build & Deploy](https://github.com/AiSenseiMY/Ai-Team_studio/actions/workflows/deploy.yml/badge.svg)](https://github.com/AiSenseiMY/Ai-Team_studio/actions/workflows/deploy.yml)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)](https://redis.io/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Prisma](https://img.shields.io/badge/Prisma-7.x-2D3748?logo=prisma&logoColor=white)](https://www.prisma.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![LLM Providers](https://img.shields.io/badge/LLM_Providers-10+-8B5CF6)]()
[![AI Agents](https://img.shields.io/badge/AI_Agents-24-F59E0B)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

> Tell us what you need. Our AI team will build and deliver it.

**Codanium** is a **full-service AI software delivery platform**. Users describe what they want built and a team of 24 specialized AI agents handles the entire development lifecycle — requirements, architecture, scaffolding, design, coding, testing, security review, and deployment.

**The user is the stakeholder, not a developer.** No technical background required.

[![Website](https://img.shields.io/badge/Website-codanium.com-F59E0B?style=for-the-badge)](https://codanium.com)
[![Desktop App](https://img.shields.io/badge/Desktop_App-Download-10B981?style=for-the-badge&logo=github)](https://github.com/AiSenseiMY/Codanium/releases)

### Live URLs

| Resource | URL |
|----------|-----|
| **Production App** | [https://codanium.com](https://codanium.com) |
| **GitHub (Web App)** | [https://github.com/AiSenseiMY/Ai-Team_studio](https://github.com/AiSenseiMY/Ai-Team_studio) |
| **GitHub (Desktop)** | [https://github.com/AiSenseiMY/Codanium](https://github.com/AiSenseiMY/Codanium) |
| **Desktop Downloads** | [https://github.com/AiSenseiMY/Codanium/releases](https://github.com/AiSenseiMY/Codanium/releases) |
| **API Health Check** | [https://codanium.com/api/llm/health](https://codanium.com/api/llm/health) |

---

## Repository Structure

This is a monorepo. Each top-level directory is an independent piece of the platform:

```
.
├── ai-team-studio/           # Web application (Next.js 16, Prisma, PostgreSQL, BullMQ)
├── codanium-desktop/         # Desktop app (Tauri v2, React 19) — primary client
├── ai-team-studio-vscode/    # VS Code extension (deprecated; replaced by codanium-desktop)
├── AI_Team_Studio_v4/        # Enterprise architecture documentation (20 docs)
└── README.md / LICENSE / CONTRIBUTING.md / CODE_OF_CONDUCT.md / SECURITY.md
```

---

## Quick Start

### Web App

```bash
cd ai-team-studio

# Start PostgreSQL + Redis (Docker required)
docker compose up -d

# Install dependencies
npm install

# Sync database schema
npx prisma db push

# Seed with sample data
npx prisma db seed

# Start dev server (http://localhost:3000)
npm run dev
```

### Docker (Production)

```bash
cd ai-team-studio
docker compose up -d    # Starts app (14001), worker, postgres (14000), redis (14003)
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16 (Turbopack, React 19, App Router) |
| Database | PostgreSQL via Docker (port 14000), Prisma 7.x |
| Auth | NextAuth.js (dev mode auto-login) |
| Styling | Tailwind CSS 4, dark theme, amber accent |
| State | Zustand, framer-motion |
| Components | Radix UI, Recharts, cmdk, @dnd-kit |
| AI Backend | Multi-provider LLM — OpenAI / Anthropic / Ollama / NVIDIA / Mistral / Groq |
| Encryption | AES-256-GCM (API keys at rest) |
| Task Queue | BullMQ + Redis for background agent work |

---

## How It Works — PM Gatekeeper Pipeline

```
User describes idea (web app)
  → PM (Product Manager) activates as gatekeeper
  → PM creates card → BA collects requirements → BRD → PM validates
  → PM creates card → SA designs architecture → SDD → PM validates
  → PM creates card → DO scaffolds project → verifies build passes
  → PM hands to TL → UX creates UI Kit → UI Designer builds interfaces
  → User approves UI in "Designs" menu
  → PM confirms all gates passed (BRD + SDD + Scaffold + UI)
  → TL assigns ONE feature card → JD/SD codes → tsc validation
  → QA + SEC + DO + PE quad sign-off → card DONE
  → TL picks next card → repeat until all features built
  → PM validates → deployment
```

### Key Principles
- **PM is gatekeeper** — creates all phase cards, validates all outputs
- **One card at a time** — dev finishes one feature fully before starting the next
- **Quad sign-off** — QA + SEC + DO + PE must all approve before a card is DONE
- **Everything on the board** — all work tracked as cards on the Work Board
- **Code must compile** — dev runs `tsc --noEmit` before requesting sign-off

---

## AI Agents (15 Pipeline + 3 Internal)

### Pipeline Agents

| Agent | Role |
|-------|------|
| **PM** | Gatekeeper — creates all phase cards, validates outputs, controls pipeline flow |
| **BA** | Requirements gathering — discovers needs from user, generates BRD |
| **SA** | Architecture — designs system, generates SDD |
| **DO** | DevOps — project scaffolding, deployment, infrastructure sign-off |
| **UX** | UX Designer — brand identity, color palette, typography, UI Kit |
| **UID** | UI Designer — builds page layouts, wireframes, components using UI Kit |
| **TL** | Tech Lead — manages UX/UI and dev card assignments, coordinates pipeline |
| **JD** | Junior Developer — implements features, writes code |
| **SD** | Senior Developer — complex features, architecture-level code |
| **QA** | QA Engineer — code review, test strategy, functional sign-off |
| **SEC** | Security — vulnerability review, security sign-off |
| **PE** | Platform Engineer — infrastructure readiness, scaling sign-off |

### Internal Agents

| Agent | Role |
|-------|------|
| **ORC** | Orchestrator — message routing, intent classification |
| **STC** | State Controller — enforces card lifecycle rules |
| **DEC** | Decision Controller — creates/manages decision records |

---

## LLM Configuration

### Multi-Provider Fallback
Configure multiple LLM providers with priority order in Admin Settings:
- **Primary** → **Fallback 1** → **Fallback 2** → **Last Resort**
- If primary fails (rate limit, timeout), automatically tries next provider
- Each provider stores: name, API key (encrypted), base URL, model, priority

### Supported Providers
OpenAI, Anthropic, Ollama, NVIDIA, Mistral, Groq, Together, Custom (OpenAI-compatible)

### Resolution Priority
1. User-level BYOK config (user's own key)
2. Agent-level override (admin per-agent)
3. Project-level override (admin per-project)
4. Platform fallback chain (ordered by priority)
5. Admin default settings (last resort)

---

## Implementation Status

| Phase | Status |
|-------|--------|
| Phase 1 — Foundation | Done |
| Phase 2 — Full SDLC Pipeline | Done |
| Phase 2.5 — Pipeline Hardening (PM gatekeeper, quad sign-off, scaffolding) | Done |
| Phase 3 — Enterprise (auth, billing, WebSocket, GitHub PR, deploy) | In Progress |
| Phase 4 — Custom Desktop App (replacing VS Code extension) | Planned |
| Phase 5 — Scale (marketplace, multi-tenant SaaS, white-label) | Planned |

### What's Built
- PM-gatekeeper SDLC pipeline with 6 phases (Requirements → Architecture → Scaffolding → UI/UX → Development → Release)
- 15 pipeline agents + 3 internal agents with system prompts and authority levels
- Multi-LLM provider fallback with encrypted API key storage
- Model-agnostic chat sanitization (strips tool call syntax from all LLM providers)
- Card lifecycle with quad sign-off (QA + SEC + DO + PE)
- Card comments + sign-off tracking (Jira-style audit trail)
- Token usage tracking per card
- Semantic card deduplication (70% word overlap detection)
- Loop detection with Decision record creation (My Decisions menu)
- "Designs" menu for UX/UI approval workflow
- Real code generation — agents write actual files to disk
- AES-256-GCM API key encryption
- Drag-and-drop kanban board, command palette, onboarding wizard

### What's Still Pending
- Production authentication (currently NextAuth dev/demo mode)
- Stripe billing & subscriptions
- Custom desktop app (replacing VS Code extension)
- GitHub/GitLab PR creation
- Cloud deployment pipeline

---

## Documentation

Full enterprise architecture docs in [`AI_Team_Studio_v4/`](AI_Team_Studio_v4/00_INDEX.md) — 20 documents covering vision, SDLC model, agent contracts, security, observability, architecture diagrams, and roadmap.

Test reports from SDLC pipeline testing in [`ai-team-studio/test-reports/`](ai-team-studio/test-reports/).

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for setup instructions per subproject and the PR workflow. By participating you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).

## Security

Found a security issue? Please report it privately — see [SECURITY.md](SECURITY.md). Do **not** open a public issue for vulnerabilities.

## License

[MIT](LICENSE) © 2026 AiSenseiMY

---

*Last updated: 2026-05-10*
