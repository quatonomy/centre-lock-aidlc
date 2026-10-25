# Component Inventory

## Application Components

- **CentreLock.Api** - ASP.NET Core API host for health, Development catalog/OpenAPI, and Development proxy endpoints.
- **Integration Catalog** - PostgreSQL-backed integration definitions, endpoint allow lists, OpenAPI documents, validation, and Star Wars seeding.
- **Integration Proxy** - Generic outbound HTTP forwarding, endpoint matching, supported API-key/bearer auth boundary, and response capture.
- **Development Catalog UI** - React/TypeScript browser UI for viewing integration entries and docs; not an authenticated tenant management interface.

## Data Components

- **PostgreSQL** - Local Compose database. Current persistence is limited to the integration catalog and its allowed endpoints.
- **EF Core migration** - Initial catalog schema migration; schema updates are intended to be controlled migration steps rather than app-startup migrations.

## Infrastructure and Delivery Components

- **Dockerfile** - Multi-stage web and API image builds.
- **Docker Compose** - Local API, Nginx UI, and PostgreSQL services.
- **GitHub Actions workflows** - Separate API and web CI jobs with formatting, tests, audits, API checks, and UI checks; a main-push workflow dispatches context-update review and does not deploy the application.
- **Nginx** - Serves built web assets and forwards API/Swagger paths to the API container.

No Terraform or cloud infrastructure package is present.

## Test Components

- **CentreLock.Api.Tests** - xUnit, ASP.NET Core test host, controlled proxy doubles, and environment-route tests.
- **PostgreSQL catalog tests** - Optional integration tests enabled by `CENTRELOCK_TEST_DATABASE`; CI provisions PostgreSQL and sets this variable.
- **Bruno API collection** - Deterministic API contract checks plus tagged live Star Wars tests.
- **Playwright UI tests** - Catalog rendering and loading, empty, and unavailable states with mocked API responses.

## Planned Product Components Not Yet Implemented

The specification and architecture describe identity/access, tenancy, tenant integration configuration, consumer API key management/authentication, tenant-scoped upstream credential storage, usage/metering, pricing/billing, and operator catalog administration. These are product objectives/open questions and are not components in the current code.

## Total Count

- **Tracked repository files**: 93.
- **C# application source files**: 16, including generated EF migration artifacts.
- **C# test files**: 5.
- **Bruno request files**: 21.
- **Playwright spec files**: 1.
- **Deployed application services in local Compose**: API, Nginx UI, and PostgreSQL.
