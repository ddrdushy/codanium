# 34. Architecture Diagrams

> **Implementation Status (2026-03-27):** Diagrams updated to reflect actual built architecture. The system is a Next.js 16 monolith with PostgreSQL, not a split FastAPI/Node.js architecture. VS Code extension handles file delivery. Multi-tenant SaaS and distributed agent modes are planned.

All diagrams below are in Mermaid format. Render them in any Mermaid-compatible viewer (GitHub, Mermaid Live Editor, MkDocs, Docusaurus).

**Diagram conventions:**
- Solid arrows: primary control flow
- Dotted arrows: read-only / reference
- Orange nodes: governance/control
- Blue nodes: delivery personas
- Green nodes: platform/ops

---

## 34.1 System Context Diagram (Actual — 2026-03-27)

```mermaid
flowchart LR
  U[User / Product Owner] -->|Requests, Approvals| UI[AI Team Studio\nNext.js 16 Web App]
  UI -->|State/Decision Actions| CP[Orchestration Engine]
  CP -->|Tool calls| WS[Workspace Controller\n/workspaces/projectId/]
  WS --> Repo[(Local Git\nauto-commit on write)]
  CP -->|Calls| LLM[LLM Gateway\nOpenAI / Anthropic / Ollama\nRaw fetch, admin-configured]
  WS --> PG[(PostgreSQL\n24+ models\nAES-256-GCM keys)]
  CP --> PG
  SSE[SSE Artifact Stream\n/api/projects/id/artifacts/stream] --> VSC[VS Code Extension\nv0.1.0]
  VSC -->|write files| DEV[Developer\nLocal Workspace]
  VSC -->|heartbeat\n/api/projects/id/vscode-ping| PG
  PG --> R[(Redis\nHeartbeat TTL 30s)]
```

---

## 34.2 High-Level Component Architecture (Actual)

```mermaid
flowchart TB
  subgraph FE[Frontend — Next.js 16 App Router]
    UI[12 Platform Pages]
    CHAT[SSE Streaming Chat]
    BOARD[Drag-Drop Kanban]
    CMD[Command Palette cmdk]
  end

  subgraph API[API Routes]
    STREAM[/api/projects/id/chat/stream SSE]
    ASTREAM[/api/projects/id/artifacts/stream SSE]
    PING[/api/projects/id/vscode-ping]
    REST[REST: projects/cards/decisions/agents/git/admin]
  end

  subgraph ORCH[Orchestration Engine]
    MR[MessageRouter\nclassify intent → pick agent]
    CB[ContextBuilder\nfetch project data → system prompt]
    AE[AgentExecutor\ncall LLM via Gateway]
    LLM[LLM Gateway\nresolve admin config → provider]
    CL[card-lifecycle.ts\ntransition validation]
    AG[authority-guard.ts\nwrite permission check]
    LD[loop-detector.ts\nfuzzy + repetition + reask]
    QG[quality-gates.ts\nDoD enforcement]
  end

  subgraph AGENTS[23 Agent Definitions]
    GOVC[Governance: ORC/SC/DEC/AUD/SEC]
    SDLC[SDLC: BA/SA/UX/PM/TL]
    ENG[Engineering: JD/SD/QA/AT/PF]
    PLAT[Platform: PE/DO/IE/SM/SR]
    AI[AI/Cost: CA/PE/TA]
  end

  subgraph TOOLS[Tool System]
    FS[write_file / edit_file / read_file\nlist_directory / glob / grep]
    GIT[git_commit / git_branch / git_diff]
    SHELL[run_command / run_tests / run_build]
    PROJ[create_card / update_card / create_document\ncreate_decision / remember]
    COMM[consult_agent / ask_user]
  end

  subgraph STORAGE[Storage]
    PG[(PostgreSQL port 14000\nPrisma 7.x — 24+ models)]
    DISK[/workspaces/projectId/\nAgent-generated source files]
    REDIS[(Redis\nHeartbeat + BullMQ)]
  end

  subgraph VSCODE[VS Code Extension v0.1.0]
    WM[WorkspaceManager\nSSE consumer + file writer]
    HB[Heartbeat\nPOST every 25s → Redis 30s TTL]
    VIEWS[TreeView: Projects/Agents/Chat]
  end

  UI --> API
  CHAT --> STREAM
  BOARD --> REST
  CMD --> REST
  STREAM --> MR
  MR --> CB
  CB --> AE
  AE --> LLM
  LLM --> AGENTS
  AGENTS --> TOOLS
  TOOLS --> FS
  TOOLS --> GIT
  FS --> DISK
  GIT --> DISK
  AE --> CL
  AE --> AG
  AE --> LD
  AE --> QG
  ORCH --> PG
  ASTREAM --> VSCODE
  WM --> HB
  HB --> REDIS
  PING --> REDIS
```

