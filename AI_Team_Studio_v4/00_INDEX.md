# AI TEAM STUDIO — Document Index

## Version 4.1 — Updated 2026-03-27 to reflect actual implementation

> **Note:** v4.1 reflects the actual built state as of 2026-03-27. Phase 1 and Phase 2 are substantially complete. Phase 3 and Phase 4 remain planned. See `13_CONTROLS_AND_ROADMAP.md` for the full status breakdown.

---

## Document Map

| # | File | Contents |
|---|------|----------|
| 00 | `00_INDEX.md` | This file — master index and reading guide |
| 01 | `01_EXECUTIVE_VISION.md` | Vision, market positioning, target users, foundational principles |
| 02 | `02_STATE_AND_MEMORY.md` | Memory architecture, deterministic state engine, resume strategy |
| 03 | `03_SDLC_OPERATING_MODEL.md` | 10-stage SDLC lifecycle, quality gates |
| 04 | `04_CARDS_AND_DECISIONS.md` | Card types, states, transitions, decision lifecycle framework |
| 05 | `05_AGENTS_AND_RACI.md` | 23 agents, persona authority, full RACI matrix, artifact ownership |
| 06 | `06_GIT_CICD_RELEASE.md` | Git governance, branch strategy, CI/CD pipeline, release process |
| 07 | `07_INTEGRATIONS_LLM_SECRETS.md` | Integration setup, secret management, BYOM/LLM architecture |
| 08 | `08_UIUX_AND_WIREFRAMES.md` | UI/UX governance, wireframe JSON schema, component model |
| 09 | `09_QUALITY_AND_SECURITY.md` | Definition of Done, security framework, risk management |
| 10 | `10_OBSERVABILITY_AND_COST.md` | Observability stack, reliability, cost governance, budget controls |
| 11 | `11_ARCHITECTURE_AND_DEPLOYMENT.md` | System architecture, multi-tenant SaaS, deployment topologies, scalability, disaster recovery |
| 12 | `12_EVENTS_REPLAY_SCHEMAS.md` | Event bus design, deterministic replay, JSON schemas for all data models |
| 13 | `13_CONTROLS_AND_ROADMAP.md` | Enterprise policy enforcement, phased roadmap |
| 14 | `14_ARCHITECTURE_DIAGRAMS.md` | All 11 Mermaid architecture diagrams |
| A | `APPENDIX_A_AGENT_CONTRACTS.md` | Behavioral contracts for all 23 agents |
| B | `APPENDIX_B_STATE_TRANSITIONS.md` | Full state transition matrix (cards + decisions + illegal transitions) |
| C | `APPENDIX_C_DECISION_TEMPLATES.md` | Templates: architecture, dependency, scope change, security |
| D | `APPENDIX_D_SECURITY_CHECKLIST.md` | Stage-by-stage security control checklist |
| E | `APPENDIX_E_KPI_DASHBOARD.md` | KPI definitions, targets, dashboard layout |
| 15 | `15_GAPS_AND_PHASE_BREAKDOWN.md` | Full gap inventory, sprint order, build status snapshot (2026-03-27) |

---

## Reading Order

**For executives / board presentations:**
→ 01 → 03 → 05 (agents overview) → 13 (roadmap) → Appendix E (KPIs)

**For architects / technical leads:**
→ 01 → 02 → 03 → 04 → 05 → 06 → 11 → 12 → 14 (diagrams)

**For developers:**
→ 04 (cards) → 06 (git) → 09 (quality/DoD) → 12 (schemas) → Appendix B (transitions)

**For security / compliance:**
→ 01 (principles) → 07 (secrets) → 09 (security) → Appendix D (checklist)

**Full read (all stakeholders):**
→ 00 through 14, then Appendices A–E

---

## Implementation Status Summary (as of 2026-03-27)

| Phase | Status | Key Deliverables |
|-------|--------|-----------------|
| **Phase 1 — Foundation** | ✅ Complete | Next.js 16 + PostgreSQL/Prisma + Docker, all 12 platform pages, 4-step project wizard, drag-and-drop kanban board, command palette, onboarding wizard |
| **Phase 2 — Full SDLC** | ✅ Mostly Complete | 23 agents with system prompts + authority, orchestration engine (MessageRouter/ContextBuilder/AgentExecutor/LLM Gateway), SSE streaming chat, tool system (13 filesystem/git/project tools), VS Code extension v0.1.0 with SSE artifact delivery, heartbeat gate, auto git-commit, loop detection, card lifecycle state machine, wireframe generation, AES-256-GCM key encryption |
| **Phase 3 — Enterprise** | 🔄 In Progress | Auth in dev mode only, no Stripe billing, WebSocket dashboard updates not yet done, GitHub/GitLab PR creation stub only, deterministic replay not yet implemented, enterprise SSO/RBAC not started |
| **Phase 4 — Scale** | 📋 Planned | Horizontal scaling, marketplace, custom agent authoring, advanced analytics, white-label |

---

## Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2024 | Initial full architecture (PDF) |
| 2.0 | 2024 | Enterprise master — expanded to 20 sections |
| 3.0 | 2024 | Board-ready blueprint — 35 sections with appendices |
| **4.0** | **2025** | **Unified master — all sources combined, gaps filled, schemas defined, split into modular docs** |
| **4.1** | **2026-03-27** | **Implementation reality update — Phase 1 & 2 actual state documented; Phase 3/4 clearly marked planned** |

---

Also available as a single combined file: `AI_TEAM_STUDIO_MASTER.md`
