# 10. Personas & Authority Model

> **Implementation Status (2026-03-27):** All 23 agents are implemented with system prompts, authority levels, tool restrictions, and context sources. Agent definitions live in `src/lib/ai/agents/definitions/`. The authority guard (`authority-guard.ts`) enforces per-agent write permissions at runtime. Dev agents (TL, JD, SD, QA, DO, SEC) are blocked from executing until the VS Code extension is connected and the heartbeat is passing.

## 10.1 Persona Definitions

| Persona | Code | Authority Domain | Writes To | Built? |
|---------|------|-----------------|-----------|--------|
| Project Lead | PL | Routing only — no authority over artifacts | N/A (router) | ✅ |
| Business Analyst | BA | Requirements & business rules | BRD, requirement cards | ✅ |
| Solution Architect | SA | Architecture & feasibility | SDD, architecture decisions | ✅ |
| UI/UX Designer | UX | Branding, wireframes, UI Kit | Wireframes, UI specifications | ✅ |
| Junior Developer | JD | Task implementation | Source code files via write_file, cards | ✅ (VS Code gate required) |
| Senior Developer | SD | Code review & quality | Review logs, card state updates | ✅ (VS Code gate required) |
| QA Engineer | QA | Functional validation | QA evidence, defect cards | ✅ (VS Code gate required) |
| Platform Engineer | PE | Environment & dependencies | Platform config, dependency manifests | ✅ |
| DevOps Engineer | DO | Release pipeline & deployment | CI/CD config, release tags | ✅ (VS Code gate required; deployment pipeline planned) |
| Tech Lead | TL | Final merge & release authority | Release approvals, merge permissions | ✅ (VS Code gate required) |
| Product Manager | PM | Scope & priority decisions | Roadmap, priority assignments | ✅ |
| Automation Test Agent | AT | Automated test suites | E2E tests, integration tests | ✅ (agent defined) |
| Performance Engineer | PF | Performance validation | Performance benchmarks, load test results | ✅ (agent defined) |
| Security & Compliance | SEC | Security review & policy | Security audit logs, compliance reports | ✅ (VS Code gate required) |
| SRE / Reliability | SR | Post-release monitoring | Incident logs, health reports | ✅ (agent defined) |

## 10.2 Authority Rules

1. **Each persona writes only to its allowed artifacts** — enforced by `authority-guard.ts`
2. **No persona can override another persona's approved work** without a recorded decision
3. **All changes requiring direction must go through the Decision Lifecycle** (see 04_CARDS_AND_DECISIONS.md)
4. **Cross-domain changes require a decision** involving both domain owners
5. **Dev agents require VS Code heartbeat** — JD, SD, QA, TL, DO, SEC are blocked without active VS Code extension connection

---

# 11. Full Agent Organization (23 Agents)

## 11.1 Governance Agents (5)

| # | Agent | Short Name | Responsibility | Built? |
|---|-------|------------|---------------|--------|
| 1 | **Orchestrator** | ORC | Routes tasks to correct persona, manages workflow sequencing | ✅ MessageRouter in orchestration engine |
| 2 | **State Controller** | SC | Validates state transitions, writes card state, blocks illegal moves | ✅ card-lifecycle.ts |
| 3 | **Decision Controller** | DEC | Manages decision lifecycle, enforces approval requirements | ✅ decision model + create_decision tool |
| 4 | **Audit & Quality Gatekeeper** | AUD | Enforces DoD at every level, blocks stage transitions if unmet | ✅ quality-gates.ts |
| 5 | **Security & Compliance Agent** | SEC | Threat modeling, dependency scanning, policy enforcement | ✅ (VS Code gate required) |

## 11.2 SDLC Delivery Agents (5)

| # | Agent | Short Name | Responsibility | Built? |
|---|-------|------------|---------------|--------|
| 6 | **Business Analyst** | BA | Gathers requirements, writes BRD, manages requirement cards | ✅ Full system prompt + create_document tool |
| 7 | **Solution Architect** | SA | Designs architecture, writes SDD, validates feasibility | ✅ Full system prompt + architecture authority |
| 8 | **UI/UX Designer** | UX | Creates wireframes, defines UI Kit, manages branding | ✅ Generates wireframe JSON artifacts (22 wireframes in sample project) |
| 9 | **Product Manager** | PM | Manages scope, priorities, roadmap, stakeholder communication | ✅ Full system prompt |
| 10 | **Tech Lead** | TL | Final technical authority, merge approvals, release sign-off | ✅ (VS Code gate required) |