---

## 34.3 Orchestration Sequence (Actual)

```mermaid
sequenceDiagram
  autonumber
  participant User
  participant UI as UI (Next.js)
  participant API as /api/chat/stream (SSE)
  participant MR as MessageRouter
  participant CB as ContextBuilder
  participant AE as AgentExecutor
  participant LLM as LLM Gateway
  participant TE as Tool Executor
  participant PG as PostgreSQL
  participant WS as /workspaces/projectId/
  participant VSC as VS Code Extension

  User->>UI: Send chat message
  UI->>API: POST with projectId + message
  API->>MR: Classify intent → select agent
  MR->>CB: Build context (project state, cards, docs)
  CB->>PG: Fetch cards, decisions, artifacts
  CB-->>AE: System prompt + context
  AE->>LLM: Call LLM (OpenAI/Anthropic/Ollama)
  LLM-->>AE: Streaming response with tool calls

  loop Tool execution
    AE->>TE: Execute tool call (e.g., write_file)
    TE->>WS: Write file to disk
    TE->>PG: Create Artifact record
    TE->>PG: Auto git_commit
    TE-->>AE: Tool result
    AE-->>API: SSE: artifact event
    API-->>VSC: SSE artifact stream
    VSC->>WS: Write file in linked workspace
  end

  AE-->>API: SSE: token stream
  API-->>UI: Stream tokens to chat
  AE->>PG: Save OrchestrationRun + Events
```

---

## 34.4 Artifacts & Data Model (ER Diagram — Actual PostgreSQL)

```mermaid
erDiagram
  PROJECT ||--o{ CARD : contains
  PROJECT ||--o{ DECISION : has
  PROJECT ||--o{ EVENT : logs
  PROJECT ||--o{ ARTIFACT : stores
  PROJECT ||--o{ ORCHESTRATION_RUN : tracks
  USER ||--o{ PROJECT : owns
  USER ||--o{ CARD : assigned_to
  USER ||--o{ DECISION : owns
  CARD ||--o{ CARD : parent_child
  ORGANIZATION ||--o{ ORG_MEMBER : has
  USER ||--o{ ORG_MEMBER : member_of

  PROJECT {
    string id
    string name
    string description
    string sdlcStage
    string ownerId
    datetime createdAt
  }
  CARD {
    string id
    string type "EPIC|FEATURE|TASK|BUG"
    string state "PLANNED|IN_PROGRESS|UNDER_REVIEW|TESTING|BLOCKED|DONE|RELEASED"
    string priority "CRITICAL|HIGH|MEDIUM|LOW"
    string module
    string parentId
    array requirementIds
    string assigneeId
    string projectId
    datetime updatedAt
  }
  DECISION {
    string id
    string title
    string description
    json options
    string recommendation
    string status "OPEN|APPROVED|REJECTED|DEFERRED"
    string selectedOption
    string ownerId
    string projectId
  }
  ARTIFACT {
    string id
    string type "BRD|SDD|API_SPEC|DESIGN_SYSTEM|TEST_PLAN|DEPLOYMENT_GUIDE|MEMORY|OTHER"
    string title
    text content
    int version
    boolean approved
    string projectId
  }
  EVENT {
    string id
    string type
    string actor
    json payload
    string projectId
    datetime createdAt
  }
  ORCHESTRATION_RUN {
    string id
    string agentId
    string status
    json inputTokens
    json outputTokens
    string projectId
    datetime createdAt
  }
```

