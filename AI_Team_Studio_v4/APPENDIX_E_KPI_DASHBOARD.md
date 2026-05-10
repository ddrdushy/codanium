# Appendix E — Enterprise KPI Dashboard Model

> **Implementation Status (2026-03-27):** KPI data sources are partially implemented. Card timestamps exist in PostgreSQL to calculate lead time and cycle time. Event records exist for deployment and QA events. The full KPI dashboard UI with live gauges and trend charts is planned for Phase 3. Basic Recharts analytics charts exist on the platform dashboard page.

## E.1 Delivery KPIs

| KPI | Definition | Target | Source |
|-----|-----------|--------|--------|
| **Lead Time** | Time from card creation to Done | < 5 days (task) | board.json timestamps |
| **Cycle Time** | Time from In Progress to Done | < 3 days (task) | board.json timestamps |
| **Deployment Frequency** | Releases per time period | Weekly+ | events.jsonl (ReleaseDeployed) |
| **Change Failure Rate** | % of releases causing incidents | < 5% | events.jsonl (IncidentRaised) |
| **MTTR** | Mean time to recover from incident | < 1 hour | events.jsonl (IncidentRaised → Resolved) |

## E.2 Quality KPIs

| KPI | Definition | Target | Source |
|-----|-----------|--------|--------|
| **Defect Rate** | Defects per feature | < 1 per feature | QA evidence |
| **Test Coverage** | % of code covered by tests | > 80% | CI pipeline reports |
| **QA Pass Rate** | % of QA scenarios passing on first run | > 90% | events.jsonl (QAResult) |
| **Decision Latency** | Time from decision requested to approved | < 24 hours | decisions.jsonl timestamps |
| **DoD Compliance** | % of cards passing DoD on first attempt | > 85% | audit.log |

## E.3 AI Cost KPIs

| KPI | Definition | Target | Source |
|-----|-----------|--------|--------|
| **Token Cost per Feature** | Total LLM tokens consumed per feature | Track trend | Cost Analytics |
| **Budget Utilization** | % of project budget consumed | < 90% | Budget Enforcer |
| **Model Efficiency** | Output quality vs. tokens consumed | Improving trend | Cost + QA Analytics |
| **Cost per Decision** | Average LLM cost per decision cycle | Track trend | Cost Analytics |

## E.4 Governance KPIs

| KPI | Definition | Target | Source |
|-----|-----------|--------|--------|
| **Audit Completeness** | % of changes with full audit trail | 100% | audit.log |
| **Authority Violations** | Attempted cross-boundary edits | 0 | State Controller logs |
| **Decision Compliance** | % of mandatory triggers that created decisions | 100% | decisions.jsonl |
| **Security Gate Pass Rate** | % passing security review on first attempt | > 90% | Security audit logs |

## E.5 Dashboard Layout

```
┌─────────────────────────────────────────────────────┐
│                 AI TEAM STUDIO KPI DASHBOARD         │
├──────────────┬──────────────┬───────────────────────┤
│  Lead Time   │  Cycle Time  │  Deploy Frequency     │
│   [gauge]    │   [gauge]    │      [counter]        │
├──────────────┴──────────────┴───────────────────────┤
│           Delivery Burndown Chart                    │
│           [line chart: cards over time]              │
├──────────────┬──────────────┬───────────────────────┤
│ Defect Rate  │  QA Pass %   │  Test Coverage %      │
│   [gauge]    │   [gauge]    │     [gauge]           │
├──────────────┴──────────────┴───────────────────────┤
│           AI Cost Breakdown                          │
│  [stacked bar: tokens by agent + model]             │
├──────────────┬──────────────┬───────────────────────┤
│ Budget Used  │ Decision Lat │  Authority Violations │
│   [gauge]    │   [gauge]    │     [counter: 0]      │
└──────────────┴──────────────┴───────────────────────┘
```
