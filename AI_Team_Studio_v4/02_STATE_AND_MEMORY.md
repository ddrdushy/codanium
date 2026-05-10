# 5. Memory & State Architecture

> **Implementation Status (2026-03-27):** State persistence is fully implemented using PostgreSQL (via Prisma), not flat JSON files. The file-based architecture described in the original v4.0 design has been implemented as a relational database model. The project workspace on disk (`/workspaces/{projectId}/`) contains generated code files. VS Code extension streams artifacts in real time.

## 5.1 Design Philosophy

Agents are stateless. All project knowledge lives in structured, version-controlled storage. This enables:

- **Resume-anytime**: Reopen the project and continue from exactly where you left off
- **Deterministic replay**: Reconstruct any historical state from event logs (events stored; replay engine planned)
- **Audit compliance**: Every change is traceable to a decision and an owner
- **Multi-session safety**: Multiple agents or sessions cannot corrupt shared state

## 5.2 Memory Layers

### Actual Implementation (PostgreSQL + Disk)

| Layer | Storage | Purpose | Status |
|-------|---------|---------|--------|
| Canonical Documents | PostgreSQL `Artifact` table (type: BRD, SDD, etc.) | Source of truth for requirements, architecture, UI design | ✅ Built |
| Card Board State | PostgreSQL `Card` table + enums | Current card states, assignments, parent/child links | ✅ Built |
| Decision Ledger | PostgreSQL `Decision` table | Non-trivial decisions with options and recommendation | ✅ Built |
| Event Log | PostgreSQL `Event` table | Chronological record of all system events | ✅ Built (events stored; replay not yet implemented) |
| Generated Code Files | Disk: `/workspaces/{projectId}/` | Agent-written source files for user projects | ✅ Built |
| Git History | Local Git in workspace dir | Source code version control; auto-commit on every file write | ✅ Built |
| LLM Provider Config | PostgreSQL `LLMProviderConfig` table | Admin-managed provider/model/key with AES-256-GCM encryption | ✅ Built |
| Project Memory (KV) | PostgreSQL `Artifact` (type: MEMORY) | Key/value memory persisted across agent sessions | ✅ Built |

### Original File-Based Design (Reference Only)

The v4.0 architecture described flat file storage (board.json, decisions.jsonl, events.jsonl). The actual implementation uses PostgreSQL for all structured state, with disk storage only for generated project code files. The relational model provides stronger querying, concurrent access safety, and foreign-key integrity.

## 5.3 Resume Strategy

On project open / system startup:

1. Load card board from PostgreSQL `Card` table → reconstruct board state
2. Load `Decision` records → identify any pending/open decisions
3. Load `Event` records → reconstruct event timeline
4. Load artifact documents (BRD, SDD) → inject into agent context via ContextBuilder
5. Resume orchestration from current state

---

# 6. Deterministic State Engine

## 6.1 State Storage

```
Web Server (Next.js 16 + Prisma):
├── PostgreSQL (port 14000)
│   ├── Project, Card, Decision, Event tables
│   ├── Artifact table (BRD, SDD, MEMORY, etc.)
│   ├── OrchestrationRun table
│   └── LLMProviderConfig table
│
├── Disk: /workspaces/{projectId}/
│   ├── src/                  ← Agent-written source files
│   ├── .git/                 ← Auto-initialized git repo
│   └── [project files]       ← All files written via write_file tool
│
VS Code Extension (v0.1.0):
├── SSE stream: /api/projects/{id}/artifacts/stream
├── Heartbeat: /api/projects/{id}/vscode-ping (Redis 30s TTL)
└── Local folder linked to project workspace
```

## 6.2 State Integrity Rules

- **Illegal transitions blocked automatically** — `card-lifecycle.ts` validates every transition against the allowed state machine at both API route level and tool executor level
- **Authority guard** — `authority-guard.ts` enforces per-agent write permissions; dev agents (TL, JD, SD, QA, DO, SEC) are blocked until VS Code is connected
- **Loop detection** — `loop-detector.ts` implements fuzzy tool-loop, text-repetition, and question-re-ask detectors to prevent runaway agent loops
- **Path traversal protection** — workspace tool executor blocks `../` path escapes
- **Dangerous command blocking** — `run_command` tool blocks destructive shell commands

## 6.3 Checkpoint & Recovery

- PostgreSQL provides durable, transactional state storage
- Git auto-commit on every `write_file` call provides code checkpoint history
- Event records stored in PostgreSQL for audit trail
- Deterministic replay from event log is planned (Phase 3) but not yet implemented
