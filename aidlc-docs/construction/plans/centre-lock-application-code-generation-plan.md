# Code Generation Plan: CentreLock Application

## Plan status and source of truth

**Status:** ABANDONED on 2026-11-03. The project was closed before this plan
was completed; unchecked items below were never implemented. See the Closure
section in `aidlc-state.md`.

## Current scope amendment (2026-10-26)

The user deferred all authentication and sign-in, including management OIDC
and consumer API keys. The original 11-story execution plan below is
superseded wherever it requires authentication, membership/roles,
tenant-private management, API keys, or authenticated proxy execution.
Current MVP work is:

1. US-04 public, read-only discovery of platform-published catalog entries
   and OpenAPI documents without tenant-specific data.
2. US-08 tenant routing from bespoke subdomains under a configured shared
   platform base domain. Host resolution selects routing context only; it
   does not authorize access or permit proxy calls.
3. Required tests and delivery for these capabilities.

US-01, US-02, US-03, US-05, US-06, US-07, US-09, US-10, and US-11 are
deferred. US-08 is narrowed to host routing only. Do not implement tenant
data access or upstream proxy execution until authentication is explicitly
brought back into scope. Existing OIDC/membership code is experimental
scaffolding and is not a current MVP deliverable. Bespoke tenant subdomains
are accepted; custom domains are deferred. The platform base domain, wildcard
DNS/TLS provisioning, reserved hostnames, and provider-specific edge routing
remain to be selected/designed.

The numbered implementation steps below retain the original approved design
history. Authentication-, identity-, key-, tenant-private-management-, and
proxy-dependent tasks in those steps are deferred and must not drive current
implementation. Update their completion state only against the amended
scope above.

**Application workspace:** `centre-lock/` (brownfield nested repository)

**Documentation workspace:** `aidlc-docs/` in the parent repository

**Deployable unit:** `centre-lock-application`, one .NET 10 modular monolith
with React/TypeScript UI, PostgreSQL, and OCI packaging.

## Scope and constraints

- Implement only the current in-scope stories identified at the top of this
  plan, following `unit-of-work-story-map.md` and the amended requirements.
- Preserve existing catalog/proxy behavior in Development and evolve the
  existing brownfield files in place; do not create duplicate variants.
- Application code and build/configuration files go under `centre-lock/`;
  markdown generation summaries go under
  `aidlc-docs/construction/centre-lock-application/code/`.
- Keep one deployable application unit and existing module boundaries.
  Preserve the provider-neutral architecture; do not add a cloud SDK, choose
  a cloud provider, or generate provider-specific Terraform.
- Implement the secret-store contract and an explicit unavailable adapter;
  use test doubles. A production provider adapter remains gated on hosting
  selection. Fail closed and return a safe, explicit unavailable response
  rather than persisting credentials in application data.
- Implement the invitation-delivery contract and test fake. No production
  email sender is selected; invitation creation must fail explicitly when no
  sender is configured, and tokens must never be logged or returned except
  through the one-time delivery contract.
- Identify platform operators by configured OIDC subject allow-list. Do not
  authorize operators from mutable email/display-name claims or tenant roles.
- Configure Google OIDC from environment/runtime configuration. Never commit
  client secrets, API keys, invitation tokens, or upstream credentials.
- Preserve user decisions that remain production gates: provider-specific
  secret and email adapters, exact RTO/RPO, GDPR policy, and measured
  capacity/latency targets.
- Keep database migrations explicit and reviewed; do not apply schema
  migrations automatically at application startup.

## Unit context and interfaces

### Stories assigned to this unit

US-01 sign-in; US-02 invitations and roles; US-03 tenant read-only access;
US-04 catalog discovery; US-05 tenant integration configuration; US-06
platform catalog administration; US-07 API-key lifecycle; US-08 tenant/key
agreement and proxy authorization; US-09 completed upstream responses;
US-10 platform proxy errors; US-11 minimal request outcomes.

### Dependencies and external boundaries

- Google OIDC is the initial management identity provider; application
  authorization and tenant membership remain local server-side decisions.
- PostgreSQL/EF Core is the persistence boundary. Preserve the existing
  `IntegrationCatalogDbContext`; keep tenant-owned persistence behind
  module-owned repositories and explicit application contracts.
- `IUpstreamCredentialProvider` is an existing proxy boundary. Evolve or
  replace it in place with the approved store/retrieve/delete contract; do
  not add a parallel credential-provider abstraction.
