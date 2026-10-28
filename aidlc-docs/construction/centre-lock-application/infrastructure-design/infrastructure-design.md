# Infrastructure Design: CentreLock Application

## Status and scope

**Scope amendment (2026-10-26):** Authentication and sign-in, including
OIDC and consumer API keys, are deferred. The current product direction
accepts tenant-specific subdomains beneath one configured platform base
domain; custom domains are deferred. Each subdomain is a routing identifier
only and does not grant data access or proxy authorization. Wildcard DNS/TLS,
reserved names, base-domain selection, and provider-specific edge routing
remain design/selection tasks.

This is the approved-plan implementation design for one CentreLock
modular-monolith deployment. Google Cloud is the selected hosting provider;
Cloud Run and Cloud SQL for PostgreSQL in `europe-west2` are the initial
targets. This design maps the accepted application, security, and resiliency
requirements to infrastructure responsibilities; it does not provision
resources, define Terraform code, or claim that any control is implemented.

Use Google Cloud managed services and a managed container runtime without
Kubernetes. Exact instance sizing, DNS zone, Terraform backend, and service
configuration remain to be selected based on cost and requirements.

## Design decisions and constraints

- Build and promote one immutable OCI application image containing the
  ASP.NET Core modular monolith and the production-shaped React UI.
- Use a provider-managed container service and managed PostgreSQL. Keep
  application instances stateless; persist tenant/catalog state in PostgreSQL
  and retrieve upstream credentials from a managed secret store.
- Use Terraform as infrastructure source of truth and GitHub Actions for
  build, validation, deployment, and promotion.
- Run production in one region across multiple availability zones. The
  provider design must offer zone-tolerant routing and database availability,
  plus in-region reduced-capacity Warm Standby for instance/zone failures.
  Full-region failure and cross-region recovery are out of scope.
- Keep public ingress at a TLS-enabled load-balancing edge; keep application
  and database endpoints private where the chosen runtime supports it.
- Use provider-managed observability, conditionally: select concrete services
  only after choosing the host. Keep application logging and metrics portable
  through standard interfaces/formats where practical.
- Preserve the approved no-Kubernetes-by-default architecture. A provider
  that cannot meet the required availability, security, and delivery controls
  with managed containers is not a suitable match without a new decision.
- Do not add a queue, broker, cache, or separately deployed service for MVP.

## Logical infrastructure mapping

| Responsibility | Design mapping | Selection status / constraints |
|---|---|---|
| Public DNS and TLS | Bespoke tenant subdomains under a shared platform base domain resolve to a managed TLS edge/load balancer; evaluate wildcard DNS and certificate coverage | Tenant-subdomain strategy is accepted; base domain, DNS zone, certificate service, and provider-specific routing remain undecided |
| Ingress protection | Edge/load balancer terminates TLS, routes only required public HTTP(S) traffic, records access logs, and supports health-based routing and rate controls | Exact load balancer, WAF, and rate-limit products are provider-dependent |
| Application compute | Provider-managed OCI container service without Kubernetes; serve built UI and API as one deployable unit; run stateless instances across zones | Concrete runtime, sizing, platform identity, and autoscaling mechanism deferred |
| Warm Standby | Keep reduced-capacity application capacity available in-region; service routing and the selected runtime must handle instance/zone loss without manual control-plane recovery | Placement and failover mechanics must be proven during provider evaluation; do not claim regional DR |
| Database | Cloud SQL for PostgreSQL; initial non-production design uses separate databases for PR previews and staging on one instance, accessed through an explicitly requested 60-minute lease that automatically stops the instance at expiry unless renewed | Region `europe-west2`; instance sizing, network configuration, lease enforcement, backup retention, and exact RTO/RPO deferred. Stopping removes instance-compute charges but not storage/backup charges, and makes all databases on that instance unavailable |
| Secret storage | Managed secret manager; application runtime identity retrieves only the integration secrets it needs; database stores opaque references, not credential values | Select with host; tenant-level access isolation and audit capabilities are provider-selection criteria |
| Application networking | Private application/database paths where supported; public ingress only at the edge; controlled egress for identity, secret service, and catalog-approved upstreams | Network constructs, DNS resolver, egress gateway, and provider endpoints are provider-dependent |
| Observability | Selected provider's managed metrics, logs, alerting, dashboards, and synthetic checks; export structured, redacted application telemetry | Concrete services and retention configuration deferred until provider and GDPR policy decisions |
| Container registry | Private registry for immutable, version-pinned images; restrict publishing and deployment identities | Provider or organizational registry selection deferred |
| CI/CD identity | GitHub Actions uses Workload Identity Federation and a dedicated non-production deployer service account, restricted to the CentreLock `main` branch; runtime uses a separate service account | Terraform bootstrap is prepared but not applied; PR deployment identity is deferred until its isolation and lease controls are implemented |
| Terraform state | GCS remote backend in `europe-west2`, with uniform bucket access, public access prevention, object versioning, and deletion protection | Terraform for the bucket is prepared but not applied. Bootstrap its state locally once, migrate it to GCS, then use distinct prefixes for state and non-production infrastructure. Production state remains a separate access boundary |
| DNS and identity integration | Resolve configured tenant subdomains at the edge/application boundary; keep future identity-provider integration independent of the hosting provider | Base domain, wildcard DNS/TLS, reserved hostnames, provider-specific routing, and any future identity configuration remain environment-specific |
| Incident operations | Human-managed response, alert routing, recovery runbooks, and evidence in an owner-designated location outside GitHub | Owner, runbook location, contact route, and drill schedule required before production |

