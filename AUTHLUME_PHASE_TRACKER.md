# AUTHLUME — PHASE TRACKER
Status: Planning | Version 0.1 | Owner: Founder | Target: quality-first, no fixed launch date

## Rules
Statuses: NOT STARTED / IN PROGRESS / BLOCKED / REVIEW / COMPLETE. Each task needs evidence (PR, test report, doc link). **No phase is complete because an AI agent says so.** Security gates require human review. Work in small reviewed PRs; record scope changes in ADRs.

| ID | Phase | Deliverables | Dependencies | Exit criteria | Status |
|---|---|---|---|---|---|
| P0 | Brand and governance | GitHub org, NPM scope, domain, Apache-2.0, SECURITY/CONTRIBUTING, CODEOWNERS, issue templates | None | Accounts secured; legal search noted; policies published | IN PROGRESS |
| P1 | Specification and threat model | Auth flows, risk model, abuse scenarios, ADRs, API contracts | P0 | Independent design walkthrough; no unresolved critical risk | IN PROGRESS |
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
Evidence checked 2026-10-23 by an AI agent; commands and raw results are in [`docs/governance/p0-verification.md`](docs/governance/p0-verification.md). Agent-drafted items stay in REVIEW until the founder approves and merges them. Agent-authored PR: https://github.com/authlume/authlume/pull/1 (branch `chore/p0-governance`).

