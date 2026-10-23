# Threat model: account recovery

> [!CAUTION]
> Draft v0.1, 2026-10-23. Drafted by an AI agent. **Not reviewed.** See the
> [threat model overview](README.md) for scope, identifiers and severity scale.

## Flows

Recovery restores access when the user has lost their primary credential.
Whoever completes recovery gains full control of the account, so every
recovery path is treated as equivalent to the strongest credential it
replaces.

1. **Recovery codes:**
   1. **Issue.** After recent reauthentication, the server generates a set of codes. It shows them once and stores only their verifiers.
   2. **Redeem.** The user submits their account identifier and a code. The code is consumed atomically.
   3. **Complete.** All sessions are revoked, and the remaining codes are invalidated and replaced with a new set (rotation, Blueprint §4).
   4. **Re-enroll.** The user is guided to enroll a new primary credential.
2. **Verified-email recovery** is available only when the account has a verified email **and** the developer has configured delivery (Blueprint §4):
   1. The request gets a generic response.
   2. A single-use link is emailed, with the same token handling as password reset.
   3. The user redeems the link and enrolls a new credential.
   4. All sessions are revoked.
3. **Unrecoverable accounts:** with no verified email and no unused recovery code, the account cannot be recovered. The Blueprint requires this to be documented, not worked around.

Related decisions:
- Token storage: [ADR-012](../adr/0012-one-time-challenge-storage.md)
- Rotation and invalidation: [ADR-013](../adr/0013-token-rotation.md)
- Throttling: [ADR-007](../adr/0007-transaction-and-rate-limit-backing-store.md)
- Verifier construction: ADR-006 (open)

## Assets

A-1, A-6, A-7, A-11, A-13 and A-14.

## Threats

### REC-T01 Online guessing of recovery codes
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:**
  - Codes are high-entropy and system-generated (INV-01). The recommendation is at least 128 bits per code, encoded for readability, for example base32 in groups. The final value is set in ADR-006.
  - Redemption requires the account identifier plus the code, and is throttled per account and per source.
  - Failures get generic responses.
- **Tests:** TC-REC-01, TC-REC-05.

### REC-T02 Offline cracking of code verifiers
- **Attacker:** ATK-7, ATK-8. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:**
  - At 128 bits or more, a SHA-256 digest makes offline search infeasible, even when server keys also leak (ATK-8).
  - If the owner chooses shorter codes for usability, storage **must** use a keyed verifier and the residual risk must be recorded (ADR-006).
  - Plaintext is never stored.
- **Tests:** TC-REC-02.

### REC-T03 Concurrent redemption of one code
- **Attacker:** ATK-5. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:** atomic conditional consumption, `used_at IS NULL` in one statement (INV-04), in the same transaction as session revocation and code rotation.
- **Tests:** TC-REC-03, TC-REC-04.

### REC-T04 Lookup by code alone
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Abuse case:** if a code is accepted without an account identifier, every guess is tested against every account, and per-account throttling cannot apply.
- **Mitigations:** an account identifier is always required, and the code is checked only against that account's verifiers.
- **Tests:** TC-REC-05.

### REC-T05 Codes exposed after issue
- **Attacker:** ATK-12, ATK-3. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:**
  - Codes are shown once and no API returns existing codes.
  - Codes are never emailed or logged.
  - The display response sends `Cache-Control: no-store`.
  - Regenerating codes invalidates the previous set atomically.
- **Tests:** TC-REC-06, TC-REC-10.

### REC-T06 Backdoor codes issued through a stolen session
- **Attacker:** ATK-3, ATK-4. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Abuse case:** an attacker with a session regenerates codes and keeps them for later.
- **Mitigations:** regeneration requires recent reauthentication, emits an audit event and a notification, and invalidates the old set, so the legitimate user's codes stop working and the change is visible.
- **Tests:** TC-REC-06.

### REC-T07 Mailbox compromise used for recovery
- **Attacker:** ATK-9. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:**
  - Email recovery is offered only when the email is verified and delivery is configured.
  - All sessions are revoked and the user is notified on completion.
  - Optionally, a configurable recovery delay: recovery completes only after a waiting period, during which a notification lets the real owner cancel it from any signed-in session. The owner decides whether to include this option.
- **Residual:** RR-REC-02. **Tests:** TC-REC-08, TC-REC-12.

