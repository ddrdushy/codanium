# Appendix D — Security Control Checklist

> **Implementation Status (2026-03-27):** Active security controls are marked ✅. Planned controls are marked 📋. The checklist below serves as both a reference for what is built and a target for remaining work.
>
> **Active controls:** AES-256-GCM API key encryption, path traversal blocking, dangerous command blocking, authority guard, loop detection, card lifecycle validation, NextAuth session management.
>
> **Planned controls:** Production auth (replace dev mode), enterprise RBAC, OS Keychain integration, pre-commit hooks, HTTPS enforcement, penetration testing.

## D.1 Architecture Stage

- [ ] Threat model completed by Security & Compliance Agent
- [ ] Authentication strategy defined and documented in SDD
- [ ] Authorization model defined (RBAC/ABAC)
- [ ] Data classification completed (PII, sensitive, public)
- [ ] Encryption strategy defined (at-rest, in-transit)
- [ ] Third-party dependency risk assessed

## D.2 Development Stage

- [ ] No hardcoded secrets in source code
- [ ] Input validation on all user-facing endpoints
- [ ] Output encoding to prevent XSS
- [ ] Parameterized queries to prevent SQL injection
- [ ] CSRF protection implemented
- [ ] Rate limiting configured on public endpoints
- [ ] Error messages do not leak internal details

## D.3 Code Review Stage

- [ ] SD reviewed for security anti-patterns
- [ ] Security & Compliance Agent scan completed
- [ ] No new Critical/High vulnerabilities introduced
- [ ] Dependency versions are current (no known CVEs)
- [ ] Authentication/authorization correctly implemented

## D.4 Testing Stage

- [ ] Security test scenarios executed
- [ ] Penetration test results reviewed (if applicable)
- [ ] OWASP Top 10 coverage validated
- [ ] Secrets scanning confirms no leaks in repository

## D.5 Release Stage

- [ ] Final dependency scan (no new vulnerabilities since review)
- [ ] Secrets rotated if any were exposed during development
- [ ] Security headers configured in deployment
- [ ] HTTPS enforced
- [ ] Monitoring and alerting active for security events

## D.6 Post-Release

- [ ] Security monitoring dashboards active
- [ ] Incident response plan documented
- [ ] Vulnerability disclosure process in place
- [ ] Regular security audit schedule established
