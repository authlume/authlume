# AUTHLUME — PHASE TRACKER
Status: Planning | Version 0.1 | Owner: Founder | Target: quality-first, no fixed launch date

## Rules
Statuses: NOT STARTED / IN PROGRESS / BLOCKED / REVIEW / COMPLETE. Each task needs evidence (PR, test report, doc link). **No phase is complete because an AI agent says so.** Security gates require human review. Work in small reviewed PRs; record scope changes in ADRs.

| ID | Phase | Deliverables | Dependencies | Exit criteria | Status |
|---|---|---|---|---|---|
| P0 | Brand and governance | GitHub org, NPM scope, domain, Apache-2.0, SECURITY/CONTRIBUTING, CODEOWNERS, issue templates | None | Accounts secured; legal search noted; policies published | IN PROGRESS |
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

## P0 working tracker
Evidence checked 2026-10-23 by an AI agent; commands and raw results are in [`docs/governance/p0-verification.md`](docs/governance/p0-verification.md). Agent-drafted items stay in REVIEW until the founder approves and merges them. Agent-authored PR: pending (branch `chore/p0-governance`).

| Task ID | Owner | PR URL | Tests/evidence | Security reviewer | Status | Blockers | Decision/ADR | Date |
|---|---|---|---|---|---|---|---|---|
| P0-01 GitHub organization `authlume` created | Founder | — | `gh api orgs/authlume`: exists, created 2026-10-23T07:38:09Z, Free plan; `9jaDevo` sole member, active admin | — | COMPLETE | — | ADR-010 | 2026-10-23 |
| P0-02 GitHub organization secured | Founder | — | Org 2FA requirement **off**; members may delete repos, change visibility and invite outside collaborators; security features off for new repos | — | NOT STARTED | Founder applies [recommended org settings](docs/governance/repository-settings.md#organization-settings) | ADR-009 | 2026-10-23 |
| P0-03 NPM organization / `@authlume` scope reserved | Founder | — | Registry `/-/org/authlume/package` returns 200 `{}` (control scope returns 404 "Scope not found"); 0 packages; npm CLI not logged in, so org roles and 2FA **not verified** | — | REVIEW | Founder runs `npm org ls authlume` and confirms owner role and 2FA | ADR-010 | 2026-10-23 |
| P0-04 Primary repository `authlume/authlume` | Founder (authorized), agent (created) | — | Initially 404. Created **private** on founder authorization: `gh api repos/authlume/authlume` shows created 2026-10-23T08:35:13Z, visibility private, default branch `main` | — | COMPLETE | Make public once P0 is reviewed and the domain is registered | ADR-009 | 2026-10-23 |
| P0-05 Domain `authlume.org` owned | Founder | — | PIR RDAP returns 404 "Object not found"; DNS NXDOMAIN; domain **not registered** | — | NOT STARTED | Founder registers domain; then org domain verification and security/conduct addresses | ADR-010 | 2026-10-23 |
| P0-06 Legal/trademark similarity review | Founder + qualified advisor | — | No search performed; no review or opinion on record. **No clearance claimed.** | — | NOT STARTED | Founder commissions or performs review | ADR-010 | 2026-10-23 |
| P0-07 Apache-2.0 LICENSE and NOTICE | Agent draft, founder approval | Pending | `LICENSE` sha256 `cfc7749b…d30`, identical to apache.org text | — | REVIEW | Copyright holder wording in `NOTICE` | ADR-010 | 2026-10-23 |
| P0-08 README with development-status disclaimer | Agent draft, founder approval | Pending | `README.md` | — | REVIEW | — | ADR-001 (package table is proposed) | 2026-10-23 |
| P0-09 SECURITY.md private disclosure | Agent draft, founder approval | Pending | `SECURITY.md` uses GitHub private vulnerability reporting | Founder (policy review) | REVIEW | Private vulnerability reporting must be enabled (public repos only on Free plan); security email waits on P0-05 | ADR-009 | 2026-10-23 |
| P0-10 CONTRIBUTING.md | Agent draft, founder approval | Pending | `CONTRIBUTING.md` | — | REVIEW | DCO decision | ADR-009 | 2026-10-23 |
| P0-11 CODE_OF_CONDUCT.md | Agent draft, founder approval | Pending | Contributor Covenant 2.0, interim enforcement contact | — | REVIEW | Dedicated conduct contact waits on P0-05 | — | 2026-10-23 |
| P0-12 Issue forms and PR template | Agent draft, founder approval | Pending | `.github/ISSUE_TEMPLATE/*`, `.github/PULL_REQUEST_TEMPLATE.md` | — | REVIEW | — | — | 2026-10-23 |
| P0-13 CODEOWNERS with verified maintainers | Agent draft, founder approval | Pending | `* @9jaDevo` (only verified maintainer); pending invitee excluded | — | REVIEW | Second owner recommended; GitHub CODEOWNERS validation after push | ADR-009 | 2026-10-23 |
| P0-14 Documentation structure and strategy docs | Agent draft, founder approval | Pending | `docs/` index, ADR register and template, ADR-009/010 (Proposed), NPM publishing and ecosystem strategy, repository settings | — | REVIEW | Founder decisions in ADR-009 and ADR-010 | ADR-009, ADR-010 | 2026-10-23 |
| P0-15 Repository protections applied | Founder | — | Not applied. Repository is private on the GitHub Free plan | — | BLOCKED | On the Free plan, rulesets and private vulnerability reporting need a public repo (or a paid plan) | ADR-009 | 2026-10-23 |

### P0 exit criteria (2026-10-23)
- [ ] **Accounts secured.** GitHub organization exists, but its 2FA requirement is off; npm ownership and 2FA are unverified; `authlume.org` is unregistered.
- [ ] **Legal search noted.** No trademark or similarity search has been performed.
- [ ] **Policies published.** Governance files are drafted on `chore/p0-governance` and not merged.

P0 remains IN PROGRESS. Note: the account-status sentence in Master Blueprint section 6 predates these checks and is left unchanged for the founder to update; this tracker records current status.
