# NFR Design Patterns: CentreLock Application

## Design status and scope

**Scope amendment (2026-10-26):** The current MVP defers all authentication
and sign-in, including consumer API keys, and does not expose tenant-private
management or proxy execution. Availability and criticality statements below
that assume those capabilities describe future-state operations. The
security baseline still applies to public catalog and tenant-routing
surfaces.

This artifact defines logical, provider-neutral patterns for the approved
single modular-monolith unit. It does not claim the patterns are implemented
or certify production readiness. Hosting-provider controls and several
production targets remain gated as listed below.

## Availability, criticality, and recovery

- Classify the production application and PostgreSQL data as **High**
  criticality. An outage blocks tenant management and consumer integration
  traffic and prevents access to tenant configuration.
- During MVP there is no availability SLA; the service may be stopped while
  idle.
- Before production customer traffic, establish the 99.95% monthly
  availability target and approve exact RTO/RPO values.
- The selected topology is **single-region, multi-zone**. Production compute,
  database, and traffic routing must tolerate an availability-zone failure
  without requiring control-plane action to restore service.
- Warm Standby is **in-region reduced-capacity standby** for instance/zone
  failures. Full-region failure is explicitly out of scope under the selected
  regional topology. Do not represent this as regional disaster recovery.
- Use provider-managed, encrypted automated backups for persistent data.
  Test restore integrity before production and periodically thereafter.
  Backup retention is set after the GDPR policy and RPO decision.
- Document zone failover/failback, recovery validation, backup restore,
  communication, and operator actions before production.
- No cross-region replica or regional failover mechanism is included in this
  design.

## Observability and health

- Collect request count/throughput, latency distributions, platform/upstream
  error rate, timeout rate, response-size rejection rate, CPU/memory,
  database connection/pool saturation, and scaling/capacity indicators.
- Measure management/API platform overhead separately from end-to-end proxy
  latency because upstream latency is external. MVP gathers baselines; numeric
  production capacity and latency targets are approved before customer
  traffic.
- Attach timestamp, severity, correlation/request ID, event code, and safe
  context to structured application logs. Redact API keys, OIDC tokens,
  upstream credentials, secret references where sensitive, and unnecessary
  personal data.
- Keep minimal usage outcomes separate from security/application logs.
  Retain security/application logs for at least 90 days, with tamper-evident
  storage and no application permission to delete or modify its own
  centralized logs.
- Audit critical administrative/catalog/tenant changes with actor, timestamp,
  action, resource, and safe before/after values. Never include credential or
  API-key material in audit data.
- Use correlation IDs across the request pipeline. Distributed tracing
  across services is N/A because the product is one service; in-process
  activities may be emitted for diagnostics.
- Define dashboards and alarms for availability, latency, errors, capacity,
  database health, backup failures, zone redundancy degradation, repeated
  authentication failures, authorization failures, and privilege changes.
  Alert routes go to the human operators identified by the external incident
  process.
- Provide shallow process-liveness and deep readiness checks. Readiness
  verifies required local dependencies such as PostgreSQL; do not require all
  upstream integrations to be healthy for the platform to accept traffic.
  Integrate health status with load balancing/routing. Use synthetic
  monitoring for public sign-in and consumer proxy journeys where the selected
  host permits safe credentials and test tenants.

## Capacity, scaling, and performance

- Keep the deployment horizontally scalable as a stateless application
  process; shared state and tenant data live in PostgreSQL and the secret
  manager.
- Production must define minimum/maximum compute capacity and an autoscaling
  policy after expected traffic is measured. Candidate triggers include
  CPU, memory, request rate, and saturation/latency metrics; avoid scaling
  only on CPU where database or upstream queues are saturated.
- Set database connection-pool limits and outbound HTTP concurrency/resource
  limits to prevent a scale-out event from overwhelming dependencies.
- Identify hosting-provider quotas and monitor utilization; plan quota
  increases before production if the load profile approaches limits.
- No numeric MVP throughput or latency SLA is set. Capture a representative
  load profile, baseline, and load-test report before selecting production
  limits. Derive alerts and scale thresholds from that evidence.
- Maintain request/body limits and the 30-second upstream deadline. Bound
  memory use by streaming or otherwise enforcing the 0.5 MB upstream body
  limit before buffering unbounded response data.

## Dependency isolation and failure handling

- Put explicit timeouts on OIDC metadata/token calls, PostgreSQL operations,
  secret-manager calls, and upstream HTTP calls. Only the upstream timeout is
  fixed at 30 seconds by the approved requirements; provider-specific
  timeout values are selected and documented before production.
