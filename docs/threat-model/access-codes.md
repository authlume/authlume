# Threat model: 12-digit access codes

> [!CAUTION]
> Draft v0.1, 2026-10-23. Drafted by an AI agent. **Not reviewed.** See the
> [threat model overview](README.md) for scope, identifiers and severity scale.
> This module is optional and **disabled by default** (Blueprint §4). Phase P6
> may not start until P1 risk acceptance.

## What the Blueprint proposes

- **Mode A, temporary:** short-lived, single-use 12-digit codes, bound to an initiated transaction or trusted context, with a short TTL and attempt limits.
- **Mode B, persistent:** long-lived 12-digit codes. These are **never** accepted as an unrestricted sole factor; they require an independent factor, a stricter policy, rotation and revocation. The Blueprint adds: "Consider removing persistent mode if safe UX cannot be demonstrated."

Both modes are secrets, not identifiers. The Blueprint (§4) forbids deriving
keys from personal questions, storing plaintext or unkeyed short-code hashes,
and looking up users by short code without strict protections.

Decisions that depend on this document:
- Code policies and whether long-lived codes survive: ADR-005 (open)
- Keyed verifiers: ADR-006 (open)
- Transaction storage: [ADR-012](../adr/0012-one-time-challenge-storage.md)
- Shared counters: [ADR-007](../adr/0007-transaction-and-rate-limit-backing-store.md)

## How strong is a 12-digit code?

The probability calculations assume uniform independent codes. Offline timing
depends on the exact verifier and measured hardware throughput; no benchmark
has been performed for this project.

| Quantity | Value |
| --- | --- |
| Code space | 10^12, about 2^39.9, so roughly **40 bits** |
| Mode A: success chance per transaction with k attempts | k / 10^12. With k = 5, that is 5 × 10^-12. |
| Guessing when codes are looked up across all accounts | Each guess succeeds with probability of about N / 10^12. With N = 10^6 accounts, an even chance of one hit takes about **700,000 guesses**, which is reachable for a botnet. |
| Offline search of an **unkeyed** hash (SHA-256) | Up to 10^12 candidates, with an expected 5 × 10^11 guesses for one uniformly selected code. At measured throughput R guesses/second, expected time is (5 × 10^11) / R seconds. |
| Offline search of a keyed HMAC when the key also leaks (ATK-8) | The same candidate space, but HMAC throughput must be measured separately; SHA-256 throughput is not an HMAC benchmark. |
| Offline search with a keyed **and** memory-hard verifier when the key leaks | Raises cost per guess; feasibility depends on parameters, hardware and budget. No cracking-resistance guarantee is established here. |

The conclusion is that 40 bits is enough only when an attacker gets very few
online guesses, every guess is bound to a short-lived transaction, and the
stored verifier is keyed. A long-lived 40-bit secret depends entirely on
server key custody and on throttling, for as long as it is valid.

## Flows

### Mode A, transaction-bound temporary code

1. A transaction is initiated, for example a sign-in attempt in browser X. The server creates a transaction record with a high-entropy transaction ID, binds it to browser X's ceremony context ([ADR-012](../adr/0012-one-time-challenge-storage.md)), and generates a code.
2. The code is delivered through a channel the developer configures, for example email, or display on an already signed-in device. SMS is not a V1 channel.
3. The user enters the code in browser X. The server checks the transaction ID, the bound context, expiry and the attempt count. Then it compares HMAC(key, code) in constant time and consumes the transaction atomically.

### Mode B, persistent code (proposed; recommended for removal)

1. A system-generated code is issued to an authenticated user and stored as a keyed verifier.
2. The code may be presented only alongside an independent factor, under a stricter policy.

## Threats: Mode A

### CODE-T01 Guessing within a transaction
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Mitigations:** an attempt limit per transaction (for example 5), after which the transaction is burned; a short TTL (for example 10 minutes or less); and atomic attempt counting (INV-04).
- **Tests:** TC-CODE-03, TC-CODE-06.

