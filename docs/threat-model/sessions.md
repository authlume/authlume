# Threat model: sessions

> [!CAUTION]
> Draft v0.1, 2026-10-23. Drafted by an AI agent. **Not reviewed.** See the
> [threat model overview](README.md) for scope, identifiers and severity scale.

## Flows

The proposed design is in
[ADR-002](../adr/0002-session-strategy-and-cookie-defaults.md), with rotation
in [ADR-013](../adr/0013-token-rotation.md).

1. **Issue:** after successful authentication, the server generates a new 256-bit token. It stores only the token's SHA-256 hash, sets a `__Host-` cookie that is `HttpOnly`, `Secure` and `SameSite=Lax`, and sends `Cache-Control: no-store`.
2. **Validate:** on each request, the server hashes the cookie value, looks up the session, and checks that it is not revoked and has not passed its idle or absolute expiry.
3. **Rotate:** on privilege-changing events ([ADR-013](../adr/0013-token-rotation.md)).
4. **Revoke:** on sign-out, sign-out everywhere, credential changes, recovery, account deletion or an administrative action.

## Assets

A-1, A-4, A-11 (session metadata), A-13 and A-14.

## Threats

### SES-T01 XSS in the host application uses the session
- **Attacker:** ATK-4. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations:**
  - `HttpOnly` stops script from reading the token, so it cannot be exfiltrated for later use.
  - Sensitive actions need recent reauthentication.
  - Tokens are never put in `localStorage` or `sessionStorage` (Blueprint §5).
- **Residual:** RR-SES-01. Injected script can still send same-origin requests while the page is open.
- **Tests:** TC-SES-09, TC-SES-18.

### SES-T02 Database leak yields usable tokens
- **Attacker:** ATK-7, ATK-8. **STRIDE:** Spoofing. **Severity:** High.
- **Mitigations:** only the SHA-256 hash of each 256-bit token is stored (INV-03). Hashes cannot be turned back into cookie values, even with server keys.
- **Tests:** TC-SES-02.

### SES-T03 Token appears in a URL, log or analytics
- **Attacker:** ATK-12. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:** the token travels only in the cookie, never in URLs or response bodies. `Cookie` and `Set-Cookie` headers are redacted in any logging Authlume provides (XC-T03).
- **Tests:** TC-SES-14.

### SES-T04 Cross-site request forgery
- **Attacker:** ATK-4. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Mitigations (layered):**
  1. `SameSite=Lax` by default.
  2. Every state-changing request is checked against the configured origins, using `Sec-Fetch-Site` and `Origin`, and rejected if neither shows a same-origin request.
  3. No state changes on `GET` or `HEAD`.
  4. An optional synchronizer token for developers who need it.
  5. Sign-in and registration endpoints are covered too, against login CSRF, where the attacker signs the victim into the attacker's account.
- `SameSite` treats sibling subdomains as same-site, so checks 2 and 3 are what protect against a hostile subdomain.
- **Tests:** TC-SES-10, TC-SES-11.

### SES-T05 Session fixation
- **Attacker:** ATK-4 (sibling subdomain), ATK-1. **STRIDE:** Spoofing. **Severity:** High.
- **Abuse case:** the attacker plants a known session identifier before the victim signs in.
- **Mitigations:**
  - A new token is always issued at authentication. A token from before sign-in is never promoted.
  - The `__Host-` prefix makes browsers reject cookies set with a `Domain` attribute or from a non-secure origin, so sibling subdomains cannot plant this cookie.
- **Tests:** TC-SES-08, TC-SES-09.

### SES-T06 Token intercepted on the network
- **Attacker:** ATK-13. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:** the `Secure` attribute (implied by `__Host-`). Configuration validation refuses `http` public origins outside the explicit development mode (XC-T05). HSTS is the developer's responsibility.
- **Tests:** TC-SES-09, TC-SES-15.

### SES-T07 Revocation does not take effect
- **Attacker:** ATK-3. **STRIDE:** Elevation of privilege. **Severity:** High.
- **Abuse case:** stateless or cached validation keeps a revoked session alive.
- **Mitigations:** opaque tokens are checked against the database on every request. Any optional cache has a documented maximum staleness and is off by default ([ADR-002](../adr/0002-session-strategy-and-cookie-defaults.md)).
- **Tests:** TC-SES-05, TC-SES-06, TC-SES-07.

### SES-T08 Expiry enforced only by the cookie
- **Attacker:** ATK-3. **STRIDE:** Elevation of privilege. **Severity:** Medium.
- **Mitigations:** idle and absolute expiry are enforced on the server, using the database clock (INV-11). Cookie `Max-Age` is a convenience, not a control.
- **Tests:** TC-SES-03, TC-SES-04.

### SES-T09 Rotation races
- **Attacker:** ATK-5, or normal concurrent browser tabs. **STRIDE:** DoS, Spoofing. **Severity:** Medium.
- **Abuse case:** concurrent requests during rotation either log the user out or leave two tokens valid without limit.
- **Mitigations:** rotation is a single transaction. If periodic rotation is enabled, the previous token gets a short, bounded grace window ([ADR-013](../adr/0013-token-rotation.md)).
- **Tests:** TC-SES-13.