- Use a per-upstream/dependency circuit breaker to stop repeatedly sending
  requests to a persistently unhealthy upstream. An open breaker returns the
  normal safe platform failure; it does not queue, replay, or retry a
  consumer request.
- Never automatically retry an upstream request. Half-open health probes, if
  used, must be separate from consumer requests and must not duplicate
  customer operations.
- Apply concurrency limits/bulkheads so one slow upstream or secret service
  cannot exhaust all API request capacity.
- Define degraded behavior: catalog browsing and tenant management remain
  available when an unrelated upstream is unavailable; proxy calls to the
  unavailable upstream fail fast with a safe 502-class error. If identity,
  tenant authorization, or secret retrieval cannot be validated, fail closed.
- Avoid introducing a cache, queue, or broker for MVP unless a later
  requirement demonstrates a need; any cache of tenant/credential data would
  require explicit isolation, expiry, and invalidation design.

## Security and privacy patterns

- Enforce TLS 1.2+ for external and data-store traffic; encrypt PostgreSQL,
  backups, and managed secret storage at rest with managed/customer-managed
  keys selected with the provider.
- Keep PostgreSQL and application services in private network segments where
  supported. Permit public ingress only through the public-facing load
  balancer on 80/443; require private database access and restrict egress to
  necessary upstream and provider endpoints.
- Apply least-privilege service identities, specific actions/resources,
  scoped secret access, and separate read/write privileges. Document any
  unavoidable provider wildcard permission.
- Validate request methods, paths, schemas, sizes, content types, and tenant
  context before processing. Use parameterized database operations and
  catalog-controlled destinations/endpoint allow lists to resist injection
  and server-side request forgery.
- Encode untrusted UI content on output and do not inject unsanitized
  user-controlled HTML into the DOM.
- Enforce tenant and role authorization at the API/service boundary for
  every operation. PostgreSQL row-level security may provide defense in
  depth only with safe connection-pool tenant context; it does not replace
  application authorization.
- Set CSP (at minimum `default-src 'self'`, without unsafe inline/eval
  unless justified), HSTS with one-year max age on HTTPS production
  responses, `nosniff`, frame denial, and strict referrer policy.
- Apply rate limits to public authentication, management, and consumer
  endpoints. Use OIDC-provider controls for password policy, brute-force
  protection, session lifetime, and administrative MFA; validate provider
  configuration before production. Application sessions/cookies must be
  secure, HttpOnly, SameSite, expiring, and invalidated on logout.
- Return the agreed JSON error envelope with generic safe messages. A
  0.5 MB over-limit response yields HTTP 502 and no truncated connector body.
- Use scoped disposal, transaction handling, and cleanup for outbound
  requests, database work, and temporary resources. A global exception
  boundary logs safe details and returns the agreed generic error.
- Pin dependencies and tool/action versions, scan dependencies, generate an
  SBOM for production artifacts, and restrict/audit CI/CD definition changes.
- Verify downloaded dependencies and published artifacts with trusted
  registry metadata, checksums/signatures where supported, and an artifact
  provenance chain. Restrict who can change workflow definitions and separate
  code authorship from production deployment approval where supported by the
  accepted release process.
- Obtain privacy/legal approval for GDPR-compliant usage/log retention and
  access before production. Security/application logs retain at least 90
  days; this minimum does not authorize retaining personal data contrary to
  the approved policy.

## Deployment and change safety

- Build one immutable OCI artifact and promote the same version through
  isolated environments using GitHub Actions.
- Use canary rollout with progressive traffic shift. Verify health, error
  rate, latency, and saturation before increasing traffic; halt promotion
  and return to the prior version when health regresses.
- Roll back by redeploying the previous version-pinned application and
  infrastructure artifact. Use backward-compatible schema migrations and
  document database recovery separately; application rollback does not undo
  data changes.
- Keep reviewed Terraform as infrastructure source of truth. Provider
  resources, state backend, keys, alarms, and autoscaling limits are selected
  after hosting-provider selection.
- Use a lightweight GitHub-visible change record, approval, and rollback
  note. Human operators handle incidents; operational documentation remains
  outside GitHub as requested.

## Resiliency testing scenarios

The user deferred execution of resiliency/DR tests to Operations and
identified no existing practice. Document these scenarios now; before
production, an owner must assign an external runbook/test record location,
operator, and schedule. Operations executes the approved tests.

1. **Instance/process loss:** Remove one application instance; confirm
   health-based routing, remaining capacity, and no tenant-state loss.
