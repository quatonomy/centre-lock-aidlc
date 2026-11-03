# NFR Requirements: CentreLock Application

## Scope and classification

**Scope amendment (2026-10-26):** All authentication and sign-in are
deferred. Current MVP scope is public catalog discovery and tenant-subdomain
routing; tenant-private management and the consumer proxy are not exposed.
Criticality statements below that assume tenant traffic describe future
scope and must be re-evaluated before production.

- **Deployable workload:** CentreLock modular-monolith application with
  PostgreSQL-backed tenant and catalog data.
- **Production criticality:** High. An outage prevents customers from
  managing integrations and using the tenant-facing integration proxy; the
  service stores tenant and configuration data.
- **MVP operating mode:** No availability SLA. The service may be turned off
  while idle to control operating costs.
- **Production availability direction:** Establish a 99.95% monthly
  availability target before production customer traffic.
- **Production recovery direction:** Warm Standby with a single-region,
  multi-zone topology. Exact RTO/RPO values are deferred during MVP and must
  be approved before production customer traffic.

This is a requirements baseline, not a claim of present implementation or
production readiness.

## Performance and capacity

- **MVP load:** No numeric capacity commitment. Measure a baseline and
  load-test a documented expected traffic profile before production
  customer traffic.
- **Latency:** No MVP latency SLA. Measure platform overhead and end-to-end
  proxy latency separately during MVP; establish numeric production
  percentile targets before production customer traffic. End-to-end proxy
  latency includes external upstream time and must be interpreted separately
  from CentreLock processing time.
- **Request behavior:** Upstream calls have a 30-second timeout, are not
  retried, and accept upstream response bodies up to 0.5 MB. Oversized
  upstream bodies return 502 without truncation.
- **Capacity evidence gate:** Before production traffic, document expected
  tenant/request profile, test results, resource scaling limits, dependency
  quotas, and the trigger for capacity review. Do not fabricate expected
  customer volume.

## Availability and recovery

- The MVP may be shut down when idle; do not present an MVP uptime commitment.
- Before production traffic, set a 99.95% monthly availability target and
  approve exact maximum RTO/RPO values.
- Use Warm Standby as the recovery strategy direction and single-region,
  multi-zone topology. The system must tolerate an availability-zone failure
  in its production design; regional failure recovery depends on the approved
  RTO/RPO values.
- Before production use, automated managed backups must cover persistent
  data, be encrypted, and have a documented restore-validation procedure.
  Select backup retention only after reconciling recovery objectives and
  GDPR policy.
- Document failover, failback, post-recovery validation, and stakeholder
  communication procedures before production. Do not represent these as
  complete until validated.
- Classify the database and application as High for production; document
  their dependencies and outage impact in resiliency design.

## Security, privacy, and observability

The Security Baseline is blocking and applies to all design and
implementation artifacts. Requirements include:

- Encrypt persistent data at rest with managed/customer-managed keys and
  enforce TLS 1.2+ in transit.
- Enable access logging on every external network intermediary and structured
  application logging with timestamps, correlation IDs, severity, and safe
  messages.
- Never log passwords, OIDC tokens, consumer API keys, upstream credentials,
  or unnecessary personal data.
- Set restrictive web security headers, validate every input and payload
  size, parameterize database operations, and enforce server-side object and
  function authorization with deny-by-default tenant isolation.
- Restrict CORS, protect credentials, use rate limiting, generic production
  errors, safe deserialization, dependency pinning/scanning, SBOM generation,
  and least-privilege infrastructure permissions.
- Configure security-event alerting for repeated authentication failures,
  authorization violations, and privilege changes. Keep logs tamper-evident,
  prevent application roles from deleting their own security logs, and
  define dashboards/alarms for operational and security metrics.
- Retain security/application logs for at least 90 days. Keep them separate
  from minimal per-request success/failure usage records.
- Approve a GDPR-compliant retention/access and permitted-use policy for
  usage and security/application records before production traffic.

All rules must be verified in the corresponding stage; unresolved findings
are blocking and must not be represented as compliant. Some rules depend on
hosting-provider controls and remain open until provider selection.

## Resiliency and change management

- Use GitHub Actions for CI and Terraform as the infrastructure source of
  truth. Keep the hosting provider and provider-specific resources
  undecided until evaluated.
- Deploy an immutable, version-pinned artifact through isolated environments;
  use canary deployment with progressive traffic shift and automated
  verification where supported.
- On failed deployment, redeploy the previous version-pinned application and
  infrastructure artifact. Database rollback must be migration-aware; do
  not assume old application code reverses data changes.
- Maintain a lightweight GitHub-visible production change record, approval,
  and rollback note.
- Production incident response is human-managed and documented outside
  GitHub for now.
- Define explicit timeouts for every external dependency, resource limits,
  health/readiness checks, and safe degraded behavior. Circuit breaking for
  upstream dependencies is a design decision, subject to preserving the
  no-retry requirement.
- The Resiliency Baseline requires a DR/resiliency testing approach and test
  scenarios. The owner decision is deferred to NFR Design; Operations will
  execute any approved drills.

## Accessibility and usability

- The tenant and operator management UI targets WCAG 2.2 AA.
- User-facing flows must support keyboard operation, semantic structure,
  accessible labels, announced status/validation messages, visible focus,
  and accessible errors.
- Verify with automated accessibility checks and manual keyboard/screen
  reader review of critical sign-in, membership, integration, and key
  workflows.

## Maintainability and testing

- Preserve the .NET modular monolith and explicit module contracts. Keep
  provider-specific infrastructure and secret management behind adapter
  boundaries.
- Apply reviewed EF Core migrations as controlled deployment steps.
- Apply full Property-Based Testing rules to business logic and data
  transformations; PBT supplements rather than replaces example-based,
  API, PostgreSQL, and Playwright tests.
- Use FsCheck with xUnit for C# properties and fast-check with Vitest for
  TypeScript logic. Pin compatible versions in NuGet/npm dependency metadata
  and lockfiles; run PBT in CI with shrinking and reproducible seed reporting.
- Use domain-specific generators with boundary cases. Stateful properties
  cover invitation, membership, API-key, and tenant configuration sequences.

## Production-readiness gates

Before production customer traffic:

1. Approve exact RTO/RPO values and confirm the 99.95% monthly target.
2. Approve GDPR-compliant retention/access policy and backup retention.
3. Select hosting provider and secret manager; validate encryption, network,
   logging, backups, and least-privilege controls.
4. Set numeric capacity and latency targets based on measured traffic and
   load tests; define scaling bounds and alert thresholds.
5. Validate restore/failover runbooks and the selected resiliency test plan.
6. Demonstrate Security Baseline and PBT compliance in CI and deployment
   artifacts.
