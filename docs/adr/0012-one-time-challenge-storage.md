# ADR-012: One-time challenge storage

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision). This is a new ADR. The P1 agent prompt asks for a challenge-storage ADR, and the Blueprint's decision register has no entry for it.
- **Date:** 2026-10-23
- **Deciders:** project owner; independent security reviewer
- **Phase:** P1 (decision), P3 to P6 (implementation)
- **Related:** Master Blueprint §7 (`one_time_challenges`); [threat model](../threat-model/README.md) INV-03, INV-04, INV-11; PKY-T01, PKY-T03; PWD-T08 to T10; REC-T09; CODE-T01 to T05; ADR-006 (verifier construction, open); [ADR-007](0007-transaction-and-rate-limit-backing-store.md)

## Context

Several flows issue something that must be **fresh, single-use, short-lived
and bound** to its purpose:

- WebAuthn registration and authentication challenges.
- Email verification tokens.
- Password reset tokens.
- Email recovery tokens.
- Mode A access-code transactions.
- Reauthentication (step-up) challenges.

The Blueprint's data model provides
`one_time_challenges(id, purpose, user_id?, verifier, expires_at, consumed_at)`.

## Options

### Option A: one server-side table, with purpose, bindings and atomic consumption

- **Advantages:** single use is enforced by the database (INV-04). The same semantics apply to every flow, and the table is easy to audit.
- **Disadvantages:** a write for every ceremony, and the table needs cleanup.
- **Security consequences:** the strongest guarantees against replay and races.
- **Migration cost:** none.

### Option B: stateless signed challenges, for example an HMAC-signed value with an expiry, held by the client

- **Advantages:** no database writes.
- **Disadvantages:** single use cannot be enforced without server state, so a challenge can be replayed until it expires (PKY-T01). Signing keys become critical.
- **Security consequences:** unacceptable for authentication ceremonies and one-time tokens.
- **Migration cost:** high to reverse.

### Option C: store challenges inside a pre-authentication session record

- **Advantages:** naturally bound to one browser.
- **Disadvantages:** needs pre-authentication session infrastructure, and email tokens opened on another device do not fit.
- **Security consequences:** good for browser ceremonies, but does not work for email flows.
- **Migration cost:** medium.

### Option D: a cache such as Redis, with TTLs

- **Advantages:** fast, with automatic expiry.
- **Disadvantages:** extra infrastructure, and single-use atomicity depends on careful scripting.
- **Security consequences:** acceptable only with atomic get-and-delete. Not a V1 store under [ADR-007](0007-transaction-and-rate-limit-backing-store.md).
- **Migration cost:** medium.

## Decision

Pending owner decision.

## Rationale (agent recommendation: Option A, plus a browser-context binding borrowed from Option C)

### Proposed columns

These extend the Blueprint model. Final schema design is in P3.

| Column | Purpose |
| --- | --- |
| `id` | A random public handle, used as the transaction ID |
| `purpose` | An enum: `webauthn_registration`, `webauthn_authentication`, `email_verification`, `password_reset`, `email_recovery`, `access_code`, `reauthentication` |
| `user_id` | Nullable. Null only for usernameless WebAuthn authentication |
| `verifier` | The hash of the secret value, or a keyed verifier for low-entropy codes (ADR-006) |
| `context_hash` | The hash of a random browser-context secret held in a short-lived `__Host-` cookie, for ceremonies that must finish in the browser that started them |
| `bound_value_hash` | The hash of the value the challenge is about, for example the email address being verified, so that changing the address invalidates the token |
| `attempts`, `max_attempts` | For codes |
| `created_at`, `expires_at`, `consumed_at` | Database-clock timestamps (INV-11) |

### Secret values

| Purpose | Secret | Stored as |
| --- | --- | --- |
| WebAuthn challenges | 32 random bytes | SHA-256 of the challenge. The WebAuthn library compares through a verification callback, so the raw value need not be stored. |
| Email tokens (verification, reset, recovery) | 32 random bytes, base64url in the link | SHA-256 of the token, which is also the lookup key, with a unique index |
| Access codes (Mode A) | A 12-digit code, plus the high-entropy transaction `id` | Keyed HMAC of the code (ADR-006). Lookup is by `id`, **never** by code. |

### Binding rules

| Purpose | Bound to |
| --- | --- |
| `webauthn_registration` | `user_id`, browser context |
| `webauthn_authentication` | Browser context, plus `user_id` if username-first |
| `email_verification` | `user_id`, the email address |
| `password_reset`, `email_recovery` | `user_id`. Not bound to a browser, because links may be opened on another device. |
| `access_code` | Browser context, plus `user_id` if known |
| `reauthentication` | `user_id`, session ID, browser context |

### Consumption

**Challenges and tokens: consume first, then verify.**

```sql
UPDATE one_time_challenges
SET consumed_at = now()
WHERE verifier = $1 AND purpose = $2 AND consumed_at IS NULL AND expires_at > now()
RETURNING ...
```

The row is consumed **before** the WebAuthn verification or the password
update runs. A failed verification therefore still burns the challenge, and
the dependent change runs in the same transaction.

**Codes: count the attempt atomically, then compare, then consume.**

```sql
UPDATE ... SET attempts = attempts + 1
WHERE id = $1 AND purpose = 'access_code' AND consumed_at IS NULL
  AND expires_at > now() AND attempts < max_attempts AND context_hash = $2
RETURNING verifier
```

Then compare in constant time. If the code matches, consume with
`UPDATE ... SET consumed_at = now() WHERE id = $1 AND consumed_at IS NULL`,
which must affect exactly one row.

### Proposed TTLs

These are for the owner to confirm. All are configurable within hard maximums.

| Purpose | Proposed TTL | Hard maximum |
| --- | --- | --- |
| WebAuthn ceremonies | 5 minutes (matches the ceremony `timeout`) | 10 minutes |
| Reauthentication | 5 minutes | 10 minutes |
| Access code (Mode A) | 10 minutes | 15 minutes |
| Password reset, email recovery | 30 minutes | 2 hours |
| Email verification | 24 hours | 7 days |

### Invalidation

- Issuing a new reset or recovery token invalidates any earlier outstanding token for the same user and purpose.
- Credential and email changes invalidate dependent tokens ([ADR-013](0013-token-rotation.md)).

### Cleanup

A maintenance function deletes rows that expired or were consumed more than a
retention period ago (default 7 days). Audit events keep the history.

## Security consequences

Not reviewed. This addresses replay, races, confusion between ceremonies and
contexts, guessing and offline exposure, for every one-time secret. It depends
on ADR-006 for the verifier construction of low-entropy codes.

## Migration cost

None before the first release. Adding purposes later is additive.

## Follow-up

- Owner: confirm option A, the binding rules and the TTLs.
- ADR-006: verifier construction and key handling.
- P3: schema and migrations. P4 to P6: tests TC-PWD-13 to 17, TC-PKY-03 to 06, TC-CODE-03 to 06.
