# Application Components

## Current scope amendment (2026-10-26)

Authentication and sign-in of every kind are deferred. The current MVP
includes read-only public catalog discovery and tenant routing from bespoke
subdomains only. A host selects routing context, not authorization. The
protected-management and proxy components below are future-state designs.

## Design principles

- Deploy as one .NET modular monolith with a React/TypeScript UI and
  PostgreSQL. Modules communicate through application contracts; no module
  accesses another module's persistence internals.
- Enforce tenant scope server-side on every tenant-owned operation. Tenant
  context comes from validated identity or the matching tenant subdomain/API
  key pair, never from an untrusted body field.
- Keep the Development catalog/proxy endpoints separate from authenticated
  tenant and operator experiences.
- Keep provider-specific hosting and secret-manager code behind deployment
  adapters. The provider is not selected.

## Components

### API Host and Composition Root

**Purpose:** Configure the application, dependency graph, HTTP pipelines,
database, authentication, authorization, health checks, and route groups.

**Responsibilities:** Register module services, apply global error handling,
request limits, rate limiting and security headers, and expose management,
operator, consumer, and Development-only route groups.

### Identity and Access

**Purpose:** Authenticate management users through OIDC and authorize users
against their tenant membership or platform-operator role.

**Responsibilities:** Validate identity-provider tokens, resolve memberships
and roles, enforce server-side function/object authorization, and keep
operator privileges distinct from tenant-admin privileges.

### Tenancy and Tenant Provisioning

**Purpose:** Represent customer organizations and establish trusted tenant
context.

**Responsibilities:** Allow an authorized platform operator to provision a
tenant and its first Tenant Admin; resolve the tenant for management
requests; and validate agreement between the consumer request subdomain and
API-key tenant.

### Integration Catalog

**Purpose:** Own platform-managed integration definitions, allowed endpoint
templates, and OpenAPI documentation.

**Responsibilities:** Provide read access to tenant users; support catalog
management through the protected operator UI; validate catalog changes; and
provide definitions to tenant configuration and proxy modules.

### Tenant Integration Configuration

**Purpose:** Store each tenant's selected integrations and local settings.

**Responsibilities:** Enforce tenant scoping; allow Tenant Admins to add,
modify, and remove configurations; keep catalog-controlled data immutable to
tenants; and store only references to upstream secrets in application data.

### Secret Store Boundary

**Purpose:** Encapsulate encrypted persistence and retrieval of upstream
credentials.

**Responsibilities:** Provide tenant- and integration-scoped store, retrieve,
and delete operations through a provider-neutral interface. The hosting
provider's secret manager is selected later. The module never exposes
credentials to UI/API responses and never logs secret values.

### Consumer API Key Management

**Purpose:** Issue and validate user-scoped API keys within a tenant.

**Responsibilities:** Generate keys, return the usable secret only once to
its owner, store only a one-way hash, validate presented keys, support
owner-managed create/replace/revoke operations, and permit a Tenant Admin to
revoke keys within their tenant without retrieving the secret.

### Consumer Proxy

**Purpose:** Authenticate tenant consumers and call catalog-allowed upstream
endpoints using the tenant's configured integration and credentials.

**Responsibilities:** Validate the key/subdomain tenant match, authorize the
key owner, enforce endpoint allow lists, retrieve tenant-scoped credentials,
make one outbound request with a 30-second timeout, enforce the 0.5 MB
response limit, and map platform/upstream results to the approved response
contract. Do not retry or forward upstream response headers.

### Usage and Security Logging

**Purpose:** Record minimal request outcomes and security/application events
as distinct record types.

**Responsibilities:** Emit structured request-correlated logs without
credentials, API keys, or unnecessary personal data; create one minimal
success/failure usage record per API request; retain security/application
logs for at least 90 days; and keep access/retention rules subject to the
pre-production GDPR review.

### Management and Operator UI

**Purpose:** Provide the authenticated interface for tenant users and
platform operators.

**Responsibilities:** Tenant views support sign-in, membership/integration
management and user-owned API key management. Tenant Admins may revoke
member keys. A distinct operator view supports tenant bootstrap and catalog
maintenance. Enforce authorization at the API even when controls are hidden
in the UI.

The accepted production packaging direction builds and serves the UI with
the application. The current local Nginx UI container can remain a
development/deployment detail if it preserves that production boundary.

### Persistence and Infrastructure

**Purpose:** Persist application data and provide portable build/deployment
resources.

**Responsibilities:** PostgreSQL/EF Core stores tenant, membership,
catalog, tenant-configuration, API-key-hash, and usage metadata. Apply
reviewed migrations as a controlled step. Terraform and GitHub Actions
provide infrastructure and delivery; specific hosting resources remain
pending provider selection.

## Runtime boundaries

- OIDC identity provider is external; the API validates identity and applies
  local tenant/platform authorization.
- Only the proxy module reads upstream secrets, using a scoped secret-store
  contract.
- Catalog definitions and endpoint allow lists are platform-owned.
- The tenant-facing consumer API is authenticated and tenant-scoped.
- Development-only routes are not made public by production catalog seeding.
- Billing, payment, invoices, usage views, GitHub connector, and
  custom-domain support are outside MVP scope.
