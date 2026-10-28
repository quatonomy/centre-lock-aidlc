# Code Quality Assessment

## Test Coverage

- **Overall**: Meaningful test layers exist, but this review did not execute tests or calculate coverage.
- **Unit/feature tests**: xUnit coverage for endpoint validation, proxy forwarding/allow-list behavior, credential forwarding, Development route behavior, and health.
- **Integration tests**: PostgreSQL persistence and application startup tests are opt-in locally via `CENTRELOCK_TEST_DATABASE`; CI provisions PostgreSQL.
- **API tests**: Bruno covers local API contracts and live-tagged Star Wars endpoint requests.
- **UI tests**: Playwright covers catalog browsing, selected details, loading, empty, and unavailable states.
- **Coverage reporting**: Coverlet is referenced, but current CI does not collect/publish coverage or enforce a threshold. The coding standard's coverage target is not currently a pipeline gate.

## Code Quality Indicators

- **Formatting**: `dotnet format --verify-no-changes` is in CI.
- **Web lint/build**: Oxlint and TypeScript/Vite build run in CI.
- **Dependency auditing**: `npm audit` runs for web and API-test dependencies.
- **API security-sensitive tests**: Existing tests exercise base URL/path validation, allow-list rejection, Development-only route mapping, and refusal of credentialed integrations on the unauthenticated Development proxy.
- **Documentation**: Setup instructions, product requirements, accepted ADRs, architecture overview, and coding/testing/security/CI standards provide substantial project context; these documents are now stored in `aidlc-docs/`.
- **CI**: API and web checks are separate jobs; Compose teardown runs with `if: always()`.

## Technical Debt and Gaps Relative to Product Goals

- Google OIDC sign-in, tenant memberships with `Admin`/`Member` roles, and operator tenant provisioning exist, but role-based authorization differences, membership management/invitation, tenancy resolution for tenant-owned data, and tenant isolation (including row-level security) are not implemented.
- Tenant integration configuration and the management UI are not implemented; the UI shows sign-in state and memberships plus a catalog preview, and members see the platform-wide enabled catalog.
- Consumer API key creation, hashing, rotation, revocation, and authentication are not implemented.
- Tenant-scoped upstream secret storage/retrieval is not implemented; only an interface and a deliberately unconfigured implementation exist.
- GitHub is an initial integration goal but has no seeded catalog entry or connector setup.
- Usage recording, metering, editable plans, billing, and operator catalog-management workflows are absent.
- The generic connector's catalog persistence is present, but no authorized production catalog administration API exists.
- Non-production delivery exists (Terraform on GCP, per-PR Cloud Run previews, staging deploy on merge). Still missing: promotion of one immutable artifact through test/staging/production, a production environment, API/feature checks against previews beyond a readiness smoke test, controlled (non-startup) production migrations, keeping the database password out of Terraform state, and tested restore/recovery. Preview and staging workflows are inert until `LEASE_CONTROLLER_URL` is set; this static review did not exercise them.
- The cloud image serves the UI with the API, while Compose still runs separate API and Nginx UI services.
- Several specification questions remain open, including tenant API error/timeout details, connector credential flows, GDPR retention/access, and production resiliency and change-management targets. The per-request amount is now £0.01 in future billing scope; usage and billing are deferred beyond the MVP.
- Active GitHub issues add questions about DTO publication, auditability, DORA metrics, AI-assisted integration/test maintenance, AI guidance structure, CI coverage thresholds, monitoring/alerting, and hosting/deployment. They are captured as future context rather than accepted requirements in `github-issue-context.md`.

## Patterns and Anti-patterns

- **Good patterns**: Small module interfaces; EF Core migrations; explicit environment gating for unauthenticated Development routes; controlled outbound HTTP test handler; PostgreSQL-backed persistence tests; method/path allow-list validation; app startup seed is idempotent; live API calls are excluded from normal Bruno tests.
- **Risks to keep visible**: The Development proxy uses a placeholder tenant ID and must not be reused as a tenant-facing route. The upstream credential provider is intentionally nonfunctional until tenant-scoped secret storage is decided and implemented. Product security properties must not be inferred from the local Development controls.

## Review Scope

Static review of tracked repository files and project documentation, plus retrieval of eight open GitHub issues. No build, test suite, live upstream request, or security audit was run.