### REC-T08 Recovery used to downgrade a strong account
- **Attacker:** ATK-9, ATK-10. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Abuse case:** a passkey-only account is taken over through email, which is weaker than the passkey.
- **Mitigations:** the developer configures which recovery methods each account may use. The documentation states that the weakest enabled method sets the account's strength. Any recovery revokes every session.
- **Residual:** RR-REC-02. **Tests:** TC-REC-11.

### REC-T09 Recovery tokens leak through the link
- **Attacker:** ATK-9, ATK-12, ATK-1. **STRIDE:** Information disclosure, Spoofing. **Severity:** High.
- **Mitigations:** identical to password reset tokens: [PWD-T08](passwords.md#pwd-t08-reset-or-verification-token-leaks-from-the-link) to [PWD-T11](passwords.md#pwd-t11-reset-poisoning-through-the-host-header).
- **Tests:** TC-REC-09.

### REC-T10 Recovery state is enumerable
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** Medium.
- **Abuse case:** responses such as "this account has no recovery email" or "no codes left" reveal account state.
- **Mitigations:** generic responses (INV-05). State is shown only to an authenticated user.
- **Tests:** TC-REC-08.

### REC-T11 Attacker keeps access after recovery
- **Attacker:** ATK-3. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:**
  - Completing recovery revokes all sessions, invalidates all outstanding one-time tokens, and rotates recovery codes.
  - Optionally, it revokes passkeys added recently ([ADR-013](../adr/0013-token-rotation.md)).
- **Tests:** TC-REC-07.

### REC-T12 Manual recovery by support staff is socially engineered
- **Attacker:** ATK-11. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:** this is outside Authlume's flows. Any administrative recovery API that Authlume exposes must require explicit host authorization, emit audit events and notify the account. The documentation includes guidance for support processes.
- **Residual:** RR-REC-03.

### REC-T13 Phishing of recovery codes
- **Attacker:** ATK-10. **STRIDE:** Spoofing. **Severity:** Medium.
- **Mitigations:** recovery pages are served only from the configured origin, and the documentation tells users to enter codes only on that origin. No complete mitigation exists.
- **Residual:** RR-REC-04.

## Residual risks

| ID | Residual risk |
| --- | --- |
| RR-REC-01 | Accounts with no verified email and no unused recovery code are **unrecoverable by design**. Developers and users must be told before this happens (Blueprint §4). |
| RR-REC-02 | Email recovery bounds the account's strength by the mailbox's strength. |
| RR-REC-03 | Human support processes are outside Authlume's control. |
| RR-REC-04 | Recovery codes are phishable. |

## Test cases

The Phase Tracker does not yet assign recovery codes to a phase.
Verified-email recovery belongs with P4. Recovery codes are marked "P4 or P5"
until the owner assigns them.

| ID | Verifies | Scenario and expected result | Gate |
| --- | --- | --- | --- |
| TC-REC-01 | T01 | Codes come from the CSPRNG at the configured entropy, with the configured set size and encoding. | P4/P5 |
| TC-REC-02 | T02 | Only verifiers are stored. A database scan finds no issued code. | P4/P5 |
| TC-REC-03 | T03 | 20 concurrent redemptions of one code produce exactly one success. | P4/P5 |
| TC-REC-04 | T03 | A used code is rejected. After recovery, every remaining code is invalid and a new set has been issued. | P4/P5 |
| TC-REC-05 | T01, T04 | Redemption without an account identifier is impossible. Failures are throttled per account and per source, with generic responses. | P4/P5 |
| TC-REC-06 | T05, T06 | Regeneration without recent reauthentication is refused. Regeneration invalidates the previous set and emits an audit event and a notification. | P4/P5 |
| TC-REC-07 | T11 | Completing recovery revokes all sessions and invalidates outstanding one-time tokens. | P4/P5 |
| TC-REC-08 | T07, T10 | Email recovery is not offered when the email is unverified or delivery is unconfigured, and the response is the same generic response. | P4 |
| TC-REC-09 | T09 | Email recovery tokens pass the password reset token tests TC-PWD-13, 14, 16 and 17. | P4 |
| TC-REC-10 | T05 | Codes never appear in logs, audit events or telemetry. No endpoint returns an existing code. | P4/P5 |
| TC-REC-11 | T08 | Recovery methods the developer has disabled for an account are neither offered nor accepted. | P4/P5 |
| TC-REC-12 | T07 | If the recovery delay is enabled, cancelling from a signed-in session or a notification stops recovery. | P4 |
