# ADR-011: Password hashing

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision). This is a new ADR. The P1 agent prompt asks for a password-hashing ADR, and the Blueprint's decision register has no entry for it.
- **Date:** 2026-10-23
- **Deciders:** project owner; independent security reviewer
- **Phase:** P1 (decision), P4 (implementation)
- **Related:** Master Blueprint §§4 and 5; [passwords threat model](../threat-model/passwords.md) PWD-T03, T04, T06, T17, T18; ADR-006 (verifier storage and secret rotation, open)

## Context

Blueprint §5 requires an established Argon2id implementation with
configurable, benchmarked parameters, safe migration of legacy hashes, and no
plaintext logging. The algorithm is therefore settled. This ADR decides:
1. the implementation,
2. the parameters,
3. pepper, normalization and length rules,
4. rehashing and migration,
5. denial-of-service controls.

## Options: implementation

### Option A: `argon2`, the widely used native binding (node-argon2)

- **Advantages:** mature and widely deployed. Produces PHC strings. Has a rehash check. Supports the Argon2 secret input, which can carry a pepper.
- **Disadvantages:** a native addon. It relies on prebuilt binaries per platform and falls back to a compile toolchain, which is supply-chain and installation risk.
- **Security consequences:** the reference C implementation runs off the event loop.
- **Migration cost:** low. The PHC format is portable between implementations.

### Option B: `@node-rs/argon2`, a Rust-based binding

- **Advantages:** prebuilt binaries for each platform, with no node-gyp fallback.
- **Disadvantages:** a different implementation from the reference C code. Feature parity, such as the secret input and rehash helpers, must be confirmed.
- **Security consequences:** comparable. The review must check how it is maintained.
- **Migration cost:** low, through the PHC format.

### Option C: the Node.js built-in Argon2 API

- **Advantages:** no third-party native dependency.
- **Disadvantages:** introduced in Node.js v24.7.0 with Stability 1.2 (release candidate), returning raw derived bytes rather than PHC strings ([versioned documentation](https://nodejs.org/download/release/v24.7.0/docs/api/crypto.html#cryptoargon2algorithm-parameters-callback)). Stability must be rechecked against the eventual ADR-008 runtime matrix; the API is absent from older runtimes and needs reviewed PHC handling.
- **Security consequences:** promising once stable. Writing PHC handling adds code to review.
- **Migration cost:** low.

### Option D: scrypt or bcrypt

Rejected. The Blueprint mandates Argon2id. bcrypt also truncates input at 72
bytes.

## Decision

Pending owner decision.

## Rationale (agent recommendation)

### Implementation

Option A for V1, behind an internal `PasswordHasher` interface, so that
Option C can replace it once it is stable across the supported Node.js range
(ADR-008). Pin the version and check prebuilt-binary provenance in CI (XC-T06).

### Parameters

| Setting | Proposal | Source |
| --- | --- | --- |
| Algorithm | Argon2id, version 0x13 | Blueprint; RFC 9106 |
| Hard floor (refused below) | m = 19 MiB, t = 2, p = 1 | OWASP Password Storage Cheat Sheet minimum for Argon2id |
| Default | Tuned by benchmark on the target host, at or above the floor. RFC 9106's second recommended option (m = 64 MiB, t = 3, p = 4) is the suggested starting point where memory allows. | RFC 9106 §4 |
| Salt | 16 bytes from the CSPRNG, per hash | RFC 9106 |
| Output | 32 bytes, stored as a PHC string with its parameters | RFC 9106; PHC format |

A benchmark command (P4) measures latency and memory on the developer's
hardware and recommends parameters for a target latency the developer
chooses. Published guidance states the hardware assumptions (Blueprint §14).

### Rehash

On successful sign-in, if the stored parameters are below the current
configuration, rehash in the same transaction (PWD-T04, TC-PWD-09).

### Pepper

Optional and **off by default**. When it is enabled, the pepper is passed
through Argon2's secret input, a standard part of Argon2, rather than through
a custom pre-hash. It is held outside the database (TB-11), and a key ID is
stored next to each hash so it can be rotated at sign-in. The documentation
must state that losing the pepper makes every password unverifiable. Key
custody rules are in ADR-006.

### Password policy

This follows NIST SP 800-63B (Revision 4) guidance:
- A length-based minimum: 15 Unicode code points for single-factor use; a minimum of 8 is permitted only when the password is always part of multi-factor authentication. A second knowledge secret does not qualify as another factor.
- No composition rules and no forced periodic changes.
- A blocklist check against common and compromised passwords, using a bundled list plus an optional developer hook for an external breach service. No Authlume-operated service is involved.
- At least 64 characters are accepted.

### Normalization

Apply Unicode NFC before hashing, following the final
[NIST SP 800-63B-4 password-verifier guidance](https://pages.nist.gov/800-63-4/sp800-63b.html#passwordver).
Earlier drafts recommended NFKC or NFKD; the final revision recommends NFC.
Count Unicode code points rather than JavaScript UTF-16 code units for length
validation. Record the normalization form with the hash and retain the legacy
form when verifying imported credentials (PWD-T18). TC-PWD-24 must cover
canonical equivalents, compatibility characters that NFC leaves distinct,
and supplementary-plane characters at the length boundary.

### Length cap

Reject passwords longer than 1,024 UTF-8 bytes after normalization, **before**
hashing (PWD-T06). This always allows 64 or more characters.

### Denial-of-service controls

- Throttling happens before hashing ([ADR-007](0007-transaction-and-rate-limit-backing-store.md)).
- A per-process concurrency limit on hash operations. The default is tied to the libuv thread-pool size, and saturation returns a retryable "busy" error.
- Documented memory sizing: memory parameter × concurrency (TC-PWD-11).

### Enumeration

A dummy Argon2id hash with the current parameters is computed at startup and
verified on the unknown-account path (PWD-T03).

### Legacy migration

At import, wrap each legacy hash with Argon2id, applied to the legacy hash
output together with its original salt and parameters. The database then
never holds a weak-only hash. The first successful sign-in replaces the
wrapped hash with a native Argon2id hash (PWD-T17). The supported legacy
schemes are listed in P4.

### Hygiene

- Passwords are carried in a redacting secret type (XC-T03).
- They are never logged.
- They are compared only through the library's verify function, which runs in constant time.

## Security consequences

Not reviewed. This addresses PWD-T03, T04, T06, T17 and T18. RR-PWD-01 remains:
weak passwords are still crackable offline, only at a higher cost.

## Migration cost

Low, because PHC strings carry their parameters. Changing the normalization
form later requires the recorded version and a rehash at sign-in.

## Follow-up

- Owner: decide the implementation, the parameter floor and default approach, whether to offer a pepper, the minimum length after confirming NIST Revision 4, and the normalization form.
- Confirm the status of Node.js's built-in Argon2 API.
- P4: implement the benchmark command and TC-PWD-08, 09, 11, 22, 23 and 24.
