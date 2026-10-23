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

From section 15 of the Master Blueprint.

| ADR | Decision | Needed by | Status |
| --- | --- | --- | --- |
| ADR-001 | Package naming and split | P1 | Open |
| ADR-002 | Session strategy and cookie defaults | P1 | Open |
| ADR-003 | Passkey credential lifecycle | P1 | Open |
| ADR-004 | Email identity normalization and duplicate policy | P1 | Open |
| ADR-005 | Access-code policies, and whether long-lived numeric codes survive review | P1 | Open |
| ADR-006 | Cryptographic verifier storage and secret rotation | P1 | Open |
| ADR-007 | Transaction and rate-limit backing store | P1 | Open |
| ADR-008 | Framework support matrix | P1 | Open |
| [ADR-009](0009-repository-and-release-process.md) | Repository and release process | P0 to P2 | Proposed |
| [ADR-010](0010-legal-trademark-and-npm-scope.md) | Legal and trademark clearance, npm scope reservation | P0 | Proposed |
