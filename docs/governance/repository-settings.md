# GitHub organization and repository settings

These are **recommendations** for the project owner. Changing organization and
repository settings requires owner approval. With that approval, an AI agent
created the repository and made it public on 2026-10-23. It has applied none of
the other settings below. Settings marked "observed" were read from the GitHub
API on 2026-10-23 (see the [P0 evidence log](p0-verification.md)).

## Plan constraint: public or private

The `authlume` organization is on the GitHub **Free** plan (observed). On that
plan:

- Branch protection and rulesets are enforced only on **public** repositories.
- Private vulnerability reporting is available only on **public** repositories.
- Secret scanning and push protection are free for **public** repositories.

The repository was created private, then made **public** on 2026-10-23 after
the owner registered `authlume.org`. All the controls below are therefore
available on the Free plan. They take effect only once they are turned on.
Observed at 2026-10-23T09:11Z: private vulnerability reporting off; secret
scanning, push protection and Dependabot security updates disabled; no
rulesets.

## Organization settings

| Setting | Observed | Recommended |
| --- | --- | --- |
| Require two-factor authentication for all members | Off | **On** (Blueprint section 10) |
| Base repository permission | Read | Read |
| Members can delete or transfer repositories | Yes | No (owners only) |
| Members can change repository visibility | Yes | No (owners only) |
| Members can invite outside collaborators | Yes | No (owners only) |
| Dependency graph for new repositories | Off | On |
| Dependabot alerts for new repositories | Off | On |
| Dependabot security updates for new repositories | Off | On |
| Secret scanning for new repositories | Off | On |
| Secret scanning push protection for new repositories | Off | On |
| Verified domain | None | Verify `authlume.org` (registered 2026-10-23) with a DNS TXT record |
| Organization profile (description, URL) | Empty | Add a description, and the project URL once a site is live |
| Owners | 1 | Add a second trusted owner when one exists |

## Repository: `authlume/authlume`

### General

- **Description:** "Self-hosted, privacy-first authentication toolkit for
  TypeScript. Pre-release: not for production use."
- **Topics:** `authentication`, `typescript`, `self-hosted`, `privacy`,
  `passkeys`, `webauthn`, `nodejs`, `nextjs`, `react`, `postgresql`.
- **Features:** Issues on. Wiki off (documentation lives in the repository).
  Discussions are optional; turn them on if the owner wants RFC discussion
  outside issues. Projects are optional.
- **Pull requests:** allow squash merging (default message: pull request title
  and description). Turn off merge commits and rebase merging for a linear
  history. Always suggest updating pull request branches. Automatically delete
  head branches.

### Branch and tag rulesets

Ruleset for the default branch `main`:

- Require a pull request before merging.
  - Required approvals: 1.
  - Require review from code owners.
  - Dismiss stale approvals when new commits are pushed.
  - Require approval of the most recent push by someone other than its author.
  - Require conversation resolution.
- Block force pushes.
- Restrict deletions.
- Require linear history.
- Require status checks to pass. Add them in P2 once CI exists: format, lint,
  typecheck, tests, build, package boundary tests and dependency scans.
- Bypass list: empty. Leave the bypass list empty, including for the owner,
  except for a documented emergency procedure.
- Commit sign-off or signing: as decided in
  [ADR-009](../adr/0009-repository-and-release-process.md).

Ruleset for release tags (from P9): restrict creation, update and deletion of
`@authlume/*` and `v*` tags to the release workflow and owners.

### Security

- Private vulnerability reporting: **on**. `SECURITY.md` depends on it.
- Dependency graph, Dependabot alerts and Dependabot security updates: on.
- Dependabot version updates: add `.github/dependabot.yml` in P2, when there
  are dependencies and workflows to update.
- Secret scanning and push protection: on.
- Code scanning (CodeQL default setup): on from P2, once there is code.

### GitHub Actions (from P2)

- Allow only actions created by GitHub and an explicit allowlist of verified
  actions, pinned to full commit SHAs.
- Default `GITHUB_TOKEN` permissions: read-only. Grant extra permissions per
  job.
- Do not allow GitHub Actions to create or approve pull requests.
- Require approval to run workflows for first-time contributors, and for all
  outside contributors on pull requests from forks.
- Never run untrusted pull request code with `pull_request_target` and secrets.

### Environments (from P9)

- `npm-publish`: deployment branches restricted to `main`, the project owner as
  required reviewer, and no long-lived npm token. Publishing uses npm trusted
  publishing (OIDC) with provenance. See the
  [npm publishing strategy](../strategy/npm-publishing.md).

### Labels

GitHub's default labels `bug`, `enhancement` and `documentation` are used by
the issue forms. Also recommended: `needs-triage`, `security`, `adr`,
`good first issue` (once there is code) and one `phase: Pn` label per phase.

## Checklist for the owner

- [x] Create `authlume/authlume` and make it public (done 2026-10-23 with
      owner authorization).
- [ ] Verify `authlume.org` on the organization.
- [ ] Turn on the organization 2FA requirement.
- [ ] Restrict repository deletion, visibility changes and outside
      collaborator invitations to owners.
- [ ] Turn on security features for new repositories.
- [ ] Turn on private vulnerability reporting on the repository.
- [ ] Create the `main` ruleset.
- [ ] Set the description and topics.
- [ ] Record each completed item, with evidence, in the Phase Tracker.
