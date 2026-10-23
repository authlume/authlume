# ADR-009: Repository and release process

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision)
- **Date:** 2026-10-23
- **Deciders:** project owner
- **Phase:** P0 (repository), P2 (tooling), P9 (first release)
- **Related:** Master Blueprint sections 6, 10 and 11;
  [npm publishing strategy](../strategy/npm-publishing.md);
  [repository settings](../governance/repository-settings.md)

## Context

The Blueprint already sets the direction:

- One TypeScript monorepo using pnpm workspaces (section 6).
- Changesets or an equivalent tool for versioning, Semantic Versioning and
  changelogs (sections 6 and 10).
- Signed or provenance-enabled npm publishing from CI, least-privilege publish
  tokens or OIDC, protected branches, CODEOWNERS, 2FA for maintainers, required
  reviews and automated dependency updates (section 10).
- A public RFC/ADR process and a "DCO or contributor policy" (section 11).

This ADR records those settled points and lists the choices that are still
open. It does not change the Blueprint.

## Settled by the Blueprint (recorded, not reopened)

1. **Single repository.** All V1 packages live in `authlume/authlume`. No
   separate repositories for core, server, postgres, node, next, react or
   access-code.
2. **Independently versionable packages** under the `@authlume` npm scope,
   with names and boundaries subject to ADR-001.
3. **Publishing only from CI**, with provenance, and never from a developer
   machine.

## Open decisions

### 1. Contributor sign-off

| Option | Summary | Consequences |
| --- | --- | --- |
| A. DCO sign-off (`git commit -s`) | Contributors certify their right to submit under the project license. | Low friction; widely used. Grants no right to relicense contributions, which matches the Blueprint's rule against relicensing community contributions. |
| B. CLA | Contributors sign an agreement, possibly granting broader rights. | Higher friction, needs legal drafting and signing tooling. Only useful if relicensing rights are wanted, which the Blueprint does not plan for. |
| C. No sign-off | Rely on Apache-2.0 section 5 (inbound = outbound). | Lowest friction, weakest provenance record. |

Agent recommendation: **Option A (DCO)**, enforced through GitHub's "require
contributors to sign off on web-based commits" setting and a DCO check in CI
from P2 onward.

### 2. Default branch protection

Agent recommendation: protect `main` with a repository ruleset as described in
the [repository settings](../governance/repository-settings.md#branch-and-tag-rulesets).
That means pull requests are required, a code-owner approval is required,
stale approvals are dismissed, force pushes and deletion are blocked, and the
required status checks are added once CI exists in P2.

Constraint to note: on GitHub's Free plan, branch protection and rulesets are
enforced only on **public** repositories. A private repository on the Free
plan cannot enforce these rules.

### 3. Signed commits and tags

| Option | Consequences |
| --- | --- |
| A. Require signed commits on `main` | Stronger authorship evidence. Adds contributor setup friction, and squash merges made through the GitHub web interface are signed by GitHub. |
| B. Require signed release tags only | Protects release points with less friction. |
| C. Neither, relying on provenance attestations | Package provenance already links each published artifact to a CI run and commit. |

Agent recommendation: **B** for now, then revisit at P9.

### 4. Version strategy within the monorepo

| Option | Consequences |
| --- | --- |
| A. Fully independent versions (Changesets default) | Matches the "independently versionable" goal; consumers must track a compatibility matrix. |
| B. Fixed (lock-step) versions | Simple compatibility story, but releases packages with no changes. |
| C. Independent, with `linked` groups for tightly coupled packages | Flexible middle ground; group membership decided after ADR-001. |

Agent recommendation: **A**, revisited once ADR-001 fixes the package split.

### 5. Release approval

Agent recommendation: publishing runs in a GitHub Actions environment (for
example `npm-publish`) restricted to `main`, with the project owner as a
required reviewer. No long-lived npm token is stored in the repository.

## Decision

Pending owner decision.

## Security consequences

Not reviewed. Relevant threats include malicious pull requests, compromised
maintainer accounts, compromised CI dependencies and publish-token theft.
Mitigations are the controls above plus 2FA on GitHub and npm. They need
security review before P9.

## Migration cost

Low while no packages exist. Changing the versioning or sign-off policy after
the first public release has a higher cost.

## Follow-up

- Owner decides items 1 to 5.
- P2: implement Changesets, the CI checks and the `npm publish --dry-run` gate.
- P9: configure trusted publishing and the release environment, then verify
  provenance on a release candidate.
