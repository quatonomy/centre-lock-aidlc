# ADR 0002: Architecture and delivery principles

- **Status:** Accepted
- **Date:** 2026-10-04

## Context

The initial product needs strong tenant isolation, clear integration
boundaries, portable packaging, and a safe path from pull-request testing to
production. The hosting provider is not yet selected. The project owner will
review all pull requests, and the main branch must remain releasable.

## Decision

### Application architecture

- Keep the product as a modular monolith, with logical boundaries for
  identity/access, tenancy, integration catalog, connectors, proxy, usage and
  billing, and application interfaces. These boundaries do not imply separate
  deployments.
- Modules collaborate through explicit contracts or application services.
  They must not access another module's persistence or internal implementation
  directly.
- Use a shared PostgreSQL database/schema. Enforce tenant scope in application
  services and data access; use PostgreSQL row-level security as defense in
  depth where it can be implemented safely with connection pooling and tenant
  context.
- Support one API version initially, while keeping API routing and contracts
  capable of adding versions when needed.

### Container and infrastructure

- Build and distribute the application as an OCI-compatible container image.
  Use Docker for image tooling and Docker Compose for local development.
- Do not adopt Kubernetes solely to achieve portability. Select a managed
  container runtime after comparing hosting options; adopt orchestration only
  if later requirements justify it.
- Use Terraform as the infrastructure-as-code source of truth and GitHub
  Actions for CI. Defer provider-specific infrastructure and Terraform
  backends until the hosting provider is chosen.
- Keep local, test, staging, and production environments isolated.

### Pull-request validation and releases

- Every pull request must pass CI and be reviewed by the project owner before
  merge. Build and test small, independently releasable features so main
  remains deployable.
- Where supported by the selected host, create an isolated, temporary
  deployment for each pull request, with a unique URL and application
  resources. Use a dedicated IP only if the host and routing design require
  and support it. Do not expose production data or credentials to these
  deployments.
- Run the full relevant feature and API checks against the pull-request
  deployment before merge. Tear down its temporary resources when the pull
  request is closed or merged.
- Build an immutable container artifact. A merge to main automatically
  deploys that same artifact through test and staging to production, with
  automated verification between environments. Production deployment does
  not require a separate manual approval for the normal merge flow.

### Database operations

- Review and commit EF Core schema migrations with application changes. Apply
  migrations as a controlled deployment step rather than as uncontrolled
  application startup behavior.
- Select a hosting provider whose managed database offering supports backups
  and restoration. Verify restoration before production use. Recovery time
  and recovery point objectives remain open until provider requirements and
  costs are understood.

## Rationale

- A modular monolith supports clear ownership and future growth without
  introducing the operational cost of distributed services prematurely.
- Application tenant checks are explicit and testable; database row-level
  security provides an additional containment layer where operationally safe.
- OCI-compatible images are portable across container platforms. Docker is
  useful for building and local development, while Kubernetes solves
  orchestration rather than containerization and is unnecessary by default.
- Per-PR deployments allow end-to-end validation before merge. Automated
  post-merge promotion preserves the requested release flow while running
  environment checks before production.
- Provider-managed database recovery avoids selecting a cloud-specific backup
  implementation before the provider is known.

## Consequences

- Hosting selection must account for ephemeral per-PR environments, unique
  routing, Terraform support, container deployment, managed PostgreSQL
  backups, and restoration.
- Dedicated per-PR IP addresses are not guaranteed; the hosting and routing
  design may meet isolation needs with unique hostnames and application
  resources instead.
- Production recovery objectives, backup retention, and the detailed
  promotion/health-check policy must be set before production launch.
- CI must eventually measure unit coverage, run PostgreSQL-backed integration
  tests and Bruno CLI API tests, and support safe teardown of ephemeral
  deployments.
