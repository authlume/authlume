# Architecture decision records

Authlume records significant decisions as architecture decision records (ADRs).
The [Master Blueprint](../../AUTHLUME_MASTER_BLUEPRINT.md) is the source of
truth; an ADR is the only way to change or refine it.

## When an ADR is required

Write an ADR before implementing any change to:

- product scope (what is in or out of V1),
- architecture or package boundaries,
- security invariants, cryptographic choices or secret handling,
- public API contracts,
- repository, release or licensing process.

If an implementation would conflict with the Blueprint, stop and propose an ADR.

## Process

1. Copy [`0000-template.md`](0000-template.md) to `NNNN-short-title.md`, using
   the number from the register below or the next free number.
2. Set the status to **Proposed** and open a pull request.
3. Record every option considered, the rationale, the **security
   consequences** and the **migration cost**, as the Blueprint requires.
4. The project owner approves or rejects the ADR. Security-relevant ADRs also
   need independent human security review.
5. On approval, set the status to **Accepted** with the date and approver, and
   update the Blueprint or Phase Tracker if the decision changes them.

Statuses: **Open** (identified, not yet drafted), **Proposed**, **Accepted**,
**Rejected**, **Superseded by ADR-NNNN**.

An AI agent may draft an ADR but can never accept one.

## Decision register

ADR-001 to ADR-010 come from section 15 of the Master Blueprint. ADR-011 to
ADR-013 are new: the P1 agent prompt asks for decisions on password hashing,
challenge storage and token rotation, which section 15 does not list. The
owner may want to add them to the Blueprint register.

| ADR | Decision | Needed by | Status |
| --- | --- | --- | --- |
| [ADR-001](0001-package-naming-and-split.md) | Package naming and split | P1 | Proposed |
| [ADR-002](0002-session-strategy-and-cookie-defaults.md) | Session strategy and cookie defaults | P1 | Proposed |
| ADR-003 | Passkey credential lifecycle | P1 | Open |
| ADR-004 | Email identity normalization and duplicate policy | P1 | Open |
| ADR-005 | Access-code policies, and whether long-lived numeric codes survive review | P1 | Open. The [threat model recommends removing Mode B](../threat-model/access-codes.md#recommendation-for-adr-005) |
| ADR-006 | Cryptographic verifier storage and secret (server key) rotation | P1 | Open |
| [ADR-007](0007-transaction-and-rate-limit-backing-store.md) | Transaction and rate-limit backing store | P1 | Proposed |
| ADR-008 | Framework support matrix | P1 | Open |
| [ADR-009](0009-repository-and-release-process.md) | Repository and release process | P0 to P2 | Proposed |
| [ADR-010](0010-legal-trademark-and-npm-scope.md) | Legal and trademark clearance, npm scope reservation | P0 | Proposed |
| [ADR-011](0011-password-hashing.md) | Password hashing (new) | P1 | Proposed |
| [ADR-012](0012-one-time-challenge-storage.md) | One-time challenge storage (new) | P1 | Proposed |
| [ADR-013](0013-token-rotation.md) | Token rotation and invalidation (new; server keys stay in ADR-006) | P1 | Proposed |
