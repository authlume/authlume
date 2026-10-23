# Authlume documentation

Authlume has no user-facing documentation yet, because there is no code to
document. The documentation site, quickstart and API reference are Phase P8
deliverables. This directory currently holds the project's design and
governance records.

## Start here

- [Master Blueprint](../AUTHLUME_MASTER_BLUEPRINT.md): the source of truth for
  scope, security invariants and architecture.
- [Phase Tracker](../AUTHLUME_PHASE_TRACKER.md): phase status, exit criteria and
  evidence.
- [Agent Prompts](../AUTHLUME_AGENT_PROMPTS.md): rules for AI-assisted work.

## Contents

```text
docs/
├── README.md                     This index
├── adr/                          Architecture decision records
│   ├── README.md                 ADR process and decision register
│   ├── 0000-template.md          Template for new ADRs
│   ├── 0001-package-naming-and-split.md
│   ├── 0002-session-strategy-and-cookie-defaults.md
│   ├── 0007-transaction-and-rate-limit-backing-store.md
│   ├── 0009-repository-and-release-process.md
│   ├── 0010-legal-trademark-and-npm-scope.md
│   ├── 0011-password-hashing.md
│   ├── 0012-one-time-challenge-storage.md
│   └── 0013-token-rotation.md
├── governance/
│   ├── p0-verification.md        Evidence for Phase P0 account checks
│   └── repository-settings.md    Recommended GitHub organization and repository settings
├── strategy/
│   ├── ecosystem.md              V1 targets and deferred ecosystems
│   └── npm-publishing.md         Future npm package and release strategy
└── threat-model/                 P1 draft, not yet reviewed
    ├── README.md                 Scope, assets, attackers, trust boundaries, invariants, cross-cutting threats
    ├── passwords.md
    ├── passkeys.md
    ├── recovery.md
    ├── sessions.md
    └── access-codes.md           Both 12-digit code modes, and unsafe designs flagged for removal
```

## Planned additions

These are added in later phases and do not exist yet:

| Content | Phase |
| --- | --- |
| ADR-003 to ADR-006 and ADR-008 (passkeys, email identity, access codes, verifiers, support matrix) | P1 |
| API contracts: input and output schemas, error taxonomy | P1 |
| Contributor development setup | P2 |
| Quickstart, guides, API reference and troubleshooting | P8 |
| Security deployment checklist and known limitations | P8 to P9 |
