# CI/CD standards

The GitHub Actions workflow in
[`centre-lock/.github/workflows/ci.yml`](../../centre-lock/.github/workflows/ci.yml) is the
current source of truth for automated checks. Update it when a change adds,
removes, or changes a required build, test, dependency, or delivery step.

Non-production deployment is handled by separate workflows
(`deploy-preview.yml`, `deploy-staging.yml`, `deploy-infrastructure.yml`) and
`terraform-validate.yml`, which validates every Terraform root under
`infrastructure/`. Preview and staging deploys are disabled until the
`LEASE_CONTROLLER_URL` repository variable is set, and any step that needs the
shared database must first renew its lease with `.github/scripts/renew-lease.sh`.
Preview deploys run only for same-repository PRs; fork PRs must never receive
cloud credentials.

## Pull request checks

- Run build, formatting/analyzer, unit, and relevant integration/API checks
  for each PR. Include web lint/build and Playwright UI tests for web changes.
- Keep dependency audits and other security checks enabled for the manifests
  and build paths they are intended to cover.
- Required checks must fail clearly when a check fails. Do not mask errors,
  continue past failed prerequisites, or report a skipped check as a pass.
- Provision test resources only as needed. Keep PR resources, data, and
  credentials isolated from other PRs and production, and clean them up even
  when a job fails.
- Grant workflow permissions only as needed. Keep secrets out of logs,
  artifacts, and PR-controlled code paths.

## Evolving the pipeline

As applications and suites grow:

- Split work into independently runnable jobs by application or check
  responsibility when that improves parallelism or failure diagnosis. Keep
  true prerequisites explicit and avoid unnecessary serial dependencies.
- Cache package downloads using the corresponding lockfiles as cache keys.
  Do not cache generated build output in a way that can hide stale or
  incompatible artifacts.
- Use path filters only when they cannot skip checks needed by shared code,
  contracts, build configuration, or workflow changes. Prefer reliable
  coverage over fragile speed optimizations.
- Keep test environments and fixtures deterministic, isolated, and
  disposable. Revisit startup, readiness, timeout, and cleanup behavior as
  test services are added.
- Build immutable container artifacts and promote the same artifact through
  delivery environments; do not rebuild different artifacts for each
  environment.
- Keep deployment and infrastructure changes aligned with the accepted
  architecture and delivery decisions. Review schema and infrastructure
  changes in the PR that introduces them.

When changing a workflow, check its triggers, job dependencies, permissions,
cache inputs, secrets exposure, failure conditions, and cleanup paths. Verify
workflow syntax and run affected commands locally where possible. If a
workflow-only behavior cannot be exercised locally, report that limitation
and rely on CI to validate it.
