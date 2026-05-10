# 29. Event Bus & Orchestration Design

> **Implementation Status (2026-03-27):** The event system is fully implemented in PostgreSQL. The `Event` and `AuditLog` tables store all system events. The event-bus module (`event-bus.ts`) and event-handlers module (`event-handlers.ts`) are built in the orchestration layer. Deterministic replay from event history is planned for Phase 3.

## 29.1 Event Types

| Event | Emitted By | Consumed By | Status |
|-------|-----------|-------------|--------|
| `ArtifactCreated` | Any agent | Orchestrator, VS Code extension stream | ✅ Built |
| `StateChanged` | card-lifecycle.ts | Orchestrator, Event table | ✅ Built |
| `DecisionRequested` | Any agent (create_decision tool) | Decision Controller | ✅ Built |
| `DecisionApproved` | Decision Controller | Orchestrator | ✅ Built |
| `TaskAssigned` | Orchestrator | Target agent | ✅ Built |
| `TaskCompleted` | Executing agent | QA Gatekeeper, Orchestrator | ✅ Built |
| `PRCreated` | JD | Observability | 📋 Planned — GitHub integration not yet active |
| `PRApproved` | SD | Orchestrator | 📋 Planned |
| `ReleaseDeployed` | DevOps | Observability, SRE | 📋 Planned — deployment pipeline not yet built |
| `QAResult` | QA / Auto Test | Orchestrator, Observability | ✅ Built |
| `IncidentRaised` | SRE | SRE, Observability | 📋 Planned |

## 29.2 Event Schema (PostgreSQL Event Model)

The actual PostgreSQL `Event` model:

```typescript
model Event {
  id         String    @id @default(cuid())
  type       String                          // Event type string
  actor      String                          // Agent short name or user ID
  payload    Json                            // Event-specific payload
  projectId  String
  createdAt  DateTime  @default(now())
  project    Project   @relation(...)
}
```

Example payload shapes match the v4.0 design:

```json
{
  "type": "TaskCompleted",
  "actor": "JD",
  "payload": {
    "card_id": "TASK-042",
    "evidence": "files-written",
    "files": ["src/components/NavBar.tsx", "src/app/about/page.tsx"]
  },
  "projectId": "cm...",
  "createdAt": "2026-03-27T14:30:00Z"
}
```

## 29.3 Event Processing Rules

- Events are **persisted** to PostgreSQL Event table
- Events are processed **in order** by the Orchestrator
- Event-handlers module routes events to appropriate handlers
- Dead-letter / failed events logged via AuditLog table
- Deterministic replay from event history is **planned Phase 3**

---

# 30. Deterministic Replay Model

> **Implementation Status (2026-03-27):** The infrastructure for deterministic replay exists — all events are stored in PostgreSQL with timestamps. The replay engine itself (iterating events chronologically to reconstruct state) is not yet implemented. This is a Phase 3 target.

## 30.1 Concept

Given the same event history in PostgreSQL:
- `Event` table (complete event history)
- `Decision` table (complete decision history)
- `Card` table (initial state snapshots)

The system **must** be able to reconstruct the exact same project state at any point in time.

## 30.2 Replay Process (Planned)

1. Load initial project state from PostgreSQL
2. Replay Event table in chronological order
3. Apply Decision records at their recorded timestamps
4. Validate that reconstructed board matches expected state
5. Report any divergence as a consistency error

**Status: Planned Phase 3** — event storage infrastructure is built; replay execution is not yet implemented.

## 30.3 Use Cases

| Use Case | Description | Status |
|----------|-------------|--------|
| **Audit review** | Regulators can replay project history | 📋 Planned (events stored now) |
| **Debugging** | Reproduce exact state when a bug was introduced | 📋 Planned |
| **Disaster recovery** | Reconstruct state from backed-up event logs | 📋 Planned |
| **Training** | Replay exemplary projects as templates | 📋 Planned |

---

# 31. Data Models & JSON Schemas

> **Implementation Status (2026-03-27):** All data models are implemented as PostgreSQL tables via Prisma schema (`prisma/schema.prisma`). The schema has 24+ models. Key models are documented below.

## 31.1 PostgreSQL Card Model

```typescript
model Card {
  id             String      @id @default(cuid())
  title          String
  description    String?
  type           CardType    // EPIC | FEATURE | TASK | BUG
  state          CardState   // PLANNED | IN_PROGRESS | UNDER_REVIEW | TESTING | BLOCKED | DONE | RELEASED
  priority       Priority    @default(MEDIUM)
  module         String?
  parentId       String?
  requirementIds String[]    // BRD requirement traceability
  assigneeId     String?
  projectId      String
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt
}
```

## 31.2 PostgreSQL Decision Model

```typescript
model Decision {
  id             String         @id @default(cuid())
  title          String
  description    String?
  options        Json           // Array of {label, pros, cons}
  recommendation String?
  status         DecisionStatus @default(OPEN)  // OPEN | APPROVED | REJECTED | DEFERRED
  selectedOption String?
  ownerId        String?
  projectId      String
  createdAt      DateTime       @default(now())
  updatedAt      DateTime       @updatedAt
}
```

## 31.3 PostgreSQL Artifact Model

```typescript
model Artifact {
  id         String       @id @default(cuid())
  type       ArtifactType // BRD | SDD | API_SPEC | DESIGN_SYSTEM | TEST_PLAN | DEPLOYMENT_GUIDE | MEMORY | OTHER
  title      String
  content    String       @db.Text
  version    Int          @default(1)
  approved   Boolean      @default(false)
  projectId  String
  createdAt  DateTime     @default(now())
  updatedAt  DateTime     @updatedAt
}
```

## 31.4 PostgreSQL Event Model

```typescript
model Event {
  id        String   @id @default(cuid())
  type      String
  actor     String
  payload   Json
  projectId String
  createdAt DateTime @default(now())
}
```

## 31.5 Wireframe Schema

See `08_UIUX_AND_WIREFRAMES.md` Section 18.2 for the full wireframe JSON schema. Wireframes are stored as `Artifact` records with `type: OTHER` and structured JSON content.
