# Technology Stack

## Programming Languages

- **C#** - .NET 10 API, data access, and tests.
- **TypeScript / JavaScript** - React UI, Vite configuration, and Playwright tests.
- **SQL** - PostgreSQL schema managed through EF Core migrations.
- **YAML** - GitHub Actions and Docker Compose configuration.

## Frameworks and Libraries

- **ASP.NET Core / .NET 10** - API host and minimal routes.
- **Entity Framework Core 10.0.0** - ORM and migrations.
- **Npgsql.EntityFrameworkCore.PostgreSQL 10.0.0** - PostgreSQL provider.
- **Swashbuckle.AspNetCore 10.2.3** - Swagger/OpenAPI UI and generation.
- **React / React DOM 19** - UI.
- **Vite 8 and TypeScript 6** - Web build and development server.
- **Oxlint** - Web linting.

## Infrastructure

- **PostgreSQL 18** - Local Compose database image.
- **Docker / Docker Compose** - OCI-compatible image build and local API/UI/database execution.
- **Nginx Alpine** - Serves the built UI and proxies `/api` and `/swagger`.
- **Hosting provider** - Not selected.
- **Terraform** - Accepted as the intended IaC source of truth, but no Terraform configuration is present.
- **GitHub Actions** - Current CI workflow.

## Build Tools

- **.NET SDK 10** - Restore, build, test, format, and publish.
- **npm / Node.js 22** - Web and Bruno package installation, builds, audits, and tests.
- **EF Core CLI 10.0.0** - Manually installed for controlled local database migration application.
- **Docker Compose** - Local dependency orchestration.

## Testing Tools

- **xUnit 2.9.3 / Microsoft.NET.Test.Sdk 17.14.1** - Backend tests.
- **Microsoft.AspNetCore.Mvc.Testing 10.0.12** - In-process API integration tests.
- **Coverlet collector 6.0.4** - Test coverage collection dependency; the current CI workflow does not publish or enforce a coverage threshold.
- **Bruno CLI 4.2.0** - API request collection.
- **Playwright 1.63** - Browser end-to-end tests.
- **dotnet format** - .NET formatting verification in CI.
- **npm audit** - Dependency audit in API-test and web jobs.

## External Integrations

- **Star Wars API (`swapi.dev`)** - Seeded Development proxy integration; live tests are opt-in.
- **GitHub** - A product candidate with no implementation or catalog definition; MVP inclusion and authorization are deferred pending use cases and permission scopes.
- **Identity, billing, secret storage** - Not integrated.
