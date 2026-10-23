# Threat model: passkeys (WebAuthn)

> [!CAUTION]
> Draft v0.1, 2026-10-23. Drafted by an AI agent. **Not reviewed.** See the
> [threat model overview](README.md) for scope, identifiers and severity scale.

## Flows

Authlume uses a maintained, standards-based WebAuthn library for every
cryptographic and parsing step (Blueprint §5, INV-02). It never parses CBOR or
verifies signatures itself. The library is chosen in ADR-003 (open).

1. **Registration ceremony:**
   1. An authenticated user requests options. The server creates a challenge ([ADR-012](../adr/0012-one-time-challenge-storage.md)).
   2. The browser calls `navigator.credentials.create()`.
   3. The server verifies the challenge, type, origin, RP ID hash, flags and algorithm, and that the credential ID is unique.
   4. The server stores the credential ID, public key, counter, transports and user handle.
2. **Authentication ceremony:**
   1. The server creates a challenge. With discoverable credentials, there is no `allowCredentials` list.
   2. The browser calls `navigator.credentials.get()`.
   3. The server verifies the challenge, type, origin, RP ID hash, flags, signature and counter.
   4. The server issues a session ([sessions.md](sessions.md)).
3. **Lifecycle:** list credentials, rename, delete, and re-enroll.

## Assets

A-1, A-5, A-8, A-11 and A-14.

## Threats

### PKY-T01 Assertion or challenge replay
- **Attacker:** ATK-3, ATK-5. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:**
  - The server generates a random challenge. The WebAuthn specification recommends at least 16 bytes; Authlume uses 32.
  - The challenge is single-use, short-lived, bound to its purpose, and consumed atomically **before** verification, so a failed verification also burns it ([ADR-012](../adr/0012-one-time-challenge-storage.md)).
- **Tests:** TC-PKY-03, TC-PKY-04, TC-PKY-05.

### PKY-T02 Wrong origin or RP ID accepted
- **Attacker:** ATK-10, ATK-4 (sibling subdomain). **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** a page on `evil.example.com` can request assertions for RP ID `example.com`. If the server checks only the RP ID hash, it accepts them.
- **Mitigations:**
  - Configure an explicit allowlist of exact expected origins (scheme, host and port). Reject any `clientDataJSON.origin` outside it.
  - The RP ID comes from configuration, never from the `Host` header (INV-07).
  - Verify the RP ID hash.
  - Related-origin requests (`/.well-known/webauthn`) are not supported in V1 without an ADR.
- **Tests:** TC-PKY-01, TC-PKY-02, TC-PKY-18.

### PKY-T03 Ceremony confusion
- **Attacker:** ATK-1. **STRIDE:** Tampering. **Severity:** Medium.
- **Abuse case:** a registration response is submitted to the authentication endpoint, or the reverse, or a challenge issued for one user or browser is used by another.
- **Mitigations:**
  - Check that `clientDataJSON.type` is `webauthn.create` for registration and `webauthn.get` for authentication.
  - Bind each challenge to its purpose, to the user for registration, and to the initiating browser context ([ADR-012](../adr/0012-one-time-challenge-storage.md)).
- **Tests:** TC-PKY-05, TC-PKY-06, TC-PKY-07.

### PKY-T04 Backdoor passkey added through a stolen session
- **Attacker:** ATK-3, ATK-4. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Abuse case:** an attacker with a session registers their own authenticator and keeps access after the password is changed.
- **Mitigations:**
  - Adding a passkey requires recent reauthentication.
  - The account is notified, and an audit event is emitted.
  - Credential lists show creation time and last use.
  - Recovery and password reset can optionally revoke all passkeys added in the last N hours (policy, [ADR-013](../adr/0013-token-rotation.md)).
- **Tests:** TC-PKY-13.

### PKY-T05 Credential ID registered to two accounts
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Mitigations:** the WebAuthn specification says a relying party should fail a registration whose credential ID is already registered. Authlume always fails it, enforced by a database unique constraint on the credential ID.
- **Tests:** TC-PKY-09.

### PKY-T06 Cloned authenticator goes undetected
- **Attacker:** ATK-3. **STRIDE:** Spoofing. **Severity:** Medium.
- **Abuse case:** a cloned hardware key is used in parallel with the original.
- **Mitigations:**
  - Follow the WebAuthn rule: when either the stored or the received signature counter is non-zero, and the received value is not greater than the stored one, treat it as a possible clone. The configurable action is to reject or to flag, and to emit an audit event (ADR-003).
  - Authenticators that always report 0, which includes most synced passkeys, never trigger this check and must be accepted.
- **Residual:** RR-PKY-02. **Tests:** TC-PKY-10.

### PKY-T07 User verification not enforced
- **Attacker:** ATK-3 (physical access to an unlocked authenticator). **STRIDE:** Spoofing. **Severity:** Medium.
- **Mitigations:**
  - Request `userVerification: "required"` when a passkey is the sole factor, and verify the UV flag server-side. The browser option alone proves nothing.
  - Always verify the UP flag.
- **Tests:** TC-PKY-08.

### PKY-T08 Trust placed in unverified attestation
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Low.
- **Abuse case:** policy decisions rely on an AAGUID or authenticator model that is not attested.
- **Mitigations:**
  - V1 requests `attestation: "none"` for privacy.
  - The AAGUID is treated as an untrusted display hint only.
  - No claims are made about authenticator certification.
  - Attestation verification would need a new ADR.
- **Tests:** review item.

