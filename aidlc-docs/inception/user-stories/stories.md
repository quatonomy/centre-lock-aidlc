# MVP User Stories

Stories use the approved hybrid approach: journeys grouped under feature
epics, with persona labels. Acceptance criteria use Given/When/Then and target
small, independently verifiable outcomes. Stories express independent,
valuable user outcomes and remain negotiable, estimable, and testable
(INVEST); delivery dependencies do not combine separate outcomes.
Requirement references point to `aidlc-docs/inception/requirements/requirements.md`.

## Scope amendment (2026-10-26)

Authentication and sign-in are entirely deferred: no OIDC sessions, tenant
membership/roles, invitations, consumer API keys, or authenticated proxy calls
are in the current MVP. Stories explicitly marked **Deferred** are retained
for future planning but must not be implemented as current MVP scope. Current
in-scope outcomes are public read-only catalog discovery and tenant routing
based on bespoke subdomains beneath a configured shared base domain. A
subdomain selects a routing context only; it does not authenticate a caller,
authorize tenant data access, or permit upstream proxy execution.

## Epic 1: Tenant access and membership (deferred)

### US-01: Sign in to a tenant account

**MVP status: Deferred.** All authentication and sign-in are out of scope for
the current MVP. The original acceptance criteria below describe future
behavior only.

**As a** tenant user, **I want** to sign in through the configured identity
provider, **so that** I can access my tenant's management experience securely.

**Persona:** Tenant Admin; Tenant Member  
**Requirements:** FR-01, NFR-01, NFR-02

**Acceptance criteria**

- **Given** a user with an existing tenant membership and a valid Google
  identity, **when** the user completes OIDC sign-in, **then** the management
  experience is available only within that user's tenant and assigned role.
- **Given** an identity provider returns an invalid, expired, or untrusted
  authentication result, **when** sign-in is attempted, **then** access is
  denied.
- **Given** a signed-in user signs out, **when** the user attempts to reuse
  the prior session, **then** the session is no longer accepted.

### US-02: Invite a tenant user and assign a role

**MVP status: Deferred.** Membership, roles, invitations, and identity
verification depend on authentication, which is out of scope.

**As a** Tenant Admin, **I want** to invite a user and assign an admin or
view-only role, **so that** the right people can access my tenancy with
appropriate permissions.

**Persona:** Tenant Admin  
**Requirements:** FR-02, NFR-01, NFR-02

**Acceptance criteria**

- **Given** an authenticated Tenant Admin, **when** the admin invites a user
  and assigns one of the supported roles, **then** the invitation and role
  are associated only with that admin's tenant.
- **Given** a pending invitation, **when** it reaches 7 days after issue,
  **then** its redemption link is expired and cannot create a membership.
- **Given** a pending invitation, **when** the Tenant Admin resends it,
  **then** the old link is invalidated, a new 7-day link is issued, and the
  invitation's tenant and assigned role are unchanged.
- **Given** an invitee redeems an invitation, **when** their OIDC identity is
  verified, **then** its email must match the invited email before membership
  is created.
- **Given** an authenticated view-only Tenant Member, **when** they attempt
  to invite a user or change a role, **then** the operation is denied.
- **Given** a user is invited to one tenant, **when** that user accesses a
  different tenant, **then** the invitation grants no access to the other
  tenant.

### US-03: View tenant information without edit access

**MVP status: Deferred.** Tenant-private information must not be exposed
without authentication and authorization.

**As a** Tenant Member, **I want** to view the integration information
available to my tenancy, **so that** I can understand what my team can use
without having administrative access.

**Persona:** Tenant Member  
**Requirements:** FR-02, FR-03, NFR-01

**Acceptance criteria**

- **Given** an authenticated view-only member, **when** they view integrations
  and associated OpenAPI documentation, **then** only information available
  to their tenant is shown.
- **Given** a view-only member, **when** they attempt to modify tenant
  settings, credentials, membership, or roles, **then** the operation is
  denied server-side.
- **Given** a view-only member, **when** they attempt to read another
  tenant's resources, **then** no other tenant's data is returned.

## Epic 2: Discover and configure integrations

### US-04: Discover available integrations and documentation

**As a** visitor, **I want** to see the platform-published integrations and
their OpenAPI documentation, **so that** I can understand the available
options.

**MVP status: In scope as public, non-tenant-specific catalog discovery.**  
**Persona:** Visitor  
**Requirements:** FR-03, FR-11, NFR-01

**Acceptance criteria**

- **Given** a visitor browses the public catalog, **when** integrations are
  listed, **then** only platform-published integrations are returned and no
  tenant-specific settings or credentials are included.
