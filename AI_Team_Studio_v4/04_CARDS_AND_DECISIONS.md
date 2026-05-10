# 8. Card & Workflow Architecture

> **Implementation Status (2026-03-27):** Cards and the board are fully implemented in PostgreSQL. The card lifecycle state machine is enforced in `card-lifecycle.ts` at both the API route level and the tool executor level. The drag-and-drop kanban board with milestones view is built and wired to the real database.

## 8.1 Card Types

| Card Type | Visibility | Purpose | Built? |
|-----------|-----------|---------|--------|
| **Epic** | User-facing | Represents a module or major feature area | ✅ PostgreSQL Card model with type=EPIC |
| **Feature** | User-facing | A user-visible capability within an Epic | ✅ type=FEATURE with parent/child relations |
| **Task** | Hidden | An implementation unit assigned to a developer agent | ✅ type=TASK with assignee, requirementIds |
| **Bug** | User-facing | A defect requiring a fix | ✅ type=BUG supported in Card schema |
| **QA Validation** | Hidden (summary visible) | Test evidence and pass/fail results | ✅ QA agent creates validation cards |
| **Decision Blocker** | User-facing | A pending decision that blocks progress | ✅ Decision model linked to cards |

## 8.2 Card States

| State | Meaning | Built? |
|-------|---------|--------|
| **PLANNED** | Card created, work not started | ✅ |
| **IN_PROGRESS** | Active work underway by assigned agent | ✅ |
| **UNDER_REVIEW** | Submitted for review (code review, design review, etc.) | ✅ (also accepts REVIEW alias) |
| **TESTING** | QA validation in progress | ✅ |
| **BLOCKED** | Cannot proceed — pending decision, failed gate, or missing artifact | ✅ |
| **DONE** | All acceptance criteria met, DoD passed | ✅ |
| **RELEASED** | Deployed to production, release tagged | ✅ (state exists; deployment pipeline planned) |

## 8.3 State Transitions

```
PLANNED → IN_PROGRESS         (start work)
IN_PROGRESS → UNDER_REVIEW    (submit for review)
IN_PROGRESS → REVIEW          (alias — also accepted)
UNDER_REVIEW → IN_PROGRESS    (changes requested)
UNDER_REVIEW → TESTING        (approved)
TESTING → BLOCKED             (defect found / decision required)
BLOCKED → IN_PROGRESS         (decision approved / fix started)
TESTING → DONE                (all scenarios pass)
DONE → RELEASED               (release approved + deployed)
```

Illegal transitions are blocked by `card-lifecycle.ts`. Attempted violations return a 400 error with the reason.

## 8.4 Card Fields (Actual Schema)

The PostgreSQL `Card` model includes:
- `id`, `title`, `description`, `type` (EPIC/FEATURE/TASK/BUG), `state`
- `priority` (CRITICAL/HIGH/MEDIUM/LOW)
- `module` — feature area label
- `parentId` — parent card for hierarchy
- `requirementIds` — array of BRD requirement IDs for traceability
- `assigneeId` — linked to user (or agent short name)
- `projectId` — project scope
- `createdAt`, `updatedAt`

## 8.5 Why Cards Exist

Cards are the **system memory**. They answer:

- **Where are we?** — Current state of every work item
- **What is next?** — Next card in the workflow
- **What is blocked?** — Pending decisions or failed gates
- **What can resume?** — After reopening, exactly where to continue

---

# 9. Decision Lifecycle Framework

> **Implementation Status (2026-03-27):** The Decision model is fully implemented in PostgreSQL. The `create_decision` tool lets agents record decisions with options and recommendations. The Decisions UI shows "Your AI Team Recommends" banner with prominent approve/reject CTAs. Full decision lifecycle state machine is implemented in the Decision model.

## 9.1 Purpose

Every non-trivial change must be recorded as a **formal decision**. This ensures:

- Pause/resume without context loss
- Full audit trail for compliance
- Accountability — every decision has an owner
- Deterministic orchestration — decisions drive state changes

## 9.2 Decision Record Fields

| Field | Type | Description | Built? |
|-------|------|-------------|--------|
| `id` | string | Unique identifier (cuid) | ✅ |
| `title` | string | Decision title | ✅ |
| `description` | string | What needs to be decided | ✅ |
| `options` | JSON | List of possible approaches with pros/cons | ✅ |
| `recommendation` | string | Agent's recommended option with rationale | ✅ |
| `status` | enum | `OPEN` / `APPROVED` / `REJECTED` / `DEFERRED` | ✅ |
| `selectedOption` | string | The option selected by the approver | ✅ |
| `ownerId` | string | User who owns/approved this decision | ✅ |
| `projectId` | string | Project scope | ✅ |
| `createdAt`, `updatedAt` | datetime | Timestamps | ✅ |

## 9.3 Decision Lifecycle States

```
[New] → OPEN               (decision triggered by agent via create_decision tool)
OPEN → APPROVED            (user selects recommended or alternative option)
OPEN → REJECTED            (user rejects, needs revision)
OPEN → DEFERRED            (user defers to later)
```

## 9.4 Mandatory Decision Triggers

A formal decision is **required** before:

- Architectural changes (new service, pattern change)
- Security-related changes (auth flow, data exposure)
- Dependency additions (new packages, libraries)
- Model configuration changes (LLM provider, budget limits)
- Scope changes (feature additions, requirement modifications)
- Technology changes (framework, language, tool switches)
