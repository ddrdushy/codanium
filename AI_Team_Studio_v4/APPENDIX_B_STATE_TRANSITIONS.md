# Appendix B — State Transition Matrix

> **Implementation Status (2026-03-27):** The full card state transition matrix is implemented in `src/lib/ai/orchestration/card-lifecycle.ts`. All legal transitions and illegal transitions listed below are enforced at both the API route level and the tool executor level. Illegal transitions return a 400 error with the specific violation reason.

## B.1 Card State Transitions

| From State | To State | Trigger | Required Conditions | Responsible |
|-----------|----------|---------|---------------------|-------------|
| `—` | Planned | Card created | Parent exists (for child cards) | Orchestrator |
| Planned | In Progress | Work started | Agent assigned, no blockers | Orchestrator |
| In Progress | Under Review | Submitted for review | Task DoD met (code + tests) | JD → SD |
| Under Review | In Progress | Changes requested | Review feedback provided | SD → JD |
| Under Review | Testing | Review approved | PR approved by SD | SD → QA |
| Testing | Blocked | Defect or decision needed | Defect card created or decision requested | QA |
| Blocked | In Progress | Blocker resolved | Decision approved or fix ready | Decision Controller |
| Testing | Done | All tests pass | Feature DoD met, no open defects | QA → Audit Gate |
| Done | Released | Deployed | Release DoD met, rollback ready | DevOps + TL |

---

## B.2 Decision State Transitions

| From State | To State | Trigger | Responsible |
|-----------|----------|---------|-------------|
| `—` | Drafted | Decision triggered | Any agent |
| Drafted | Options Collected | Options gathered | Owner agent |
| Options Collected | Recommended | Recommendation added | Owner agent |
| Recommended | Awaiting Approval | Submitted for approval | Decision Controller |
| Awaiting Approval | Approved | Option selected | User / TL |
| Awaiting Approval | Rejected | Needs revision | User / TL |
| Rejected | Options Collected | Revise options | Owner agent |
| Approved | Implemented | Changes applied | Executing agent |
| Implemented | Verified | QA confirms | QA / Audit Gate |

---

## B.3 Illegal Transitions (Blocked by State Controller)

| Attempted Transition | Reason Blocked |
|---------------------|---------------|
| Planned → Done | Cannot skip development and testing |
| Planned → Released | Cannot skip entire lifecycle |
| In Progress → Released | Cannot skip review, testing, and release process |
| Under Review → Done | Cannot skip testing |
| Testing → Released | Cannot skip Done state (DoD enforcement) |
| Done → In Progress | Regression — must go through decision process |
| Released → Planned | Invalid — released items cannot be un-released |
