# ADR-013: Token rotation and invalidation

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision). This is a new ADR. The P1 agent prompt asks for a token-rotation ADR. It covers **user-facing** tokens and codes only. Rotation of **server-held keys** (verifier keys, peppers) stays in ADR-006, "secret rotation".
- **Date:** 2026-10-23
- **Deciders:** project owner; independent security reviewer
- **Phase:** P1 (decision), P3 to P6 (implementation)
- **Related:** Master Blueprint §§4 and 5; [sessions threat model](../threat-model/sessions.md) SES-T05, T07, T09; PWD-T13, PWD-T15; PKY-T04; REC-T06, REC-T11; CODE-T18; [ADR-002](0002-session-strategy-and-cookie-defaults.md); [ADR-012](0012-one-time-challenge-storage.md)

## Context

Blueprint §5 requires secure rotation and invalidation of sessions. Blueprint
§4 requires recovery codes to be rotated after recovery. Stale tokens that
survive a security event are a common takeover path, for example sessions
that outlive a password reset (PWD-T13) or pre-hijacking (PWD-T15). This ADR
decides **when** user-facing tokens rotate, and **what each event
invalidates**.

## Options: session token rotation

### Option A: rotate on every request

- **Advantages:** a stolen token is useful only until the victim's next request.
- **Disadvantages:** a write on every request. Concurrent tabs and requests race constantly (SES-T09). Users get logged out spuriously.
- **Security consequences:** the theoretical benefit is mostly lost to the grace windows needed to make it usable.
- **Migration cost:** low.

### Option B: rotate on security events only

The events are: sign-in, reauthentication, credential change and recovery.

- **Advantages:** simple, with no races during normal use. Defeats session fixation (SES-T05).
- **Disadvantages:** a stolen cookie stays useful until it expires or is revoked.
- **Security consequences:** good, combined with server-side expiry and revocation.
- **Migration cost:** none.

### Option C: Option B plus periodic rotation, with a grace window and reuse detection

For example, rotate after 24 hours of use. The previous token stays valid for
a short grace window (for example 60 seconds). Presenting a superseded token
after the grace window revokes the whole session, because it signals that
someone else holds a copy (RR-SES-02).

- **Advantages:** turns cookie theft into a detectable event.
- **Disadvantages:** complexity. False positives from network retries and restored tabs.
- **Security consequences:** better than B against long-lived cookie theft, if false positives are tolerable.
- **Migration cost:** low. It can be enabled later.

### Option D: separate access and refresh tokens, as in OAuth

- **Advantages:** well understood for API clients.
- **Disadvantages:** unnecessary for first-party cookie sessions, and it adds a second secret to protect.
- **Security consequences:** neutral.
- **Migration cost:** high.

## Decision

Pending owner decision.

## Rationale (agent recommendation)

**Sessions:** Option B by default. Option C is available as an opt-in, to be
enabled as the default only if the P3 concurrency tests (TC-SES-13) show
acceptable false-positive rates. Every rotation is one transaction: insert the
new token hash, and mark the old one superseded, with a reference to its
successor.

### Invalidation matrix

Every row is atomic with the triggering change and emits an audit event
(INV-10). "Notify" means a message through the developer's configured
channel.

| Event | Sessions | One-time tokens and challenges | Recovery codes | Passkeys | Notify |
| --- | --- | --- | --- | --- | --- |
| Sign-in | New session, never promote a prior token | Consume the ceremony challenge | No change | Update the counter | Optional |
| Reauthentication (step-up) | Rotate the current session | Consume the challenge | No change | No change | No |
| Password changed | Revoke all others, rotate the current one | Invalidate reset and recovery tokens | No change | No change | Yes |
| Password reset completed | Revoke **all** | Invalidate all reset and recovery tokens | No change | Optional: revoke passkeys added in the last N hours | Yes |
| Email changed (after verifying the new address) | Optional: revoke others (proposed default: yes) | Invalidate verification, reset and recovery tokens for the old address | No change | No change | Yes, to both addresses |
| First proof of email ownership on a pre-registered account (PWD-T15) | Revoke **all** created before verification | Invalidate all | Invalidate any | Revoke any added before verification | Yes |
| Passkey added | No change | Consume the challenge | No change | Add | Yes |
| Passkey removed | Optional: revoke sessions created with it | No change | No change | Remove immediately | Yes |
| Recovery codes regenerated | No change | No change | Replace the whole set atomically | No change | Yes |
| Recovery completed (code or email) | Revoke **all** | Invalidate all | Invalidate the remaining codes and issue a new set (Blueprint §4) | Optional: revoke passkeys added in the last N hours | Yes |
| Access code (Mode A) redeemed or exhausted | New session on success | Transaction consumed or burned | No change | No change | Optional |
| Sign-out | Revoke the current one | No change | No change | No change | No |
| Sign out everywhere | Revoke all | Invalidate reauthentication challenges | No change | No change | Optional |
| Account disabled or deleted | Revoke all | Invalidate all | Invalidate all | Disable or delete | Developer hook |

### Mass revocation

A per-user `session_epoch` counter is incremented on "revoke all" events and
checked during validation. This makes revocation O(1) and atomic. Revoked rows
are still marked for the audit history.

### Recovery code rotation

The Blueprint chooses full-set rotation after recovery. This ADR records that
choice. When a code is used for recovery, the rest of the set is replaced.

### Access codes

Mode A codes never rotate. Each transaction has its own code. Mode B rotation
is moot if ADR-005 removes Mode B, which is the threat model's
recommendation.

### Future

Device-bound session credentials, where browsers support them, could replace
Option C's reuse detection. Tracked as a future option only.

## Security consequences

Not reviewed. This addresses SES-T05, T07 and T09, PWD-T13 and T15, PKY-T04,
REC-T06 and T11, and CODE-T18. RR-SES-02 remains: a copied cookie stays usable
until rotation, revocation or expiry.

## Migration cost

Low. The invalidation rules can be tightened in later minor versions, which
users may experience as extra sign-outs.

## Follow-up

- Owner: decide between options B and C for sessions, and confirm each optional cell in the matrix.
- ADR-006: server-key rotation.
- P3: implement the rotation transaction, `session_epoch` and TC-SES-13. P4 to P6: tests for each matrix row.