## 11.3 Engineering Agents (5)

| # | Agent | Short Name | Responsibility | Built? |
|---|-------|------------|---------------|--------|
| 11 | **Junior Developer** | JD | Implements tasks, writes real files via write_file tool | ✅ Writes to /workspaces/{projectId}/ (VS Code gate required) |
| 12 | **Senior Developer** | SD | Reviews code, ensures quality | ✅ (VS Code gate required) |
| 13 | **QA Engineer** | QA | Writes and executes test scenarios, validates functionality | ✅ (VS Code gate required) |
| 14 | **Automation Test Agent** | AT | Builds and maintains automated test suites (E2E, integration) | ✅ Agent defined |
| 15 | **Performance Engineer** | PF | Load testing, performance benchmarks, optimization recommendations | ✅ Agent defined |

## 11.4 Platform & Operations Agents (5)

| # | Agent | Short Name | Responsibility | Built? |
|---|-------|------------|---------------|--------|
| 16 | **Platform Engineer** | PE | Environment setup, dependency management, infrastructure config | ✅ Agent defined |
| 17 | **DevOps Engineer** | DO | CI/CD pipelines, deployment automation, release tagging | ✅ (VS Code gate required; trigger_deploy is stub) |
| 18 | **Integration Engineer** | IE | Third-party API integrations, connectivity testing | ✅ Agent defined |
| 19 | **Secrets & Key Management Agent** | SM | Secure storage, rotation policies, access audit | ✅ AES-256-GCM admin key encryption implemented |
| 20 | **SRE / Reliability Engineer** | SR | Monitoring, alerting, incident response, postmortems | ✅ Agent defined |

## 11.5 AI & Cost Governance Agents (3)

| # | Agent | Short Name | Responsibility | Built? |
|---|-------|------------|---------------|--------|
| 21 | **LLM Gateway Manager (BYOM)** | CA | Model routing, provider configuration, fallback logic | ✅ Admin-configured LLM provider (OpenAI/Anthropic/Ollama) |
| 22 | **Prompt & Policy Engineer** | PE | Prompt templates, policy filters, output quality control | ✅ Agent defined |
| 23 | **Cost & Telemetry Analyst** | TA | Token tracking, budget enforcement, cost-per-feature reporting | 🔄 Telemetry schema defined; dashboard not yet built |

---

# 12. RACI Matrix (Full 23 Agents)

> **R** = Responsible (does the work) | **A** = Accountable (owns the outcome) | **C** = Consulted | **I** = Informed

