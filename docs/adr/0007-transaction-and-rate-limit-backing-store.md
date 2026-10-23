# ADR-007: Transaction and rate-limit backing store

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision)
- **Date:** 2026-10-23
- **Deciders:** project owner; independent security reviewer
- **Phase:** P1 (decision), P3 to P6 (implementation)
- **Related:** Master Blueprint §§5 and 7; [threat model](../threat-model/README.md) INV-04, INV-08, XC-T01, XC-T02; [ADR-012](0012-one-time-challenge-storage.md)

## Context

Blueprint §5 requires:
- rate limits per account, network and risk signal;
- exponential backoff where suitable;
- distributed, shared storage for counters;
- atomic single-use operations.

Blueprint §7 requires database constraints and atomic transactions, not
check-then-insert. PostgreSQL is the only V1 database. Throttling that lives
in process memory is bypassed as soon as there is more than one instance
(U-08, CODE-T08).

This ADR decides:
1. where counters live,
2. the transaction semantics for single-use operations,
3. the throttling policy model.

## Options for counter storage

### Option A: PostgreSQL, the same database as authentication data

- **Advantages:** no new infrastructure, because V1 already requires PostgreSQL. Counters and authentication state fail together, which avoids fail-open splits. Atomic `INSERT ... ON CONFLICT ... DO UPDATE` is available.
- **Disadvantages:** write load on the primary under attack, and table bloat that needs cleanup.
- **Security consequences:** consistent and shared across instances. Under a flood, the database becomes the bottleneck, which XC-T02 and capacity guidance must address.
- **Migration cost:** none.

### Option B: Redis or Valkey

- **Advantages:** fast, native TTLs, atomic increments.
- **Disadvantages:** extra infrastructure for every user. A new package and adapter (ADR-001 amendment). Separate availability and fail-open versus fail-closed choices.
- **Security consequences:** strong when it is available. If it is down while PostgreSQL is up, Authlume must choose between denying service and allowing unthrottled attempts.
- **Migration cost:** medium.

### Option C: in-process memory

- **Advantages:** zero setup.
- **Disadvantages:** not shared across instances, and lost on restart.
- **Security consequences:** unsafe in production (U-08).
- **Migration cost:** none.

### Option D: a pluggable interface, with Option A as the default implementation

- **Advantages:** V1 ships with one supported store. Others can be added later without changing callers.
- **Disadvantages:** the interface must be designed carefully so it guarantees atomicity.
- **Security consequences:** the same as A for V1.
- **Migration cost:** low.

## Decision

Pending owner decision.

## Rationale (agent recommendation: Option D, with PostgreSQL as the only V1 production store)

### Interface

`RateLimitStore` is defined in `@authlume/core`. Its one atomic operation is
"consume k units for key K under policy P, and return allowed, retry-after
and remaining". The PostgreSQL implementation lives in `@authlume/postgres`.

An in-memory implementation exists **only** for tests and single-process
development. Configuration validation refuses it unless an explicit
`singleInstance: true` acknowledgement is given, and logs a warning (XC-T05).

### Algorithm

A single-row-per-key algorithm, so each check is one atomic statement. The
candidates are GCRA, a token bucket stored as one timestamp, or an approximate
sliding window. The final choice follows P3 benchmarks.

### Keys

| Dimension | Notes |
| --- | --- |
| Normalized account identifier | Stored as HMAC(key, identifier), so counter tables do not duplicate email addresses |
| Source address | Taken from trusted proxy configuration only (XC-T01). IPv4 per address; IPv6 per prefix, default /64 |
| Endpoint, globally | A circuit breaker for floods |
| Transaction or challenge | Attempt counts live on the challenge row ([ADR-012](0012-one-time-challenge-storage.md)), not in this store |

### Policy model

- Throttle with exponential backoff. There is no permanent lockout by default (XC-T02).
- An account-level limit applies across all sources, which stops distributed stuffing of one account.
- To protect victims from lockout DoS, an optional recognised-device allowance gives a browser with a valid signed device cookie from an earlier successful sign-in its own separate budget.
- All thresholds are configurable. Defaults are published with their reasoning in P4.

### Failure behaviour

Fail **closed**: if the counter statement fails, the attempt is refused with a
retryable error. With Option A, an unavailable store means authentication is
unavailable anyway.

Counter consumption commits before credential verification, in a separate
short transaction or autocommit statement. A rejected credential, application
exception or rollback of the authentication transaction must not refund the
attempt. This also applies to failed access-code attempts in ADR-012.
TC-PWD-03 must include repeated invalid credentials and injected failures
after counter consumption on two instances; the shared budget must still be
exhausted.

### Transaction semantics, for all stores and all single-use operations

- Single-use consumption is one conditional statement. For example:

  ```sql
  UPDATE ... SET consumed_at = clock_timestamp()
  WHERE id = $1 AND consumed_at IS NULL AND expires_at > clock_timestamp()
  RETURNING ...
  ```

  A row count of 0 means failure. Never `SELECT` and then `UPDATE`.
- Uniqueness, such as normalized email, credential ID and token hash, is enforced by unique constraints. Constraint violations map to generic errors.
- `READ COMMITTED` is enough for these patterns. Multi-step changes (for example resetting the password, revoking sessions and consuming the token) run in one transaction.
- Expiry comparisons use the database's `clock_timestamp()` (INV-11), not
  transaction-start `now()`. Recheck expiry after waiting for locks and before
  the protected state change, within a bounded transaction. PostgreSQL's
  [time-function semantics](https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-CURRENT)
  distinguish actual time from transaction-start time. TC-XC-10 must also hold
  a transaction or row lock across expiry and require rejection.

### Cleanup

A maintenance function deletes expired counter rows. It is documented for
scheduling by the developer.

## Security consequences

Not reviewed. This addresses INV-04, INV-08, XC-T01, XC-T02 and CODE-T08.
RR-XC-01 remains: a determined attacker can still degrade service for one
account.

## Migration cost

Low. Adding a Redis adapter later is additive.

## Follow-up

- Owner: decide the option, the fail-closed default and whether to include the recognised-device allowance.
- P3: choose the algorithm from benchmarks, and implement and test TC-XC-01, TC-XC-02 and TC-PWD-03.
- P4: publish the default thresholds.
