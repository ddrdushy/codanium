# 1. Executive Vision & Strategic Intent

> **Implementation Status (2026-03-27):** The vision and principles in this document are fully reflected in the built system. The platform is operational with Phase 1 and Phase 2 substantially complete. Multi-tenant SaaS and enterprise SSO remain Phase 3 targets.

AI Team Studio is a **deterministic AI-native Product Delivery Operating System**.

It replaces traditional IDE-driven, chat-driven development with structured, state-driven, governance-controlled SDLC orchestration.

The system embeds a **complete product delivery organization** inside software:

- Business Analyst
- Solution Architect
- UI/UX Designer
- Developers (Junior & Senior)
- QA Engineers
- Platform & DevOps Engineers
- Tech Lead
- Product Manager
- Security & Compliance
- SRE & Reliability
- AI Cost Governance

**Core Outcomes:**

- Predictable, deterministic delivery
- Fully auditable change history
- Controlled AI model usage with budget enforcement
- Resume-anytime capability — no context loss
- Enterprise compliance readiness from day one

---

# 2. Market Positioning & Differentiation

## Traditional AI IDEs

| Weakness | Impact |
|----------|--------|
| Chat-driven | Progress lost when context window resets |
| No governance layer | No authority boundaries, no approval gates |
| Context-loss prone | Cannot resume reliably after interruption |
| Weak audit trails | No decision history, no traceability |
| No SDLC structure | Skips requirements, design, QA stages |

## AI Team Studio

| Strength | Impact | Built? |
|----------|--------|--------|
| State-driven engine | Progress persisted in PostgreSQL, not memory | ✅ Complete |
| Decision ledger enforcement | Every change is traceable and auditable | ✅ Complete |
| Persona authority routing | Each agent has bounded responsibilities | ✅ Complete |
| Deterministic orchestration | Predictable, repeatable outcomes | ✅ Complete |
| Enterprise-grade compliance | Built-in governance, security, audit trails | 🔄 Partial (events stored; replay engine planned) |
| Resume-anytime | Reopen project and continue from exact state | ✅ Complete (via PostgreSQL state) |
| BYOM (Bring Your Own Model) | No vendor lock-in for AI providers | ✅ Complete (OpenAI/Anthropic/Ollama, admin-configured) |

---

# 3. Target Users

| User Type | Use Case |
|-----------|----------|
| Non-technical founders / ideators | Build software products without coding expertise |
| Product managers | Manage full delivery lifecycle with AI agents |
| Technical leads | Orchestrate teams with governance guardrails |
| Small teams | Replace missing roles with AI personas |
| Enterprise teams | Achieve compliance, audit, and governance goals |

---

# 4. Foundational Principles

These principles are **non-negotiable** and govern all system behavior:

| # | Principle | Rationale | Implementation Status |
|---|-----------|-----------|----------------------|
| 1 | **Agents do not remember. State remembers.** | State is persisted in PostgreSQL (cards, decisions, artifacts, events), not in agent memory or chat history. | ✅ Built — 24+ model Prisma schema with full persistence |
| 2 | **State + Decisions drive progress.** | Chat is an interface layer only. Progress is tracked through card states and recorded decisions. | ✅ Built — card lifecycle state machine enforced in API routes and tool executor |
| 3 | **No approval bypass.** | Every quality gate must be passed. No shortcutting from requirements to code. | ✅ Built — authority guard + card state validation |
| 4 | **Wireframes precede UI code.** | Structured wireframes must be approved before any UI implementation begins. | ✅ Built — UX agent generates wireframe JSON artifacts |
| 5 | **Secrets never stored in Git.** | API keys, tokens, and credentials are stored encrypted at rest. Only references appear in project files. | ✅ Built — AES-256-GCM encryption for LLM API keys at rest |
| 6 | **Release requires a rollback path.** | No deployment without a tested rollback strategy. | 📋 Planned — deployment pipeline not yet built |
| 7 | **Deterministic replay must be possible.** | Given the same event history, the system must reconstruct the same state. | 🔄 Partial — events are stored in PostgreSQL; replay engine not yet implemented |
| 8 | **Every artifact has a single owner.** | One persona owns each artifact. No cross-editing permitted without a formal decision. | ✅ Built — authority guard enforces per-agent write permissions |
