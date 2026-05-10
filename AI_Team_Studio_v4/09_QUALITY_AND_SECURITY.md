# 19. Quality Assurance & Definition of Done

> **Implementation Status (2026-03-27):** The core DoD enforcement mechanisms are built. Card lifecycle validation in `card-lifecycle.ts` and `quality-gates.ts` enforces state transitions. The loop detector in `loop-detector.ts` prevents runaway agent loops. Authority guard prevents cross-boundary edits. Path traversal protection and dangerous command blocking are active in the tool executor.

## 19.1 Task DoD

| Criterion | Verification | Status |
|-----------|-------------|--------|
| Code implemented | Source files exist in /workspaces/{projectId}/ | ✅ JD writes files via write_file tool |
| Unit tests passing | Test runner reports all pass | ✅ run_tests tool available to QA agent |
| Error handling added | Try/catch, validation, edge cases covered | ✅ JD system prompt requires complete error handling |
| No new dependencies without approval | Decision required for any new package | ✅ Enforced via create_decision requirement |
| Code follows project patterns | SD confirms during review | ✅ SD review authority enforced |
| Card marked DONE after completion | JD must call update_card(state: DONE) | ✅ JD system prompt critically enforces this |

## 19.2 Feature DoD

| Criterion | Verification | Status |
|-----------|-------------|--------|
| End-to-end flow works | QA validates complete user journey | ✅ QA agent authority |
| All QA scenarios pass | QA evidence logged | ✅ QA creates validation cards |
| No open defects | All defect cards resolved or deferred (with decision) | ✅ BLOCKED state + decision requirement |
| Integrated safely | No regressions in existing features | ✅ run_tests tool available |
| Accessibility checked | Basic a11y validation | 📋 Planned |

## 19.3 Module (Epic) DoD

| Criterion | Verification | Status |
|-----------|-------------|--------|
| All features Done | Every child feature card is Done | ✅ Card hierarchy + state validation |
| Stable interfaces | API contracts confirmed, no breaking changes | ✅ SA artifact authority |
| Regression passed | Full regression suite passes | 📋 CI pipeline planned |
| Documentation updated | BRD/SDD reflect current implementation | ✅ update_document tool available |

## 19.4 Release DoD

| Criterion | Verification | Status |
|-----------|-------------|--------|
| All modules ready | Every module in scope passes Module DoD | ✅ Card state validation |
| Rollback tested | Rollback artifact generated and verified | 📋 Deployment pipeline planned |
| Release notes complete | Stakeholder-facing release notes written | 📋 Planned |
| Stakeholders notified | PM confirms communication sent | 📋 Planned |
| Monitoring active | SRE confirms dashboards and alerts are live | 📋 Planned |

## 19.5 Enforcement

The following quality enforcement mechanisms are **built and active**:

| Mechanism | File | What It Enforces |
|-----------|------|-----------------|
| Card lifecycle state machine | `card-lifecycle.ts` | Blocks illegal card state transitions at API + tool level |
| Authority guard | `authority-guard.ts` | Blocks agents from writing outside their authority |
| Loop detector | `loop-detector.ts` | Fuzzy tool-loop + text-repetition + question-reask detection |
| Quality gates | `quality-gates.ts` | DoD checks for stage transitions |
| Path traversal protection | `workspace.ts` | Blocks `../` escapes in file paths |
| Dangerous command blocking | `tool-executor.ts` | Blocks rm -rf, sudo, and similar shell commands |
| VS Code heartbeat gate | `/api/projects/[id]/vscode-ping` | Dev agents blocked without active extension connection |

---

# 20. Security & Compliance Framework

> **Implementation Status (2026-03-27):** Core security controls are active. API key encryption, path traversal protection, command blocking, and authority guard are all built. Production authentication, enterprise RBAC, and advanced security dashboards are Phase 3 targets.

## 20.1 Security Controls

| Control | Stage | Responsible Agent | Status |
|---------|-------|------------------|--------|
| Threat modeling | Architecture (SDD) | Security & Compliance | ✅ SEC agent defined with threat modeling system prompt |
| Dependency scanning | Development + CI | Security & Compliance | ✅ check_dependencies tool available |
| Code security review | Code Review | SD + Security & Compliance | ✅ review_changes + validate_code tools |
| Secret leak prevention | Pre-commit (planned) + encryption | Secrets Agent | ✅ AES-256-GCM encryption active; git hooks planned |
| Audit trail integrity | Continuous | Audit Gatekeeper | ✅ Event table + AuditLog table in PostgreSQL |
| Role-based authority | Continuous | State Controller | ✅ authority-guard.ts |
| Path traversal prevention | Tool execution | Tool Executor | ✅ workspace.ts blocks ../  escapes |
| Dangerous command blocking | Tool execution | Tool Executor | ✅ rm -rf, sudo, etc. blocked |
| API key encryption at rest | Admin config | Secrets Agent | ✅ AES-256-GCM in encryption.ts |
| Production authentication | Auth layer | Platform | 🔄 NextAuth dev mode; production auth is Phase 3 |
| HTTPS enforcement | Deployment | DevOps | 📋 Production deployment planned |
| Enterprise RBAC | Multi-tenant | Platform | 📋 Phase 3 |

## 20.2 Compliance Features

| Feature | Status |
|---------|--------|
| Immutable event log | ✅ PostgreSQL Event table (append pattern enforced) |
| Full traceability (code → task → feature → epic → BRD) | ✅ Card requirementIds + parent/child chain |
| Role-based access (agent authority boundaries) | ✅ authority-guard.ts |
| Audit log | ✅ PostgreSQL AuditLog table |
| Incident traceability | 📋 Planned (SRE monitoring Phase 3) |
| Deterministic replay | 📋 Events stored; replay engine planned Phase 3 |

---

# 21. Risk Management Model

## 21.1 Risk Categories

| Category | Assessed By | Timing | Status |
|----------|------------|--------|--------|
| **Architectural Risk** | Solution Architect | SDD stage | ✅ SA agent with risk assessment in system prompt |
| **Security Risk** | Security & Compliance Agent | Architecture + Code Review | ✅ SEC agent (VS Code gate required) |
| **Performance Risk** | Performance Engineer | Development + Testing | ✅ PE agent defined |
| **Cost Risk** | Cost & Telemetry Analyst | Continuous | 🔄 Schema defined; dashboard planned |
| **Dependency Risk** | Platform Engineer | Planning + Development | ✅ check_dependencies tool |
| **Schedule Risk** | Product Manager | Planning + Sprint | ✅ PM agent with roadmap authority |

## 21.2 Risk Scoring

| Rating | Definition | Action |
|--------|-----------|--------|
| **Low** | Unlikely to impact delivery | Monitor only |
| **Medium** | Could impact delivery if unaddressed | Mitigation plan required |
| **High** | Will likely impact delivery | Decision required before proceeding |
| **Critical** | Blocks delivery entirely | Immediate escalation to TL + PM |

## 21.3 Risk Tracking

All risks are recorded in the **Decision Ledger** (PostgreSQL Decision table) with:
- Risk rating attached to the relevant decision
- Mitigation strategy documented as the recommended option
- Risk owner assigned
- Review cadence set (e.g., re-evaluate at next quality gate)