### CODE-T02 Transaction farming
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Abuse case:** the attacker opens many transactions to multiply guesses.
- **Mitigations:** limits on transaction creation per account and per source, using shared counters ([ADR-007](../adr/0007-transaction-and-rate-limit-backing-store.md)). Delivery limits also prevent code-spamming the user.
- **Tests:** TC-CODE-07, TC-CODE-08.

### CODE-T03 Code relayed through phishing or social engineering
- **Attacker:** ATK-10, ATK-11. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** the attacker starts the transaction on their own device and persuades the victim to read out the code or type it in. This is the same pattern as device-code phishing.
- **Mitigations:**
  - Context binding stops codes from being used in any context other than the initiating one, but it does not stop an attacker who initiated the transaction.
  - Show context to the person approving: requesting device, approximate location, time.
  - Warn "never share this code".
  - Keep the TTL short.
  - Prefer passkeys.
- **Residual:** RR-CODE-01. **Tests:** review item; TC-CODE-05.

### CODE-T04 Code used outside its bound context
- **Attacker:** ATK-10. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:** redemption requires the transaction ID **and** the initiating browser's context secret. A different context is rejected and source-throttled without consuming the legitimate transaction's attempt budget (ADR-012).
- **Tests:** TC-CODE-05.

### CODE-T05 Concurrent redemption
- **Attacker:** ATK-5. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:** attempt counting and consumption use conditional atomic statements ([ADR-012](../adr/0012-one-time-challenge-storage.md)), so only one correct submission can succeed.
- **Tests:** TC-CODE-04.

### CODE-T06 Offline brute force of stored verifiers
- **Attacker:** ATK-7, ATK-8. **STRIDE:** Information disclosure. **Severity:** High for Mode B; Low for Mode A, whose codes expire within minutes.
- **Mitigations:** keyed HMAC verifiers only, with the key outside the database (TB-11, ADR-006). Never plaintext or unkeyed hashes.
- **Tests:** TC-CODE-02.

### CODE-T07 Code leaks through the delivery channel
- **Attacker:** ATK-9, ATK-12. **STRIDE:** Information disclosure. **Severity:** Medium.
- **Abuse case:** lock-screen previews, email logs, provider logs.
- **Mitigations:** a short TTL, and keeping the code out of subject lines. Codes never appear in Authlume logs or audit events.
- **Residual:** RR-CODE-02. **Tests:** TC-CODE-11.

### CODE-T08 Counters bypassed by load balancing
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:** INV-08. Attempt counts live on the transaction row and rate limits live in the shared store, never in process memory in production.
- **Tests:** TC-CODE-07.

### CODE-T09 Biased or predictable code generation
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Medium.
- **Mitigations:** draw from the CSPRNG with rejection sampling over 10^12, so there is no modulo bias, and preserve leading zeros (XC-T07).
- **Tests:** TC-CODE-01.

### CODE-T10 Transaction endpoint enables enumeration
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** Low.
- **Mitigations:** transaction IDs are 256-bit random values. Responses do not reveal whether an account exists (INV-05).
- **Tests:** TC-CODE-09.

## Threats: Mode B

### CODE-T11 A disclosed persistent code works until rotation
- **Attacker:** ATK-10, ATK-11, ATK-3. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** the code is phished once, seen over the user's shoulder, or written down. Unlike a one-time code, it stays valid for days or months.
- **Mitigations:** forced expiry and rotation, plus immediate revocation. These shorten the window; they do not close it.
- **Residual:** RR-CODE-03.

### CODE-T12 Persistent code accepted as the sole factor
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** misconfiguration, a skipped step or reordered requests let the code alone yield a session.
- **Mitigations:** the code can only be verified inside a transaction that has already passed an independent factor in the same ceremony. Configuration validation refuses any sole-factor configuration.
- **Tests:** TC-CODE-12.

### CODE-T13 Persistent code used as an account identifier
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** "enter your 12-digit code" finds the account, so every guess is tested against every account (see the table above).
- **Mitigations:** forbidden. The Blueprint says codes are not user IDs.
- **Tests:** TC-CODE-09.

