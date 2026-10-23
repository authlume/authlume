# Ecosystem strategy

- **Status:** Strategy. V1 implements only the targets listed under "V1
  targets". Everything under "Future ecosystems" is deferred and will not be
  implemented, scaffolded or published during V1.
- **Source:** Master Blueprint sections 3, 4, 12 and 13, and the deferred rows
  (FUT1 to FUT4) of the Phase Tracker.

## V1 targets

| Area | V1 target | Notes |
| --- | --- | --- |
| Language | TypeScript and JavaScript | Strict TypeScript, ESM first |
| Runtime | Node.js | Supported LTS range defined at release (ADR-008) |
| Server framework | Next.js | Server adapters; supported versions in ADR-008 |
| Client | React | Optional headless hooks and helpers; no secrets |
| Database | PostgreSQL | Adapter, versioned migrations, transactional semantics |
| Distribution | npm, `@authlume` scope | See the [npm publishing strategy](npm-publishing.md) |
| Repository | One monorepo, `authlume/authlume` | See [ADR-009](../adr/0009-repository-and-release-process.md) |

V1 is about getting one stack secure, well documented and easy to adopt. It
does not try to cover many stacks at once.

## Future ecosystems (deferred)

Each item below needs evidence of demand, an available maintainer and its own
ADR before any work starts. None of them is promised.

| Possible future work | Tracker row | Gate before starting |
| --- | --- | --- |
| UI kit: optional accessible login components | FUT1 | V1 beta complete; accessible and framework-tested components |
| PHP SDK distributed through Composer | FUT2 | Demand validated and maintainers available |
| MySQL and SQLite adapters | FUT2 | Demand validated; adapter conformance tests |
| More framework integrations | FUT2 | Demand validated and maintainers available |
| Social authentication (OAuth providers) | FUT2 | Demand validated; dedicated threat model |
| Federation and a self-hosted OIDC provider | FUT3 | Mature core; protocol conformance and independent audit |
| Optional commercial modules, support and hosting | FUT4 | Willingness to pay and operating model validated |

### Principles for future ecosystems

- **Same security invariants.** Any new language, database or integration must
  meet the invariants in Blueprint section 5 and pass the same categories of
  security tests.
- **Shared contracts, not shared code.** The V1 database schema, error taxonomy
  and policy contracts define what other SDKs must interoperate with. A
  cross-language design needs its own ADR.
- **Repository placement is a future decision.** Packagist reads a package's
  `composer.json` from the root of its repository. A PHP SDK would therefore
  need its own repository or a read-only split from this one. That choice
  belongs to a future ADR. V1 creates no extra repositories.
- **Name reservations are tracked, not assumed.** On 2026-10-23 Packagist
  listed no packages under the `authlume` vendor name. Packagist assigns a
  vendor name when the first package under it is published, so the name is
  not reserved. Names in other registries have not been checked. Reserve them
  only when an ecosystem is approved, and record the evidence in the Phase
  Tracker.
- **Federation is not marketed in V1.** The Blueprint states that federation
  must not be advertised until it exists and has been audited.

## Commercial boundaries

From Blueprint sections 11 and 12:

- The core stays Apache-2.0. Essential security controls and fixes stay free.
- Premium offerings may use separate proprietary licenses, but community
  contributions are not relicensed without the appropriate rights.
- No pricing, hosted service or enterprise features are promised before they
  are validated.