### Google Cloud service selection

Select Google Cloud services against the complete workload, not compute alone:

1. Managed PostgreSQL and application runtime support multi-zone operation and
   in-region reduced-capacity standby without manual recovery actions.
2. Private networking, restricted egress, TLS, encrypted storage/backups,
   scoped runtime identity, and auditable managed secrets are available.
3. Access logs, tamper-evident centralized security/application logs with at
   least 90-day retention, metrics, alerts, dashboards, and synthetic checks
   can meet the approved GDPR access/retention policy.
4. GitHub Actions can use short-lived federated identity, and Terraform can
   use encrypted, locked remote state with environment isolation.
5. Health-gated canary deployment, immutable image promotion, rollback, and
   resource bounds are practical on the managed container runtime.
6. Cost, quotas, regional availability, and operational burden are documented
   for the expected workload before provisioning and before production use.

## Production topology and data flow

```text
Users / consumer clients
        |
 Public DNS + TLS
        |
 Managed edge / load balancer -- access logs, health routing, rate controls
        |
 Private managed-container service (one application image)
   |            |              |
   |            |              +--> Approved upstream endpoints (HTTPS)
   |            +--> Managed secret manager (runtime identity)
   +--> Private managed PostgreSQL (multi-zone, encrypted backups)
        |
        +--> Provider-managed observability and alerting

GitHub Actions -- short-lived federated identity --> Terraform/provider APIs
      |                                          --> private image registry
      +--> build, test, scan, publish, and promote the same image
```

- Tenant subdomain resolution selects routing context only. It is not
  authorization and must not expose tenant-private resources or permit proxy
  execution while authentication remains out of scope. If proxy auth returns
  to scope, the subdomain and tenant-scoped key must identify the same tenant.
- The proxy retrieves credentials only at execution time and sends traffic
  only to catalog-allowed destinations. Network egress controls supplement,
  but do not replace, application allow-list and SSRF protections.
- Upstream response headers are not forwarded. Enforce the 30-second timeout,
  no-retry rule, and 0.5 MB response limit in the application.
- Keep minimal request outcomes separate from security/application logs.
  Never log API keys, OIDC tokens, upstream credentials, or unnecessary
  personal information.
- Usage and audit data remain tenant-scoped in PostgreSQL; central security
  and application logs are exported to a sink that application identities
  cannot alter or delete.

## Availability, backups, and recovery

- Distribute the production application and traffic routing across at least
  two availability zones. Keep reduced-capacity in-region standby available
  for instance/zone failures; the provider design must explain capacity,
  health detection, routing, and restoration behavior.
- Configure managed PostgreSQL for multi-zone availability and automatic
  failover where supported. Validate tenant data consistency and application
  reconnection behavior after failover.
- Use encrypted automated backups. Choose retention only after exact RTO/RPO
  and the GDPR-compliant data-retention/access policy are approved.
- Document and test database restore, zone failover/failback, application
  recovery, and post-recovery validation before production. Operations owns
  test execution and retains evidence outside GitHub.
