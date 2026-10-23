# AUTHLUME — MASTER BLUEPRINT
Version: 0.1 planning draft | 23 October 2026 | Status: approved direction, implementation not started

## 1. Mission and positioning
Authlume: **Authentication, on your terms.** An Apache-2.0-licensed, self-hosted, privacy-first authentication toolkit for developers. No Authlume-operated server is necessary. Primary message: **Authentication developers control**. Secondary: minimal data collection and excellent developer experience. Future: portable/federated identities. Do not advertise federation in V1.

## 2. Success criteria
First validate security and usability; then adoption. Measurable pilot objectives (not promises): 5–10 independent integrators; two reproducible demo apps; installation from zero in under 30 minutes for an experienced developer; documented threat model; passing security tests; independent review before recommending production use. Track downloads, retained integrations, open issues, contribution turnaround and vulnerabilities; stars are a secondary awareness metric. 100,000 stars is aspirational, not a delivery guarantee.

## 3. Audiences and boundaries
V1: TypeScript/JavaScript teams using Node.js, Next.js and React; PostgreSQL; application-specific users. Library manages authentication identities, credentials, sessions, recovery state, email-verification state and audit records. Application owns profile and business data linked by immutable opaque `user_id`. No Authlume-hosted auth, no universal cross-app identity in V1, no cryptocurrency or blockchain, no multi-tenant admin console.

## 4. Product requirements
### V1 capabilities
- Server-side signup, login, logout, session refresh/invalidation, change credentials, revoke other sessions, account deletion hooks.
- Conventional email/password with verified email flow when configured, strong password hashing, password reset via single-use expiring tokens, account-enumeration resistant responses.
- WebAuthn/passkeys using maintained standards-based libraries, correct RP ID/origin validation, challenges, credential counters and authenticator metadata handling.
- Recovery: cryptographically random high-entropy one-time recovery codes, hashed at rest, single-use atomically consumed, rotated after recovery; verified-email recovery only when verified email exists and developer configures delivery. Explicitly document that no verified channel + no recovery secret may mean unrecoverable account.
- Optional access-code module, **disabled by default**: (A) short-lived single-use 12-digit codes bound to an initiated transaction/trusted context, short TTL and attempt limits; (B) long-lived 12-digit codes **never accepted as unrestricted sole factor**; require independent factor and stricter policy, rotation and revocation. Numeric codes are secrets, not permanent user IDs. Do not use personal questions to derive keys. No offline brute-forceable plaintext or unkeyed short-code hashes: store server-keyed HMAC/peppered verifier with protected key management, plus online throttling. Consider removing persistent mode if safe UX cannot be demonstrated.
- Framework-neutral headless API; Node adapter, Next.js adapter, React headless helpers; PostgreSQL adapter; versioned migrations.
- Documentation site, reference, example apps, troubleshooting, security and migration guides.

### Out of scope V1
OAuth social login, universal login/federation, full OIDC provider, PHP SDK, MySQL/SQLite, premium UI kit, hosted SaaS, organizations/enterprise SSO, user analytics, payments, mobile SDK, user-defined security questions.

## 5. Threat model and security invariants
Assume remote attackers can enumerate endpoints, steal client tokens, perform credential stuffing, exploit CSRF/XSS in host apps, race recovery flows, compromise third-party packages and access leaked database backups. Specify host-application trust boundaries and residual risks.
- Cryptographic randomness from audited runtime primitives; never custom cryptography.
- Passwords: established Argon2id implementation with configurable, benchmarked parameters; migrate legacy hashes safely; no plaintext logging.
- Password reset and email verification: single-use, short expiry, hashed random tokens, replay-safe atomic database transactions, generic public responses.
- Sessions: opaque random session tokens; hash at rest where practical; HttpOnly Secure SameSite cookies for browser defaults, CSRF protection appropriate to deployment, secure rotation and invalidation, configurable idle/absolute expiry; avoid localStorage token defaults.
- Passkeys: follow WebAuthn spec and origin/RP constraints; challenge lifecycle and replay protection; no claims that biometrics are transmitted to Authlume.
- Access codes: rate limits per account, network and risk signals; exponential backoff where suitable; distributed/shared storage for counters; atomic single-use; risk-tier policy; no user lookup by short code without strict protections. Consider server-held lookup identifiers separate from secrets.
- Security operations: audit event hooks without secrets, redaction, safe errors, dependency pinning, SBOM, SAST, secret scanning, vulnerability intake, advisory publication and supported-version policy.
- Critical code requires independent human security review, including AI-generated code. Passing tests is insufficient for production endorsement.
- Document limits: developer controls TLS, DNS, infrastructure, email provider, key custody, database backups, rate-limit infrastructure and incident response.

