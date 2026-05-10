# 7. SDLC Operating Model

> **Implementation Status (2026-03-27):** The SDLC stage model is fully implemented. The 10-stage lifecycle is enforced through the card lifecycle state machine and agent authority restrictions. Quality gates are enforced structurally — dev agents are blocked until the VS Code extension is connected and the heartbeat gate passes. Document approval gates are enforced in the orchestration engine.

## 7.1 Lifecycle Stages

| Stage | # | Owner | Gate to Pass | Built? |
|-------|---|-------|-------------|--------|
| Business Analysis | 1 | BA Agent | BRD approved | ✅ BA agent with system prompt and create_document/approve_document tools |
| Solution Architecture | 2 | SA Agent | SDD approved, feasibility confirmed | ✅ SA agent with architecture authority |
| UI/UX Design | 3 | UI/UX Agent | Wireframes approved | ✅ UX agent generates wireframe JSON artifacts (22 wireframe files in sample project) |
| Planning & WBS | 4 | Tech Lead / PM | Epics, features, tasks created on board | ✅ TL/PM agents with create_card tool and board management |
| Development | 5 | Junior Developer | Code implemented per task DoD | ✅ JD agent writes real files to /workspaces/{projectId}/ via write_file tool |
| Code Review | 6 | Senior Developer | PR approved, quality confirmed | ✅ SD agent with code review authority; create_pr tool defined (stub — GitHub PR creation not yet active) |
| Testing | 7 | QA Engineer | All test scenarios pass | ✅ QA agent with run_tests/validate_code tools |
| Release | 8 | DevOps + Tech Lead | DoD met, rollback ready, deployed | 🔄 DO agent defined; trigger_deploy tool stub only — cloud pipeline not built |
| Post-Release Monitoring | 9 | SRE / Platform | Health checks pass, no critical incidents | 📋 SRE/PE agents defined; monitoring infrastructure not yet built |
| Resume / Iterate | 10 | Orchestrator | Next cycle begins from current state | ✅ PostgreSQL state enables resume from exact last state |

## 7.2 Core Rule

> **Chat does NOT drive progress. State + Decisions drive progress.**

The chat interface is for user interaction only. All progress is tracked through card state transitions and the decision ledger. If the system restarts, chat history is irrelevant — the PostgreSQL board state and decisions are the source of truth.

## 7.3 Quality Gates

Quality gates are enforced between **every** stage:

| Gate | Condition | Blocker If Failed | Status |
|------|-----------|-------------------|--------|
| BRD → SDD | BRD approved by BA + user | SA cannot begin | ✅ Enforced via document approval state |
| SDD → UI/UX | SDD approved by SA + user | UI/UX cannot begin | ✅ Enforced via document approval state |
| UI/UX → Planning | Wireframes approved by UI/UX + user | Planning cannot begin | ✅ Enforced via artifact approval |
| Planning → Dev | WBS complete, all cards created | Development cannot begin | ✅ Enforced via card state machine |
| Dev → Review | Task DoD met (code + tests + error handling) | PR not created | ✅ Card lifecycle validation in tool executor |
| Review → Testing | PR approved by SD | QA cannot begin | ✅ Under Review → Testing transition gated on SD authority |
| Testing → Release | All QA scenarios pass, no open defects | Release blocked | ✅ card-lifecycle.ts blocks invalid transitions |
| Release → Monitoring | DoD met, rollback ready, deployed | Not released | 🔄 DevOps pipeline not yet built; gate logic defined |
| **VS Code Gate** | VS Code extension connected + heartbeat passing | Dev agents blocked | ✅ Redis heartbeat gate (30s TTL) — JD/SD/QA/TL/DO/SEC blocked without extension |