| Task ID | Owner | PR URL | Tests/evidence | Security reviewer | Status | Blockers | Decision/ADR | Date |
|---|---|---|---|---|---|---|---|---|
| P0-01 GitHub organization `authlume` created | Founder | — | `gh api orgs/authlume`: exists, created 2026-10-23T07:38:09Z, Free plan; `9jaDevo` sole member, active admin | — | COMPLETE | — | ADR-010 | 2026-10-23 |
| P0-02 GitHub organization secured | Founder | — | Org 2FA requirement **off**; members may delete repos, change visibility and invite outside collaborators; security features off for new repos; no verified domain | — | NOT STARTED | Founder applies [recommended org settings](docs/governance/repository-settings.md#organization-settings), including verifying `authlume.org` on the org | ADR-009 | 2026-10-23 |
| P0-03 NPM organization / `@authlume` scope reserved | Founder | — | Registry `/-/org/authlume/package` returns 200 `{}` (control scope returns 404 "Scope not found"); 0 packages; npm CLI not logged in, so org roles and 2FA **not verified** | — | REVIEW | Founder runs `npm org ls authlume` and confirms owner role and 2FA | ADR-010 | 2026-10-23 |
| P0-04 Primary repository `authlume/authlume` | Founder (authorized), agent (created) | — | Initially 404. Created **private** on founder authorization at 2026-10-23T08:35:13Z; made **public** on founder authorization at 2026-10-23T09:11Z (`visibility: public`, anonymous API access HTTP 200); default branch `main` | — | COMPLETE | — | ADR-009 | 2026-10-23 |
| P0-05 Domain `authlume.org` owned | Founder | — | First check: RDAP 404 (unregistered). Re-check 2026-10-23T09:10Z: PIR RDAP shows registered 2026-10-23T09:08:53Z, registrar Spaceship, Inc., expires 2027-10-23, status client transfer prohibited; registrant redacted. Ownership confirmed by the founder in writing on 2026-10-23 | — | COMPLETE | Follow-ups: enable auto-renew before 2027-10-23; verify the domain on the GitHub org (DNS TXT proof of control); set up security and conduct addresses | ADR-010 | 2026-10-23 |
| P0-06 Legal/trademark similarity review | Founder + qualified advisor | — | No search performed; no review or opinion on record. **No clearance claimed.** | — | NOT STARTED | Founder commissions or performs review | ADR-010 | 2026-10-23 |
| P0-07 Apache-2.0 LICENSE and NOTICE | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `LICENSE` sha256 `cfc7749b…d30`, identical to apache.org text | — | REVIEW | Copyright holder wording in `NOTICE` | ADR-010 | 2026-10-23 |
| P0-08 README with development-status disclaimer | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `README.md` | — | REVIEW | — | ADR-001 (package table is proposed) | 2026-10-23 |
| P0-09 SECURITY.md private disclosure | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `SECURITY.md` uses GitHub private vulnerability reporting, enabled on the repository 2026-10-23 (`enabled: true`) | Founder (policy review) | REVIEW | No security mailbox set up yet | ADR-009 | 2026-10-23 |
| P0-10 CONTRIBUTING.md | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `CONTRIBUTING.md` | — | REVIEW | DCO decision | ADR-009 | 2026-10-23 |
| P0-11 CODE_OF_CONDUCT.md | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | Contributor Covenant 2.0, interim enforcement contact | — | REVIEW | No dedicated conduct mailbox set up yet | — | 2026-10-23 |
| P0-12 Issue forms and PR template | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `.github/ISSUE_TEMPLATE/*`, `.github/PULL_REQUEST_TEMPLATE.md` | — | REVIEW | — | — | 2026-10-23 |
| P0-13 CODEOWNERS with verified maintainers | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `* @9jaDevo` (only verified maintainer); pending invitee excluded; `gh api repos/authlume/authlume/codeowners/errors?ref=chore/p0-governance` returns `{"errors":[]}` | — | REVIEW | Second owner recommended | ADR-009 | 2026-10-23 |
| P0-14 Documentation structure and strategy docs | Agent draft, founder approval | [PR 1](https://github.com/authlume/authlume/pull/1) | `docs/` index, ADR register and template, ADR-009/010 (Proposed), NPM publishing and ecosystem strategy, repository settings | — | REVIEW | Founder decisions in ADR-009 and ADR-010 | ADR-009, ADR-010 | 2026-10-23 |
| P0-15 Repository protections applied | Founder (authorized), agent (applied) | — | Applied 2026-10-23T09:15Z and verified by API: private vulnerability reporting `enabled: true`; secret scanning and push protection enabled; Dependabot alerts on (HTTP 204) and security updates enabled; ruleset `main` (ID 25710980) active on the default branch with no bypass actors, blocking deletion and force pushes, requiring linear history and a pull request with 0 approvals and conversation resolution. PR 1 still `MERGEABLE`/`CLEAN` | — | COMPLETE | Follow-ups: when a second maintainer joins, set 1 approval and turn on code-owner review and most-recent-push approval; P2 adds required status checks and CodeQL | ADR-009 | 2026-10-23 |

### P0 exit criteria (2026-10-23)
- [ ] **Accounts secured.** The repository is public with its security settings and `main` ruleset applied, but the org 2FA requirement is off and org member permissions are not tightened; npm ownership and 2FA are unverified; `authlume.org` is registered (founder-confirmed) but not yet verified on the GitHub org.
- [ ] **Legal search noted.** No trademark or similarity search has been performed.
- [ ] **Policies published.** Governance files are drafted on `chore/p0-governance` and not merged.

P0 remains IN PROGRESS. Note: the account-status sentence in Master Blueprint section 6 predates these checks and is left unchanged for the founder to update; this tracker records current status.

## P1 working tracker
P1 drafting started 2026-10-23 at the founder's direction, while P0 exit criteria remain open (see above). All items are AI-drafted and stay in REVIEW until the founder approves them; the security-relevant ones also need independent human security review. Nothing is implemented. Branch `p1/threat-model-and-adrs` (stacked on `chore/p0-governance`); PR not yet opened.

| Task ID | Owner | PR URL | Tests/evidence | Security reviewer | Status | Blockers | Decision/ADR | Date |
|---|---|---|---|---|---|---|---|---|
| P1-01 Threat model: passwords, passkeys, recovery, sessions, access codes (both modes), cross-cutting | Agent draft, founder approval | — | [`docs/threat-model/`](docs/threat-model/README.md): 11 trust boundaries, 15 assets, 13 attacker capabilities, 11 invariants, 93 threats, 25 residual risks, 98 test cases mapped to the P3–P7 and P9 gates | Required, not assigned | REVIEW | Independent design walkthrough (P1 exit criterion) | ADR-001 to ADR-013 | 2026-10-23 |
| P1-02 Unsafe long-lived numeric login designs flagged | Agent draft, founder decision | — | [Unsafe designs U-01 to U-12](docs/threat-model/access-codes.md#unsafe-numeric-code-designs); recommendation: remove Mode B from V1 | Required, not assigned | REVIEW | Founder decision in ADR-005 | ADR-005 (Open) | 2026-10-23 |
| P1-03 ADR-001 package naming and split | Agent draft, founder approval | — | [ADR-001](docs/adr/0001-package-naming-and-split.md) (Proposed) | Required for the server/browser boundary | REVIEW | Founder decision | ADR-001 | 2026-10-23 |
| P1-04 ADR-002 session strategy and cookie defaults | Agent draft, founder approval | — | [ADR-002](docs/adr/0002-session-strategy-and-cookie-defaults.md) (Proposed) | Required, not assigned | REVIEW | Founder decision on expiry, `SameSite` and metadata defaults | ADR-002 | 2026-10-23 |
| P1-05 ADR-007 rate-limit persistence and transaction semantics | Agent draft, founder approval | — | [ADR-007](docs/adr/0007-transaction-and-rate-limit-backing-store.md) (Proposed) | Required, not assigned | REVIEW | Founder decision | ADR-007 | 2026-10-23 |
| P1-06 ADR-011 password hashing (new) | Agent draft, founder approval | — | [ADR-011](docs/adr/0011-password-hashing.md) (Proposed) | Required, not assigned | REVIEW | Confirm NIST SP 800-63B Rev. 4 minimum length and Node.js built-in Argon2 status | ADR-011 | 2026-10-23 |
| P1-07 ADR-012 one-time challenge storage (new) | Agent draft, founder approval | — | [ADR-012](docs/adr/0012-one-time-challenge-storage.md) (Proposed) | Required, not assigned | REVIEW | Depends on ADR-006 for low-entropy verifiers | ADR-012 | 2026-10-23 |
| P1-08 ADR-013 token rotation and invalidation (new) | Agent draft, founder approval | — | [ADR-013](docs/adr/0013-token-rotation.md) (Proposed) | Required, not assigned | REVIEW | Founder decision on options B and C and the optional matrix cells | ADR-013 | 2026-10-23 |
| P1-09 Remaining ADRs: 003, 004, 005, 006, 008 | — | — | Not drafted | Required for 003, 005, 006 | NOT STARTED | — | ADR-003 to 006, 008 | 2026-10-23 |
| P1-10 API contracts: input and output schemas, error taxonomy, CSRF and rate-limit behaviour per endpoint | — | — | Not drafted | Required | NOT STARTED | Needs ADR-001, 002 and 012 decisions | Blueprint §8 | 2026-10-23 |
| P1-11 Assign a phase for recovery codes | Founder | — | The tracker does not name the phase in which recovery codes ship | — | NOT STARTED | Founder decision | — | 2026-10-23 |
| P1-12 Independent design walkthrough | Founder + independent reviewer | — | Not held | Required | NOT STARTED | P1-01 to P1-10 | — | 2026-10-23 |

### P1 exit criteria (2026-10-23)
- [ ] **Independent design walkthrough.** Not held.
- [ ] **No unresolved critical risk.** The Mode B persistent access codes (ADR-005) and every Proposed ADR are unresolved.