- No second region, regional replica, or full-region recovery is designed.
- The 99.95% monthly availability target, exact RTO/RPO, backup retention,
  numeric scale bounds, and alert thresholds remain production gates.

## Security and privacy controls

- Permit public network access only through the TLS edge on required ports.
  Keep application and database endpoints private where supported; do not
  expose PostgreSQL publicly.
- Enforce TLS 1.2+ externally and for service/data connections; encrypt
  database storage, backups, registry artifacts, state, and managed secrets.
- Use separate least-privilege identities for application runtime, CI build,
  Terraform plan, infrastructure apply, and production deployment. Prefer
  short-lived federation; production apply/deploy require protected approval.
- Runtime access to secrets is scoped to required secret identifiers and
  actions. Never pass secret values through Terraform outputs, source control,
  build logs, container images, or application logs. Provision secret
  containers/references separately from populating secret values.
- Restrict egress to required identity, provider, and approved integration
  destinations where the provider permits. Application catalog allow lists,
  endpoint normalization, input validation, and SSRF defenses remain
  mandatory.
- Centralize redacted structured logs, access logs, security events, and
  metrics. Prevent application roles from modifying/deleting centralized
  security logs; retain them for at least 90 days subject to GDPR approval.
- Keep production data and secrets out of PR environments. Avoid production
  data in lower environments; use generated or sanitized test data.
- Apply managed identity/MFA controls to operator access and audit privileged
  infrastructure, catalog, tenant, and key lifecycle actions.

## Observability and capacity

Collect the signals defined by the NFR design, including request count,
platform and upstream latency separately, status/error/timeout rates,
response-size rejections, authentication/authorization failures, application
resource saturation, database connectivity/pool saturation, secret-service
health, autoscaling/quota state, backup health, and zone redundancy.

Define dashboards and alert thresholds after measured load and production
targets are approved. Use liveness and database-aware readiness checks;
upstream health must not make the whole application unready. Add safe
synthetic sign-in and proxy journeys where the selected provider supports
credential handling for test tenants. Route alerts to human operators using
the external incident process.

Application compute is horizontally scalable and bounded by explicit
minimum/maximum capacity. Set database connection-pool and outbound
concurrency limits so scale-out cannot overwhelm PostgreSQL or upstreams.
Select trigger signals, thresholds, quotas, and cost controls using a
representative load test; no numeric MVP capacity or latency commitment is
implied.

## Terraform and infrastructure lifecycle

- Organize Terraform by reusable provider-neutral conventions and
  environment-specific configuration. Keep provider modules behind
  environment boundaries; do not invent a provider module before selection.
- Use encrypted, locked remote state with restricted read/write access and
  audit. Isolate state and apply permissions by environment, especially
  production. Do not commit state, plan files containing sensitive data, or
  credentials.
- CI validates formatting, configuration, and plans for review. Pull
  requests do not apply infrastructure. Protected environment approval is
  required for staging/production applies and production changes.
- Provision secret metadata and access policy through reviewed infrastructure
  where appropriate, but populate secret values through an approved secure
  process outside Terraform state. Never output secret values.
- Pin Terraform/provider versions and container base images; review changes,
  scan dependencies/images, and publish an SBOM for release artifacts.
- Use narrowly scoped federated GitHub Actions identities and separate
  Terraform read/plan from privileged apply roles.

## Production-readiness gates

Before customer production traffic, all of the following remain required:

1. Select and document a hosting provider after hosting/cost comparison;
   confirm managed container, PostgreSQL, networking, secret, DNS/TLS, and
   observability services meet the selection criteria above.
2. Approve 99.95% monthly availability, exact RTO/RPO, backup retention, and
   GDPR-compliant retention/access policy.
3. Measure expected traffic, run representative load tests, and approve
   numeric latency targets, capacity bounds, scaling policy, quotas, and
   alert thresholds.
4. Validate security baseline controls, identity federation, least
   privilege, egress restrictions, logging/retention, and secret handling.
5. Exercise multi-zone failover, backup restore, warm-standby recovery,
   canary rollback, and operational response; record acceptance evidence in
   the Operations-owned location.
6. Complete change, rollback, failback, and incident runbooks and identify
   their owners and alert escalation route.

These are design gates, not evidence of implemented or production-ready
controls.
