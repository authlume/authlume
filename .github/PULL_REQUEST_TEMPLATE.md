## Summary

<!-- What does this change do, and why? -->

## Phase and related work

- Phase / task: <!-- e.g. P0, P1 threat model -->
- Issue: <!-- e.g. #123 -->
- ADRs affected: <!-- e.g. ADR-002, or "none" -->

## Type of change

- [ ] Documentation or planning documents
- [ ] Governance or repository configuration
- [ ] Architecture decision record (ADR)
- [ ] Implementation (Phase P2 or later)
- [ ] Security fix

## Public API changes

<!-- New or changed exports, configuration or behaviour. Write "none" if there are none. -->

## Evidence

<!-- Commands you ran and their actual results, test reports, screenshots or links. Do not summarise results you did not observe. -->

## Security checklist

- [ ] No secrets, credentials, tokens or real user data in code, logs, fixtures, tests or documentation.
- [ ] No custom cryptography; only audited runtime primitives or established libraries.
- [ ] No server-only code, configuration or secrets can reach a browser bundle.
- [ ] No claim of security, production readiness or compliance without linked evidence.
- [ ] This change touches authentication, cryptography, recovery, sessions or access codes.
      If checked, an independent human security reviewer is requested: <!-- @reviewer -->

## Definition of Done (implementation pull requests)

See the [Definition of Done](https://github.com/authlume/authlume/blob/main/AUTHLUME_PHASE_TRACKER.md#definition-of-done-all-implementation-prs) in the Phase Tracker.

- [ ] Scope and public API documented; ADR updated where necessary.
- [ ] Tests cover success, failure, replay, concurrency and abuse cases relevant to this change.
- [ ] Formatting, lint, typecheck, tests, build and security scanners pass.
- [ ] Docs, examples and changelog (changeset) updated where user-visible.

## AI assistance

- [ ] AI tools were used to produce part of this change. I have reviewed every line and take responsibility for it.

## Known limitations and follow-ups

<!-- Anything not done, not verified or deferred. -->