2. **Availability-zone loss:** Simulate loss of a zone; confirm the service
   continues in another zone without control-plane recovery action.
3. **Database failover/restore:** Exercise the provider's multi-zone database
   failover and a backup restore; validate data integrity and tenant
   isolation.
4. **Upstream outage/latency:** Use a controlled fixture to trigger timeout,
   transport errors, and circuit breaking; confirm no retries or resource
   exhaustion.
5. **Secret service outage:** Confirm proxy requests fail closed and no
   credential is logged or returned.
6. **Canary regression/rollback:** Inject a controlled health/error
   regression, halt rollout, and restore the prior application/infrastructure
   artifact with schema compatibility.
7. **Security event alerting:** Generate controlled authentication failures
   and authorization denials; confirm alerts route to human operators.
8. **Full-region failure:** Not covered by the selected single-region
   topology. A future topology change and user decision are required to test
   regional failover.

Acceptance evidence includes scenario, environment, observed result, data
integrity/tenant isolation checks, recovery duration, operator actions, and
tracked corrective actions. Exact RTO/RPO pass thresholds are set only after
owner approval before production traffic.

## Design-stage compliance status

Status evaluates whether the required design is represented in this
artifact; it is not implementation verification.

| Rule | Status | Design coverage |
|---|---|---|
| SECURITY-01 | Compliant (design) | TLS, encrypted data/backups/secrets; provider configuration gated |
| SECURITY-02 | Compliant (design) | Access logs required for each external intermediary |
| SECURITY-03 | Compliant (design) | Structured, correlated, centralized, redacted logs |
| SECURITY-04 | Compliant (design) | Restrictive CSP, HSTS, nosniff, frame and referrer policy |
| SECURITY-05 | Compliant (design) | Schema, format, length, size, allow-list, and parameterized-query validation |
| SECURITY-06 | Compliant (design) | Least privilege, scoped actions/resources, documented exceptions |
| SECURITY-07 | Compliant (design) | Deny-by-default ingress/egress, private data tier |
| SECURITY-08 | Compliant (design) | Server-side object/function authorization and tenant checks |
| SECURITY-09 | Compliant (design) | Hardened runtime, generic errors, no default credentials |
| SECURITY-10 | Compliant (design) | Pinning, scanning, SBOM, trusted images/actions |
| SECURITY-11 | Compliant (design) | Rate limiting, misuse cases, layered controls |
| SECURITY-12 | Compliant (design) | OIDC, provider MFA/brute-force controls, secure expiring sessions |
| SECURITY-13 | Compliant (design) | Safe deserialization, CI controls, critical-change audit records |
| SECURITY-14 | Compliant (design) | Security alerts, tamper-evident 90-day logs, dashboards |
| SECURITY-15 | Compliant (design) | Explicit timeouts, fail closed, cleanup, global safe error handling |
| RESILIENCY-01 | Compliant (design) | High criticality, impact, and dependency map |
| RESILIENCY-02 | Compliant (design gate) | 99.95% and exact RTO/RPO required before production; not an MVP SLA |
| RESILIENCY-03 | Compliant (design) | GitHub-visible change record, approval, rollback notes |
| RESILIENCY-04 | Compliant (design) | Immutable artifact, canary, verification, previous-version rollback |
| RESILIENCY-05 | Compliant (design) | Metrics, logs, correlation, dashboards; tracing N/A for single service |
| RESILIENCY-06 | Compliant (design) | Shallow/deep health, routing, synthetic checks |
| RESILIENCY-07 | Compliant (design gate) | Capacity/resiliency alarms and provider tool selection before production |
| RESILIENCY-08 | Compliant (design) | User-selected single-region multi-zone; no regional recovery claimed |
| RESILIENCY-09 | Compliant (design gate) | Autoscaling and bounds required after measured load/provider choice |
| RESILIENCY-10 | Compliant (design) | Timeouts, circuit breaker, bulkhead, graceful degradation, no retries |
| RESILIENCY-11 | Compliant (design gate) | In-region Warm Standby, failover/failback; exact RTO/RPO production gate |
| RESILIENCY-12 | Compliant (design gate) | Encrypted automated backups, restore tests; retention tied to GDPR/RPO |
| RESILIENCY-13 | Compliant (design gate) | Runbooks, communication, and validation required before production |
| RESILIENCY-14 | Compliant (design) | Scenarios documented; execution explicitly deferred to Operations |
| RESILIENCY-15 | Compliant (design gate) | Human-managed external incident practice; owner/runbook routing required |
