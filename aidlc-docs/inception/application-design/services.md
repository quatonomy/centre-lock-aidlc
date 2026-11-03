# Application Services

## Current scope amendment (2026-10-26)

All authentication and sign-in are deferred, including OIDC and consumer
API keys. Current services are limited to public read-only catalog discovery
and tenant-subdomain routing. The authentication-dependent flows below
describe future-state behavior and must not be implemented as current MVP
features. Host-based routing never authorizes tenant-private data or proxy
execution.

## Request-level orchestration

The API host composes middleware and application services. Authentication
establishes a principal; authorization and tenant resolution establish a
trusted context before a module can read or mutate tenant-owned data.

## Service responsibilities

### OIDC Authentication and Authorization

1. Complete the configured OIDC flow and validate issuer, audience, signature,
   and expiration.
2. Resolve the authenticated principal and active membership/role.
3. Authorize the requested tenant and operation server-side.
4. Keep platform-operator access distinct from tenant roles.

### Tenant Provisioning

1. Require an authorized platform operator.
2. Create the tenant and initial Tenant Admin membership.
3. Persist tenant identity and membership through the tenancy boundary.
4. Return non-secret onboarding information to the operator UI.

### Catalog Administration

1. Authenticate a platform operator and enforce catalog-management permission.
2. Validate integration definition, allowed endpoint templates, and OpenAPI
   document.
3. Persist the platform-owned catalog change.
4. Make only published definitions available to tenant users and the proxy.

### Tenant Integration Management

1. Resolve authenticated tenant context.
2. Require Tenant Admin authorization for add/update/remove operations.
3. Verify the selected integration is published.
4. Store credentials through the secret-store boundary; persist only the
   returned secret reference and tenant-local settings.
5. Return configuration metadata without returning secret values.

### Consumer API-Key Lifecycle

1. Authenticate a tenant user for key creation/owner operations.
2. Generate a high-entropy key, store a one-way hash, and return the secret
   only at creation/replacement.
3. For revocation, authorize either the key owner or a Tenant Admin acting
   within the same tenant.
4. Never disclose another user's usable key to an administrator.
5. Enforce the configurable 12-month default rotation interval.

### Tenant-Facing Proxy

1. Validate request shape and size bounds.
2. Resolve tenant from the request subdomain and validate the presented key;
   reject any mismatch before upstream access.
3. Resolve the published integration and enforce the catalog endpoint
   allow list.
4. Retrieve that tenant's configured credentials through the secret-store
   interface.
5. Send one upstream request with a 30-second timeout and no automatic retry.
6. For a completed response, return HTTP 200 with upstream status/body under
   `connector`; do not forward upstream headers, parse valid JSON, otherwise
   return text, and enforce the 0.5 MB response limit.
7. For platform failures, use 400/401/403/502/504 as specified and return a
   JSON platform error object.
8. Record one minimal request outcome and emit correlated structured logs
   without secrets or API keys.

**Design proposal for review:** Treat an upstream response exceeding 0.5 MB
as a platform-side 502 error rather than truncating it; use a stable JSON
error code and safe message. The exact error schema is not fixed by
requirements.

### Management and Operator UI

- Tenant UI routes call the authenticated API for membership, integration
  configuration, documentation, and consumer API-key lifecycle.
- Operator UI routes call protected APIs for tenant bootstrap and catalog
  maintenance.
- UI affordances reflect roles, but API handlers remain the authorization
  enforcement point.
- Build/serve the UI with the application in the accepted production shape;
  preserve local Nginx behavior where useful for development.

## Cross-cutting behavior

- Centralize validation, authorization, exception mapping, correlation IDs,
  security headers, rate limiting, and structured logging in the API
  pipeline and module boundaries.
- Separate usage records from security/application logs. Keep secret and API
  key values out of logs.
- Apply database transactions within owning module services; use reviewed
  EF Core migrations as controlled deployment steps.
- Do not introduce direct module-to-module persistence access or separate
  deployable services.

## Deferred service integrations

- Hosting-provider secret manager implementation: select with the hosting
  provider, then implement the existing provider-neutral interface.
- Billing, payment, invoices, and usage presentation: outside MVP.
- GitHub connector: defer until use cases and permission scopes are defined.
