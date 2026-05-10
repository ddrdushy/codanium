# Security Policy

## Supported Versions

Only the latest `main` branch receives security updates.

| Version | Supported          |
| ------- | ------------------ |
| `main`  | :white_check_mark: |
| `< 1.0` | :x:                |

## Reporting a Vulnerability

**Please do NOT open a public issue for security vulnerabilities.**

Instead, report privately by either:

- Opening a [private security advisory](../../security/advisories/new) on GitHub, or
- Emailing the maintainers directly (contact details in the repo's GitHub profile).

When reporting, please include:

- A description of the issue and its impact
- Steps to reproduce (proof-of-concept if possible)
- The affected subproject (`ai-team-studio`, `codanium-desktop`, etc.) and version/commit
- Any suggested mitigation

## What to expect

- Acknowledgement within 72 hours
- An initial assessment within 7 days
- Coordinated disclosure once a fix is ready — credit given to reporters who request it

## Out of scope

- Vulnerabilities in third-party dependencies (please report upstream)
- Issues requiring physical access to a user's machine
- Self-XSS or social-engineering attacks
- Findings from automated scanners without a working PoC
