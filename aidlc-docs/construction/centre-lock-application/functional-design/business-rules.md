# Business Rules: CentreLock Application

## Current MVP scope amendment (2026-10-26)

All authentication and sign-in are deferred. Only public read-only catalog
discovery and tenant routing from bespoke subdomains are in scope. The
identity, membership, invitation, credential, and proxy rules below describe
future-state behavior and are not current MVP requirements. A subdomain
selects routing context only and does not authorize tenant-private data or
upstream calls.

## Identity, tenancy, and authorization

- **BR-01:** A tenancy is the customer organization isolation boundary.
- **BR-02:** Tenant access requires an authenticated identity with membership
  in that tenant. Resolve tenant context server-side; a caller-supplied
  tenant ID is not authorization.
- **BR-03:** Initial tenant roles are Admin and view-only Member. Keep role
  definitions extensible.
- **BR-04:** Only an authorized platform operator may provision a tenant and
  its first Admin.
- **BR-05:** Tenant Admin may invite users, assign supported roles, manage
  tenant integrations, and revoke API keys within the same tenant.
- **BR-06:** A Member may read permitted tenant integrations and
  documentation but may not mutate membership, roles, configuration, or
  another user's keys.
- **BR-07:** Platform operator permissions are distinct from tenant roles.
  Only an operator may manage catalog definitions, endpoint allow lists, and
  associated OpenAPI documents.
- **BR-08:** Authorization is checked server-side for every operation and
  resource. Deny by default and fail closed.

## Invitations

- **BR-09:** An invitation binds one normalized email, tenant, assigned role,
  expiry, state, and single-use token.
- **BR-10:** Invitation validity lasts 7 days from issue; expiry is enforced
  server-side at redemption.
- **BR-11:** Redemption requires a valid OIDC identity with a verified email
  matching the invited email.
- **BR-12:** Successful redemption atomically consumes the invitation and
  creates the membership in the invitation's tenant and role.
- **BR-13:** A pending invitation may be revoked by an authorized Tenant
  Admin in its tenant. Revocation invalidates the token.
- **BR-14:** Resending a pending invitation invalidates the prior token,
  issues a new token with a new 7-day expiry, and does not change tenant or
  role.
- **BR-15:** Expired, revoked, or consumed invitations cannot be redeemed.
  Invitation tokens are secrets and must not be logged or persisted in
  recoverable/plaintext form.

## Catalog and tenant integration configuration

- **BR-16:** Catalog definitions and endpoint allow lists are
  platform-managed. Tenant users cannot modify them.
- **BR-17:** Only published catalog integrations are available for tenant
  configuration and proxy requests.
- **BR-18:** A tenant may have at most one configuration for a given catalog
  integration.
- **BR-19:** Tenant configuration can modify tenant-local settings and
  credentials only; catalog-owned fields remain immutable to the tenant.
- **BR-20:** Upstream credentials are stored and retrieved through the
  tenant-scoped secret-store boundary. Application persistence stores only
  secret references; UI/API responses and logs never contain credential
  values.
- **BR-21:** Removing a tenant integration prevents its future use. Secret
  deletion/revocation must follow the secret-store contract and not leak the
  credential.

## Consumer API keys

- **BR-22:** A key is scoped to one user and one tenant. Tenant context for
  key operations comes from authenticated membership.
- **BR-23:** Store only a one-way hash of a high-entropy API key. Return the
  usable value only at key creation/replacement.
- **BR-24:** Users may create, replace, and revoke their own keys. Tenant
  Admins may revoke keys in their tenant but cannot read or retrieve the
  usable key.
- **BR-25:** Multiple keys may be active for one user to enable non-disruptive
  rotation. Replacing a key does not implicitly revoke existing keys.
- **BR-26:** Default rotation interval is 12 months and must be configurable.
  Enforcement and notices must not disclose key material.
- **BR-27:** Revoked, expired, unknown, malformed, or hash-mismatched keys
  fail authentication.

## Proxy authorization and forwarding

