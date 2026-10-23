# Testing standards

Choose tests that verify the behavior changed, at the layer where that
behavior is observable. A change may need more than one test type. Passing
unit tests or reaching a line-coverage target does not replace API contract
or UI behavior tests.

## Unit and feature tests

- Aim for 100% unit-test coverage of meaningful, testable production
  behavior. Do not add tests that only exercise mocks or test the test
  itself. Exclude generated or genuinely valueless code only with a
  documented rationale.
- Unit-test business rules and edge cases without external services.
- Add feature tests for implemented behavior wherever practical, including
  authorization, tenant isolation, invalid input, and failure cases.
- Test PostgreSQL persistence and tenant isolation against PostgreSQL, not
  only an in-memory substitute.
- Test proxy and connector behavior with controlled test doubles or local
  fixtures, not live third-party services.
- Maintain at least one successful behavioral test for every enabled
  integration endpoint (each catalogued HTTP method and path-template
  combination), using controlled upstream responses so the tests can run in
  CI. Keep opt-in live smoke cases for validating the external provider itself
  in the same Bruno collection, tagged so the normal CI run
  can exclude it. When a catalog endpoint is added, removed, or changed,
  update this endpoint-level test coverage with it.
- Keep tests deterministic and isolated. Do not depend on production data,
  shared mutable state, or credentials belonging to a real integration.

## API integration tests (Bruno)

When a change adds or changes an API endpoint or externally observable API
behavior:

- Update the version-controlled Bruno collection under `tests/api/`.
- Cover every endpoint and representative input classes: valid, invalid,
  empty or malformed payloads where relevant, unauthenticated and
  unauthorized access, and malicious input such as SQL-injection attempts.
- Assert expected status codes and response shapes, including relevant
  error responses.
- Include or link to endpoint-level successful proxy cases for each enabled
  integration method/path combination. Verify forwarding against a controlled
  upstream fixture in CI; tag tests against public third-party services as
  `live` in the same collection and run them explicitly.
- Run the collection through the Bruno CLI in CI against isolated test
  resources. Use Compose services and controlled fixtures; do not call live
  third-party services.
- Review adjacent requests when a shared contract or authorization behavior
  changes so the collection remains consistent with the API.

API behavioral coverage is distinct from source-code line coverage. A new
endpoint is not covered merely because its implementation has unit tests.

## UI end-to-end tests (Playwright)

When a change alters user-visible web behavior, add or update Playwright tests
under `web/e2e/`:

- Exercise the user-visible result and important interactions, including
  relevant loading, empty, validation, and error states.
- Prefer accessible roles, labels, and visible text over implementation
  details such as CSS classes or component structure.
- Keep tests isolated from real identity providers and third-party
  integrations. Use controlled test data and service doubles for behavior
  that needs a backend.
- Run the web lint/build checks and `npm run test:e2e` from `web/`. The
  Playwright configuration starts the local Vite server; CI installs the
  configured Chromium browser before running the suite.

Do not add browser automation for changes with no user-visible UI impact
unless it verifies a relevant cross-layer behavior that cannot be adequately
covered at a lower layer.

## Selecting checks for a change

For each change, identify the affected behavior and run its relevant checks:

| Change area | Expected test consideration |
| --- | --- |
| Business logic or API implementation | Unit/feature tests; Bruno tests for changed API behavior |
| Persistence, migrations, or tenant scoping | PostgreSQL-backed tests and tenant-isolation cases |
| Proxy or connector behavior | Controlled upstream fixtures; Bruno tests for changed API contract |
| User-visible web behavior | Playwright end-to-end test(s), plus web lint and build |
| Shared contracts or cross-layer behavior | Tests at each affected boundary |
| Build, dependency, or CI configuration | Run the affected checks locally where possible and verify the workflow still covers them |

If a relevant test cannot be added or run, state why and what remains
unverified in the change review. Do not silently treat a skipped check as
passing.
