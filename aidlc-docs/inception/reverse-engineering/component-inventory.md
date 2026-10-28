# Component Inventory

## Application Components

- **CentreLock.Api** - ASP.NET Core API host for health/readiness, identity, operator tenant provisioning, tenant catalog read, Development catalog/OpenAPI, and Development proxy endpoints.
- **Tenant Identity** - Google OIDC cookie sign-in, identities, tenants, and memberships (`Admin`/`Member`) in a separate EF Core context.
- **Security middleware** - Safe exception handling, security headers/correlation ID, and cookie-session CSRF validation.
- **Integration Catalog** - PostgreSQL-backed integration definitions, endpoint allow lists, OpenAPI documents, validation, and Star Wars seeding.
- **Integration Proxy** - Generic outbound HTTP forwarding, endpoint matching, supported API-key/bearer auth boundary, and response capture.
- **Web UI** - React/TypeScript browser UI for sign-in/out, viewing memberships, and viewing integration entries and docs; not a tenant management interface.

## Data Components

- **PostgreSQL** - Local Compose database and a shared non-production Cloud SQL instance. Persistence covers the integration catalog and tenant identity (identities, tenants, memberships).
- **EF Core migrations** - Catalog and `TenantIdentity` migrations. Schema updates are intended to be controlled steps; app-startup migration (`Database:MigrateOnStartup`) is used only for non-production cloud environments.

## Infrastructure and Delivery Components

- **Dockerfile** - Multi-stage web and API image builds.
- **Docker Compose** - Local API, Nginx UI, and PostgreSQL services.
- **GitHub Actions workflows** - Separate API and web CI jobs with formatting, tests, audits, API checks, and UI checks; a main-push workflow dispatches context-update review and does not deploy the application.
- **Nginx** - Serves built web assets and forwards API/Swagger paths to the API container.

- **Terraform roots (`infrastructure/state`, `bootstrap`, `shared`, `environment`)** - Non-production GCP infrastructure; see `architecture.md`.
- **Lease controller (`infrastructure/shared/lease_controller`)** - Python Cloud Run function that starts/stops the shared database on a time-capped lease.
- **Deploy workflows** - `deploy-preview.yml`, `deploy-staging.yml`, `deploy-infrastructure.yml`, `terraform-validate.yml`, and `.github/scripts/renew-lease.sh`.
- **Dockerfile `cloud-run-runtime` target** - API image bundling the built web assets.

No production infrastructure is present.

## Test Components

- **CentreLock.Api.Tests** - xUnit, ASP.NET Core test host, controlled proxy doubles, and environment-route tests.
- **PostgreSQL catalog tests** - Optional integration tests enabled by `CENTRELOCK_TEST_DATABASE`; CI provisions PostgreSQL and sets this variable.
- **Bruno API collection** - Deterministic API contract checks plus tagged live Star Wars tests.
- **Playwright UI tests** - Catalog rendering and loading, empty, and unavailable states with mocked API responses.

## Planned Product Components Not Yet Implemented

The specification and architecture describe tenant isolation, tenant integration configuration, consumer API key management/authentication, tenant-scoped upstream credential storage, usage/metering, pricing/billing, and operator catalog administration. These are product objectives/open questions and are not components in the current code.

## Total Count

- **Tracked repository files**: 93.
- **C# application source files**: 16, including generated EF migration artifacts.
- **C# test files**: 5.
- **Bruno request files**: 21.
- **Playwright spec files**: 1.
- **Deployed application services in local Compose**: API, Nginx UI, and PostgreSQL.
