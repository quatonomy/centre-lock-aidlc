# Business Logic Model: CentreLock Application

## Current MVP scope amendment (2026-10-26)

All authentication and sign-in are deferred. Current implementation scope is
public catalog discovery and tenant host routing only. Identity, membership,
invitation, API-key, tenant-private management, and proxy workflows below are
future-state design, not authorization to implement them in the current MVP.
A subdomain identifies routing context only.

## Scope and boundaries

The model describes business behavior for the one-deployable-unit MVP. It
does not prescribe a hosting provider, secret-manager product, database
schema technology, or independently deployed service. Module ownership and
contracts are in the approved Application Design.

Business actors are Tenant Admin, Tenant Member (view-only), Tenant API
Consumer, and Platform Operator. Authentication and authorization are
server-enforced. A tenant identifier received from an untrusted request body
is never proof of access.

## Core concepts and invariants

- A **Tenant** is the organization-level isolation boundary.
- A **Membership** links a user to one tenant with an extensible role;
  initial roles are Admin and view-only Member.
- A **Pending Invitation** binds an email, tenant, assigned role, expiry,
  and single-use redemption token.
- A **Catalog Integration** is platform-owned and includes endpoint
  allow-list definitions and OpenAPI documentation.
- A **Tenant Integration Configuration** selects one catalog integration
  per tenant and contains tenant-local settings plus a secret reference.
- An **API Key** belongs to one user in one tenant. Multiple keys may be
  active for a user. The raw key is returned only at creation/replacement;
  persistent data contains only a one-way hash and non-secret metadata.
- A **Proxy Request** is authorized only when its tenant subdomain and API
  key resolve to the same tenant, the key is active, and the requested
  integration endpoint is allowed.
- A **Usage Outcome** records a minimal success/failure result. It is
  distinct from security/application logs.
- Tenant-owned resources and secrets never cross tenant boundaries.

## Workflows

### Operator provisions the first tenant admin

1. An authenticated principal requests tenant provisioning.
2. Server-side authorization verifies a platform-operator role.
3. The system creates the tenant and an initial Admin membership as one
   logical operation.
4. The operator receives non-secret provisioning confirmation.
5. A non-operator is denied; no tenant or membership is created.

### Tenant admin invites a user

1. Authenticate the Tenant Admin and resolve their trusted tenant context.
2. Validate invitee email and requested initial role (Admin or view-only
   Member).
3. Create a pending invitation scoped to that tenant, expiring 7 days after
   issue.
4. Deliver a redemption link using a single-use token; do not log the token.
5. On redemption, require a valid OIDC identity with a verified email that
   matches the invitation.
6. Reject expired, revoked, already-consumed, mismatched-email, or
   otherwise-invalid tokens without creating membership.
7. Atomically consume a valid invitation and create membership in its
   recorded tenant and role.

### Tenant admin resends or revokes an invitation

- **Resend:** Only a pending invitation in the admin's tenant may be resent.
  Invalidate the previous token, issue a new token with a fresh 7-day expiry,
  and retain the invitation's tenant and role.
- **Revoke:** Mark a pending invitation revoked and invalidate its token.
- Expired, revoked, or consumed invitations are not redeemable.

### Tenant user discovers and configures integrations

1. Authenticate the user and resolve tenant membership.
2. List only published catalog integrations and their associated
   documentation.
3. A Tenant Admin may add one configuration per catalog integration to
   their tenant, update its tenant-local settings, or remove it.
4. The system rejects a second configuration for the same tenant/integration.
5. Catalog definitions, allow lists, and documentation remain
   platform-owned; tenant updates cannot mutate them.
6. Credentials are written through the secret-store boundary; application
   data retains only a scoped reference. Responses never return a usable
   credential.
7. View-only members may read permitted integration information but cannot
   mutate configuration.

### Platform operator manages catalog

1. Authenticate the operator and authorize catalog administration.
2. Validate integration metadata, endpoint templates, and OpenAPI document.
3. Create or update catalog data and publish it through the protected
   operator UI workflow.
4. Only published integrations and allow-listed endpoint definitions are
   available to tenant configuration and proxy resolution.
5. Tenant principals cannot invoke catalog write operations, regardless of
   UI visibility.

### User creates, replaces, and revokes consumer keys