### CODE-T14 Key leak makes every persistent code recoverable
- **Attacker:** ATK-8. **STRIDE:** Information disclosure. **Severity:** High.
- **Abuse case:** with the HMAC key, an attacker can search the 40-bit code space offline; any recovered code stays useful until rotation. Recovery time depends on measured verifier throughput.
- **Mitigations:** a keyed **and** memory-hard verifier (ADR-006), and rotation.
- **Residual:** RR-CODE-04.

### CODE-T15 A false sense of multi-factor protection
- **STRIDE:** Spoofing. **Severity:** Medium.
- **Abuse case:** a password plus a persistent code are both things the user knows. Together they are not independent factors, yet they may be presented as MFA.
- **Mitigations:** documentation must not describe password plus code as multi-factor. The only independent factor that combines meaningfully with Mode B is a passkey, which on its own (with user verification) already suffices.

### CODE-T16 Persistent code becomes a recovery downgrade
- **Attacker:** ATK-10. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:** forbidden. Codes never act as a recovery method.
- **Tests:** TC-CODE-12.

### CODE-T17 User-chosen or derived codes
- **Attacker:** ATK-2. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** dates, phone numbers and repeated digits are guessable.
- **Mitigations:** codes are always system-generated, never chosen by users and never derived from personal data.
- **Tests:** TC-CODE-13.

### CODE-T18 Rotation and revocation gaps
- **Attacker:** ATK-3. **STRIDE:** Elevation of privilege. **Severity:** Medium.
- **Mitigations:** rotation replaces the code atomically. Revocation takes effect at the next verification. Credential changes and recovery revoke the code ([ADR-013](../adr/0013-token-rotation.md)).
- **Tests:** TC-CODE-13.

## Unsafe numeric code designs

Each of these designs is flagged for **removal** or **redesign**. Any proposal
that matches one of them must be rejected in review.

| ID | Design | Why it is unsafe | Verdict |
| --- | --- | --- | --- |
| U-01 | A long-lived 12-digit code as the **sole** sign-in factor | A 40-bit static secret that is phishable and replayable for its whole lifetime; the Blueprint forbids it | **Remove** |
| U-02 | A code used to **find** the account ("enter your code") | Every guess is tested against every account, so the effective guess space shrinks by the number of accounts (T13) | **Remove** |
| U-03 | A code stored as plaintext or as an **unkeyed** hash | Offline search of the whole space takes about a minute or two | **Remove** |
| U-04 | A code **derived** from personal data or security questions | Guessable from public information; the Blueprint forbids it | **Remove** |
| U-05 | A code with **no expiry, rotation or revocation** | Indefinite replay after a single disclosure | **Remove** |
| U-06 | A code that acts as a **recovery** method | A downgrade path that bypasses stronger credentials | **Remove** |
| U-07 | Password plus persistent code presented as **MFA** | Two knowledge factors are not independent; the claim is misleading | **Remove** the claim |
| U-08 | Attempt counters **in process memory**, or per-instance limits | Bypassed by spreading attempts across instances | **Remove** |
| U-09 | **User-chosen** codes | Very low effective entropy | **Remove** |
| U-10 | A Mode A code **not bound** to its initiating context | Can be relayed or used from any device | **Redesign:** bind to the transaction and context |
| U-11 | Mode A with a long TTL or unlimited attempts | Turns a temporary code into a guessable one | **Redesign:** TTL of 10 minutes or less, attempt limit of about 5 |
| U-12 | Any Mode B as specified, kept in V1 | Adds little security over a passkey, and keeps all the risks of a static, low-entropy, phishable secret | **Remove from V1** (recommended) |

## Recommendation for ADR-005

