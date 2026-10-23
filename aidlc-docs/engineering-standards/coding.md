# Coding and engineering standards

These standards apply across the repository. Enforce mechanical rules with
formatters, analyzers, linters, and CI checks as the application is scaffolded.
Update this document when the selected tools establish more specific
conventions.

## Engineering principles

- Prefer clear, explicit code and small, cohesive changes over speculative
  abstractions.
- Build incrementally as small, independently releasable features. Keep the
  main branch releasable at all times.
- Pull requests (PRs) must be reviewed by the project owner before merge.
  Keep each PR focused, with passing required checks and enough context to
  review safely.
- Preserve the modular monolith and its module boundaries. Do not introduce
  cloud-specific dependencies or independently deployed services without an
  accepted architecture decision.
- Make failures explicit. Validate untrusted input at system boundaries;
  handle errors deliberately. Avoid broad catches, silent defaults, and
  success-shaped fallbacks.
- Document non-obvious decisions and behavior. Record material architecture
  choices as architecture decision records (ADRs).
- Version database changes as reviewed Entity Framework Core migrations.
- Keep local development reproducible with the repository's Docker Compose
  setup.

## .NET

- Follow the [Microsoft C# coding conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) and standard ASP.NET Core conventions.
- Preserve nullable reference type and compiler/analyzer feedback; do not
  suppress warnings without a documented reason.
- Use asynchronous I/O for network and database operations.
- Keep transport concerns, application behavior, and persistence concerns
  separated according to the architecture boundaries.
- Make tenant scope explicit in operations involving tenant-owned data.
- Use dependency injection and typed interfaces at module and connector
  boundaries; avoid service-location and cross-module persistence access.

## TypeScript

- Keep TypeScript strict and avoid `any`; model API inputs and outputs with
  explicit types.
- Follow the repository's configured formatter and lint rules.
- Keep UI components focused. Enforce authentication and authorization on the
  server, not only through UI visibility.

## Security and data handling

- Apply least privilege and enforce authorization on the server for every
  protected operation.
- Resolve tenant context from trusted authenticated/routing context; never
  trust an unverified tenant ID supplied by a caller.
- Keep secrets out of source control, container images, responses, and logs.
  Use the hosting environment's secret manager after a provider is selected.
- Do not log credentials, API keys, or sensitive upstream payloads. Keep
  request logging to the minimum required by the product specification.
- Validate untrusted data, including proxy paths and payloads, against the
  appropriate contract and allow list.
- Include dependency/security scanning in CI once the application pipeline is
  established.
- See [`security.md`](security.md) for the security review checklist for each
  change.

## Tests and quality gates

- Aim for 100% unit-test coverage of meaningful, testable production behavior.
  Do not add tests that only exercise mocks or test the test itself. Exclude
  generated or genuinely valueless code only with a documented rationale.
  Select and document the coverage metric when CI coverage reporting is
  introduced.
- Add feature tests for implemented behavior wherever practical, including
  authorization, tenant isolation, and failure cases.
- Maintain a version-controlled Bruno collection that exercises every API
  endpoint and representative classes of input: valid, invalid, empty or
  malformed payloads where relevant, unauthenticated/unauthorized requests,
  and malicious input such as SQL-injection attempts. Assert expected status
  codes and response shapes. Run the collection in CI using the Bruno CLI
  against isolated test resources.
- Aim for complete endpoint and representative-input API test coverage. This
  behavioral coverage is distinct from source-code line coverage.
- Test PostgreSQL persistence and tenant isolation against PostgreSQL, not
  only an in-memory substitute.
- Test proxy and connector behavior without relying on live third-party
  services; use controlled test doubles or local fixtures.
- Run build, formatting/analyzer, unit, and relevant integration/API checks in
  CI for each PR. A failed required check blocks merge.
- For PR-specific deployments, keep test resources, data, and credentials
  isolated from other PRs and production, and tear them down when no longer
  needed.
- Add or update Playwright tests for changes to user-visible web behavior.
  See [`testing.md`](testing.md) for guidance on selecting test layers and
  running the UI suite.

## Delivery and review

- Use GitHub Actions for CI and Terraform as the infrastructure-as-code
  source of truth.
- Build an immutable container artifact and promote the same artifact through
  the delivery environments.
- A merge to main triggers automatic deployment through the configured
  environments to production after required automated checks pass.
- Keep infrastructure and application configuration separate from secrets;
  never commit credentials or place them in build artifacts.
- Review schema and infrastructure changes in the PR that introduces them.
- See [`ci-cd.md`](ci-cd.md) for pipeline change and scaling guidance.