---

## 34.5 Card State Machine (Actual Implementation)

```mermaid
stateDiagram-v2
  [*] --> PLANNED
  PLANNED --> IN_PROGRESS: start work
  IN_PROGRESS --> UNDER_REVIEW: submit for review\n(also accepts REVIEW alias)
  UNDER_REVIEW --> IN_PROGRESS: changes requested
  UNDER_REVIEW --> TESTING: approved
  TESTING --> BLOCKED: defect / decision required
  BLOCKED --> IN_PROGRESS: decision approved / fix started
  TESTING --> DONE: all scenarios pass
  DONE --> RELEASED: release approved + deployed

  note right of BLOCKED
    Blocked if:
    - Pending Decision
    - Failed DoD gate
    - Missing artifacts
    - VS Code not connected (dev agents)
  end note

  note right of UNDER_REVIEW
    Enforced by card-lifecycle.ts
    Illegal transitions return 400
  end note
```

---

## 34.6 Decision Lifecycle (Actual)

```mermaid
stateDiagram-v2
  [*] --> OPEN: agent calls create_decision tool
  OPEN --> APPROVED: user selects option in Decisions UI
  OPEN --> REJECTED: user rejects
  OPEN --> DEFERRED: user defers
  APPROVED --> [*]: orchestration continues
  REJECTED --> [*]: agent revises approach
  DEFERRED --> OPEN: revisit later

  note right of OPEN
    "Your AI Team Recommends" banner
    shows recommendation prominently.
    User can choose recommended or
    alternative option.
  end note
```

---

## 34.7 VS Code Extension Integration Flow

```mermaid
flowchart LR
  subgraph WEBAPP[Web App — Next.js 16]
    CHAT[Agent generates code\nvia write_file tool]
    ARTIFACT[Artifact saved to PostgreSQL\n+ written to /workspaces/projectId/]
    SSE[SSE: /api/projects/id/artifacts/stream\nartifact event emitted]
    PING[/api/projects/id/vscode-ping\nRedis TTL 30s heartbeat]
  end

  subgraph VSCODE[VS Code Extension v0.1.0]
    WM[WorkspaceManager\nlistens to SSE stream]
    HB[Heartbeat\nPOST every 25s]
    FW[File Writer\nwrites to linked folder]
    FT[FileTracker\nprevents duplicates]
    OPEN[Opens new files\nin editor]
  end

  subgraph DEVAGENT[Dev Agents Gate]
    BLOCK{VS Code\nConnected?}
    YES[JD / SD / QA / TL / DO / SEC\ncan execute]
    NO[Dev agents\nblocked]
  end

  CHAT --> ARTIFACT
  ARTIFACT --> SSE
  SSE --> WM
  WM --> FW
  FW --> FT
  FT --> OPEN
  HB --> PING
  PING --> BLOCK
  BLOCK -->|Yes| YES
  BLOCK -->|No| NO
```

---

## 34.8 LLM Gateway (Actual)

