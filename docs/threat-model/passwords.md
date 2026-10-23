# Threat model: passwords

> [!CAUTION]
> Draft v0.1, 2026-10-23. Drafted by an AI agent. **Not reviewed.** See the
> [threat model overview](README.md) for scope, identifiers and severity scale.

## Flows

1. **Registration:** email and password are submitted. Authlume creates the user, an unverified email and an Argon2id password credential. If configured, it sends a verification email.
2. **Sign-in:** email and password are submitted. Authlume returns a generic failure or a new session.
3. **Password change:** requires the current password or recent reauthentication. Other sessions are revoked.
4. **Password reset:**
   1. A request is made for an email. The response is always generic.
   2. If an eligible account exists, Authlume emails a single-use, expiring token.
   3. The token is redeemed together with a new password.
   4. All sessions are revoked.
5. **Email verification:** a single-use token is emailed and bound to the address. Redeeming it sets `verified_at`.

Related decisions:
- Password hashing: [ADR-011](../adr/0011-password-hashing.md)
- Token storage: [ADR-012](../adr/0012-one-time-challenge-storage.md)
- Invalidation rules: [ADR-013](../adr/0013-token-rotation.md)
- Throttling: [ADR-007](../adr/0007-transaction-and-rate-limit-backing-store.md)
- Email normalization and duplicates: ADR-004 (open)

## Assets

