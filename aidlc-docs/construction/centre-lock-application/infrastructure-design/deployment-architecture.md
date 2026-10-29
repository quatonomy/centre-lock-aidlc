# Deployment Architecture: CentreLock Application

## Deployment unit

CentreLock deploys to Cloud Run as one versioned OCI image containing the
ASP.NET Core modular monolith and built React/TypeScript UI, served from the
same origin. The local Docker Compose setup retains separate API and Nginx
targets for development. The application is a single deployable unit with
logical modules, not a set of independently deployed services. PostgreSQL,
the secret manager, ingress, and observability are managed infrastructure
dependencies selected with the hosting provider.

Docker Compose remains the local development environment. Google Cloud is
the selected hosting provider, with Cloud Run and Cloud SQL for PostgreSQL
as the initial deployment targets in `europe-west2`. Exact instance sizing,
DNS zone, registry, and observability services remain to be selected.
Production topology and readiness requirements still need validation
against these services before production use.

## Environment model

| Environment | Purpose and data | Provisioning and access | Lifecycle |
|---|---|---|---|
| Local | Developer feedback using Docker Compose and synthetic data/credentials | No production credentials; local-only configuration and isolated database | Developer controlled |
| Test / CI | Automated unit, integration, API, and UI checks; ephemeral databases or isolated test database/schema | GitHub Actions, short-lived test credentials, mocked external identity/upstreams except controlled integration tests | Created for CI and removed after completion |
| Pull request | Isolated testing of that pull request without overwriting another PR's environment | Temporary Cloud Run service and separate database on the shared non-production Cloud SQL instance; no production data/secrets, resource limits, restricted access | Created and updated by its PR workflow; automatically destroyed on close/merge or expiry |
| Staging | Stable shared environment for regression testing changes together, migration rehearsal, and release-candidate validation | Cloud Run service and separate database on the non-production Cloud SQL instance; protected deployment | Updated by CD from commits to `main`; database requires an explicit 60-minute lease |
| Production | Customer management and proxy traffic; single region, multi-zone | Strongly isolated production boundary; narrow federated identities; protected approval and human change record | Persistent; deploy only approved immutable artifacts |

There is no shared devtest environment. PR environments provide isolated
testing; staging is the shared stable environment for regression testing.
Never clone production customer data into non-production environments.
The preview databases and staging database are logically separate but share
one Cloud SQL instance. CI or an authorized operator explicitly starts or
renews a 60-minute access lease when testing is needed. A lease-expiry
mechanism automatically stops the instance when the lease expires unless it
has been explicitly renewed. While stopped, every environment using that
instance is unavailable. Database storage and retained backups continue
to incur charges while the instance is stopped. The lease-enforcement
mechanism must be implemented and verified before enabling deployment
workflows.

## Infrastructure and trust boundaries

```text
GitHub repository
   |
   +--> GitHub Actions: build, test, scan, SBOM, publish immutable image
   |          |
   |          +--> private OCI registry
   |          +--> short-lived federated identity for reviewed Terraform/deploy
   |
   +--> Pull request: Terraform apply to its own isolated preview only
        (see Automation model); shared/staging/production apply from main

Public clients
   |
   +--> public DNS / TLS edge / load balancer / access logs
             |
             +--> private managed-container service
                       |--> private multi-zone managed PostgreSQL
                       |--> managed secret manager via runtime identity
                       |--> catalog-allowed upstreams via controlled egress
                       +--> provider-managed logs / metrics / alerts
```

Provider-specific network constructs and trust policies are selected later.
The chosen implementation must preserve public edge-only ingress, private
data access, restricted egress, tenant isolation, and least privilege.
The Terraform bootstrap and validation workflow are scaffolding only.
Application deployment workflows remain disabled until remote state,
federated identity, and the verified one-hour database lease controller
are configured.

## Build and release flow

1. A pull request runs formatting, build, unit and property-based tests,
   PostgreSQL-backed tests where configured, UI/API checks, dependency and
   image scans, and Terraform validation/plan review. Its workflow deploys an
   isolated preview for testing, requesting a non-production database lease
   if needed; PR workflows never apply production infrastructure.
2. On an approved merge, GitHub Actions builds one immutable image, pins its
   version/digest, generates the release SBOM, and publishes it to the
   restricted OCI registry. Credentials are not baked into the image. CD
   deploys that artifact to staging.
3. Staging verifies health/readiness, database migrations, tenant isolation,
   proxy contracts, security controls, and regression tests against the
   integrated changes on `main`, requesting a non-production database lease
   if needed.
4. After staging acceptance, promote the same immutable image digest to
   production; environment configuration and secret references vary by
   environment, not the artifact.
