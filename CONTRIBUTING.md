# Contributing to Authlume

Thank you for your interest in Authlume. This guide explains how to take part
and what we expect from contributions to a security-sensitive project.

Everyone who takes part agrees to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Where the project is today

Authlume is in **Phase P0 (governance)**. The next phase, P1, produces the
threat model and API contracts. No source code or build tooling exists yet;
the monorepo arrives in Phase P2, after P1 is approved.

The most valuable contributions right now are:

- Reviewing the [Master Blueprint](AUTHLUME_MASTER_BLUEPRINT.md), especially the
  threat model and security invariants in section 5.
- Commenting on open [architecture decisions](docs/adr/README.md).
- Improving documentation: clarity, accuracy, accessibility and typos.

**We are not yet accepting code contributions.** Pull requests that add
packages, build tooling or authentication logic before the P1 contracts are
approved will be closed, with thanks, and pointed to the relevant ADR.

## Ground rules

- **The Master Blueprint is the source of truth.** Changes to scope,
  architecture, package boundaries or security invariants need an ADR approved
  by the project owner before implementation. See the
  [ADR process](docs/adr/README.md).
- **Never report vulnerabilities in public.** Use the private process in
  [SECURITY.md](SECURITY.md).
- **No unverified claims.** Do not describe anything as secure, audited,
  production-ready or compliant without evidence that a reviewer can check.
- **No custom cryptography.** Use audited runtime primitives and established,
  maintained libraries.
- **Small, reviewable changes.** One concern per pull request.

## Opening an issue

Use the issue forms when you open a new issue:

- **Bug report**: something in the repository is wrong or broken.
- **Proposal**: a new capability or a change to a design decision. Larger
  proposals become ADRs.
- **Documentation**: something is missing, unclear or incorrect.

Search existing issues first. Questions about using Authlume will be easier to
answer once there is something to use; until then, open a documentation issue.

## Pull request process

1. For anything beyond a small documentation fix, open an issue first so the
   approach can be agreed before you invest time.
2. Fork the repository and create a branch from `main`.
3. Make your change. Keep the diff focused.
4. Fill in every section of the pull request template, including the security
   checklist.
5. A code owner reviews every pull request. Changes that touch
   authentication, cryptography, recovery, sessions or access codes also
   require an **independent human security reviewer**. Passing tests alone are
   never enough to approve a security-sensitive change.

Implementation pull requests (from Phase P2 onward) must meet the
[Definition of Done](AUTHLUME_PHASE_TRACKER.md#definition-of-done-all-implementation-prs)
in the Phase Tracker.

### Commit messages

Write clear commit messages in the imperative mood ("Add security policy", not
"Added security policy"). Short type prefixes such as `docs:`, `fix:` or
`chore:` are encouraged.

### AI-assisted contributions

You may use AI tools, but you are responsible for every line you submit. Say in
the pull request when AI assistance was used. AI-generated changes go through
exactly the same review, and security-critical AI-generated code requires
independent human security review like any other code.

## Licensing of contributions

Authlume is licensed under the [Apache License 2.0](LICENSE). Under section 5 of
that license, any contribution you intentionally submit for inclusion is
provided under the same license, unless you explicitly state otherwise.

The project has not yet decided whether to require a Developer Certificate of
Origin (DCO) sign-off on commits. That decision is tracked in
[ADR-009](docs/adr/0009-repository-and-release-process.md). If it is adopted,
this section will explain how to sign off.

Any dependency proposed for a future package must use a license compatible with
Apache 2.0.

## Development setup

There is nothing to build yet. Phase P2 will add a pnpm workspace, a pinned
toolchain and a documented supported Node.js LTS range, and this section will
describe how to install, test and build locally.

## Getting help

Open a documentation issue if anything in this guide is unclear.