```mermaid
flowchart TB
  AE[AgentExecutor] --> GW[LLM Gateway\nresolve provider config]
  GW --> PRIO{Resolution Priority}
  PRIO -->|1st| AO[Agent-level override]
  PRIO -->|2nd| PO[Project-level override]
  PRIO -->|3rd| AD[Admin platform default\nLLMProviderConfig table]
  AD --> ENC[AES-256-GCM decrypt API key]
  ENC --> PROV{Provider}
  PROV -->|OpenAI| OAI[openai-adapter.ts\nRaw fetch — no SDK]
  PROV -->|Anthropic| ANT[anthropic-adapter.ts\nRaw fetch — no SDK]
  PROV -->|Ollama| OLL[ollama-adapter.ts\nRaw fetch — no SDK]
  OAI --> SSE_OUT[SSE token stream\nback to client]
  ANT --> SSE_OUT
  OLL --> SSE_OUT
```

---

## 34.9 Deployment Topology (Current + Planned)

```mermaid
flowchart LR
  subgraph CURRENT[Current — Dev Mode]
    UI1[Browser\nlocalhost:3000] --> NX1[Next.js 16\nTurbopack]
    NX1 --> PG1[(PostgreSQL\nDocker port 14000)]
    NX1 --> R1[(Redis\nDocker)]
    NX1 --> WS1[/workspaces/\nLocal disk]
    WS1 --> GIT1[(Local Git\nauto-commit)]
    NX1 -->|Raw fetch| LLM1[OpenAI/Anthropic/Ollama]
    VSC1[VS Code Extension\nv0.1.0] -->|SSE + Heartbeat| NX1
    VSC1 --> WS1
  end

  subgraph PLANNED[Phase 3 — SaaS]
    UI2[Browser] --> CDN2[CDN / Load Balancer]
    CDN2 --> NX2[Next.js\nCloud instances]
    NX2 --> PG2[(PostgreSQL\nManaged cloud DB)]
    NX2 --> R2[(Redis\nManaged)]
    NX2 --> S3[Cloud Storage\nWorkspaces]
    NX2 -->|BYOM| LLM2[LLM Gateway\nOpenAI/Anthropic/Ollama/Custom]
    NX2 --> BQ2[BullMQ Workers\nParallel agents]
    VSC2[VS Code Extension] -->|SSE + Heartbeat| NX2
  end
```

---

## 34.10 Security & Secrets Flow (Actual)

```mermaid
flowchart TB
  ADMIN[Admin Settings Page] -->|Enter LLM API key| ENC[AES-256-GCM encrypt\nencryption.ts]
  ENC -->|Store encrypted| PG[(PostgreSQL\nLLMProviderConfig table)]
  PG -->|Decrypt at runtime| GW[LLM Gateway]
  GW -->|Auth headers| LLM[OpenAI / Anthropic / Ollama]
  GW -->|Never stores in| GIT[(Git / Workspace)]

  subgraph PLANNED[Phase 3 — Extended Secret Management]
    VAULT[OS Keychain / HashiCorp Vault]
    AWSSM[AWS Secrets Manager]
    AKV[Azure Key Vault]
  end

  subgraph ACTIVE[Active Controls]
    R1[AES-256-GCM for LLM keys at rest]
    R2[Path traversal blocked in workspace]
    R3[Dangerous commands blocked]
    R4[No secrets in git commits]
    R5[Authority guard prevents cross-agent writes]
  end
```

---

## 34.11 Observability & Cost Governance (Target State)

```mermaid
flowchart LR
  CP[Orchestration Engine] --> EVTS[Event table\nPostgreSQL]
  CP --> ORC[OrchestrationRun table\ntoken counts]
  CP --> AUDIT[AuditLog table]
  LLM[LLM Gateway] --> TOK[Token Usage\nper request]
  TOK --> ORC
  EVTS --> REPLAY[Replay Engine\nPlanned Phase 3]
  ORC --> COST[Cost Dashboard\nPlanned Phase 3]
  AUDIT --> COMP[Compliance Reports\nPlanned Phase 3]
  COST --> BUD[Budget Enforcer\nPlanned Phase 3]
  BUD -->|Throttle/deny| LLM
  COST --> DASH[Exec Dashboards\nPlanned Phase 3]
```