- The secret-manager and email-delivery production adapters remain
  unselected. Tests use deterministic fakes; unconfigured runtime adapters
  fail explicitly and safely.
- Upstream HTTP calls use the existing allow-list-aware proxy and
  `HttpClient` boundary, with no retries, 30-second timeout, no response
  headers forwarded, JSON-or-text response parsing, and a 0.5 MB maximum
  body.
- Provider-specific cloud resources, Terraform state, DNS, production email
  service, and cloud secret manager are outside this implementation plan.

### Data owned by the application

Tenant, OIDC subject reference, membership/role, invitation hash and
lifecycle, tenant integration configuration and opaque secret reference,
consumer API-key hash and lifecycle metadata, and minimal usage outcomes.
Catalog data remains owned by the existing Catalog module. Keep tenant
isolation enforced in application contracts and database access; never return
raw credentials or key/token hashes to clients.

## Numbered implementation steps

### Step 1: Verify brownfield baseline and implementation boundaries

- [x] Reconfirm the application repository is clean before edits and inspect
  target files in `centre-lock/`.
- [x] Preserve the existing `Integrations/Catalog` and
  `Integrations/Proxy` contracts where compatible; record necessary
  in-place changes before making them.
- [x] Keep Development-only catalog/proxy routes restricted to Development.
- [x] Do not add real identity, email, secret-manager, upstream, or cloud
  credentials to tests or documentation.

### Step 2: Implement authentication, session, operator, and request foundations

- [x] Modify `centre-lock/src/CentreLock.Api/Program.cs`,
  `appsettings.json`, and
  `centre-lock/src/CentreLock.Api/CentreLock.Api.csproj` to configure Google
  OIDC, secure cookie sessions, authorization, CSRF protections for
  cookie-authenticated writes, and safe error handling from runtime
  configuration.
- [x] Add the Identity/Access module under
  `centre-lock/src/CentreLock.Api/Identity/` for OIDC principal handling,
  sign-in/sign-out routes, and configured platform-operator subject
  authorization.
- [x] Require non-empty OIDC/runtime configuration outside Development and
  tests; do not
  provide a production or generally accessible authentication bypass.
- [x] Add correlation, safe JSON error-envelope mapping, security headers,
  and health/readiness checks without changing
  existing Development-only route exposure.
- [ ] Add request validation and limits.
- [x] Add xUnit tests under
  `centre-lock/tests/CentreLock.Api.Tests/` for authentication rejection,
  operator allow-list matching, authorization defaults, CSRF, safe errors,
  and production-vs-Development route behavior.
- [x] Add authenticated-session and failure-state UI tests in
  `centre-lock/web/e2e/`.
- [ ] Story traceability: US-01, US-06, US-10.

### Step 3: Add tenant, membership, invitation, and provisioning persistence

**Progress (partial):** Tenant identity, tenant, and membership entities,
validation, EF Core persistence, an explicit migration, verified-email/
membership-gated OIDC sign-in, and operator first-tenant provisioning are
implemented. Invitation lifecycle and delivery, tenant-scoped member
management, broader persistence coverage, and a complete onboarding UI remain
outstanding; this step and US-01 are not complete.

- [ ] Add tenant-domain entities, validation, and services under
  `centre-lock/src/CentreLock.Api/Tenancy/`,
  `Identity/`, and `Invitations/`; use trusted authenticated context, not
  caller-supplied tenant IDs, for authorization.
- [ ] Add a tenant-application EF Core context and module-owned repositories
  for tenant, membership, and invitation data. Preserve the existing catalog
  context and migrations; avoid cross-module table access.
- [ ] Add a reviewed migration for tenant data with unique constraints,
  required indexes, concurrency/lifecycle fields, normalized email, and
  one-way invitation-token hashes.
- [ ] Implement operator-only first-tenant/admin provisioning, tenant-scoped
  member listing/role management, and admin-only invitation issue/resend/
  revoke/redemption services.
- [ ] Implement invitation expiry at 7 days; resend invalidates the previous
  hash and resets expiry while preserving tenant and role; redemption
  requires a verified OIDC email match and atomically creates membership.
- [ ] Define `IInvitationDelivery` and a deterministic test fake. With no
  production sender configured, fail the operation safely without storing a
  usable pending link; never log or expose the raw link/token.
- [ ] Add xUnit unit/property and PostgreSQL-backed tests for isolation,
  roles, uniqueness, atomic redemption, expiry, resend, revocation, and
  email matching.
