# Technology and Testing Decisions

## Current scope amendment (2026-10-26)

OIDC/Google authentication and consumer API-key validation are deferred from
the current MVP. Do not configure identity-provider credentials as a
prerequisite for current product operation. Bespoke tenant subdomains under
a shared base domain are accepted for routing; domain/TLS provisioning is
still provider-dependent.

## Accepted application stack

| Area | Decision | Notes |
|---|---|---|
| Backend/API | C# / ASP.NET Core on .NET 10 | Existing codebase and accepted ADR |
| Architecture | Modular monolith | One deployable application; explicit logical module boundaries |
| UI | React with TypeScript | Build and serve with application in accepted production shape; local Nginx arrangement may remain a development detail |
| Persistence | PostgreSQL with EF Core/Npgsql | Shared database; reviewed EF Core migrations applied as controlled deployment steps |
| Packaging | OCI-compatible container | Docker tooling and Compose for local development; Kubernetes is not required |
| CI/CD | GitHub Actions | Accepted delivery standard; canary, promotion, rollback, and environment implementation remain to be designed |
| Infrastructure as code | Terraform | Accepted source of truth; provider and backend remain undecided |
| Identity | Deferred | No application authentication or sign-in in current MVP |
| Upstream secrets | Provider-neutral application interface | Hosting provider's secret manager selected after provider choice |

## Testing stack

### C# application logic

- Existing runner: xUnit 2.9.3 in the API test project.
- Property framework: FsCheck with xUnit integration (`FsCheck.Xunit`).
- Use FsCheck generators for valid tenants, memberships, invitations, roles,
  API keys, endpoint templates, and bounded proxy responses.
- Preserve automatic shrinking; report seed and minimized counterexample on
  failure. Pin package versions and include properties in CI.
- The exact FsCheck package version should be selected as a compatible,
  supported version when the dependency is added; do not use an unpinned
  floating version.

### TypeScript logic

- Property framework: fast-check with Vitest for pure frontend/domain logic,
  form validation, normalization, and serialization where those behaviors
  are implemented in TypeScript.
- Add Vitest and fast-check as pinned development dependencies with a lockfile
  update when frontend property tests are introduced.
- Existing Playwright end-to-end tests remain responsible for user-visible
  workflows; they do not replace PBT for pure transformations.
- Keep external identity and upstream behavior mocked in tests. Do not call
  real providers from PBT.

### Complementary checks

- Unit and feature tests cover business rules and failure paths.
- PostgreSQL-backed tests verify persistence and tenant isolation against
  PostgreSQL.
- Bruno checks externally observable API contracts and status/body behavior.
- Playwright validates UI workflows and accessibility-relevant interactions.
- Keep all test inputs isolated and deterministic; no production data or
  credentials.

## Operational tooling and constraints

- Use structured centralized logs and provider-managed monitoring where
  available; exact monitoring/logging product awaits hosting selection.
- Managed PostgreSQL backups must be encrypted and restore-tested before
  production. Exact retention follows approved RTO/RPO and GDPR rules.
- Use dependency lockfiles, dependency vulnerability scanning, pinned build
  tools/base images, and SBOM generation per Security Baseline.
- Do not add a cloud SDK or secret-manager package until the hosting provider
  and its supported secret service are selected.

## Deferred technical choices

- Hosting provider, container runtime, Terraform backend, and concrete
  managed monitoring/logging service.
- Exact versions for new PBT libraries, selected at dependency addition.
- Capacity sizing, production latency SLO percentiles, scaling bounds, and
  alert thresholds after load-profile measurement.
- DR/chaos test schedule and execution procedure, to be decided in NFR Design.