- **BR-28:** Tenant subdomain and API key must independently identify the
  same tenant. Reject missing, invalid, revoked, expired, or mismatched
  credentials before catalog lookup or outbound network access.
- **BR-29:** The key owner must remain authorized in the resolved tenant.
- **BR-30:** Forward only methods and normalized paths permitted by the
  platform catalog. Reject disallowed endpoints before outbound network
  access.
- **BR-31:** Retrieve credentials only for the resolved tenant and selected
  tenant integration. Do not permit tenant-controlled hosts, catalog
  definitions, or allow lists to redirect proxy traffic.
- **BR-32:** Proxy makes one outbound request per accepted consumer request;
  automatic retries are prohibited.
- **BR-33:** Upstream timeout is 30 seconds. Timeout responses use platform
  HTTP 504 and indicate timeout.
- **BR-34:** Maximum upstream response body is 0.5 MB. If exceeded, return
  HTTP 502 with a platform error; do not truncate and return a normal
  connector response.
- **BR-35:** For every completed upstream response within the size limit,
  return platform HTTP 200 with a top-level `connector` object containing
  upstream status and body, including upstream 4xx/5xx responses.
- **BR-36:** Do not forward upstream response headers. Parse valid JSON
  response bodies as JSON; otherwise represent them as text.

## Platform errors and sensitive information

- **BR-37:** Use HTTP 400 for invalid requests, 401 for missing/invalid API
  keys, 403 for authenticated but unauthorized requests, 502 for upstream
  transport/over-limit failures, and 504 for timeout.
- **BR-38:** Platform errors use the JSON envelope
  `{"error":{"code":"...","message":"..."}}`. Codes are stable identifiers;
  messages must be safe and generic. Exact code names remain for
  implementation design.
- **BR-39:** Production errors never expose stack traces, framework details,
  internal paths, database details, secrets, API keys, or raw exception
  messages.
- **BR-40:** Validate request types, formats, lengths, and payload size before
  processing. Use parameterized persistence operations; do not interpolate
  untrusted input into queries or commands.

## Usage and logging

- **BR-41:** Record one minimal success/failure outcome for each handled
  consumer proxy request. Do not include API-key values, upstream
  credentials, or unnecessary personal data.
- **BR-42:** Keep minimal usage records separate from security/application
  logs. Include correlation/request ID, timestamp, severity, and safe
  contextual information in structured logs.
- **BR-43:** Retain security/application logs for at least 90 days.
  GDPR-compliant retention, access, and permitted-use policy must be approved
  before production customer traffic.
- **BR-44:** Do not claim production readiness or a production SLA during MVP.
  Approve 99.95% monthly availability and exact Warm Standby RTO/RPO values
  before production customer traffic.

## Property-based test obligations

- **PBT-01:** Functional Design identifies properties for each logic-bearing
  component in `business-logic-model.md`.
- **PBT-02:** Use generated tenant, invitation, key, catalog path, response,
  and request data; generators must satisfy domain constraints and include
  relevant boundary cases.
- **PBT-03:** Test invariants for tenant isolation, single-use invitations,
  role enforcement, key revocation, catalog allow lists, bounded response
  data, and safe error mapping.
- **PBT-04:** Test idempotence for catalog seeding and any operation whose
  contract promises idempotency.
- **PBT-05:** Compare endpoint-template matching and normalization against a
  simple reference model where feasible.
- **PBT-06:** Stateful/model-based tests cover sequences of invitation,
  membership, API-key, and tenant-configuration operations.
- **PBT-07:** Generators are reusable, domain-specific, bounded, and include
  empty, boundary, and adversarial cases.
- **PBT-08:** Shrinking and reproducible seeds remain enabled and failures
  report the seed and minimal counterexample.
- **PBT-09:** Select and integrate an appropriate framework for applicable
  project languages during NFR Requirements; C# application logic requires a
  .NET PBT framework.
- **PBT-10:** PBT complements example-based regression and contract tests;
  it does not replace explicit story acceptance tests.
