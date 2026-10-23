# AUTHLUME — PHASE TRACKER
Status: Planning | Version 0.1 | Owner: Founder | Target: quality-first, no fixed launch date

## Rules
Statuses: NOT STARTED / IN PROGRESS / BLOCKED / REVIEW / COMPLETE. Each task needs evidence (PR, test report, doc link). **No phase is complete because an AI agent says so.** Security gates require human review. Work in small reviewed PRs; record scope changes in ADRs.

| ID | Phase | Deliverables | Dependencies | Exit criteria | Status |
|---|---|---|---|---|---|
| P0 | Brand and governance | GitHub org, NPM scope, domain, Apache-2.0, SECURITY/CONTRIBUTING, CODEOWNERS, issue templates | None | Accounts secured; legal search noted; policies published | NOT STARTED |
| P1 | Specification and threat model | Auth flows, risk model, abuse scenarios, ADRs, API contracts | P0 | Independent design walkthrough; no unresolved critical risk | NOT STARTED |
| P2 | Monorepo foundation | pnpm workspace, TS config, CI, release tooling, examples skeleton | P1 | Clean clone installs; type/lint/test/build green; protected branches | NOT STARTED |
| P3 | Persistence and sessions | PostgreSQL adapter, migrations, transactional sessions, audit hooks | P2 | Concurrency/replay/revocation and migration tests pass | NOT STARTED |
| P4 | Email/password | Register/login, verify email, password reset, rate limiting | P3 | Enumeration, reset replay, brute force and token-leak tests pass | NOT STARTED |
| P5 | Passkeys | WebAuthn registration/login, challenge and credential lifecycle | P3, P4 | RP/origin/challenge tests and independent review pass | NOT STARTED |
| P6 | Access codes | Temporary flow; optional long-lived + mandatory factor policy | P3–P5 | Explicit risk signoff; single-use and abuse tests pass; drop unsafe mode | NOT STARTED |
| P7 | Integrations | Node/Next adapters, React headless API, two real examples | P4–P6 | Fresh-project install; no server secrets in browser bundles | NOT STARTED |
| P8 | Documentation and DX | docs site, API reference, quickstart, migration/security guides | P7 | External developers complete integration unassisted | NOT STARTED |
| P9 | Public beta readiness | Security review, remediation, versioning, NPM provenance, disclosure policy | P1–P8 | Critical/high findings resolved; beta limitations public | NOT STARTED |
| P10 | Community launch | Announcement, starter issues, RFC process, demos, feedback triage | P9 | First external integrations; maintainers responsive | NOT STARTED |
| FUT1 | UI kit | Optional accessible login components | P9 | Accessible, framework-tested components | DEFERRED |
| FUT2 | Ecosystem | MySQL/SQLite, PHP, other frameworks, social providers | Traction | Demand validated and maintainers available | DEFERRED |
| FUT3 | Federation | Self-hosted OIDC identity-provider option | Mature core | Protocol conformance and independent audit | DEFERRED |
| FUT4 | Commercial | Support, enterprise modules, optional hosting | Traction | Willingness-to-pay and operating model validated | DEFERRED |

## Detailed security test gates
- P3: simultaneous token redemption, expired sessions, revocation, migration rollback, isolation, backups and secret handling.
- P4: duplicate email race, account enumeration, weak password defenses, credential stuffing throttling, reset token reuse, timing leaks, logging redaction.
- P5: mismatched RP ID, origin spoofing, reused challenge, invalid signatures, credential deletion/re-enrollment and counter behavior.
- P6: guessed codes, concurrent redemption, distributed rate-limit enforcement, persistent code second-factor bypass, rotation, audit trails.
- P7: CSRF, cookie settings, XSS-sensitive storage, SSR/browser package separation, proxy and origin configuration.
- P9: SBOM, dependency scans, malicious PR protections, npm publish dry-run, signed/provenance release, incident simulation.

## Definition of done (all implementation PRs)
1. Scope and public API documented; ADR updated if necessary.
2. Unit/integration tests cover success, failure, replay, concurrency and security abuse relevant to change.
3. No secrets in logs, telemetry, fixtures or browser bundle.
4. Formatting, lint, typecheck, tests, build and security scanners pass.
5. Human review completed; independent security reviewer required for auth/crypto/recovery/session changes.
6. Docs, examples and changelog updated where user-visible.

## Metrics and review cadence
Weekly: CI health, vulnerability queue, open PR age, docs friction, real installation attempts. Monthly: npm downloads, successful integrations, active contributors, adoption feedback. GitHub stars are secondary. Never manufacture downloads, endorsements or stars.

## Working tracker template
For each task record: `Task ID | Owner | PR URL | Tests/evidence | Security reviewer | Status | Blockers | Decision/ADR | Date`.