A-1, A-2, A-3, A-7, A-11, A-13 and A-14 (see the [asset list](README.md#assets)).

## Threats

### PWD-T01 Credential stuffing and online guessing
- **Attacker:** ATK-1, ATK-2. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** the attacker replays leaked email and password pairs, or sprays common passwords across many accounts.
- **Mitigations:**
  - Shared-store throttling ([ADR-007](../adr/0007-transaction-and-rate-limit-backing-store.md)), keyed per normalized account, per source address or prefix, and globally per endpoint.
  - Backoff instead of permanent lockout (XC-T02).
  - A password blocklist at registration and change.
  - An optional developer hook for breached-password checks.
  - Passkeys promoted as the stronger credential.
- **Residual:** RR-PWD-01, RR-PWD-04. **Tests:** TC-PWD-01 to TC-PWD-03, TC-PWD-22.

### PWD-T02 Account enumeration through responses
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** Medium.
- **Abuse case:** different messages, status codes or headers for "no such account", "wrong password", "email already registered" and "reset sent".
- **Mitigations:**
  - Identical public responses (INV-05).
  - Registration with an existing email returns the normal response and emails the existing owner instead. This requires the verified-email flow; see RR-PWD-05 and ADR-004.
  - Reset requests always return the same response.
- **Tests:** TC-PWD-04, TC-PWD-06, TC-PWD-07.

### PWD-T03 Account enumeration through timing
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** Medium.
- **Abuse case:** an unknown email skips password hashing, or a known email waits for email delivery, so response times differ.
- **Mitigations:**
  - Verify against a dummy Argon2id hash, with the same parameters, when the account does not exist.
  - Send email asynchronously, through a queue or after the response, so delivery time never shows.
  - Run the same database queries on both paths where practical.
- **Tests:** TC-PWD-05, TC-PWD-07.

### PWD-T04 Offline cracking after a database leak
- **Attacker:** ATK-7. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:**
  - Argon2id with benchmarked parameters above a hard floor and a per-hash random salt ([ADR-011](../adr/0011-password-hashing.md)).
  - An optional server-held pepper ([ADR-011](../adr/0011-password-hashing.md), ADR-006).
  - Rehash on sign-in when parameters are upgraded.
  - No plaintext passwords anywhere.
- **Residual:** RR-PWD-01. **Tests:** TC-PWD-08, TC-PWD-09.

### PWD-T05 Passwords leak into logs, errors or telemetry
- **Attacker:** ATK-12. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:** XC-T03. Password fields are carried in a redacting secret type from the moment of parsing, and are never included in validation errors.
- **Tests:** TC-PWD-10, TC-XC-03.

### PWD-T06 Denial of service through password hashing
- **Attacker:** ATK-1. **STRIDE:** DoS. **Severity:** Medium.
- **Abuse case:** a flood of sign-in attempts exhausts CPU and memory, because Argon2id is deliberately expensive.
- **Mitigations:**
  - Throttle before hashing.
  - Cap concurrent hash operations per process, returning a "busy" error when saturated.
  - Cap password length in bytes before hashing.
  - Publish capacity guidance from benchmarks.
- **Residual:** RR-PWD-06. **Tests:** TC-PWD-11.

### PWD-T07 Duplicate registration race
- **Attacker:** ATK-5. **STRIDE:** Tampering. **Severity:** Medium.
- **Abuse case:** concurrent registrations for the same email create two accounts.
- **Mitigations:** a unique database constraint on the normalized email. A constraint violation maps to the generic registration response. No check-then-insert (Blueprint §7).
- **Tests:** TC-PWD-12.

### PWD-T08 Reset or verification token leaks from the link
- **Attacker:** ATK-9, ATK-12. **STRIDE:** Information disclosure. **Severity:** High.
- **Abuse case:** the token in the emailed URL leaks through the `Referer` header to third-party resources, through browser history, proxy logs or analytics, or through email security scanners that fetch links.
- **Mitigations:**
  - Opening the link (`GET`) only renders a form; the token is consumed only by `POST`. This also stops scanners from burning tokens.
  - The landing page sends `Referrer-Policy: no-referrer` and `Cache-Control: no-store`, and loads no third-party resources.
  - A short TTL, single use, and storage as a hash only ([ADR-012](../adr/0012-one-time-challenge-storage.md)).
  - Logging guidance tells developers to strip the token parameter from URLs.
- **Residual:** RR-PWD-03. **Tests:** TC-PWD-17.

### PWD-T09 Reset token replay or race
- **Attacker:** ATK-5. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Abuse case:** the same token is redeemed twice concurrently, setting two passwords or creating two sessions.
- **Mitigations:** atomic conditional consumption (INV-04) in the same transaction as the password update.
- **Tests:** TC-PWD-13, TC-PWD-14.

### PWD-T10 Reset token guessing
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Low with mitigations.
- **Mitigations:** 256-bit random tokens (INV-01), looked up by hash, with a throttled redemption endpoint.
- **Tests:** TC-PWD-13.

### PWD-T11 Reset poisoning through the Host header
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** the attacker requests a reset for the victim with a forged `Host` or `X-Forwarded-Host`. The emailed link points to the attacker's domain, and the victim's click delivers the token to the attacker.
- **Mitigations:** reset and verification links are built only from the configured public origin (INV-07, XC-T08).
- **Tests:** TC-PWD-16.

### PWD-T12 Mailbox compromise leads to account takeover
- **Attacker:** ATK-9. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:**
  - Notify the account when a reset is completed.
  - Let developers disable email reset for accounts that have a passkey, or require an existing passkey as well (policy option).
  - Document that email strength bounds account strength.
- **Residual:** RR-PWD-03. **Tests:** TC-PWD-18.

### PWD-T13 Attacker sessions survive a password reset or change
- **Attacker:** ATK-3. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:**
  - Reset revokes all sessions.
  - Change revokes all other sessions and rotates the current one ([ADR-013](../adr/0013-token-rotation.md)).
  - Both invalidate outstanding reset tokens.
- **Tests:** TC-PWD-15, TC-PWD-18.

### PWD-T14 Credentials changed through a stolen session
- **Attacker:** ATK-3, ATK-4. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Abuse case:** an attacker with a stolen session changes the password or email to keep access.
- **Mitigations:**
  - Credential and email changes require the current password or recent reauthentication.
  - A changed email is verified before it takes effect, and the old address is notified.
  - CSRF defenses ([sessions.md](sessions.md#ses-t04-cross-site-request-forgery)).
- **Tests:** TC-PWD-19, TC-PWD-20.

### PWD-T15 Account pre-hijacking
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Abuse case:** the attacker registers the victim's email before the victim does, without verifying it, and keeps a session or a known password. The victim later "recovers" or verifies the account while the attacker keeps access. This pattern is described in published research on pre-hijacked accounts (Sudhodanan and Paverd, USENIX Security 2022).
- **Mitigations:**
  - When ownership of an email is first proven by verification or reset, revoke every session and invalidate credentials created before verification ([ADR-013](../adr/0013-token-rotation.md)).
  - Unverified accounts expire after a configurable period.
  - The duplicate and verification policy is decided in ADR-004.
- **Tests:** TC-PWD-21.

### PWD-T16 Email normalization confusion
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Abuse case:** case, Unicode or IDN variants create duplicate accounts, or match the wrong account.
- **Mitigations:** one canonicalization function, used both for the uniqueness constraint and for lookup (ADR-004, open). The original address is stored for display.
- **Tests:** to be defined with ADR-004.

### PWD-T17 Weak legacy hashes after migration
- **Attacker:** ATK-7. **STRIDE:** Information disclosure. **Severity:** High.
- **Abuse case:** imported bcrypt, SHA-1 or MD5 hashes sit in the database until users sign in.
- **Mitigations:** wrap legacy hashes in Argon2id at import time, so the database never holds a weak-only hash, and upgrade to a native Argon2id hash on next sign-in ([ADR-011](../adr/0011-password-hashing.md)).
- **Tests:** TC-PWD-23.

### PWD-T18 Unicode mismatches lock users out
- **Attacker:** none (correctness). **STRIDE:** DoS. **Severity:** Low.
- **Abuse case:** the same password typed on different devices produces different bytes.
- **Mitigations:** normalize before hashing, with the form recorded as a versioned decision in [ADR-011](../adr/0011-password-hashing.md).
- **Tests:** TC-PWD-24.

### PWD-T19 Real-time phishing of passwords
- **Attacker:** ATK-10. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:** none that are complete for passwords. Promote passkeys and allow passkey-only accounts.
- **Residual:** RR-PWD-02.

## Residual risks

| ID | Residual risk | Owner |
| --- | --- | --- |
| RR-PWD-01 | Weak but policy-compliant passwords remain crackable after a database leak. Argon2id only raises the cost. | Accepted; documented |
| RR-PWD-02 | Passwords can be phished in real time. Only passkeys resist this. | Accepted; documented |
| RR-PWD-03 | Whoever controls the mailbox can reset the password. Email strength bounds account strength. | Accepted; developer policy options |
| RR-PWD-04 | Large botnets can run slow credential stuffing below the thresholds. | Accepted; developer monitoring |
| RR-PWD-05 | If the developer disables the verified-email flow, registration cannot hide whether an email exists. | Accepted; documented trade-off |
| RR-PWD-06 | Hashing cost can be used for denial of service. Providing capacity is the developer's responsibility. | Accepted; guidance provided |

## Test cases

| ID | Verifies | Scenario and expected result | Gate |
| --- | --- | --- | --- |
| TC-PWD-01 | T01 | Repeated failures for one account from many sources trigger account-level throttling, with a generic response. | P4 |
| TC-PWD-02 | T01 | Spraying one password across many accounts from one source or prefix triggers source-level throttling. | P4 |
| TC-PWD-03 | T01 | Failures spread across two application instances count against one shared limit (INV-08). Invalid credentials and injected authentication rollbacks do not refund consumed attempts. | P4 |
| TC-PWD-04 | T02 | Unknown email and wrong password return byte-identical status, headers (other than date) and body. | P4 |
| TC-PWD-05 | T03 | Latency distributions for unknown and known accounts cannot be told apart within a documented statistical threshold (dummy hash active). | P4 |
| TC-PWD-06 | T02 | Registering with an existing email returns the same response as a new registration, and the existing owner receives a notice. | P4 |
| TC-PWD-07 | T02, T03 | Reset requests for unknown and known emails return the same response, with no timing difference caused by sending email. | P4 |
| TC-PWD-08 | T04 | Stored credentials are Argon2id PHC strings with parameters at or above the floor. A database scan finds no test plaintext. | P4 |
| TC-PWD-09 | T04 | A hash below the current parameters is rehashed on successful sign-in, in the same transaction. | P4 |
| TC-PWD-10 | T05 | Every flow is run with a sentinel password. The sentinel is absent from logs, errors, audit events and telemetry. | P4 |
| TC-PWD-11 | T06 | An oversized password is rejected before hashing. Saturating the hash-concurrency limit returns "busy" without memory growth. | P4 |
| TC-PWD-12 | T07 | 50 concurrent registrations for one email produce exactly one account. | P4 |
| TC-PWD-13 | T09, T10 | 20 concurrent redemptions of one reset token produce exactly one success. Random tokens fail. | P4 |
| TC-PWD-14 | T09 | An expired token is rejected, judged by the database clock. | P4 |
| TC-PWD-15 | T13 | A password change invalidates outstanding reset tokens. | P4 |
| TC-PWD-16 | T11 | Forged `Host` and `X-Forwarded-Host` headers do not change emailed link origins. | P4 |
| TC-PWD-17 | T08 | `GET` on the reset link does not consume the token, and `POST` does. The landing page sends `Referrer-Policy: no-referrer` and `Cache-Control: no-store`. | P4 |
| TC-PWD-18 | T12, T13 | Completing a reset revokes all sessions and sends a notification. A password change revokes all other sessions. | P4 |
| TC-PWD-19 | T14 | Password and email change without the current password or recent reauthentication is refused. | P4 |
| TC-PWD-20 | T14 | Changing the email invalidates verification tokens for the previous address, and the old address is notified. | P4 |
| TC-PWD-21 | T15 | When a pre-registered, unverified account is claimed by verification or reset, the earlier party's sessions, passkeys and password stop working. A fresh sign-in with the old password fails; only a newly established owner credential works. | P4 |
| TC-PWD-22 | T01 | Blocklisted passwords are rejected. Composition rules are not required. | P4 |
| TC-PWD-23 | T17 | Imported legacy hashes are stored wrapped, and the first successful sign-in upgrades them. | P4 |
| TC-PWD-24 | T18 | NFC-equivalent passwords verify identically; compatibility characters that NFC leaves distinct remain distinct. Minimum lengths count Unicode code points, including supplementary-plane characters. Imported credentials retain their recorded normalization form. | P4 |
