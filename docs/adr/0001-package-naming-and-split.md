# ADR-001: Package naming and split

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision)
- **Date:** 2026-10-23
- **Deciders:** project owner; independent security reviewer for the server and browser boundary
- **Phase:** P1 (decision), P2 (implementation)
- **Related:** Master Blueprint §§6 and 8; [threat model](../threat-model/README.md) INV-09, XC-T04, SES-T15; [npm publishing strategy](../strategy/npm-publishing.md); ADR-008 (open)

## Context

Blueprint §6 proposes seven packages: `core`, `server`, `postgres`, `node`,
`next`, `react` and `access-code`. It requires:
- No circular imports.
- Server code never in client bundles.
- Publishing only once a package has stable entrypoints, tests and release checks.

Blueprint §8 requires browser initiation and server verification to be
distinct. The threat model makes **INV-09** (server-only packages cannot be
bundled for the browser) a security invariant, because a mistake here exposes
server secrets (XC-T04).

## Options

### Option A: the Blueprint split, with enforced boundaries

Seven packages under `@authlume`, with explicit dependency rules and
build-time guards.

- **Advantages:** small surface per package. The opt-in `access-code` module can be dropped or added without touching the others. Server and browser code are separated physically, not only by convention.
- **Disadvantages:** seven version lines and a compatibility matrix to maintain.
- **Security consequences:** the strongest separation. Each server package can refuse browser bundling on its own.
- **Migration cost:** none now. Later merges are possible but break import paths.

### Option B: one package with subpath exports

A single `authlume` package with `authlume/server`, `authlume/react` and so on.

- **Advantages:** one version, simple installation.
- **Disadvantages:** every user installs every dependency, including native Argon2 and the PostgreSQL driver, even for browser-only use. It cannot be versioned independently, which the founder's brief requires.
- **Security consequences:** a weaker boundary. A wrong import of a server subpath from a client file is easier to make, and harder to block across all bundlers.
- **Migration cost:** splitting later breaks every import.

### Option C: a finer split

For example separate `@authlume/passwords`, `@authlume/passkeys` and `@authlume/sessions`.

- **Advantages:** users install only the flows they use.
- **Disadvantages:** more packages and more cross-package contracts before the API is stable. Higher risk of version-skew bugs between flows that share session state.
- **Security consequences:** neutral to slightly negative, because there are more seams to secure.
- **Migration cost:** high if merged later.

## Decision

Pending owner decision.

## Rationale (agent recommendation: Option A)

Option A is what the Blueprint proposes, it meets the requirement for
independent versioning, and it gives the strongest technical enforcement of
INV-09. The proposed rules:

**Dependency graph** (an arrow means "may depend on"; nothing else is allowed):

```text
core        <- (nothing inside @authlume)
server      -> core
postgres    -> core, server (contracts only)
node        -> core, server
next        -> core, server
access-code -> core, server
react       -> core (type-only imports)
```

**Boundary enforcement:**

1. `core` has no database, Node-only or browser-only APIs, holds no secrets, and has minimal or no runtime dependencies.
2. Every server-only package (`server`, `postgres`, `node`, `next`, `access-code`) maps the `browser` export condition to a stub that throws at import time ("this package must not be bundled for the browser"). Bundlers that honour the `browser` condition then fail loudly.
3. `next` additionally uses the Next.js server-only marker convention, so importing it from a client component fails the build.
4. `react` has no runtime dependency on any server package. WebAuthn browser calls can use a maintained browser helper library, chosen in ADR-003, instead of a new `@authlume/browser` package. V1 adds no eighth package.
5. Configuration types are split into server configuration, which may hold secrets, and public configuration. Only public configuration types are exported from `react`.
6. CI from P2 onward:
   - an import-graph check (no cycles, no forbidden edges),
   - bundle tests with at least two bundlers, which must fail when a server package is imported from client code,
   - a production-bundle scan for server modules and sentinel secrets (TC-XC-04, TC-SES-18).
7. Packages are ESM-first. Whether to also ship CommonJS, and the supported Node.js range, are decided in ADR-008.

**Naming:** the npm name equals `@authlume/` plus the directory name.
`access-code` stays singular to match the Blueprint.

## Security consequences

Not reviewed. With Option A, a developer who imports `@authlume/server` into
client code gets a build-time failure in common bundlers rather than a silent
leak. The residual risk is bundlers that ignore the `browser` condition, which
the CI bundle matrix and documentation must cover.

## Migration cost

None yet. After the first release, renaming or merging packages is a breaking
change for every importer.

## Follow-up

- Owner: decide between options A, B and C.
- P2: implement the export maps, the browser stubs, the import-graph check and the bundle tests.
- ADR-003: choose the WebAuthn server library and any browser helper library.
- ADR-008: decide the Node.js range and whether to ship CommonJS.
