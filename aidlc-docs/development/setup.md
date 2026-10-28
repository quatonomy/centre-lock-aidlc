# centre-lock local development

The concise product and codebase summary is
[`centre-lock/SPEC.md`](../../centre-lock/SPEC.md). Detailed requirements,
decisions, architecture, and standards are maintained under `aidlc-docs/`.

## Getting started

Useful context: [project guidance](../project-guidance.md),
[architecture](../architecture/overview.md),
[accepted decisions](../architecture/decisions/),
[engineering standards](../engineering-standards/), and
[reverse-engineering notes](../inception/reverse-engineering/).

## Local development

Prerequisites: .NET 10 SDK, Node.js 22 or later, and Docker with Docker Compose.

Start PostgreSQL, apply its schema migration, then start the API and React UI:

```sh
docker compose up --detach postgres
```

If Compose is installed as a standalone CLI, use `docker-compose up --build`
and `docker-compose down`. The API is available at <http://localhost:8080>
(API health: <http://localhost:8080/api/health>, development Swagger UI:
<http://localhost:8080/swagger>), the separately hosted UI is available at
<http://localhost:5173/>, and PostgreSQL is on port `5432`. The API is served
under `/api`; Swagger generates its OpenAPI document from the API routes. The
database credentials in `compose.yaml` are for local development only; never
reuse them outside the local environment.

Database migrations are applied as a controlled step, not on application
startup (the `Database:MigrateOnStartup` option exists only for non-production
cloud environments). Both the catalog and identity contexts have migrations;
`dotnet ef database update` needs `--context` for each. After starting the Compose PostgreSQL service, install the EF CLI if
needed and apply the catalog migration:

```sh
dotnet tool install --global dotnet-ef --version 10.0.0
ConnectionStrings__CentreLock='Host=localhost;Port=5432;Database=centre_lock;Username=centre_lock;Password=local-development-only' \
  dotnet ef database update --project src/CentreLock.Api
```

Then launch the API and UI; startup idempotently adds the Star Wars API
definition if it is not already in the catalog:

```sh
docker compose up --build
```

The Star Wars API catalog definition is seeded in every environment as a
trial integration. Catalog browsing, OpenAPI document routes, and the
unauthenticated proxy remain Development-only until tenant authentication and
authorization are implemented. In Development, request a Star Wars person
through centre-lock:

```sh
curl --fail-with-body http://localhost:8080/api/integrations/proxy \
  --header 'Content-Type: application/json' \
  --data '{"integration":"star-wars","method":"GET","path":"/people/1"}'
```

The platform responds with a `connector` object containing the upstream HTTP
status and parsed JSON response body. The route is not mapped outside
Development and refuses integrations that require credentials; it is not the
authenticated tenant proxy API.

The web app at <http://localhost:5173/> includes a Development-only integration
catalog browser with each enabled integration's upstream URL, authentication
scheme, allow-listed endpoints, and OpenAPI document link. Swagger UI at
<http://localhost:8080/swagger> remains the interactive API documentation
surface and includes the catalogued integration documents. These catalog
read endpoints and the proxy are not available outside Development;
tenant-scoped integration management is not implemented yet.

Google sign-in is optional locally. Set `CENTRELOCK_GOOGLE_CLIENT_ID` and
`CENTRELOCK_GOOGLE_CLIENT_SECRET` (and `CENTRELOCK_PLATFORM_OPERATOR_SUBJECT`
to the Google `sub` of the first platform operator) in the environment before
`docker compose up`. Without them, sign-in reports "not configured"; outside
Development and Testing the API refuses to start without Google configuration.
Only users with a tenant membership or a configured operator subject can sign
in. Cloud deployment of non-production environments is described in the
`infrastructure/*/README.md` files of the application repository.

Stop the services with `docker compose down` (or `docker-compose down`). Add
`--volumes` only when you intend to remove the local PostgreSQL data.

Run the Bruno API collection against the running app:

```sh
npm ci --prefix tests/api
npm test --prefix tests/api
```

The `centre-lock` collection has deterministic catalog/proxy contract cases
and tagged live Star Wars API cases. Its default run skips `live`-tagged
requests and does not call the upstream service:

The same collection contains one successful live case for each of the 12
enabled Star Wars API method/path combinations (the list and representative
item endpoint for films, people, planets, species, starships, and vehicles).
Run those public-upstream checks explicitly:

```sh
npm run test:live --prefix tests/api
```

The .NET proxy tests also exercise every seeded Star Wars endpoint against a
controlled HTTP response, so endpoint allow-list and forwarding coverage runs
without a network dependency as part of `dotnet test`.

Run checks without containers:

```sh
dotnet test CentreLock.slnx
dotnet format CentreLock.slnx --verify-no-changes
cd web && npm ci && npm run lint && npm run build
```

The PostgreSQL catalog persistence test runs when
`CENTRELOCK_TEST_DATABASE` is set. CI starts the Compose PostgreSQL service and
sets this variable; without it, that integration test is skipped.

Run Playwright UI tests from `web/` (install the Chromium browser once):

```sh
npm ci
npx playwright install chromium
npm run test:e2e
```
