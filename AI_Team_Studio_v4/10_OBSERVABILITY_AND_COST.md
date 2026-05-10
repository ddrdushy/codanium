# 22. Observability & Reliability Engineering

> **Implementation Status (2026-03-27):** Basic observability infrastructure exists. The PostgreSQL `Event`, `AuditLog`, and `OrchestrationRun` tables provide an audit trail. The telemetry module (`telemetry.ts`) is present in the orchestration layer. Full observability dashboards, alerting stacks, SLO tracking, and SRE automation are Phase 3 targets.

## 22.1 Observability Stack

| Signal | Source | Consumer | Status |
|--------|--------|----------|--------|
| **Audit Events** | All agents via Event table | PostgreSQL audit trail | ✅ Built |
| **OrchestrationRun records** | Orchestration engine | PostgreSQL OrchestrationRun table | ✅ Built |
| **Card state changes** | card-lifecycle.ts | Event table | ✅ Built |
| **Agent SSE stream** | Chat API stream endpoint | Frontend `useAgentStream` hook | ✅ Built |
| **VS Code artifact stream** | `/api/projects/[id]/artifacts/stream` | VS Code extension | ✅ Built |
| **Structured Logs** | All agents, Orchestrator | Application logs | 🔄 Partial — console logging; structured log pipeline planned |
| **Metrics dashboards** | State Controller, LLM Gateway | Recharts UI components | 🔄 Partial — basic charts built; advanced dashboards planned |
| **Distributed traces** | Orchestrator, Agent Runtime | Performance analysis | 📋 Planned Phase 3 |
| **Token usage events** | LLM Gateway | Cost Analytics | 📋 Schema defined; dashboard planned |

## 22.2 Monitoring & Alerting

| Monitor | Threshold | Alert Target | Status |
|---------|-----------|-------------|--------|
| Agent task completion time | > P95 threshold | SRE | 📋 Planned |
| Failed state transitions | Any | State Controller + TL | ✅ Returns 400 error; planned: alerting |
| LLM API errors | > 5% error rate | LLM Gateway Manager | 🔄 Errors logged; alerting planned |
| Budget utilization | > 80% of cap | Cost Analyst + PM | 📋 Planned |
| Build/test failures | Any | DevOps + JD | 📋 Planned with CI/CD pipeline |
| Post-release error rate | > baseline | SRE + TL | 📋 Planned |
| VS Code heartbeat loss | > 30s gap | Dev agents blocked | ✅ Redis TTL heartbeat gate active |

## 22.3 Reliability Practices

| Practice | Status |
|----------|--------|
| Error budgets per project | 📋 Planned |
| Incident logging tied to decision ledger | 📋 Planned |
| Automated alerts routed to responsible agent | 📋 Planned |
| Postmortem workflow on Critical incidents | 📋 Planned |
| SLO tracking | 📋 Planned |
| Loop detection (prevents runaway agents) | ✅ loop-detector.ts — fuzzy tool-loop + text-repetition + question-reask |

---

# 23. Cost Governance & AI Budget Controls

> **Implementation Status (2026-03-27):** The telemetry schema is defined and the `telemetry.ts` module exists in the orchestration layer. Token usage is tracked per request in the OrchestrationRun table. Full budget enforcement, per-agent caps, and executive cost dashboards are Phase 3 targets.

## 23.1 Token Accounting

| Metric | Granularity | Purpose | Status |
|--------|------------|---------|--------|
| Tokens consumed | Per-request | Raw usage tracking | ✅ OrchestrationRun table records token counts |
| Tokens per agent | Per-agent | Identify costly agents | 📋 Planned aggregation |
| Tokens per feature | Per-feature card | ROI per feature | 📋 Planned |
| Tokens per project | Per-project | Project budget tracking | 📋 Planned |

## 23.2 Budget Controls

| Control | Behavior | Status |
|---------|----------|--------|
| **Project budget cap** | Hard limit — blocks LLM calls when reached | 📋 Planned |
| **Agent budget cap** | Per-agent limit — throttles expensive agents | 📋 Planned |
| **Budget alarms** | Alert at 50%, 80%, 95% utilization | 📋 Planned |
| **Model tiering** | Use cheaper models for simple tasks | 📋 Planned |
| **ROI reporting** | Cost per feature delivered | 📋 Planned |
| **Admin LLM config** | Admin controls platform-wide model selection | ✅ Built — admin settings page |

## 23.3 Executive Dashboards

- Real-time token usage by project, agent, and model — 📋 Planned
- Cost trends and forecasts — 📋 Planned
- Budget utilization gauges — 📋 Planned
- Feature delivery cost breakdown — 📋 Planned
- Model performance vs. cost comparison — 📋 Planned
- Basic Recharts analytics charts on platform dashboard — ✅ Built (delivery metrics)
