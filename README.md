# Authlume

**Authentication, on your terms.**

Authlume is an open-source, self-hosted, privacy-first authentication toolkit for
TypeScript and JavaScript developers. It runs inside your own application and
infrastructure. No Authlume-operated server is required, and no user data leaves
the systems you control.

> [!WARNING]
> **Development status: planning (Phase P0, governance).**
> There is no usable code yet and no packages have been published to npm.
> Authlume makes no security guarantees at this stage. Do not use it to protect
> real users. Authlume will not be recommended for production use until it has
> a documented threat model, passing security tests and an independent security
> review.

## Goals

Authlume is being designed around three priorities, in this order:

1. **Authentication developers control.** You host it, you own the database, you
   choose the infrastructure.
2. **Minimal data collection.** Authlume stores only what authentication needs.
   Profile and business data stay in your application, linked by an immutable,
   opaque `user_id`.
3. **Good developer experience.** Headless, typed APIs; secure defaults you can
   copy and paste; clear documentation of limits and trade-offs.

## Planned V1 scope

The capabilities below are **planned, not implemented**. Each one ships only
after its phase passes the security gates in the
[Phase Tracker](AUTHLUME_PHASE_TRACKER.md).

- Server-side sign-up, sign-in, sign-out, session refresh and revocation
- Email and password with a verified-email flow, Argon2id password hashing and
  single-use, expiring reset tokens
- Passkeys (WebAuthn) built on a maintained, standards-based library
- Recovery codes that are high-entropy, hashed at rest, and single-use
- An optional access-code module, **disabled by default**, whose design is
  still under security review
- Integrations for Node.js, Next.js and React (headless helpers), with a
  PostgreSQL adapter and versioned migrations

### Not in V1

Hosted authentication, social login, federation and OpenID Connect provider
features, a PHP SDK, MySQL and SQLite adapters, a UI kit, enterprise SSO and
user analytics are all out of scope for V1. See the
[ecosystem strategy](docs/strategy/ecosystem.md) for what may come later.

## Planned packages

Authlume will be a single TypeScript monorepo publishing independently
versioned packages under the [`@authlume`](https://www.npmjs.com/org/authlume)
npm scope. The split below is the Master Blueprint's proposal and is
**subject to ADR-001**. None of these packages exist yet.

| Package | Purpose | Runtime |
| --- | --- | --- |
| `@authlume/core` | Domain types, interfaces and policy contracts | Shared, no DB or browser APIs |
| `@authlume/server` | Server-only flows, sessions and credential orchestration | Server |
| `@authlume/postgres` | PostgreSQL adapter, migrations and transaction semantics | Server |
| `@authlume/node` | Node.js HTTP integration | Server |
| `@authlume/next` | Next.js server adapters | Server |
| `@authlume/react` | Optional headless hooks and client helpers, no secrets | Browser |
| `@authlume/access-code` | Opt-in access-code policy module | Server |

See the [npm publishing strategy](docs/strategy/npm-publishing.md) for how these
packages will be versioned and released.

## Project status

| Phase | Focus | Status |
| --- | --- | --- |
| P0 | Brand, accounts and governance | In progress |
| P1 | Specification and threat model | Not started |
| P2 | Monorepo foundation and CI | Not started |
| P3 to P8 | Persistence, credentials, passkeys, integrations, documentation | Not started |
| P9 | Public beta security gate | Not started |
| P10 | Community launch | Not started |

The [Phase Tracker](AUTHLUME_PHASE_TRACKER.md) holds the authoritative status
and the evidence for each task. Authlume has no fixed launch date; quality and
security come first.

## Planning documents

- [Master Blueprint](AUTHLUME_MASTER_BLUEPRINT.md): mission, scope, threat model,
  architecture and decision register. This is the source of truth.
- [Phase Tracker](AUTHLUME_PHASE_TRACKER.md): phases, exit criteria, security
  test gates and evidence.
- [Agent Prompts](AUTHLUME_AGENT_PROMPTS.md): rules for AI-assisted
  development, including mandatory human and security review.
- [Documentation index](docs/README.md): architecture decisions, governance
  and strategy documents.

## Security

Please do not report security vulnerabilities through public issues. Follow the
private process in [SECURITY.md](SECURITY.md).

## Contributing

Authlume is at the design stage, so the most useful contributions right now are
feedback on the threat model, the architecture decisions and the documentation.
Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening an issue or pull request.
Everyone who takes part agrees to follow the
[Code of Conduct](CODE_OF_CONDUCT.md).

## License

Authlume is licensed under the [Apache License, Version 2.0](LICENSE).
See [NOTICE](NOTICE) for attribution.