- **Given** the catalog is seeded, **when** a visitor views available
  integrations in any environment where the catalog UI is deployed, **then**
  the Star Wars API is available as the trial integration.
- **Given** a visitor opens integration documentation, **when** the
  documentation is returned, **then** it corresponds to the selected
  integration and contains no tenant credentials.
- **Given** a visitor attempts to edit a catalog definition or endpoint
  allow list, **when** the request is processed, **then** no public write
  operation is available.

### US-05: Configure an integration for my tenant

**MVP status: Deferred.** Tenant configuration and credential management
require authenticated, tenant-scoped authorization.

**As a** Tenant Admin, **I want** to add, update, or remove an available
integration and manage its tenant-local settings, **so that** my team can use
the integration with our account.

**Persona:** Tenant Admin  
**Requirements:** FR-03, FR-04, NFR-01, NFR-02

**Acceptance criteria**

- **Given** an integration is published in the platform catalog, **when** a
  Tenant Admin adds it, **then** the configuration is attached only to that
  tenant.
- **Given** an integration is configured for a tenant, **when** an
  authorized admin updates or removes it, **then** no other tenant's
  configuration is changed.
- **Given** the configuration includes an upstream credential, **when** it
  is saved or used by the proxy, **then** the credential is encrypted,
  tenant-scoped, and never returned through the management UI or API after
  setup.
- **Given** a user lacks edit permission, **when** they attempt to change
  tenant integration settings, **then** the change is denied.
- **Given** an integration's endpoints are configured by the platform,
  **when** a tenant edits its local settings, **then** the catalog definition
  and endpoint allow list remain unchanged.

### US-06: Maintain the platform integration catalog

**MVP status: Deferred.** Catalog mutation and publication administration
require a separately approved operator authentication mechanism. The current
MVP may serve already-published/seeded catalog entries read-only.

**As a** Platform Operator, **I want** to maintain integration definitions,
endpoint allow lists, and OpenAPI documentation, **so that** tenants can use
only integrations and endpoints approved by the platform.

**Persona:** Platform Operator  
**Requirements:** FR-03, NFR-01, NFR-02

**Acceptance criteria**

- **Given** a platform operator is authorized for catalog administration,
  **when** they publish or update an integration definition, **then** the
  published definition and allowed endpoints are available to tenants.
- **Given** a tenant user attempts a catalog write, **when** the request is
  processed, **then** it is denied regardless of client-side UI state.
- **Given** an endpoint is not in the catalog allow list, **when** a consumer
  requests it, **then** the proxy does not call the upstream endpoint.

**Design note:** The exact operator identity and catalog administration
workflow are not yet defined; acceptance assumes an explicitly authorized
platform operator and does not prescribe a UI or API.

## Epic 3: Tenant routing and future consumer proxy

### US-07: Create and manage consumer API keys

**MVP status: Deferred.** Consumer API-key authentication and key lifecycle
are out of scope along with all other authentication.

**As a** tenant API Consumer, **I want** a revocable API key scoped to my user
and tenant, **so that** my application can authenticate to the proxy.

**Persona:** Tenant API Consumer  
**Requirements:** FR-05, FR-10, NFR-01, NFR-02

**Acceptance criteria**

- **Given** an authenticated tenant user creates an API key through the UI,
  **when** the key is issued, **then** it is scoped to that user and tenant
  and the usable secret is displayed only at creation.
- **Given** a key is stored, **when** key data is inspected in persistent
  storage, **then** only a one-way hash is stored.
- **Given** a key is revoked or replaced, **when** the former key is used,
  **then** authentication fails.
- **Given** a Tenant Admin manages their tenant, **when** they revoke another
  tenant user's key, **then** that key stops authenticating and the admin
  cannot retrieve its usable secret.
- **Given** a Tenant Admin attempts to revoke a key from another tenant,
  **when** the request is processed, **then** it is denied.
- **Given** the configured rotation interval elapses (12 months by default),
  **when** key rotation is due, **then** the key can be replaced and the
  rotation interval can be changed through configuration.

### US-08: Resolve a tenant from its bespoke subdomain

**As a** visitor, **I want** a tenant-specific subdomain to select its tenant
routing context, **so that** tenant routes can be established without
confusing one tenant's address with another.

**MVP status: In scope for routing only. This does not grant access to tenant
data or enable proxy calls.**  
**Persona:** Visitor  
**Requirements:** FR-12, NFR-01

**Acceptance criteria**

- **Given** a request uses a configured tenant subdomain beneath the
  platform base domain, **when** the host is resolved, **then** it selects
  only the matching tenant-routing context.
