# Security Policy

Authlume is authentication software, so we treat security reports as our top
priority. Thank you for helping keep Authlume and its users safe.

## Project status

Authlume is in the **planning phase**. No packages have been released and no
version is suitable for production. This policy applies now so that the
reporting channel exists before any code ships.

## Supported versions

| Version | Supported |
| --- | --- |
| No releases yet | Not applicable |

This table will list supported release lines once packages are published. The
policy for how long each line receives security fixes will be published before
the public beta (Phase P9).

## Reporting a vulnerability

**Do not report security vulnerabilities through public GitHub issues,
discussions or pull requests.**

Report privately through GitHub's private vulnerability reporting:

1. Go to the repository's **Security** tab.
2. Select **Report a vulnerability**.
3. Fill in the advisory form.

Direct link: <https://github.com/authlume/authlume/security/advisories/new>

If that form is unavailable, open a public issue titled "Request for private
security contact" **without any vulnerability details**, and a maintainer will
arrange a private channel.

A dedicated security email address will be added here once it has been set up.

### What to include

- The affected package, file or design document, and the version or commit.
- A description of the issue and its impact: what an attacker can do, and under
  what conditions.
- Step-by-step reproduction, a proof of concept, or the relevant section of the
  design if the issue is in a specification.
- Any suggested mitigation.

Please do not include real user data, credentials or secrets in your report.

## What happens next

While Authlume is pre-release, it is maintained on a best-effort basis by a
small team. We will:

1. Acknowledge your report as soon as practical.
2. Confirm or dispute the issue and agree on a severity with you.
3. Develop and review a fix privately. Fixes to authentication, cryptography,
   recovery or session handling require independent human security review.
4. Publish a GitHub Security Advisory, requesting a CVE where appropriate, and
   credit you unless you prefer to remain anonymous.

Formal response-time targets will be published before the public beta.

We follow coordinated disclosure. Please give us a reasonable opportunity to
release a fix before you disclose publicly, and let us know your intended
timeline.

## Scope

In scope:

- Code, configuration and release artifacts in this repository.
- Published `@authlume/*` npm packages, once they exist.
- Design flaws in the threat model or specifications that would lead to an
  insecure implementation.

Out of scope:

- Vulnerabilities in third-party dependencies that do not affect Authlume.
  Please report those to the upstream project.
- Problems caused by host-application configuration that Authlume documents as
  the developer's responsibility, such as TLS, DNS, email delivery, key custody,
  database backups and rate-limit infrastructure.
- Social engineering, physical attacks and volumetric denial of service.

## Good-faith research

We will not pursue legal action against anyone who researches and reports
vulnerabilities in good faith under this policy: by avoiding privacy
violations, data destruction and service disruption, by testing only against
systems you own or have permission to test, and by giving us reasonable time
to respond before disclosure.

Authlume does not currently run a bug bounty programme.

## Security design principles

The security invariants Authlume is built against are in
[section 5 of the Master Blueprint](AUTHLUME_MASTER_BLUEPRINT.md#5-threat-model-and-security-invariants).
They include: no custom cryptography, established Argon2id password hashing,
opaque hashed session tokens, single-use expiring secrets, and mandatory
independent review of security-critical code.
