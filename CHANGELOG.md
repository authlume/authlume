# Changelog

This file records notable repository-level changes. Once packages exist
(Phase P2 onward), each published `@authlume/*` package will keep its own
changelog, generated from changesets, and follow
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- Planning documents: Master Blueprint, Phase Tracker and Agent Prompts (v0.1).
- Apache License 2.0 and NOTICE.
- Project README with a development-status disclaimer.
- Security policy, contribution guide and Code of Conduct.
- GitHub issue forms, pull request template and CODEOWNERS.
- Documentation structure: ADR register, governance and strategy documents.
- Draft P1 threat model for passwords, passkeys, recovery, sessions and both
  access-code modes (not yet reviewed).
- Proposed ADRs: ADR-001 (package split), ADR-002 (sessions and cookies),
  ADR-007 (rate-limit store), and new ADR-011 (password hashing), ADR-012
  (challenge storage) and ADR-013 (token rotation).

### Fixed

- P1 draft specifications: invalidate pre-verification password credentials on
  account claim, persist failed-attempt budgets across authentication rollback,
  separate WebAuthn challenge burning from dependent state transactions, and
  recheck expiry with actual database time.
- Correct final NIST password normalization guidance to NFC, distinguish
  browser runtime guards from build rejection, and replace unsupported
  access-code cracking timings with explicit throughput assumptions.
- Align public phase status, proposed ADR choices and access-code context/TTL
  requirements with the P1 drafts. All ADRs remain Proposed and not reviewed.

No packages have been released.