1. **Remove Mode B (persistent codes) from V1.** The analysis finds no
   configuration in which Mode B adds meaningful security:
   - With a password, it is a second knowledge factor (T15).
   - With a passkey, the passkey already suffices.
   - On its own, it is forbidden.

   It also keeps every risk in U-01 to U-09. This matches the Blueprint's
   instruction to consider removal, and the P6 exit criterion "drop unsafe
   mode". If the underlying product need is a memorable numeric step, the owner
   can consider instead:
   - Mode A delivered on demand.
   - Passkeys.
   - A standards-based one-time password (TOTP, RFC 6238). This would widen V1 scope and needs its own ADR.
2. **Keep Mode A as an opt-in module, pending P6 risk sign-off,** with these minimum requirements:
   - Bound to the transaction and its context.
   - TTL of 10 minutes or less.
   - About 5 attempts per transaction.
   - Shared counters.
   - Keyed verifiers.
   - Limits on transaction creation.
   - Context shown to the approver.
   - No lookup by code.
3. **If the owner keeps Mode B anyway,** these are the minimum redesign
   conditions. Each of them must be proven by tests before P6 sign-off:
   - Codes are system-generated.
   - The verifier is keyed and memory-hard (ADR-006).
   - The code is usable only as a step-up after a passkey or password sign-in, never for sign-in or recovery.
   - Throttling per account, with exponential backoff.
   - Maximum lifetime with forced rotation.
   - Immediate revocation.
   - The code is never shown again after issue.
   - It is not described as MFA.
   - Explicit risk acceptance by the owner is recorded in ADR-005.

## Residual risks

| ID | Residual risk |
| --- | --- |
| RR-CODE-01 | Mode A codes can be socially engineered by an attacker who started the transaction. |
| RR-CODE-02 | Delivery channels such as email and notifications can expose codes during their TTL. |
| RR-CODE-03 | (Mode B) A disclosed persistent code stays usable until rotation or revocation. |
| RR-CODE-04 | (Mode B) If the verifier key leaks, every persistent code is exposed unless the verifier is also memory-hard, and targeted cracking remains feasible even then. |

## Test cases

| ID | Verifies | Scenario and expected result | Gate |
| --- | --- | --- | --- |
| TC-CODE-01 | T09 | Codes are uniform over 10^12 (statistical test), use rejection sampling, and preserve leading zeros. | P6 |
| TC-CODE-02 | T06 | Stored verifiers are keyed HMACs. A database scan finds no plaintext or unkeyed hash of any issued code. | P6 |
| TC-CODE-03 | T01 | After k failed attempts, the transaction is burned, and the correct code is rejected too. Invalid codes and injected downstream rollbacks do not refund attempts. | P6 |
| TC-CODE-04 | T05 | 20 concurrent submissions of the correct code produce exactly one success. An injected failure rolls back consumption together with the protected change and audit event. | P6 |
| TC-CODE-05 | T03, T04 | A correct code submitted from a different context is rejected and source-throttled without spending the legitimate transaction's attempt budget. The initiating browser can still complete it. | P6 |
| TC-CODE-06 | T01 | An expired transaction is rejected, judged by the database clock. | P6 |
| TC-CODE-07 | T02, T08 | Guesses spread across several application instances are limited by shared counters. | P6 |
| TC-CODE-08 | T02 | Limits on transaction creation per account and per source are enforced. | P6 |
| TC-CODE-09 | T10, T13 | No endpoint accepts a code without a transaction or account reference. A code alone never identifies an account. | P6 |
| TC-CODE-10 | All | With the module disabled (the default), its endpoints do not exist and configuration validation reports the module as off. | P6 |
| TC-CODE-11 | T07 | Issue, failure, success and exhaustion each emit an audit event. No event, log or telemetry record contains a code. | P6 |
| TC-CODE-12 | T12, T16 | (Mode B, only if kept) A code alone, a skipped factor step, reordered requests, or use in recovery never yields a session. | P6 |
| TC-CODE-13 | T17, T18 | (Mode B, only if kept) User-chosen codes are impossible. Rotation and revocation take effect at the next verification. | P6 |
| TC-CODE-14 | T01 | Verifiers are compared in constant time. Code review plus a unit test confirm `crypto.timingSafeEqual` or the equivalent is used. | P6 |