## 6. Proposed architecture
TypeScript monorepo, pnpm workspaces; changesets or equivalent versioning; strict TypeScript; ESM-first with documented Node support. Suggested boundaries (adjust after ADR):
```
authlume/
  packages/
    core/          # domain types, interfaces, policy contracts (no DB/browser)
    server/        # server-only flows, sessions, credential orchestration
    postgres/      # adapter, migrations, transaction semantics
    node/          # Node HTTP integration
    next/          # Next.js server adapters
    react/         # optional headless hooks/client helpers; no secrets
    access-code/   # opt-in policy module
  examples/
    node-postgres/
    next-react-postgres/
  docs/
  tests/
    conformance/
    security/
  .github/
  LICENSE
  SECURITY.md
  CONTRIBUTING.md
  CODE_OF_CONDUCT.md
  CHANGELOG.md
```
Avoid circular imports. Server code must not enter client bundles. A package is publishable only when it has stable documented entrypoints, tests, and release checks. NPM scope `@authlume` pending reservation; domain `authlume.org` reported available, not yet confirmed purchased. GitHub organization `authlume` reported registrable, not yet confirmed created.

## 7. Data model (conceptual)
`auth_users(id, created_at, status)`; `auth_emails(user_id, normalized_email, verified_at)`; `password_credentials(user_id, password_hash, updated_at)`; `passkey_credentials(id, user_id, public_key, counter, transports, created_at)`; `access_code_credentials(id, user_id, policy, verifier, expires_at, revoked_at)`; `sessions(id, user_id, token_hash, expires_at, revoked_at, metadata)`; `recovery_codes(id, user_id, verifier, used_at)`; `one_time_challenges(id, purpose, user_id?, verifier, expires_at, consumed_at)`; `audit_events(id, user_id?, action, timestamp, safe_metadata)`.
Enforce uniqueness with DB constraints and atomic transactions, not check-then-insert. Minimize stored PII and define retention/deletion hooks. Never use numeric codes as database primary keys.

## 8. API design principles
Public contracts: `createAuth(config)`, `auth.register(...)`, `auth.signIn(...)`, `auth.signOut(...)`, `auth.getSession(...)`, `auth.recover(...)`; exact signatures require design review before implementation. Browser initiation and server verification must be distinct. All endpoints define input schemas, output schemas, error taxonomy, CSRF and rate-limit behavior, idempotency and audit events. Server-only secrets never appear in frontend configuration. Do not publish misleading pseudo-APIs as production examples.

## 9. Developer experience
One-page quickstart, local Docker PostgreSQL example, working end-to-end Next.js app, framework-specific guides, copy/pasteable safe defaults, migration command, configuration reference, security checklist, FAQ and API docs. Headless first; polished UI kit later. Accessibility and localization-ready API surfaces. Test quickstart with an external developer unfamiliar with codebase.

## 10. Compatibility and releases
Node LTS versions defined at release; explicit Next.js/React support matrix. SemVer and changelogs; signed or provenance-enabled NPM publishing via CI; least-privilege publish tokens/OIDC; protected branches, CODEOWNERS, 2FA for maintainers, required reviews, automated dependency updates. Supply-chain incident plan and reproducible build checks. No production-ready claim until independent security review and deployment validation.

## 11. Open-source governance and licensing
Apache License 2.0 for core, include copyright notices and attribution obligations; confirm dependencies are license-compatible. Founder-led maintainership with public RFC/ADR process, contribution guide, security reporting channel, moderation rules, DCO or contributor policy. Premium offerings may use separate proprietary licences; do not relicense community contributions without appropriate rights. Essential security controls and fixes remain free.

## 12. Commercial path
Stage 1: free core and documentation. Stage 2: paid integrations, enterprise support, implementation, premium UI/admin tools. Stage 3 (optional): managed hosting and enterprise features if traction justifies operations. Do not promise future pricing or hosted service before validation.

## 13. Roadmap
R0 brand/account reservation and threat model; R1 monorepo + contracts; R2 database + sessions; R3 email/password + verified recovery; R4 passkeys; R5 optional code policies; R6 integrations + docs + examples; R7 independent review and public beta; R8 stability and ecosystem. Later: UI components, other databases, PHP Composer SDK, social login, federation/OIDC, premium modules, optional hosting.

## 14. Nonfunctional and operational acceptance
Security tests for replay, concurrency, brute-force limits, account enumeration, token leakage, cross-origin misuse and session revocation. Performance tests with published hardware assumptions; no invented SLA. CI must run unit/integration/e2e, lint, types, migration tests, package import/bundle boundary tests and dependency scans. Public documentation must state known limitations.

## 15. Decision register / open ADRs
ADR-001 package naming and split; ADR-002 session strategy and cookie defaults; ADR-003 passkey credential lifecycle; ADR-004 email identity normalization and duplicate policy; ADR-005 code policies and whether long-lived numeric codes survive review; ADR-006 cryptographic verifier storage and secret rotation; ADR-007 transaction/rate-limit backing store; ADR-008 framework support matrix; ADR-009 repository and release process; ADR-010 legal/trademark clearance and NPM scope reservation. Every ADR records options, rationale, security consequences and migration cost.

## 16. Launch checklist
Reserve `authlume.org`, GitHub org, NPM scope; perform similarity trademark review. Publish documentation and security policy. Validate install on clean machine. Independent reviewer signs off threat model and sensitive flows. Resolve critical/high vulnerabilities. Beta disclaimer, supported versions and response contact published. No real-user rollout without deployment-specific review.
