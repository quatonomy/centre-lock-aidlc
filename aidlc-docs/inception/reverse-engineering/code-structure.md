# Code Structure

## Build System

- **Backend**: .NET 10 / MSBuild solution `CentreLock.slnx`.
- **Frontend**: npm workspaces are not used; `web/` has its own npm package and lock file.
- **API tests**: `tests/CentreLock.Api.Tests/` is a .NET test project in the solution.
- **API contract tests**: `tests/api/` has an independent npm package and Bruno collection.
- **Local infrastructure**: Docker Compose builds/runs the API and web image targets with PostgreSQL.
- **CI and automation**: GitHub Actions workflow `.github/workflows/ci.yml` runs checks on pull requests, then calls reusable Terraform-validation and preview-deployment workflows; on pushes to `main`, `.github/workflows/cd.yml` dispatches `context-update-review.yml` with the source commit SHA and runs independent (no `needs`) jobs for path-filtered shared-infrastructure deployment and staging deployment. Terraform roots are in `infrastructure/` (`state`, `bootstrap`, `shared`, `environment`).

## Key Modules

### API host
- `src/CentreLock.Api/Program.cs` - dependency injection, PostgreSQL connection, HTTP client timeout/handler configuration, Swagger, CORS, seeding, and route mapping.
- `src/CentreLock.Api/CentreLock.Api.csproj` - .NET target and EF Core, Npgsql, and Swagger dependencies.

### Catalog
- `Integrations/Catalog/IntegrationDefinition.cs` - catalog definition, authentication scheme, and allowed endpoint model.
- `Integrations/Catalog/IntegrationDefinitionValidator.cs` - validates definitions, HTTPS DNS base URLs, authentication headers, OpenAPI JSON shape, methods, and path templates.
- `Integrations/Catalog/IntegrationCatalogDbContext.cs` - EF Core mappings for catalog and owned allow-list rows.
- `Integrations/Catalog/EfIntegrationCatalog.cs` and `IIntegrationCatalog.cs` - persistence/query implementation and interface.
- `Integrations/Catalog/IntegrationCatalogSeed.cs` - Star Wars definition, OpenAPI document, and 12 GET method/path combinations.
- `Integrations/Catalog/PublicIntegrationCatalogEndpoints.cs` - public read-only catalog-list and OpenAPI routes (`MapPublicIntegrationCatalogEndpoints`; the containing static class is still named `DevelopmentIntegrationCatalogEndpoints`).
- `Integrations/Catalog/Migrations/` - initial schema migration, generated designer, and model snapshot.

### Proxy
- `Integrations/Proxy/DevelopmentProxyEndpoint.cs` - Development-only request validation, credentialed-integration refusal, response wrapping, and error mapping.
- `Integrations/Proxy/DevelopmentProxyRequest.cs` - Development request shape.
- `Integrations/Proxy/IntegrationProxy.cs` - generic HTTP request forwarding and allow-list enforcement.
- `Integrations/Proxy/IIntegrationProxy.cs` - proxy contract and response record.
- `Integrations/Proxy/IUpstreamCredentialProvider.cs` - tenant/integration-scoped credential contract and unconfigured implementation.

### Web application
- `web/src/App.tsx` - Development integration catalog view, selection, and loading/empty/error states.
- `web/src/App.css`, `web/src/index.css` - UI styles.
- `web/src/main.tsx`, `web/index.html` - React entry and document.
- `web/vite.config.ts` - development server and API/Swagger proxies.
- `web/nginx.conf` - production-image static serving and API/Swagger reverse proxy.
- `web/e2e/homepage.spec.ts` - Playwright tests using a mocked catalog API.

### Tests and API collection
- `tests/CentreLock.Api.Tests/` - five C# test files covering health, Development routes, catalog validation, proxy behavior, and opt-in PostgreSQL persistence/startup behavior.
- `tests/api/centre-lock/` - Bruno requests for health, Development catalog/OpenAPI/proxy behavior, and 12 Star Wars live endpoints; live requests are tagged separately from the default test run.
- `web/e2e/` - browser-level catalog UI behavior tests.

### Delivery files
- `Dockerfile`, `compose.yaml` - local multi-container build and services.
- `.github/workflows/ci.yml` - PR and main-branch checks.
- `.github/workflows/cd.yml` - Main-branch push: context-update review dispatch, path-filtered shared-infrastructure deployment, and staging deployment (non-production).
- `aidlc-docs/development/setup.md` - local setup, migrations, API/UI use, tests, and current Development-only feature limits (moved from the application folder during context consolidation).

## Existing File Inventory

Tracked application and test source comprises 16 C# files under `src/`, five C# test files, React/TypeScript UI source and configuration, 21 Bruno request files, and one Playwright spec. The repository contains 93 tracked files in this snapshot. Generated EF migration designer/snapshot files are included in the C# source count.

## Design Patterns

### Modular monolith
- **Location**: One ASP.NET Core application with `Integrations/Catalog` and `Integrations/Proxy` namespaces.
- **Purpose**: Keep deployment unified while defining logical module boundaries.
- **Implementation**: Catalog and proxy interact through interfaces, not another module's persistence context.

### Repository/service interfaces
- **Location**: `IIntegrationCatalog`, `IIntegrationProxy`, `IUpstreamCredentialProvider`.
- **Purpose**: Separate application behavior from persistence, proxy transport, and credentials.
- **Implementation**: EF catalog and HTTP proxy implementations are registered by the API host; tests replace interfaces with controlled doubles.

### Declarative integration configuration
- **Location**: `IntegrationDefinition`, PostgreSQL schema, and `IntegrationCatalogSeed`.
- **Purpose**: Define upstream host, auth scheme, OpenAPI document, and endpoint allow list as data.
- **Implementation**: A generic proxy resolves an enabled definition at runtime; callers provide a slug, method, and path, not an upstream host.

### Environment-gated development routes
- **Location**: `Program.cs`.
- **Purpose**: Keep unauthenticated catalog and proxy routes unavailable outside Development.
- **Implementation**: Routes are mapped only when `app.Environment.IsDevelopment()` is true.

## Critical Dependencies

- **Entity Framework Core 10 / Npgsql EF provider 10**: PostgreSQL persistence and migrations for the API.
- **Swashbuckle.AspNetCore**: API and Development integration OpenAPI/Swagger surface.
- **ASP.NET Core/.NET 10**: API hosting, DI, HTTP client factory, and test host.
- **React / React DOM 19**: UI rendering.
- **TypeScript 6 / Vite 8 / Oxlint**: UI typing, build/dev server, and linting.
- **Playwright 1.63**: UI end-to-end tests.
- **Bruno CLI 4.2**: API contract collection runner.
- **Docker / Compose and Nginx**: Local service and UI container packaging.

No external cloud SDK, identity provider, payment provider, or secret-management dependency is present.
