# API Documentation

## API Surface

The currently implemented API has health/readiness routes, sign-in and identity routes, an operator tenant-provisioning route, authenticated tenant catalog read routes, and catalog/OpenAPI and proxy routes mapped only in the Development environment. Swagger UI is enabled by the API host. There are no tenant-owned integration configuration, membership-management, consumer-authentication, usage, or billing endpoints in the tracked implementation.

### API readiness
- **Method**: `GET`
- **Path**: `/api/ready`
- **Purpose**: Readiness; checks connectivity of the catalog and identity database contexts.
- **Response**: `200` with `{"status":"ready"}`, otherwise `503`.
- **Environment**: All environments. Used as the deployed-environment smoke test.

### Identity and tenant routes
- `GET /api/auth/status` (anonymous): `{signInConfigured, provider}`.
- `GET /api/auth/csrf` (anonymous): `{requestToken}` for the `X-CSRF-TOKEN` header.
- `GET /api/auth/sign-in?returnUrl=`: starts Google OIDC; only local paths are honored; `503 sign_in_not_configured` when unconfigured. Callback is `/api/auth/callback`; failures redirect to `/?signIn=failed`.
- `POST /api/auth/sign-out`: authenticated and CSRF-protected; `204`.
- `GET /api/me`: authenticated profile with email, display name, `isPlatformOperator`, and memberships (tenant ID, name, slug, role); `401` when unauthenticated or unconfigured.
- `POST /api/operator/tenants` `{name, slug}`: requires the platform-operator policy (subject in `PlatformOperators:SubjectIds`) and CSRF token; creates a tenant with the caller as `Admin`; `201`, `400 invalid_tenant_request`, or `409 tenant_slug_exists`. Mapped only when OIDC is configured.
- `GET /api/tenant/integrations` and `GET /api/tenant/integrations/{slug}/openapi.json`: authenticated and require at least one tenant membership (`403` otherwise; `503` when sign-in is unconfigured). They return the platform-wide enabled catalog, not tenant-scoped data.

## REST APIs

### API health
- **Method**: `GET`
- **Path**: `/api/health`
- **Purpose**: Liveness response.
- **Request**: None.
- **Response**: `200` with `{"status":"ok"}`.
- **Environment**: All environments.

### List Development catalog entries
- **Method**: `GET`
- **Path**: `/api/integrations`
- **Purpose**: List enabled integrations for the Development catalog preview.
- **Request**: None.
- **Response**: `200` array with `slug`, `name`, `baseUrl`, `authenticationScheme`, `endpoints` (`method`, `pathTemplate`), and `openApiUrl`.
- **Environment**: Development only. Not a tenant-facing catalog API.

### Get a Development integration OpenAPI document
- **Method**: `GET`
- **Path**: `/api/integrations/{slug}/openapi.json`
- **Purpose**: Return the stored OpenAPI document for an enabled definition.
- **Request**: URL slug.
- **Response**: `200 application/json` with the stored document; `404` with an error for an unavailable integration.
- **Environment**: Development only.

### Development proxy
- **Method**: `POST`
- **Path**: `/api/integrations/proxy`
- **Purpose**: Local-only proxy to an enabled, credential-free integration using a catalog-allowed method/path.
- **Request**: JSON with required `integration`, `method`, and `path`; optional `queryString`, JSON `body`, `contentType`, and `accept`. The route is unauthenticated and must not be treated as the tenant proxy API.
- **Response**:
  - `200` for a completed upstream response, regardless of the upstream status. Body contains `connector.statusCode` and `connector.body`; body formatting is parsed JSON or text.
  - `400` when required fields or forwarded media-type headers are invalid.
  - `403` when upstream credentials are required or the endpoint is not allow-listed.
  - `404` when the integration is unavailable.
  - `502` when an upstream transport request fails.
  - `504` with an explicit timeout error when the upstream request times out.
- **Environment**: Development only. The endpoint passes an empty tenant ID placeholder and does not support credentialed integrations.
- **Timeout/retry**: Named HTTP client timeout is 30 seconds. No retry policy is configured.

## API Documentation Surface

- **Swagger UI**: `/swagger`.
- **OpenAPI**: ASP.NET Core API document plus Development catalog documents registered with Swagger UI.
- **API contract tests**: Bruno collection under `tests/api/centre-lock/`, with default non-live tests and explicitly tagged live Star Wars calls.

## Internal APIs

### `IIntegrationCatalog`
- `ListEnabledAsync(CancellationToken)` - return enabled definitions.
- `FindEnabledAsync(string slug, CancellationToken)` - find an enabled definition.
- `SaveAsync(IntegrationDefinitionInput, CancellationToken)` - insert or update a definition.
- `AddIfMissingAsync(IntegrationDefinitionInput, CancellationToken)` - idempotently insert a definition when its slug does not exist.

The current HTTP API exposes no catalog-write route; `SaveAsync` is an internal catalog operation.

### `IIntegrationProxy`
- `SendAsync(Guid tenantId, string integrationSlug, string method, string path, string queryString, ReadOnlyMemory<byte> body, string? contentType, string? accept, CancellationToken)` - validate and forward an allowed request.
- Returns `ProxiedIntegrationResponse` with upstream status code, response bytes, and content type.

### `IUpstreamCredentialProvider`
- `GetAsync(Guid tenantId, Guid integrationId, CancellationToken)` - retrieve a scoped upstream credential.
- The registered implementation is `UnconfiguredUpstreamCredentialProvider`, which throws. Credential persistence and retrieval are not implemented.

## Data Models

### `IntegrationDefinition`
- **Fields**: ID, unique slug, name, HTTPS base URL, authentication scheme, optional API-key header, OpenAPI JSON, enabled flag, allowed endpoint records.
- **Relationships**: One definition owns its allowed method/path endpoint entries.
- **Validation**: Slug/name/base URL/auth header/OpenAPI shape/method/path constraints enforced by `IntegrationDefinitionValidator`.

### `DevelopmentProxyRequest`
- **Fields**: Integration slug, HTTP method, path, optional query string, optional JSON body, content type, and Accept.
- **Validation**: Required fields and media-type values are checked by the Development endpoint; method/path authorization is enforced by `IntegrationProxy`.

## Unresolved Contract Details

The tenant-facing proxy returns HTTP 200 for completed upstream responses, with upstream status/body under `connector`. Exact platform/upstream error representation, forwarded headers, response-size limits, and timeout details remain open in `aidlc-docs/inception/requirements/requirements-clarification-questions.md`. The current Development route has separate concrete behavior and is not the tenant-facing API.