1. Authenticate the user and resolve tenant context.
2. Generate a high-entropy key and store only its one-way hash and metadata.
3. Return the usable key once to the owner; never return it on subsequent
   reads or to a Tenant Admin.
4. Permit multiple active keys for one user. Replacement issues a new active
   key; the owner may verify it before revoking the previous key.
5. Permit the owner to revoke their own key. Permit a Tenant Admin to revoke
   any key within that admin's tenant, but not to inspect or retrieve its
   secret.
6. Enforce the default 12-month rotation interval, configurable for future
   policy changes.
7. Revoked, expired, malformed, or unknown keys do not authenticate.

### Consumer invokes an integration

1. Validate request syntax, method/path, query, body size, and required
   fields.
2. Resolve the tenant from the trusted tenant subdomain and presented key.
   Require both to identify the same tenant and require the key owner to
   remain an authorized tenant member.
3. Resolve exactly one tenant configuration for the selected catalog
   integration.
4. Match method and normalized path against the platform-owned allow list.
   Reject before outbound network access when not allowed.
5. Retrieve the tenant-scoped upstream credential via its secret reference,
   if the integration requires one.
6. Send one upstream request with a 30-second timeout; do not automatically
   retry.
7. If the upstream responds within the limit, return platform HTTP 200 with
   upstream status and body under `connector`. Do not forward upstream
   response headers. Parse valid JSON as JSON; otherwise represent the body
   as text.
8. If the upstream response exceeds 0.5 MB, stop processing and return HTTP
   502 with a platform JSON error; never return a truncated body as a
   successful connector response.
9. Record one minimal success/failure usage outcome and emit correlated,
   redacted security/application logs.

### Platform-side failure mapping

| Condition | Platform status | Response |
|---|---:|---|
| Invalid request | 400 | JSON platform error object |
| Missing/invalid/revoked/expired API key | 401 | JSON platform error object |
| Authenticated but unauthorized | 403 | JSON platform error object |
| Upstream transport failure or upstream response over the size cap | 502 | JSON platform error object |
| Upstream timeout after 30 seconds | 504 | JSON platform error object indicating timeout |

The agreed envelope is `{"error":{"code":"...","message":"..."}}`. Error
codes are stable machine-readable identifiers; messages are safe and do not
expose stack traces, credentials, internal paths, or upstream secrets.
Detailed error-code taxonomy remains for design/implementation.

## Testable properties (PBT-01)

| Component/logic | Property category | Property to verify |
|---|---|---|
| Tenant resolver | Invariant | A request is authorized only when validated key tenant equals trusted subdomain tenant; mismatches never resolve to either tenant's resources |
| Tenant-scoped repositories/services | Invariant | An operation with tenant context T can return or mutate only records owned by T |
| Invitation token lifecycle | Invariant | A token can produce at most one membership and only before expiry/revocation with a matching verified email |
| Invitation resend | Idempotence/invariant | After resend, the old token is invalid and the new token retains tenant/role scope and has a fresh expiry |
| Endpoint matcher | Invariant/oracle | Matching never authorizes a method/path outside the catalog allow list; compare normalization/matching to a simple reference matcher |
| API-key hashing/validation | Round-trip boundary | A generated raw key validates against its stored hash; different keys do not validate against that hash |
| API-key revocation | Invariant | Revoked keys never authenticate; admin revocation cannot affect keys outside the admin's tenant |
| Catalog/configuration operations | Idempotence/invariant | Reapplying an idempotent catalog seed does not duplicate entries; tenant configuration remains unique per tenant/integration |
| Error/response mapping | Invariant | Completed upstream responses map to HTTP 200; platform failure classes map to their specified statuses and never expose upstream headers |
| Response size limiter | Invariant | Returned upstream body is never larger than 0.5 MB; over-limit data is not emitted as a truncated successful result |
| JSON/text decoder | Round-trip/invariant | Valid JSON remains structurally valid JSON; non-JSON content is represented as text without changing the status/body relationship |
| Request recording | Invariant | Each handled proxy request produces exactly one minimal outcome and no credential/key values |
| State-changing workflows | Stateful/model-based | Random invitation, membership, key, and configuration command sequences preserve role, tenant, uniqueness, and revocation invariants |

External OIDC providers, secret managers, PostgreSQL I/O, and upstream HTTP
systems are exercised with deterministic fakes in property tests and with
focused integration tests at their boundaries. Do not generate property-test
cases that call real external services.
