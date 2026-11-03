# NFR Logical Components: CentreLock Application

## Current scope amendment (2026-10-26)

Authentication, identity-provider integration, tenant authorization, and the
consumer proxy are deferred from the current MVP. Their entries below are
future-state design. Security baseline requirements still apply to public
catalog and host-routing surfaces; a tenant subdomain is not an access
control.

All components below are logical responsibilities within one deployable
modular monolith unless identified as external managed infrastructure.
Concrete products and settings are deferred until the hosting provider is
selected.

## Component inventory

| Component | Responsibility | Placement/boundary |
|---|---|---|
| Secure ingress and load balancer | TLS termination, public routing, access logs, health-based multi-zone traffic distribution, WAF/rate controls where supported | Provider-managed edge; only public 80/443 ingress |
| API request pipeline | Correlation ID, validation, payload limits, authentication/authorization, rate limiting, security headers, global safe error handler | Application host |
| Identity adapter | OIDC token/session validation and provider-mediated MFA/brute-force controls | Application boundary plus external identity provider |
| Tenant authorization context | Resolve trusted tenant and enforce role/object scope before data access | Application domain/service boundary |
| Tenant-safe persistence adapters | Parameterized EF Core access, tenant scoping, optional safe PostgreSQL RLS defense in depth | Application modules and private PostgreSQL |
| Secret-store adapter | Tenant/integration scoped credential access; encryption; audit; no secret in application logs | Application contract plus provider-managed secret manager |
| Consumer proxy and connector | Allow-list matching, bounded outbound requests, 30-second timeout, circuit breaker/bulkhead, no retry, bounded response parsing | Application module |
| Usage outcome recorder | Persist one minimal request success/failure record | Application module and separate usage data |
| Audit event recorder | Record actor/time/action/resource and safe before/after values for critical administrative changes | Application module; secrets excluded |
| Central log/metric/trace exporter | Export structured redacted logs, metrics, and correlation; maintain tamper-evident centralized logs | Application integration and managed observability service |
| Health and synthetic monitoring | Liveness/readiness, database readiness, public journey probes, routing signals | Application endpoints plus provider monitoring |
| Capacity controller | Scale application instances within configured minimum/maximum; monitor saturation and quotas | Provider runtime/autoscaling plus application metrics |
| PostgreSQL backup/recovery | Multi-zone data availability, encrypted automated backups, restore validation | Provider-managed database |
| Deployment controller | Immutable artifact, canary progression, health gates, rollback to previous version | GitHub Actions plus provider deployment service |
| Terraform modules/state | Provision network, compute, database, monitoring, IAM, backup, and secrets integration | Terraform/provider boundary |
| External incident record | Human response, communication, runbook, and corrective actions outside GitHub | Owner-designated operations system/location; name pending |

## Trust and data flow

```text
Internet
  -> TLS/load balancer/access logs/rate controls
  -> API pipeline (correlation, limits, authentication, authorization)
      -> tenant-scoped application modules
          -> private PostgreSQL
          -> secret-store adapter -> provider secret manager
          -> consumer proxy -> catalog-allowed upstream
      -> usage recorder (minimal request outcomes)
      -> audit recorder (safe critical-change history)
      -> central logs/metrics/alerts
```

The application never gives browser clients direct access to PostgreSQL,
secret storage, or provider control planes. Tenant credentials are available
only to the proxy execution boundary. The public consumer endpoint does not
forward upstream response headers.

## Availability and recovery relationships

- The load balancer distributes traffic across at least two zones.
- Application instances are stateless and scale within configured bounds.
- PostgreSQL is configured for multi-zone availability and encrypted backup.
- In-region Warm Standby covers instance/zone failure at reduced capacity.
- Full-region failure is not covered. Do not configure a secondary region
  without a new owner decision.
- Backup retention, exact RTO/RPO, provider-level failover mechanics, and
  production scale bounds are owner/provider gates.

## Monitoring and alert signals

### Service and API

- Request count, latency (platform and upstream separately), status class,
  validation/auth denials, timeout and over-limit counts.
- Availability and synthetic sign-in/proxy checks.
- Canary health and release regression signals.

### Dependencies and saturation

- PostgreSQL connectivity, query/pool saturation, storage health, and
  failover/replication status available from provider.
- Secret-manager availability and latency.
- Upstream circuit-breaker state, timeout/error rates, and outbound
  concurrency.
- CPU, memory, request concurrency, scaling bounds, and service quota
  utilization.
- Backup completion, backup age, restore-test status, and multi-zone
  degradation.

### Security and audit

- Repeated authentication failures, access denials, privilege changes,
  catalog changes, tenant provisioning, key lifecycle actions, and sensitive
  configuration changes.
- Log tamper/retention controls and alerts for backup, encryption, or
  access-log configuration drift.
- Alert events route to human operators using the operational contact path
  established before production.

## Deployment and recovery controls

- GitHub Actions builds/tests/scans and publishes a pinned immutable image
  with an SBOM.
- Terraform provisions isolated environments and least-privilege identities;
  production resources use supported, pinned images and tools.
- Canary promotion requires passing health and regression thresholds;
  automatic rollback returns the prior pinned artifact/infrastructure.
- Schema changes are backward-compatible across the rollback window and
  applied as controlled EF Core migrations.
- Failover and restore scenarios have documented acceptance criteria now;
  execution is deferred to Operations, per owner decision.

## Provider decision gates

Before production, select a host and verify it supports:

- Multi-zone load balancing, app runtime, and PostgreSQL.
- In-region reduced-capacity standby and controlled failover.
- Managed encrypted backups and restore validation.
- Private networking, scoped IAM, secret manager, centralized logs/metrics,
  tamper-resistant retention, alerts, and quotas.
- Autoscaling bounds, canary/health-gated rollout, and previous-version
  rollback.
- Terraform state security and GitHub Actions identity federation.
