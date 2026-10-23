# Architecture overview

## System shape

centre-lock is a .NET modular monolith: one ASP.NET Core application contains
the management API and integration proxy, with a React/TypeScript management
UI and PostgreSQL database. The accepted technology and packaging choices are
recorded in [ADR 0001](decisions/0001-tech-stack.md).

The application is packaged as a portable container image. Docker is the
tooling for building and running containers locally; the image should follow
the OCI container image standard so it can run on compatible hosting
platforms. Docker Compose supports local development. Kubernetes is not
required for portability and is not part of the initial architecture; choose a
managed container runtime after evaluating hosting providers.

Architecture and delivery decisions are captured in
[ADR 0002](decisions/0002-architecture-and-delivery.md).

## Initial module boundaries

These are logical boundaries within the monolith, not independently deployed
services. Refine their interfaces as requirements become concrete while
preserving the rules below.

- **Identity and access:** Federated sign-in, tenant membership, roles, and
  server-side authorization.
- **Tenancy:** Organizations, tenant context, and tenant-owned settings.
- **Integration catalog:** Platform-managed integration definitions, endpoint
  allow lists, OpenAPI documentation, and declarative authentication settings,
  stored as PostgreSQL data.
- **Connectors:** A generic HTTP connector applies catalog definitions at
  runtime. Provider-specific code is reserved for protocols that cannot be
  safely expressed through the supported declarative settings.
- **Proxy:** Consumer authentication, tenant resolution, allow-list
  enforcement, request forwarding, timeout handling, response mapping, and
  usage recording.
- **Usage and billing:** Tenant-scoped usage records and data-driven plan and
  per-request pricing.
- **Application interfaces:** Management API and UI. Interfaces call module
  contracts; authorization is enforced by the server, not just by the UI.

Modules collaborate through explicit contracts or application services. A
module must not read or write another module's persistence directly or depend
on its internal implementation details. Keep the deployment and database
shared for the initial product.

## Boundaries and rules

- **Tenant isolation is mandatory.** A tenancy is one customer organization.
  Resolve tenant context from authenticated identity and trusted routing
  information at the application boundary; never treat a caller-supplied
  tenant ID as proof of access. Every operation on tenant-owned data must
  enforce tenant scope.
- **Enforce tenant scope in the application and database.** Use shared
  PostgreSQL tables with explicit tenant-scoped data access. Use PostgreSQL
  row-level security as defense in depth where it can be applied safely with
  connection pooling and tenant context. Database policies do not replace
  application authorization.
- **Keep tenant data distinct from platform-managed catalog data.** The
  platform operator owns integration definitions and endpoint allow lists.
  Tenants may configure their own use of an available integration but cannot
  change the catalog or its allow lists.
- **Keep credentials behind the proxy boundary.** The proxy uses a tenant's
  upstream credentials to call third-party services. Credentials must not be
  returned to consumers or exposed in logs. Consumer API keys authenticate
  calls to centre-lock and are distinct from upstream credentials.
- **Keep integration-specific behavior behind connector boundaries.** A
  connector handles communication and configured authentication for its
  third-party service. Prefer catalog data over provider-specific code for
  standard HTTP integrations. Shared tenant, authorization, and usage rules
  belong to the platform, not duplicated inconsistently across connectors.
- **Preserve the modular monolith.** Do not introduce independently deployed
  services without an accepted architecture decision.
- **Version API contracts deliberately.** The initial product supports one API
  version while keeping the routing and contract design able to introduce
  additional versions without an unplanned breaking change.
- **Record usage in the platform.** API usage is associated with the correct
  tenant so it can support usage views and future billing reports.
- **Keep operations portable.** Use Terraform for infrastructure as code and
  GitHub Actions for CI. Keep provider-specific infrastructure behind
  configuration and deployment boundaries until a hosting provider is
  selected.

## Delivery shape

- Run the app and dependencies locally with Docker Compose.
- Use isolated local, test, staging, and production environments. A pull
  request should get a temporary, isolated deployment with a unique URL and
  application resources so the deployed feature can be tested before merge.
  Provide a dedicated IP only where required and supported by the selected
  host. Do not use production data or credentials in these environments.
- Build an immutable container artifact. On merge to the main branch, deploy
  automatically through test and staging to production, with automated
  verification between stages. The main branch must remain releasable.
- Version database schema changes as reviewed EF Core migrations and apply
  them as a controlled deployment step.
- Before production use, select a host with managed database backup and
  restore capabilities and verify restoration. Recovery time and recovery
  point objectives remain to be decided.

## To define during implementation

- Concrete module interfaces and dependency direction.
- Tenant resolution, database access enforcement, and safe row-level security
  context handling.
- Detailed API response and error contracts.
- Hosting provider, container runtime, DNS/IP requirements, and Terraform
  state/secrets integration.
- Recovery time and recovery point objectives, backup retention, and restore
  cadence.
- Exact test-environment lifecycle and resource cleanup implementation.