### SES-T10 Token guessing
- **Attacker:** ATK-1. **STRIDE:** Spoofing. **Severity:** Low with mitigations.
- **Mitigations:** 256-bit CSPRNG tokens, looked up by hash.
- **Tests:** TC-SES-01.

### SES-T11 A CDN or shared cache stores authenticated responses
- **Attacker:** another user of the same cache. **STRIDE:** Information disclosure. **Severity:** High.
- **Abuse case:** a response carrying `Set-Cookie` or personalized server-rendered content is cached and served to other users.
- **Mitigations:** Authlume sets `Cache-Control: no-store` on every response it sends that sets or depends on a session. The Next.js adapter documents and tests caching behaviour (P7).
- **Tests:** TC-SES-12.

### SES-T12 Session data crosses between users
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:** every query is scoped to the token hash and the user identifier, and there is no shared mutable per-request state in the library.
- **Tests:** TC-SES-16.

### SES-T13 Session table growth is abused
- **Attacker:** ATK-1. **STRIDE:** DoS. **Severity:** Low.
- **Mitigations:** throttled sign-in, an optional cap on sessions per user, and a cleanup job for expired and revoked rows.
- **Tests:** TC-SES-20.

### SES-T14 Session metadata becomes a privacy liability
- **Attacker:** ATK-7, ATK-12. **STRIDE:** Information disclosure. **Severity:** Low.
- **Mitigations:** only configured metadata is stored (by default a coarse user agent and a truncated IP address, to be decided in [ADR-002](../adr/0002-session-strategy-and-cookie-defaults.md)), with a retention purge (Blueprint §7).
- **Tests:** TC-SES-19.

### SES-T15 Server code or secrets in the browser bundle
- **Attacker:** ATK-1. **STRIDE:** Information disclosure. **Severity:** High.
- **Mitigations:** [ADR-001](../adr/0001-package-naming-and-split.md) boundaries and XC-T04.
- **Tests:** TC-SES-18.

## Residual risks

| ID | Residual risk |
| --- | --- |
| RR-SES-01 | XSS in the host application lets an attacker act as the user while the page is open. `HttpOnly` prevents theft of the token, not its use. |
| RR-SES-02 | Malware on the user's device can copy cookies. Rotation with reuse detection ([ADR-013](../adr/0013-token-rotation.md) option) limits, but does not stop, this. Device-bound session credentials are a possible future option, not a V1 dependency. |
| RR-SES-03 | If the developer enables a validation cache, revocation is delayed up to the configured staleness. |

## Test cases

| ID | Verifies | Scenario and expected result | Gate |
| --- | --- | --- | --- |
| TC-SES-01 | T10 | Tokens are 32 bytes from the CSPRNG, base64url-encoded, with no collisions in a large sample (sanity check). | P3 |
| TC-SES-02 | T02 | Only token hashes are stored. Presenting a stored hash as a cookie value fails. | P3 |
| TC-SES-03 | T08 | A session idle past its limit is rejected, even though the cookie is still present. | P3 |
| TC-SES-04 | T08 | A session past its absolute limit is rejected, even with continuous activity. | P3 |
| TC-SES-05 | T07 | A revoked session is rejected on the very next request. With the cache enabled, within the configured staleness. | P3 |
| TC-SES-06 | T07 | Sign-out revokes the server record and clears the cookie. Replaying the old cookie fails. | P3 |
| TC-SES-07 | T07 | "Sign out everywhere" revokes every session for the user. | P3 |
| TC-SES-08 | T05 | A pre-authentication cookie value supplied by an attacker is never promoted. Sign-in always issues a new token. | P3 |
| TC-SES-09 | T01, T05, T06 | The cookie has the `__Host-` prefix, `HttpOnly`, `Secure`, `SameSite=Lax`, `Path=/` and no `Domain`. | P3, P7 |
| TC-SES-10 | T04 | A cross-site `POST` is rejected. A request missing both `Origin` and `Sec-Fetch-Site` is rejected under the default policy. Same-origin requests are accepted. Login CSRF is rejected. | P7 |
| TC-SES-11 | T04 | No Authlume endpoint changes state on `GET` or `HEAD`. | P3 |
| TC-SES-12 | T11 | Responses that set or depend on a session carry `Cache-Control: no-store`. A Next.js example check confirms no cacheable response carries `Set-Cookie`. | P7 |
| TC-SES-13 | T09 | Under concurrent requests during rotation, no request fails spuriously and no more than the bounded number of tokens is valid. | P3 |
| TC-SES-14 | T03 | Sentinel tokens never appear in URLs, logs, audit events or telemetry. | P3 |
| TC-SES-15 | T06 | Development cookie mode refuses to start with an `https` or non-localhost public origin. | P3 |
| TC-SES-16 | T12 | Concurrent requests for many users never return another user's session. | P3 |
| TC-SES-17 | Tracker P3 | Rolling back a migration preserves the integrity of existing session rows. | P3 |
| TC-SES-18 | T01, T15 | The `@authlume/react` production bundle contains no server modules or server configuration. | P7 |
| TC-SES-19 | T14 | Only configured metadata fields are stored, and the retention purge removes expired metadata. | P3 |
| TC-SES-20 | T13 | Cleanup removes expired and revoked rows without touching active sessions. | P3 |
