# 14. Git Governance & Branch Strategy

> **Implementation Status (2026-03-27):** Git is auto-initialized in each project's `/workspaces/{projectId}/` directory when the workspace is linked via the VS Code extension. The `git_commit` tool is called automatically after every `write_file` operation by agents. Branch management tools (`git_branch`, `git_diff`) are defined. PR creation (`create_pr`) is a stub — GitHub/GitLab integration is not yet active. Full CI/CD pipeline is planned for Phase 3.

## 14.1 Branch Structure

| Branch | Protection | Purpose | Status |
|--------|-----------|---------|--------|
| `main` | Protected — no direct push | Production-ready code | 📋 Planned with full pipeline |
| `develop` | Protected — PR required | Integration branch | 📋 Planned |
| `feature/*` | Unprotected | Feature development by JD | ✅ JD writes to workspace; git_branch tool available |
| `hotfix/*` | Protected — expedited review | Emergency production fixes | 📋 Planned |

## 14.2 Workflow

### Current (Phase 2)
```
1. VS Code extension linked to project workspace folder
2. JD writes files via write_file tool → git_commit called automatically
3. Files appear immediately in VS Code via SSE artifact stream
4. SD reviews code changes via review_changes / git_diff tools
5. Card state updated: IN_PROGRESS → UNDER_REVIEW → DONE
```

### Target (Phase 3)
```
1. JD creates feature/* branch from develop
2. JD implements task + unit tests
3. JD creates Pull Request → develop (GitHub/GitLab integration)
4. SD reviews PR (code quality, patterns, tests)
5. TL gives final technical approval
6. CI/CD runs: build → test → security scan
7. Audit Gatekeeper confirms DoD met
8. DevOps merges to develop
9. On release: develop → main (tagged release)
10. Rollback package generated automatically
```

## 14.3 Merge Requirements (Target State)

Every merge requires ALL of:

- [ ] PR approved by Senior Developer (SD)
- [ ] Tech Lead (TL) sign-off
- [ ] CI pipeline passes (build + test + security scan)
- [ ] Audit Gatekeeper clearance (DoD met)
- [ ] No open Decision Blockers on related cards

---

# 15. CI/CD & Release Governance

## 15.1 Pipeline Stages

| Stage | Tool/Action | Failure Behavior | Status |
|-------|-------------|-----------------|--------|
| Build | Compile/bundle source | Block pipeline | 📋 Planned |
| Unit Test | Run unit test suites | Block pipeline | 📋 Planned |
| Integration Test | Run integration suites | Block pipeline | 📋 Planned |
| Security Scan | Dependency + code scanning | Block on High/Critical | 📋 Planned |
| Artifact Package | Build deployable artifact | Block pipeline | 📋 Planned |
| Deploy (Staging) | Deploy to staging environment | Block pipeline | 📋 Planned |
| Smoke Test | Automated smoke tests on staging | Block release | 📋 Planned |
| Deploy (Production) | Deploy to production | Requires TL approval | 📋 Planned |
| Monitor | Health checks + alerting | SRE notified | 📋 Planned |

> Note: The `run_command`, `run_tests`, `run_build`, and `trigger_deploy` tools are defined in the tool system. `trigger_deploy` is currently a stub. Full pipeline execution requires Phase 3 cloud deployment infrastructure.

## 15.2 Release Requirements

- All modules must pass their Module DoD
- Rollback package generated and tested before deploy
- Release notes updated
- Decision ledger has no open blockers for release scope
- TL has given explicit release approval

## 15.3 Rollback Strategy

- Every release generates a rollback artifact (planned)
- Rollback can be triggered by: TL, DevOps, or SRE
- Rollback automatically restores previous deployment
- Post-rollback: incident logged, decision created for resolution
- Current fallback: git revert via git history in workspace folder
