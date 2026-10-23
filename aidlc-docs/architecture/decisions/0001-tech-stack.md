# ADR 0001: Initial technology stack

- **Status:** Accepted
- **Date:** 2026-09-30

## Context

centre-lock is a new integrations platform with a tenant-facing management UI,
a proxy API for third-party integrations, tenant and user management, and
per-request usage tracking. The initial implementation should be straightforward
to develop and operate without prematurely introducing distributed services.
Hosting cost predictability is important before go-to-market.

## Decision

Build the initial product as a **.NET modular monolith**:

- **Backend and proxy API:** ASP.NET Core, organized into modules within one
  application and deployed as one service.
- **Management UI:** React with TypeScript, built and served with the
  application rather than deployed as a separate service initially.
- **Database:** PostgreSQL, accessed from .NET using Entity Framework Core and
  the Npgsql provider.
- **Authentication:** Use OpenID Connect-based federated sign-in for the
  management UI. Google is the first provider for personal testing; keep the
  design open to additional identity providers.
- **Packaging:** Build OCI-compatible container images using Docker. Use
  Docker Compose for local development and select a compatible hosting
  container runtime later. Kubernetes is not required for the initial
  product.

Keep the application and database portable: do not make the initial design
dependent on a specific cloud provider.

## Rationale

- A modular monolith keeps the initial deployment and operations simpler than
  microservices while preserving code boundaries as the product grows.
- ASP.NET Core provides a mature foundation for the proxy API and the
  application backend.
- React and TypeScript provide a flexible customer-facing UI and a typed
  frontend codebase.
- PostgreSQL is a no-license-cost relational database well suited to tenant,
  user, integration, and usage data, with JSON support available for
  integration-specific configuration where justified.
- OCI-compatible images make the application portable across compatible
  container runtimes. Docker provides familiar local build/run tooling without
  committing the product to Kubernetes or a particular cloud provider.

## Alternatives considered

- **MySQL:** A capable relational alternative. PostgreSQL is the selected
  default, but MySQL would also meet the core relational requirements.
- **SQL Server:** A strong option for .NET, but not selected as the default
  because of potential licensing or hosting cost considerations.
- **SQLite:** Useful for lightweight local applications, but not selected for
  the initial multi-tenant service database.
- **Microservices:** Not selected because the initial product does not justify
  the additional deployment and operational complexity.
- **Kubernetes:** Not selected for the initial product. Container images
  provide portability; orchestration can be selected with the hosting provider
  if product scale or operational requirements justify it.

## Deferred decisions

- Hosting provider, deployment topology, and specific cost-throttling or
  shutdown mechanism.
- Connector-specific authentication methods.
- Detailed module interfaces, API response contract, and role-assignment
  workflows.