### PKY-T09 Weak or unexpected algorithms
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Low.
- **Mitigations:** offer an explicit `pubKeyCredParams` allowlist (for example ES256 and RS256, final list in ADR-003). Reject other algorithms at registration. Verify that the assertion algorithm matches the stored key.
- **Tests:** TC-PKY-17.

### PKY-T10 Account enumeration through credential lists
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** Medium.
- **Abuse case:** in a username-first flow, `allowCredentials` reveals whether an account exists and lists its credential IDs.
- **Mitigations:**
  - Prefer discoverable credentials with an empty `allowCredentials`.
  - If username-first is enabled, return a stable, deterministic decoy list for unknown users, derived with a server key, with the same shape and timing.
- **Tests:** TC-PKY-14.

### PKY-T11 Deleted credential still accepted
- **Attacker:** ATK-3. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:**
  - Look up the credential in the database at every verification, with no cache.
  - Deletion takes effect immediately and emits an audit event.
  - Deleting the last credential is refused unless another sign-in or recovery method exists (policy; see RR-REC-01).
- **Tests:** TC-PKY-11, TC-PKY-12.

### PKY-T12 Malformed or oversized ceremony data
- **Attacker:** ATK-1. **STRIDE:** DoS. **Severity:** Medium.
- **Mitigations:** limit request body size before parsing; rely on the maintained library for parsing, kept up to date; reject unknown fields at the schema layer.
- **Tests:** TC-PKY-15.

### PKY-T13 Personal data in the user handle
- **Attacker:** ATK-12, or anyone with authenticator access. **STRIDE:** Information disclosure. **Severity:** Low.
- **Mitigations:** the user handle is random and opaque, at most 64 bytes as the specification allows, and contains no email or name (ADR-003).
- **Tests:** TC-PKY-16.

### PKY-T14 Misleading privacy claims about biometrics
- **STRIDE:** Information disclosure (trust). **Severity:** Low.
- **Mitigations:** the documentation states that biometric data never leaves the authenticator. Authlume does not claim to receive or protect biometrics, as the Blueprint (§5) requires. This is checked in documentation review.

### PKY-T15 Phishing resistance undermined by fallbacks
- **Attacker:** ATK-10. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** the account keeps a password or an email reset, which the attacker phishes instead.
- **Mitigations:** allow passkey-only accounts, and document that the weakest enabled method sets the account's strength.
- **Residual:** RR-PKY-01.

### PKY-T16 Platform account compromise exposes synced passkeys
- **Attacker:** an attacker with the user's platform account. **STRIDE:** Spoofing. **Severity:** Medium.
- **Mitigations:** none within Authlume. Documented.
- **Residual:** RR-PKY-03.

## Residual risks

| ID | Residual risk |
| --- | --- |
| RR-PKY-01 | Phishing resistance holds only if the account has no phishable fallback. |
| RR-PKY-02 | Clone detection is unavailable for authenticators that always report counter 0, which includes most synced passkeys. |
| RR-PKY-03 | The security of a synced passkey depends on the user's platform account, which Authlume does not control. |
| RR-PKY-04 | Script running in the host origin can start ceremonies. It still needs a user gesture and verification, but can trick the user into approving. |

## Test cases

| ID | Verifies | Scenario and expected result | Gate |
| --- | --- | --- | --- |
| TC-PKY-01 | T02 | An RP ID hash that does not match the configured RP ID is rejected. | P5 |
| TC-PKY-02 | T02 | Origins not on the allowlist are rejected: sibling subdomain, `http` scheme, different port, trailing variants. | P5 |
| TC-PKY-03 | T01 | Replaying a previously accepted assertion or challenge is rejected. | P5 |
| TC-PKY-04 | T01 | A challenge past its TTL is rejected using actual database time. Invalid signatures and injected failures after a committed claim leave the challenge burned; replay fails. | P5 |
| TC-PKY-05 | T01, T03 | A registration challenge used for authentication, or the reverse, is rejected. | P5 |
| TC-PKY-06 | T03 | A challenge submitted from a different browser context is rejected without consuming the initiating browser's challenge; that browser can still complete it. | P5 |
| TC-PKY-07 | T03 | A tampered `authenticatorData`, an invalid signature, or a `clientDataJSON.type` mismatch is rejected. | P5 |
| TC-PKY-08 | T07 | When user verification is required, an assertion with UV=0 is rejected. UP=0 is always rejected. | P5 |
| TC-PKY-09 | T05 | Registering a credential ID that already exists, on the same or another account, is rejected. | P5 |
| TC-PKY-10 | T06 | A counter that is not greater than the stored one, when either is non-zero, triggers the configured action and an audit event. Counters that stay at 0 are accepted. | P5 |
| TC-PKY-11 | T11 | An assertion from a deleted credential is rejected on the next attempt. | P5 |
| TC-PKY-12 | T11 | After deletion, re-enrollment succeeds and the old credential stays rejected. | P5 |
| TC-PKY-13 | T04 | Adding a passkey without recent reauthentication is refused. Success emits an audit event and a notification. | P5 |
| TC-PKY-14 | T10 | In a username-first flow, responses for unknown and known users cannot be told apart in shape or timing. | P5 |
| TC-PKY-15 | T12 | Oversized and malformed attestation, assertion and `clientDataJSON` inputs are rejected without crashing (fuzz suite). | P5 |
| TC-PKY-16 | T13 | The user handle is random and contains no account email or name. | P5 |
| TC-PKY-17 | T09 | Registration with an algorithm outside the allowlist is rejected. | P5 |
| TC-PKY-18 | T02 | Changing the `Host` or forwarded headers does not change the expected RP ID or origins. | P5 |
