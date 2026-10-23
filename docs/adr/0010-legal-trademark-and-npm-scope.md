# ADR-010: Legal and trademark clearance, npm scope reservation

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision)
- **Date:** 2026-10-23
- **Deciders:** project owner, with qualified legal advice for trademark matters
- **Phase:** P0
- **Related:** Master Blueprint sections 6, 11 and 16;
  [P0 verification evidence](../governance/p0-verification.md)

> This ADR is not legal advice. It records what has and has not been verified,
> and the decisions the owner needs to make.

## Context

Blueprint section 16 requires reserving `authlume.org`, the GitHub
organization and the npm scope, and performing a similarity trademark review.
Section 11 requires Apache-2.0 attribution and license-compatible dependencies.

Verified status on 2026-10-23 (details and commands in the
[evidence log](../governance/p0-verification.md)):

| Item | Status |
| --- | --- |
| GitHub organization `authlume` | Exists; `9jaDevo` is its only member and an active admin. Organization 2FA requirement is off. |
| npm organization `authlume` (`@authlume` scope) | Exists in the public registry with zero packages. Organization roles and 2FA could not be verified from the agent environment. |
| Domain `authlume.org` | Unregistered at the first check. **Registered** on 2026-10-23 at 09:08 UTC through Spaceship, Inc., expiring 2027-10-23. The founder confirmed ownership; registrant details are redacted in RDAP. |
| Trademark or similarity review | **Not performed.** No evidence of a search or legal opinion exists. |
| Copyright holder wording | Provisionally "The Authlume Authors" in `NOTICE`. |

## Open decisions

### 1. Trademark and similarity review

Nothing in this repository claims trademark clearance, and nothing should
until a review is documented. The owner should decide:

- Who performs the review: a qualified trademark professional (recommended
  before public launch), or a documented preliminary self-search first.
- Which jurisdictions to search, for example the founder's home jurisdiction,
  the United States, the European Union and the WIPO Global Brand Database.
- Which goods and services classes are relevant, for example software and
  software services.
- Whether to search for similar names in authentication and identity products,
  not only exact matches.

Record the outcome, date and reviewer here. Do not record the review as
complete without that evidence.

### 2. Domain registration

Done: the founder registered `authlume.org` on 2026-10-23, before the
repository was made public. Remaining owner actions:

- Turn on auto-renew. The registration is for one year and expires on
  2027-10-23. A lapsed domain can be taken over and used to impersonate the
  project.
- Verify the domain on the GitHub organization. This adds the verified-domain
  badge and gives DNS proof of control.
- Create security and conduct contact addresses and add them to `SECURITY.md`
  and `CODE_OF_CONDUCT.md`.
- Optionally, register defensive domains. This is a budget decision.

### 3. npm scope hardening

- Confirm from the founder's npm account that the founder holds the **owner**
  role: `npm org ls authlume`.
- Require 2FA for every npm account with publish rights.
- Add a second trusted owner when one exists, to avoid a single point of
  failure.
- Do not create long-lived automation tokens. Publishing will use CI with
  provenance (see [ADR-009](0009-repository-and-release-process.md)).
- The unscoped package name `authlume` is unclaimed. Decide whether it needs
  protecting. Check npm's package-name policies first, because no package is
  published during P0.

### 4. Copyright holder wording

| Option | Consequences |
| --- | --- |
| A. "The Authlume Authors" | Common for community projects; copyright stays with each contributor. |
| B. Founder's legal name | Clear single holder for existing work; less inclusive as contributors join. |
| C. A legal entity | Requires forming or naming an entity; useful if commercial offerings follow. |

Agent recommendation: **A** for the community project. Confirm with legal
advice if commercial modules are planned.

## Decision

Pending owner decision.

## Security consequences

A lapsed or unverified domain can be taken over and used for phishing that
impersonates the project. An npm scope with a single owner and no enforced 2FA
is a supply-chain risk. Both should be resolved before any package is
published.

## Migration cost

Low now. Renaming the project after a public launch would be expensive, which
is why the trademark review should happen before P10.

## Follow-up

- Owner: turn on auto-renew for `authlume.org` and verify it on the GitHub
  organization, run `npm org ls authlume` and confirm 2FA, then commission or
  perform the trademark review.
- Update the Phase Tracker with evidence for each item.
