# Phase P0 verification evidence

This log records how each P0 account and reservation item was checked, and
what the result was. It backs the evidence column of the
[Phase Tracker](../../AUTHLUME_PHASE_TRACKER.md). An AI agent ran the checks on
**2026-10-23, between 08:20 and 08:35 UTC**, from a local development machine,
using the GitHub CLI authenticated as `9jaDevo` and unauthenticated HTTPS
requests.

Results are reported as observed. Where something could not be verified, this
log says so.

## 1. GitHub organization `authlume`

```text
$ gh api orgs/authlume
login: authlume   id: 346786269   type: Organization
created_at: 2026-10-23T07:38:09Z
plan: free
two_factor_requirement_enabled: false
public_repos: 0   total_private_repos: 0
```

```text
$ gh api user/memberships/orgs/authlume
user: 9jaDevo   state: active   role: admin
```

```text
$ gh api orgs/authlume/members          ->  9jaDevo (1 member)
$ gh api orgs/authlume/invitations      ->  1 pending invitation (direct_member, not yet accepted)
$ gh api orgs/authlume/teams            ->  none
```

**Result:** the organization exists, and `9jaDevo` is its only member and an
active admin. The organization 2FA requirement is **off**. The pending invitee
is not a verified maintainer, so is not listed in `CODEOWNERS`.

## 2. Primary repository `authlume/authlume`

```text
$ gh api repos/authlume/authlume
{"message":"Not Found","status":"404"}

$ gh repo list authlume
(no repositories)
```

**Result:** the repository **did not exist** when checked. Creating it needs the
owner's explicit authorization. Its creation, if authorized, is recorded in the
Phase Tracker.

## 3. npm organization `authlume` and scope `@authlume`

The registry's organization endpoint distinguishes existing scopes from
missing ones. A control request for a made-up scope returns `404 Scope not found`:

```text
$ curl https://registry.npmjs.org/-/org/authlume/package
HTTP 200  {}                               # organization exists, 0 packages

$ curl https://registry.npmjs.org/-/org/zzq-nonexistent-org-8f3k2x/package
HTTP 404  {"error":"Scope not found"}      # control
```

```text
$ curl https://registry.npmjs.org/@authlume%2f<name>
core, server, postgres, node, next, react, access-code  ->  HTTP 404 each (not published)

$ curl https://registry.npmjs.org/authlume
HTTP 404                                   # unscoped name unclaimed

$ curl "https://registry.npmjs.org/-/v1/search?text=scope:authlume"
{"objects":[],"total":0}
```

```text
$ npm whoami
npm error code ENEEDAUTH                   # npm CLI not logged in on this machine
```

**Result:** the npm organization `authlume` exists and has published nothing.
**Not verified:** who the organization's members and owners are, and whether
2FA is enforced. The agent deliberately did not log in to npm. The founder
should confirm ownership with:

```sh
npm org ls authlume
npm profile get
```

The first command should list the founder's npm account with the `owner` role.
The second shows whether 2FA is turned on for the founder's npm account.

## 4. Domain `authlume.org`

```text
$ curl https://rdap.publicinterestregistry.org/rdap/domain/authlume.org
HTTP 404  {"errorCode":404,"title":"Object not found"}

$ nslookup authlume.org
Non-existent domain (NXDOMAIN)
```

**Result:** `authlume.org` is **not registered**, so the founder does not own
it. Requirement status: pending founder registration.

## 5. Legal and trademark review

**Result:** no trademark or similarity search was performed, and no review or
legal opinion was provided as evidence. Status: not started. Nothing in this
repository claims trademark clearance. See
[ADR-010](../adr/0010-legal-trademark-and-npm-scope.md).

## 6. License text

```text
$ curl https://www.apache.org/licenses/LICENSE-2.0.txt | sha256sum
cfc7749b96f63bd31c3c42b5c471bf756814053e847c10f3eb003417bc523d30

$ sha256sum LICENSE
cfc7749b96f63bd31c3c42b5c471bf756814053e847c10f3eb003417bc523d30
```

**Result:** `LICENSE` is byte-identical to the canonical Apache License 2.0 text
published by the Apache Software Foundation.

## 7. Other registries

```text
$ curl "https://packagist.org/packages/list.json?vendor=authlume"
{"packageNames":[]}
```

**Result:** no Packagist packages use the `authlume` vendor name. The name is
not reserved, which is expected because PHP is out of scope for V1.