5. A production change record identifies the commit/image digest,
   infrastructure plan, approval, rollout, verification, and rollback
   version. Production deployment uses a protected environment and
   short-lived federated identity.
6. Deploy progressively as a canary. Increase traffic only after health,
   error, latency, and saturation checks pass. Halt and return to the previous
   version-pinned image if regression thresholds are exceeded.

Concrete canary percentages, observation windows, and thresholds are set
after measured load and provider selection; they are not invented here.

## Database migrations and rollback

- Apply reviewed EF Core migrations as a controlled, observable release step.
- Use expand-and-contract changes that remain compatible with both the
  current and immediately previous application version during the rollout
  and rollback window.
- Back up before changes with material data risk and verify restore procedures
  before production.
- Application rollback redeploys the previous image digest; it does not
  reverse database writes or schema changes. Database recovery is a separate
  operator-approved procedure.
- Stop promotion if migration, readiness, tenant-isolation, or post-deploy
  checks fail. Document whether recovery requires application rollback,
  forward-fix, or database restore.

## Infrastructure change flow

- Keep Terraform configuration and reusable modules reviewed and versioned.
- Pull requests run formatting, validation, security checks, and a plan
  against the target environment. Plans that contain sensitive data are
  protected and not committed as artifacts.
- Use the versioned GCS remote state bucket with locking, audit, and
  environment-level access separation. Bootstrap the bucket once with local
  state, migrate that state into GCS, then use distinct prefixes for the
  state-bucket root and non-production bootstrap. Do not use local state for
  ongoing infrastructure changes.
- Apply changes only through protected GitHub Actions environments using
  short-lived federated identities. Require explicit production approval;
  do not use long-lived cloud keys in repository secrets.
- Separate plan/read permissions from privileged apply permissions. Restrict
  production apply to reviewed changes and authorized operators.
- Review infrastructure rollback independently from application rollback;
  avoid destructive database or state changes as an automated rollback.

## Automation model (decided 2026-10-28)

Nothing is applied manually long-term. Only the one-time bootstrap (state
bucket, Workload Identity Federation, pipeline service accounts) is manual.

- **CI, per same-repo pull request:** applies a per-PR Terraform root with
  state prefix `previews/pr-<n>` and deploys the image. Destroys it on PR
  close or expiry. It uses a separate least-privilege identity limited to
  `previews/` state and Cloud Run in the non-production project (Cloud Run create cannot be scoped by service name), with no IAM administration and no access to staging or
  production state. Fork PRs receive no credentials.
- **CD, `main`:** applies the staging root, then deploys the image. Shared
  infrastructure (Cloud SQL, lease controller, registry) is applied from
  `main` only, after a reviewed plan.
- **Production:** applies the production root and promotes the same image
  digest after GitHub Environment approval, using a separate identity.
- The lease must be started before any apply that needs the database.
- This supersedes "PR workflows plan only; no apply" above for previews.
- Implemented as three workflows (`deploy-preview.yml`, `deploy-staging.yml`,
  `deploy-infrastructure.yml`) over the Terraform roots `bootstrap` (manual,
  once), `shared`, and `environment` (one root for `pr-<n>` and `staging`).
  Identities are bound to GitHub OIDC subjects: `pull_request` (preview),
  environment `staging`, environment `shared-infrastructure` (approval
  gated). A production root and identity are added when production is built.
- Known gaps: abandoned preview databases need a scheduled sweeper; the
  non-production database password is in Terraform state; migrations run at
  startup in non-production only.

## Environment promotion and cleanup

- Promote the same immutable application image from staging to production;
  do not rebuild for production.
- Keep environment-specific database, identity configuration, secret
  references, domains, and network boundaries separate.
- PR environments use unique names/hostnames, strict quotas,
  short-lived data, restricted access, and automatic teardown after merge,
  closure, or expiry. Cleanup failures alert the maintainers and require
  explicit operator cleanup.
- Serialize Cloud SQL start/stop decisions so cleanup for one PR cannot stop
  the shared instance while another PR or staging test still needs it.
- Test and staging deployments must not share production credentials or
  production database access. Production access is limited to runtime and
  deployment identities that require it.
- Retain deployment records, image digests, scan/SBOM evidence, approvals,
  and rollback references according to the approved policy.

## Operations and recovery

- Route health, security, capacity, database, backup, and deployment alerts
  to the human operator contact path established before production.
- Operations owns external incident and recovery runbooks and executes the
  documented resiliency scenarios. Record test evidence, recovery duration,
  data validation, and corrective actions outside GitHub.
- Production runs in one region and multiple zones with reduced-capacity
  in-region Warm Standby. No cross-region failover is provided.
- Before production, approve availability and RTO/RPO targets, GDPR retention
  and access policy, scale limits, and restore/failover acceptance criteria.
