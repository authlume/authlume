# npm publishing strategy

- **Status:** Strategy for future phases. Nothing described here is
  implemented. No `package.json`, build tooling, publishing credentials or
  packages exist, and none will be created during P0.
- **Decisions:** package names and boundaries are subject to **ADR-001**. The
  release process is subject to
  [ADR-009](../adr/0009-repository-and-release-process.md).

## Scope and ownership

All packages are published under the **`@authlume`** scope, which belongs to
the npm organization `authlume`. The organization exists in the public
registry and has no packages (verified 2026-10-23; see the
[P0 evidence log](../governance/p0-verification.md)). Within a scope, only
members of that organization can publish, so every `@authlume/*` name is
reserved by the organization without publishing placeholders.

## Proposed packages

These follow the Master Blueprint's directory layout. Final names and
boundaries are decided in ADR-001.

| Directory | npm name | Runtime | May depend on |
| --- | --- | --- | --- |
| `packages/core` | `@authlume/core` | Shared, with no database, Node-only or browser-only APIs | Nothing else in `@authlume` |
| `packages/server` | `@authlume/server` | Server only | `core` |
| `packages/postgres` | `@authlume/postgres` | Server only | `core`, `server` contracts |
| `packages/node` | `@authlume/node` | Server only (Node.js HTTP) | `core`, `server` |
| `packages/next` | `@authlume/next` | Server only (Next.js) | `core`, `server` |
| `packages/react` | `@authlume/react` | Browser and React; **never** server code or secrets | `core` types only |
| `packages/access-code` | `@authlume/access-code` | Server only, opt-in | `core`, `server` |

Dependencies must not form cycles. The dependency direction above is the
Blueprint's intent ("avoid circular imports") and is verified by tests from P2.

Example applications under `examples/` are never published. Every example
`package.json` will set `"private": true`.

## Principles

### Independent entrypoints

- Each package declares an explicit `exports` map. Deep imports into
  unexported files are not supported.
- ESM first, with the supported Node.js LTS range documented and enforced
  through `engines` (decided in P2 and ADR-008).
- Type declarations ship with every package.
- Each package's `files` allowlist includes only build output, `LICENSE`,
  `NOTICE` and `README.md`. Tests, fixtures and source maps that could contain
  sensitive paths are excluded. CI checks the contents with `npm pack --dry-run`.

### Strict server and browser separation

Server secrets must never reach a browser bundle (Blueprint sections 6 and 8).
Planned controls, to be confirmed in ADR-001 and implemented in P2 and P7:

- Server-only packages do not export a `browser` condition and fail fast if
  they are bundled for the client. One way to do this in Next.js is to import
  the `server-only` marker module, but the exact mechanism is an ADR-001
  decision.
- `@authlume/react` has no runtime dependency on `server`, `postgres`, `node`,
  `next` or `access-code`.
- Configuration types separate server-only settings from public settings, so
  secrets cannot appear in frontend configuration.
- CI runs import and bundle boundary tests (Blueprint section 14). These tests
  build a client bundle and fail if server modules or known secret identifiers
  appear in it.

### Semantic Versioning

- Every package follows [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html).
- Before 1.0.0, a minor version bump may contain breaking changes. Changelogs
  must call these out explicitly.
- Pre-releases use the `next` or `beta` dist-tags. `latest` is reserved for
  releases that have passed the release gates for their stage.
- Packages are versioned independently (ADR-009 option 4A), unless ADR-009
  decides otherwise.

### Changesets

- Every pull request that changes a publishable package includes a changeset
  describing the user-visible change and its SemVer impact.
- The Changesets bot opens a "Version packages" pull request. A human reviews
  and merges it. Merging it triggers the publish workflow.
- Each package's changelog is generated from its changesets. The root
  `CHANGELOG.md` covers repository-level changes only.

### Provenance and credentials

- Packages are published **only** from GitHub Actions. Nobody publishes from a
  developer machine.
- Publishing uses npm trusted publishing (OIDC) with provenance attestations,
  so each package version links to the exact workflow run and commit. No
  long-lived npm token is stored as a repository secret.
- The publish job runs in a protected environment that requires owner
  approval (see [repository settings](../governance/repository-settings.md#environments-from-p9)).
- Every npm account with publish rights uses 2FA.
- AI agents never receive publishing credentials and never publish (Agent
  Prompts, "Agent workflow rule").
- To verify in P9: whether the first publish of a brand-new package name can
  use trusted publishing, or needs a one-time, short-lived, least-privilege
  token that is revoked immediately afterwards. Record the answer in ADR-009.

## When packages get published

The Blueprint states that "a package is publishable only when it has stable
documented entrypoints, tests, and release checks".

| Phase | Publishing activity |
| --- | --- |
| P0 (now) | None. Scope verified, strategy documented. |
| P1 | None. ADR-001 fixes names and boundaries. |
| P2 | Workspace, Changesets and `npm publish --dry-run` in CI. Nothing is published. |
| P3 to P8 | Dry runs only. No public release before the beta gate. |
| P9 | First provenance-enabled release candidate, labelled beta, after the security gate passes and the owner signs off. |
| P10 onward | Regular releases through the Changesets flow. |

## Supported versions and deprecation

Before the public beta, the project will publish which release lines receive
security fixes and for how long, in `SECURITY.md`. Vulnerable versions are
handled with GitHub Security Advisories and `npm deprecate` messages that point
to the fixed version.
