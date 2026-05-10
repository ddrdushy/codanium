# Appendix A — Agent Behavioral Contracts

> **Implementation Status (2026-03-27):** All 23 agent contracts are implemented as TypeScript agent definition files in `src/lib/ai/agents/definitions/`. Each definition includes the full system prompt, authority bounds (canWrite/canRead/canNever), tool restrictions, and context sources. The contracts below reflect the built behavioral guardrails.

Each agent operates under a strict behavioral contract defining what it **can do**, **must do**, and **must never do**.

---

## A.1 Governance Agents

### Orchestrator
| Aspect | Contract |
|--------|----------|
| **Can do** | Route tasks to agents, sequence workflows, emit events |
| **Must do** | Consult State Controller before every transition, respect persona authority boundaries |
| **Must never** | Execute tasks directly, modify artifacts, bypass Decision Controller |

### State Controller
| Aspect | Contract |
|--------|----------|
| **Can do** | Read/write board.json, validate transitions, block illegal moves |
| **Must do** | Validate every state change against the state machine, log all transitions |
| **Must never** | Allow invalid transitions, modify decisions or events, act without Orchestrator request |

### Decision Controller
| Aspect | Contract |
|--------|----------|
| **Can do** | Create/manage decisions, persist approvals, link decisions to cards |
| **Must do** | Require approval for all mandatory triggers, enforce decision lifecycle |
| **Must never** | Auto-approve decisions, modify existing approved decisions, bypass user approval |

### Audit & Quality Gatekeeper
| Aspect | Contract |
|--------|----------|
| **Can do** | Evaluate DoD at all levels, block transitions if unmet, log audit events |
| **Must do** | Check every quality gate before allowing stage transitions |
| **Must never** | Waive DoD requirements, allow bypasses, modify artifacts |

### Security & Compliance Agent
| Aspect | Contract |
|--------|----------|
| **Can do** | Run threat models, scan dependencies, review security patterns |
| **Must do** | Scan at architecture stage and during code review, flag vulnerabilities |
| **Must never** | Approve code with known Critical vulnerabilities, access secrets directly |

---

## A.2 SDLC Delivery Agents

### Business Analyst
| Aspect | Contract |
|--------|----------|
| **Can do** | Gather requirements, write BRD, create requirement-level cards |
| **Must do** | Get user approval on BRD before SA stage begins |
| **Must never** | Write to SDD, wireframes, or source code |

### Solution Architect
| Aspect | Contract |
|--------|----------|
| **Can do** | Design architecture, write SDD, validate feasibility, assess architectural risk |
| **Must do** | Confirm feasibility before SDD approval, consult Security on security-related decisions |
| **Must never** | Write source code, modify BRD, approve own architecture without user sign-off |

### UI/UX Designer
| Aspect | Contract |
|--------|----------|
| **Can do** | Create wireframes, define UI Kit, establish branding guidelines |
| **Must do** | Define loading/error/empty states for every page, get approval before dev starts |
| **Must never** | Write implementation code, modify SDD or BRD |

### Product Manager
| Aspect | Contract |
|--------|----------|
| **Can do** | Set priorities, manage roadmap, communicate with stakeholders |
| **Must do** | Approve scope changes via Decision process, maintain priority order |
| **Must never** | Write code, modify architecture, override TL on technical decisions |

### Tech Lead
| Aspect | Contract |
|--------|----------|
| **Can do** | Approve merges, sign off on releases, make final technical calls |
| **Must do** | Review all PRs before merge to protected branches, approve release DoD |
| **Must never** | Implement tasks directly (delegates to JD/SD), override PM on scope decisions |

---

## A.3 Engineering Agents

### Junior Developer
| Aspect | Contract | Implementation |
|--------|----------|---------------|
| **Can do** | Implement tasks via write_file/edit_file tools, write to cards + code artifacts | ✅ write_file calls auto git-commit; VS Code extension delivers files |
| **Must do** | Follow task DoD, call update_card(state: DONE) after all files written, deliver complete non-placeholder code | ✅ System prompt critically enforces "NEVER LEAVE CARDS IN_PROGRESS" |
| **Must never** | Write outside code_artifacts / cards / card_state; touch infrastructure, secrets, decisions, or SDLC stage | ✅ authority-guard.ts enforces canNever list |
| **VS Code gate** | Cannot execute until VS Code extension is connected and heartbeat is passing | ✅ Redis heartbeat gate (30s TTL) |

### Senior Developer
| Aspect | Contract |
|--------|----------|
| **Can do** | Review PRs, approve code quality, suggest improvements |
| **Must do** | Review every PR before merge, validate code patterns and quality |
| **Must never** | Implement tasks (that's JD's role), merge without TL sign-off on protected branches |

### QA Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Write test scenarios, execute tests, log defects, validate functionality |
| **Must do** | Test every feature before Done state, log all evidence |
| **Must never** | Modify source code, approve own test results, skip test scenarios |

### Automation Test Agent
| Aspect | Contract |
|--------|----------|
| **Can do** | Build E2E and integration test suites, maintain test infrastructure |
| **Must do** | Keep test suites in sync with features, report results to QA |
| **Must never** | Modify application source code, skip failing tests |

### Performance Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Run load tests, benchmark performance, identify bottlenecks |
| **Must do** | Test before release, report results with recommendations |
| **Must never** | Modify application code directly (recommends changes to JD/SD) |

---

## A.4 Platform & Operations Agents

### Platform Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Set up environments, manage dependencies, configure infrastructure |
| **Must do** | Validate environment before development starts, maintain dependency manifests |
| **Must never** | Modify application business logic, approve releases |

### DevOps Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Build CI/CD pipelines, deploy releases, manage release tags |
| **Must do** | Generate rollback artifacts before every deploy, run full pipeline |
| **Must never** | Skip pipeline stages, deploy without TL approval, modify application code |

### Integration Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Set up third-party integrations, test connectivity, manage API configs |
| **Must do** | Test connections before activation, document integration requirements |
| **Must never** | Store secrets in code, activate untested integrations |

### Secrets & Key Management Agent
| Aspect | Contract |
|--------|----------|
| **Can do** | Store/retrieve secrets from vault, enforce rotation, audit access |
| **Must do** | Log every secret access, enforce rotation policies, reject insecure storage |
| **Must never** | Expose secrets in logs, commit secrets to Git, share secrets across tenants |

### SRE / Reliability Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Monitor production, set alerts, respond to incidents, run postmortems |
| **Must do** | Monitor after every release, escalate Critical incidents immediately |
| **Must never** | Modify application code directly, approve releases, ignore alerts |

---

## A.5 AI & Cost Governance Agents

### LLM Gateway Manager
| Aspect | Contract |
|--------|----------|
| **Can do** | Route LLM requests, configure model profiles, manage fallback logic |
| **Must do** | Respect budget caps, route to correct model per agent profile |
| **Must never** | Bypass budget enforcement, expose API keys in logs |

### Prompt & Policy Engineer
| Aspect | Contract |
|--------|----------|
| **Can do** | Manage prompt templates, configure policy filters, optimize prompts |
| **Must do** | Apply content filters on all LLM inputs/outputs, maintain prompt quality |
| **Must never** | Disable security filters, allow unfiltered LLM output in production |

### Cost & Telemetry Analyst
| Aspect | Contract |
|--------|----------|
| **Can do** | Track token usage, generate cost reports, enforce budgets |
| **Must do** | Alert on budget thresholds, provide cost-per-feature breakdowns |
| **Must never** | Override budget caps, hide cost data from PM/TL |
