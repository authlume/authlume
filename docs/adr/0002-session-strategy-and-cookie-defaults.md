# ADR-002: Session strategy and cookie defaults

- **Status:** Proposed (drafted by an AI agent; awaiting owner decision)
- **Date:** 2026-10-23
- **Deciders:** project owner; independent security reviewer
- **Phase:** P1 (decision), P3 (implementation), P7 (framework integration)
- **Related:** Master Blueprint §§5 and 7; [sessions threat model](../threat-model/sessions.md); [ADR-013](0013-token-rotation.md) (rotation); ADR-006 (verifier storage, open)

## Context

Blueprint §5 requires:
- opaque random session tokens, hashed at rest where practical;
- `HttpOnly`, `Secure` and `SameSite` cookies as browser defaults;
- CSRF protection appropriate to the deployment;
- secure rotation and invalidation;
- configurable idle and absolute expiry;
- no `localStorage` token defaults.

The data model has
`sessions(id, user_id, token_hash, expires_at, revoked_at, metadata)`.

## Options

### Option A: opaque server-side sessions, validated on every request

- **Advantages:** revocation takes effect at once. A leaked database cannot be replayed if tokens are hashed. Simple to reason about.
- **Disadvantages:** one database read per authenticated request, plus periodic writes to track idle time.
- **Security consequences:** the best fit for SES-T02 and SES-T07.
- **Migration cost:** none.

### Option B: stateless signed tokens (for example JWT) in a cookie

- **Advantages:** no database read per request.
- **Disadvantages:** revocation needs a deny list, which brings back the database read. Signing keys become critical secrets. The Blueprint asks for opaque tokens.
- **Security consequences:** revoked or stolen tokens stay valid until they expire (SES-T07). Key compromise allows forging any session.
- **Migration cost:** high if adopted and later reversed.

### Option C: opaque sessions with a short in-process validation cache

- **Advantages:** fewer database reads.
- **Disadvantages:** revocation is delayed by the cache TTL, and caches differ between instances.
- **Security consequences:** RR-SES-03.
- **Migration cost:** low. It can be added to Option A as an option.

## Decision

Pending owner decision.

## Rationale (agent recommendation: Option A, with Option C as an explicit opt-in that is off by default)

### Token

- 32 bytes from the CSPRNG, base64url-encoded.
- Stored as `token_hash = SHA-256(token)`, which is enough for a 256-bit random value. Formal verifier rules are in ADR-006.
- Looked up by `token_hash`.
- The session `id` is a separate, non-secret identifier, used for "list and revoke my sessions".

### Cookie defaults

| Attribute | Default | Reason |
| --- | --- | --- |
| Name | `__Host-authlume_session` (configurable suffix) | The `__Host-` prefix forces `Secure` and `Path=/` and forbids `Domain`, which blocks cookie tossing from sibling subdomains (SES-T05) |
| `HttpOnly` | On, and cannot be disabled | SES-T01 |
| `Secure` | On | SES-T06 |
| `SameSite` | `Lax`; `Strict` is configurable | `Lax` keeps users signed in when they arrive from external links. `Strict` is safer but breaks that. CSRF checks do not rely on `SameSite` alone. |
| `Path` | `/` | Required by `__Host-` |
| `Domain` | Not set | Required by `__Host-` |
| `Max-Age` | The remaining absolute lifetime | A convenience only. Expiry is enforced on the server. |

**Development mode:** an explicit `insecureDevelopmentCookies` flag drops the
prefix and `Secure`, for plain-HTTP local development. Configuration validation
refuses it when the configured public origin is `https` or not localhost, and
a warning is logged at startup (XC-T05, TC-SES-15).

### Expiry defaults

These are proposed values for the owner to confirm. Both limits are
configurable and both are enforced on the server with the database clock.

| Profile | Idle timeout | Absolute lifetime |
| --- | --- | --- |
| Default | 7 days | 30 days |
| High assurance (preset) | 30 minutes | 12 hours |

The idle timestamp (`last_seen_at`) is updated at most once per configurable
interval (default 5 minutes), to limit write load.

### CSRF

The layered policy from SES-T04:
1. `SameSite`.
2. For every state-changing method, require `Sec-Fetch-Site: same-origin`, or an `Origin` that matches the configured origins. Reject when neither is present, by default.
3. No state changes on safe methods.
4. An optional synchronizer token.

It applies to sign-in and registration as well. Framework adapters must not
weaken it. For example, the Next.js adapter keeps the checks even where the
framework has its own.

### Validation and caching

Option A by default. Option C requires explicit configuration of a maximum
staleness of 60 seconds or less (proposed), and the documentation must
describe the revocation delay.

### Metadata

The default is a coarse user-agent family and a truncated IP address (IPv4
/24, IPv6 /48), for "your sessions" screens. Both are configurable down to
nothing. Metadata is purged with the session (SES-T14).

### Out of scope for V1

Bearer tokens in an `Authorization` header for non-browser clients. Adding
them needs a new ADR.

## Security consequences

Not reviewed. This addresses SES-T01 to T08 and T10 to T15. RR-SES-01 (host XSS
can use the session while the page is open) and RR-SES-02 (device malware can
copy cookies) remain.

## Migration cost

None before the first release. Changing the cookie name or the token format
later logs out every user once, unless both formats are accepted for a
transition window.

## Follow-up

- Owner: confirm option A or C, the expiry defaults, the `SameSite` default and the metadata defaults.
- ADR-006: formalize SHA-256 for high-entropy tokens.
- [ADR-013](0013-token-rotation.md): rotation policy.
- P3: implement TC-SES-01 to TC-SES-17 and TC-SES-19 to 20. P7: TC-SES-09 to 12 and TC-SES-18.
