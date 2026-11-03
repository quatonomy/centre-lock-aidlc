# Functional Design Plan: CentreLock Application

## Current scope amendment (2026-10-26)

All authentication and sign-in, including consumer API keys, are deferred.
The approved functional design below is historical future-state detail where
it covers authentication, tenant-private management, or proxy access. Current
scope is public read-only catalog discovery and tenant routing from bespoke
subdomains only. Do not treat a hostname as authorization.

## Scope

Define technology-agnostic business behavior for the single
`centre-lock-application` unit. Use approved requirements, stories,
application design, and unit artifacts. Preserve their boundaries:

- Tenant Admin invitations and role assignment; view-only member access.
- Operator-provisioned first tenant/admin and protected catalog management.
- Tenant-specific integration configuration and secret references.
- User-owned API-key lifecycle with tenant-admin revocation.
- Tenant subdomain/API-key agreement, endpoint allow lists, upstream
  response mapping, and minimal request outcomes.
- Development-only routes remain distinct from authenticated tenant routes.

Security, logging, response limits, and errors must fail closed and must not
expose credentials, API keys, or sensitive internal details.

## Functional questions

Please answer each `[Answer]:` tag. These choices affect business rules,
entities, and acceptance scenarios.

### Question 1: Tenant invitation lifecycle
How should tenant-admin invitations expire and be redeemed?

A) Expire after 7 days; admins may revoke or resend; redemption requires a
verified OIDC email matching the invitation (recommended)

B) Expire after 30 days; admins may revoke or resend; redemption requires a
verified OIDC email matching the invitation

C) Do not expire automatically; admins may revoke or resend; redemption
requires a verified OIDC email matching the invitation

X) Other (describe expiry, revocation/resend, and identity matching after
`[Answer]:` below)

[Answer]: A) Expire after 7 days; admins may revoke or resend; redemption requires a verified OIDC email matching the invitation

### Question 2: Integration configuration cardinality
Can a tenant configure more than one account/instance of the same catalog
integration?

A) One tenant configuration per catalog integration (recommended for MVP)

B) Multiple configurations per catalog integration, each with a distinct
tenant-local identity/name

X) Other (describe the cardinality and how instances are distinguished after
`[Answer]:` below)

[Answer]: A) One tenant configuration per catalog integration

### Question 3: Active consumer API keys
How should key replacement support rotation without disrupting consumers?

A) Allow multiple active keys per user; replacement keys may be verified
before the previous key is revoked (recommended for non-disruptive rotation)

B) Allow exactly one active key per user; replacement immediately invalidates
the previous key

X) Other (describe the active-key limit and rotation behavior after
`[Answer]:` below)

[Answer]: A) Allow multiple active keys per user; replacement keys may be verified before the previous key is revoked

### Question 4: Oversized upstream response and error shape
The response limit is 0.5 MB. The application design proposed rejecting an
oversized upstream body with HTTP 502 rather than truncating it. It also
proposed a JSON error object with `error.code` and a safe `error.message`.

A) Accept HTTP 502 for an oversized upstream response, do not return a
truncated body, and use `{"error":{"code":"...","message":"..."}}` for
platform errors (recommended)

B) Accept the error-object shape but choose another status/behavior for
oversized responses; describe it after `[Answer]:` below

C) Define a different platform error-object schema and oversized-response
behavior after `[Answer]:` below

[Answer]: A) Accept HTTP 502 for an oversized upstream response, do not return a truncated body, and use `{"error":{"code":"...","message":"..."}}` for platform errors

## Functional design execution checklist

- [x] Analyze answers for ambiguity and resolve material gaps.
- [x] Model the tenant, user, membership, invitation, catalog, tenant
  configuration, secret reference, API key, proxy request/result, and usage
  outcome concepts and relationships.
- [x] Specify workflows and business rules for sign-in, tenant provisioning,
  invitations, catalog publishing, tenant configuration, keys, proxy calls,
  and request recording.
- [x] Specify validation, authorization, errors, timeout, response-size, and
  alternative-flow behavior.
- [x] Identify testable properties by category for each unit with logic, per
  PBT-01.
- [x] Generate `business-logic-model.md`.
- [x] Generate `business-rules.md`.
- [x] Generate `domain-entities.md`.
- [x] Generate `frontend-components.md` for tenant and operator UI workflows.
- [x] Validate every story and approved requirement is represented; record
  remaining design/production gates without inventing answers.
- [x] Present the functional design for user approval.

## Planned artifacts

- `aidlc-docs/construction/centre-lock-application/functional-design/business-logic-model.md`
- `aidlc-docs/construction/centre-lock-application/functional-design/business-rules.md`
- `aidlc-docs/construction/centre-lock-application/functional-design/domain-entities.md`
- `aidlc-docs/construction/centre-lock-application/functional-design/frontend-components.md`

## Completion gate

Functional Design is complete only after answers are analyzed, artifacts are
generated, and the user explicitly approves them. NFR Requirements follows
for this unit.

Additional clarification: resending a pending invitation invalidates the old
link and issues a new link with a fresh 7-day expiry; tenant and assigned
role remain unchanged.

**Status:** Functional design artifacts are generated and awaiting user
review and approval.

**Clarification answers:** A = 7-day invitation expiry with verified OIDC
email match; one integration configuration per tenant/catalog integration;
multiple active consumer API keys allowed; oversized upstream response is
HTTP 502 without truncation; platform errors use the agreed
`error.code`/`error.message` JSON envelope. Resending replaces the old link
and starts a new 7-day expiry while retaining tenant and role.