- **Given** a request uses an unknown, malformed, reserved, or unconfigured
  subdomain, **when** the host is resolved, **then** it is rejected without
  selecting another tenant.
- **Given** a request resolves to a tenant from its subdomain, **when** the
  caller requests tenant-private data or an upstream integration, **then**
  routing alone does not authorize the request and no upstream call is made.
- **Given** a caller uses a custom domain, **when** the request is routed,
  **then** custom-domain mapping is not supported in the MVP.

**Future requirement:** Before tenant proxy calls are enabled, consumer
authentication must be approved and implemented; the tenant subdomain and
tenant-scoped API key must then identify the same tenant.

### US-09: Receive a completed upstream response

**MVP status: Deferred.** Tenant-facing proxy calls are disabled until
authentication is in scope and implemented.

**As a** tenant API Consumer, **I want** a consistent proxy response that
contains the upstream result, **so that** my application can inspect the
integration response.

**Persona:** Tenant API Consumer  
**Requirements:** FR-06, FR-08, FR-09

**Acceptance criteria**

- **Given** an upstream request completes with any HTTP status, **when** the
  proxy returns the completed response, **then** the platform HTTP status is
  200 and the upstream status and body are included under a top-level
  `connector` object.
- **Given** an upstream response contains valid JSON, **when** it is returned
  under `connector`, **then** the body is represented as JSON.
- **Given** an upstream response is not valid JSON, **when** it is returned
  under `connector`, **then** the body is represented as text.
- **Given** an upstream response includes headers, **when** the platform
  responds, **then** upstream response headers are not forwarded.
- **Given** an upstream body exceeds 0.5 MB, **when** it is received, **then**
  the over-limit body is not returned as a normal completed proxy response.

### US-10: Understand platform-side proxy failures

**MVP status: Deferred.** The proxy and its authentication-dependent error
contract are not exposed in the current MVP.

**As a** tenant API Consumer, **I want** platform-side failures to use
distinguishable HTTP status codes and a JSON error object, **so that** my
application can respond appropriately.

**Persona:** Tenant API Consumer  
**Requirements:** FR-07, FR-09

**Acceptance criteria**

- **Given** a request is invalid, **when** the platform rejects it, **then**
  it returns HTTP 400 and a JSON platform error object.
- **Given** the API key is missing or invalid, **when** authentication
  rejects the request, **then** it returns HTTP 401 and a JSON platform error
  object.
- **Given** the caller is authenticated but unauthorized, **when** access is
  rejected, **then** it returns HTTP 403 and a JSON platform error object.
- **Given** an upstream transport failure occurs, **when** the platform
  returns the failure, **then** it uses HTTP 502 and a JSON platform error
  object.
- **Given** an upstream request exceeds the 30-second timeout, **when** the
  platform returns the failure, **then** it uses HTTP 504 and indicates that
  a timeout occurred.
- **Given** an upstream call fails or times out, **when** the platform
  handles the failure, **then** it does not automatically retry the request.

## Epic 4: Minimal operational records

### US-11: Record minimal request outcomes safely

**MVP status: Deferred with consumer proxy execution.** No tenant proxy
requests are handled in the current MVP.

**As a** Platform Operator, **I want** a minimal success/failure record for
each API request, **so that** request outcomes can be understood without
collecting unnecessary sensitive data.

**Persona:** Platform Operator  
**Requirements:** NFR-02, NFR-03

**Acceptance criteria**

- **Given** an API request is processed, **when** request recording occurs,
  **then** one minimal record identifies whether it succeeded or failed.
- **Given** a request record is written, **when** its contents are reviewed,
  **then** it contains no consumer API key, upstream credential, or other
  unnecessary sensitive data.
- **Given** security/application logs are retained, **when** retention is
  configured, **then** they are kept for at least 90 days and remain distinct
  from minimal request usage records.
- **Given** production customer traffic is planned, **when** production
  readiness is reviewed, **then** GDPR-compliant retention and access rules
  have been approved before traffic begins.

## Coverage and unresolved design points

- **Requirement coverage:** FR-01 through FR-11 and NFR-01 through NFR-03
  are represented in the stories or their acceptance criteria.
- **MVP exclusions:** Usage views, pricing, billing, payment collection,
  invoices, custom domains, and GitHub connector functionality are not
  included.
- **Design decisions:** Initial tenant/admin provisioning, exact catalog
  operator workflow, exact JSON platform error-object properties, over-limit
  response handling/status, and the hosting-provider secret manager require
  definition during design. These stories do not assume their implementation
  details.
- **Production gates:** Exact RTO/RPO values and GDPR retention/access rules
  must be approved before production customer traffic.