- [ ] Story traceability: US-01, US-02, US-03, US-06.

### Step 4: Implement tenant integration configuration and secret boundary

- [ ] Add tenant-configuration domain/service/repository code under
  `centre-lock/src/CentreLock.Api/Integrations/TenantConfiguration/`.
- [ ] Enforce one configuration per tenant/catalog integration and
  admin-only writes; resolve published catalog integrations through
  `IIntegrationCatalog`, never by direct catalog table access.
- [ ] Evolve the existing
  `centre-lock/src/CentreLock.Api/Integrations/Proxy/IUpstreamCredentialProvider.cs`
  in place to the provider-neutral store/retrieve/delete contract used by
  configuration and proxy modules. Add a safe unconfigured adapter; do not
  add a cloud-specific adapter.
- [ ] Persist only opaque secret references and validated tenant-local
  settings. Fail closed if secret store operations are unavailable; do not
  persist credential values as a fallback.
- [ ] Add reviewed schema migration, API endpoints, tenant/admin authorization
  tests, tenant-isolation tests, and fake secret-store tests.
- [ ] Story traceability: US-03, US-04, US-05, US-08.

### Step 5: Implement protected catalog administration and tenant discovery

**Progress (partial):** Authenticated tenant catalog and OpenAPI read routes
are implemented for enabled catalog entries; requests require a persisted
tenant membership, and catalog write routes are not exposed. The seeded
integration is available in all environments. The current `IsEnabled` flag
acts as the availability/publication gate. Operator catalog create/update/
publish workflows and their validation remain outstanding.

- [ ] Extend the existing
  `centre-lock/src/CentreLock.Api/Integrations/Catalog/` service and endpoint
  patterns with protected operator create/update/publish workflows; modify
  existing catalog code in place.
- [ ] Validate metadata, allowed method/path templates, and OpenAPI content
  before publishing. Keep catalog writes unavailable to tenant principals.
- [x] Add tenant-scoped published-catalog and OpenAPI read APIs without
  exposing credentials or enabling tenant mutation of catalog-owned data.
- [ ] Add xUnit tests for operator authorization, catalog validation,
  published-only discovery, and blocked tenant writes. Preserve current
  Development catalog and seed tests.
- [ ] Story traceability: US-03, US-04, US-06, US-08.

### Step 6: Implement consumer API-key lifecycle and tenant authorization

- [ ] Add the API-key domain, service, repository, and routes under
  `centre-lock/src/CentreLock.Api/ApiKeys/`; use a cryptographically secure
  high-entropy key and a one-way verifier.
- [ ] Return usable keys only on creation/replacement; allow overlapping
  active keys; implement owner create/revoke and same-tenant admin revoke;
  enforce expiry/rotation policy with a configurable 12-month default.
- [ ] Add key/subdomain tenant resolution using an environment-configured
  base domain; reject missing, invalid, revoked, expired, mismatched, or
  unauthorized-owner credentials before catalog lookup/outbound calls.
- [ ] Add migration, API tests, PostgreSQL tenant-scope tests, property tests,
  and Bruno contract coverage. Do not expose hashes or usable keys in list,
  error, audit, or log responses.
- [ ] Story traceability: US-07, US-08.

### Step 7: Implement tenant proxy behavior and minimal request outcomes

- [ ] Extend the existing
  `centre-lock/src/CentreLock.Api/Integrations/Proxy/` implementation and
  routes in place, reusing catalog endpoint validation and the existing
  no-auto-redirect/no-cookie HTTP handler.
- [ ] Authenticate the tenant subdomain/API-key pair and active membership;
  resolve the tenant configuration and scoped secret only after successful
  authorization.
- [ ] Enforce one upstream request, the 30-second timeout, no retries,
  response-body cap of 0.5 MB without truncation, valid-JSON/text handling,
  no upstream response-header forwarding, and the agreed HTTP 200
  `connector` response.
- [ ] Map invalid request/key/authorization/transport/over-limit/timeout to
  HTTP 400/401/403/502/504 with the agreed safe
  `{"error":{"code":"...","message":"..."}}` envelope. Secret-store
  unavailability fails closed with a safe platform failure.
- [ ] Add separate minimal usage outcome persistence with exactly one
  outcome per handled proxy request; keep it distinct from structured
  security/application logs and exclude keys, tokens, credentials, and
  unnecessary personal data.