| Activity | Orchestrator | State Ctrl | Decision Ctrl | Audit Gate | Sec & Comp | BA | SA | UI/UX | PM | TL | JD | SD | QA | Auto Test | Perf Eng | Platform | DevOps | Integration | Secrets | SRE | LLM Mgr | Prompt Eng | Cost Analyst |
|----------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Requirements Gathering | I | - | - | - | - | **R/A** | C | C | C | I | - | - | - | - | - | - | - | - | - | - | - | - | - |
| BRD Approval | I | R | - | A | - | R | C | - | C | I | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Architecture Design | I | - | - | - | C | C | **R/A** | C | I | C | - | - | - | - | C | C | - | - | - | - | - | - | - |
| SDD Approval | I | R | - | A | C | I | R | - | I | C | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Wireframe Design | I | - | - | - | - | C | C | **R/A** | C | I | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Wireframe Approval | I | R | - | A | - | I | I | R | C | I | - | - | - | - | - | - | - | - | - | - | - | - | - |
| WBS & Planning | **R** | R | - | - | - | C | C | C | C | **A** | I | I | I | - | - | I | I | - | - | - | - | - | - |
| Task Implementation | I | I | - | - | - | - | - | - | - | I | **R/A** | C | - | - | - | - | - | - | - | - | - | - | - |
| Code Review | I | I | - | - | - | - | - | - | - | C | I | **R/A** | - | - | - | - | - | - | - | - | - | - | - |
| QA Testing | I | I | - | C | - | - | - | - | - | I | I | I | **R/A** | C | - | - | - | - | - | - | - | - | - |
| Automated Testing | I | - | - | C | - | - | - | - | - | I | - | I | C | **R/A** | - | - | - | - | - | - | - | - | - |
| Performance Testing | I | - | - | C | - | - | C | - | - | I | - | - | I | - | **R/A** | - | - | - | - | - | - | - | - |
| Decision Management | C | I | **R/A** | I | C | C | C | C | C | C | - | - | - | - | - | - | - | - | - | - | - | - | - |
| State Transitions | R | **R/A** | I | C | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| DoD Enforcement | I | C | - | **R/A** | C | - | - | - | - | C | - | - | C | - | - | - | - | - | - | - | - | - | - |
| Security Review | I | - | C | C | **R/A** | - | C | - | I | I | - | C | - | - | - | - | - | - | C | - | - | - | - |
| Environment Setup | I | - | - | - | C | - | C | - | - | I | I | - | - | - | - | **R/A** | C | C | - | - | - | - | - |
| CI/CD Pipeline | I | - | - | C | C | - | - | - | - | C | - | - | - | - | - | C | **R/A** | - | - | - | - | - | - |
| Integration Setup | I | - | - | - | C | - | C | - | - | I | - | - | - | - | - | C | - | **R/A** | C | - | - | - | - |
| Secret Management | I | - | - | C | C | - | - | - | - | I | - | - | - | - | - | - | - | C | **R/A** | - | - | - | - |
| Release Deployment | I | R | - | A | C | - | - | - | I | **A** | - | - | - | - | - | - | **R** | - | - | I | - | - | - |
| Post-Release Monitor | I | - | - | - | - | - | - | - | I | I | - | - | - | - | - | C | C | - | - | **R/A** | - | - | - |
| LLM Config & Routing | I | - | - | - | - | - | - | - | - | I | - | - | - | - | - | - | - | - | - | - | **R/A** | C | C |
| Prompt Policy Mgmt | I | - | - | - | C | - | - | - | - | I | - | - | - | - | - | - | - | - | - | - | C | **R/A** | C |
| Cost & Budget Tracking | I | - | - | - | - | - | - | - | I | I | - | - | - | - | - | - | - | - | - | - | C | C | **R/A** |
| Incident Response | I | - | C | - | C | - | - | - | I | I | - | - | - | - | - | C | C | - | - | **R/A** | - | - | - |

---

# 13. Artifact Ownership Matrix

| Artifact | Owner Agent | Can Read | Cannot Edit | Built? |
|----------|-----------|----------|-------------|--------|
| BRD (Business Requirements) | BA | All agents | All except BA | ✅ create_document(type: BRD) |
| SDD (Solution Design) | SA | All agents | All except SA | ✅ create_document(type: SDD) |
| Wireframes (JSON) | UI/UX | All agents | All except UI/UX | ✅ UX generates wireframe artifacts |
| Source Code | JD | SD, QA, TL, Platform | All except JD | ✅ write_file/edit_file via VS Code |
| Code Review Logs | SD | JD, QA, TL | All except SD | ✅ SD review authority |
| QA Evidence & Test Results | QA | All agents | All except QA | ✅ QA creates validation cards |
| CI/CD Pipeline Config | DevOps | TL, Platform, SRE | All except DevOps | 📋 Pipeline planned |
| Release Tags & Notes | DevOps + TL | All agents | All except DevOps + TL | 📋 Deployment pipeline planned |
| Decision Ledger | Decision Controller | All agents | All except Decision Controller | ✅ create_decision tool; PostgreSQL Decision table |
| Board State | State Controller | All agents | Validated by card-lifecycle.ts | ✅ PostgreSQL Card table |
| Event Log | State Controller | All agents | Append-only | ✅ PostgreSQL Event table |
| Security Audit Logs | Security & Compliance | TL, Audit Gatekeeper | All except Security | ✅ Agent defined |
| LLM Provider Config | Admin (platform level) | Admin only | Non-admin users | ✅ AES-256-GCM encrypted in LLMProviderConfig table |

**Rule: No cross-editing permitted. Any change to another agent's artifact requires a formal Decision.**