- [ ] Add unit, property-based, PostgreSQL-backed, Bruno, and controlled
  upstream tests for tenant agreement, allow-list rejection, response
  mapping, body bounds, errors, no retries, and safe recording.
- [ ] Story traceability: US-08, US-09, US-10, US-11.

### Step 8: Generate tenant/operator management UI

- [ ] Evolve `centre-lock/web/src/App.tsx` and add focused modules under
  `centre-lock/web/src/` for sign-in/session state, tenant membership and
  invitations, catalog browsing/docs, tenant integration settings, API-key
  lifecycle, tenant provisioning, and protected catalog administration.
- [ ] Preserve the Development catalog preview where useful; use server-side
  authorization as the authority and do not infer tenant scope from editable
  browser state.
- [ ] Never redisplay saved credentials, invitation tokens, or API keys;
  show a usable API key only once after creation.
- [ ] Use accessible labels, keyboard operation, announced loading/success/
  error states, confirmation for destructive actions, and stable
  `data-testid` attributes on interactive controls.
- [ ] Add focused UI tests and expand `centre-lock/web/e2e/homepage.spec.ts`
  or add journey specs in `centre-lock/web/e2e/` for sign-in, member,
  integration, operator, and key flows.
- [ ] Use React Router only if route separation requires it; pin any added
  dependencies and update `web/package-lock.json`.
- [ ] Story traceability: US-01 through US-07.

### Step 9: Add property-based tests and complete cross-boundary coverage

- [ ] Add pinned `FsCheck.Xunit` to
  `centre-lock/tests/CentreLock.Api.Tests/CentreLock.Api.Tests.csproj` and
  domain generators/properties for tenant isolation, invitations, key
  lifecycle, configuration uniqueness, allow-list matching, response size,
  error mapping, and request outcomes.
- [ ] Add pinned `fast-check` and Vitest to `centre-lock/web/package.json`
  and lockfile; test pure TypeScript validation/normalization properties.
- [ ] Keep PBT deterministic, bounded, shrinking-enabled, and seeded in
  failure output; supplement rather than replace example, PostgreSQL,
  Bruno, and Playwright tests.
- [ ] Ensure CI runs the new test suites, but do not add production deployment
  or provider-specific Terraform workflows.
- [ ] Story traceability: US-01 through US-11; NFR PBT-01 through PBT-10.

### Step 10: Implement production-shaped OCI packaging and update developer docs

- [ ] Modify `centre-lock/Dockerfile` to build the React UI and include its
  static output in the .NET application image, preserving the separate
  local-development Compose workflow where needed.
- [ ] Configure ASP.NET static-file and SPA fallback behavior without
  swallowing API/health/Swagger errors; preserve local Nginx development
  behavior if it remains in Compose.
- [ ] Add/modify focused health, readiness, and configuration checks; do not
  apply database migrations on startup.
- [ ] Update `aidlc-docs/development/setup.md` and the relevant application
  documentation/API collection with environment variables, local identity/
  adapter limits, migration commands, test setup, and routes.
- [ ] Do not create Terraform or cloud provisioning code while provider
  selection is deferred.
- [ ] Story traceability: US-01 through US-11 and deployment architecture.

### Step 11: Verify implementation scope and generate summaries

- [ ] Run targeted .NET tests/format verification, PostgreSQL migration and
  persistence checks, Bruno contract tests, web lint/build, and Playwright
  tests appropriate to the changes. Report commands/results without
  representing unrun checks as passing.
- [ ] Verify every story acceptance criterion is implemented or explicitly
  marked blocked by the deferred production email/secret adapter decision;
  keep provider-specific production gates visible.
- [ ] Check `centre-lock/` for accidental duplicate/renamed source files and
  verify secrets, test credentials, and generated build artifacts are not
  added to source control.
- [ ] Write factual implementation summaries under
  `aidlc-docs/construction/centre-lock-application/code/` and update this
  plan's step and story checkboxes as work completes.
- [ ] Prepare the Code Generation completion review; do not begin Build and
  Test stage until explicit approval of generated code.

## Completion conditions

- All steps above are completed and checked, or explicitly reported as
  blocked with the reason and remaining decision.
- All 11 stories are implemented or their deferred production dependency is
  clearly identified; no feature is represented as complete when its
  delivery adapter is unavailable.
- Tests, migrations, deployment packaging, and documentation reflect the
  code actually generated.
- The user reviews and approves the generated code before transition to
  Build and Test.
