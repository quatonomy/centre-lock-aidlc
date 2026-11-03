# Infrastructure Design Plan: CentreLock Application

## Accepted infrastructure constraints

- One deployable OCI-compatible .NET modular monolith, with the React UI built
  and served in the accepted production shape.
- PostgreSQL is the shared application database; use a managed PostgreSQL
  service for production.
- Terraform is the infrastructure source of truth; GitHub Actions is the CI
  and delivery tool.
- Docker Compose remains for local development. Kubernetes is not required
  solely for portability.
- Isolated local, test, staging, and production environments; PR environments
  should be temporary and isolated where supported.
- Production topology is single-region, multi-zone, with in-region
  reduced-capacity Warm Standby. Full-region recovery is out of scope.
- Public ingress is through TLS/load balancing; database and application
  resources use private networking where supported.
- The secret store must be selected with the hosting provider. Do not store
  or expose tenant upstream secrets in application data.
- No queue, broker, or asynchronous processing service is required by current
  MVP workflows.

## Owner decisions

### Question 1: Hosting provider
Which provider should the infrastructure design target?

A) AWS

B) Microsoft Azure

C) Google Cloud Platform

D) Keep the design provider-neutral until a hosting/cost comparison is
completed (recommended if provider selection is not yet ready)

X) Other (identify the provider or decision process after `[Answer]:` below)

[Answer]: D — Keep the design provider-neutral until a hosting/cost
comparison is completed. No provider is selected.

### Question 2: Production compute model
Which compute model should host the OCI application?

A) Provider-managed container runtime/service without Kubernetes
  (recommended; aligns with the accepted ADR)

B) Managed Kubernetes

C) Virtual machines running the OCI container

D) Defer the concrete runtime until provider selection; retain the managed
container-runtime preference

X) Other (describe the runtime and rationale after `[Answer]:` below)

[Answer]: A — Prefer a provider-managed container runtime/service without
Kubernetes. Select the concrete runtime after the hosting provider is chosen.

### Question 3: Monitoring and logging service
How should production metrics, logs, alerts, dashboards, and synthetic checks
be hosted?

A) Use the selected cloud provider's managed observability services
  (recommended for initial operations)

B) Use an existing organizational observability platform; identify it after
`[Answer]:` below

C) Keep an observability-provider-neutral interface and defer selection
until hosting/provider evaluation

X) Other (describe the service and rationale after `[Answer]:` below)

[Answer]: A — Use the selected provider's managed observability services.
Select the concrete services after the hosting provider is chosen.

## Answer analysis

- The decisions are consistent with the accepted architecture: provider
  selection is deferred, while managed container hosting without Kubernetes
  remains the compute preference.
- Observability is conditionally provider-native; no service or provider is
  selected by this answer.
- Infrastructure artifacts must distinguish provider-neutral requirements
  and preferences from provider-specific mappings, which remain deferred to a
  hosting and cost comparison.
- The plan was approved before infrastructure mapping and artifact
  generation. The generated artifacts preserve the provider-specific
  decisions as deferred gates and are now awaiting review.

## Mapping checklist

- [x] Analyze answers and resolve ambiguity.
- [x] Map app compute, managed PostgreSQL, load balancing, DNS/TLS, networking,
  secret storage, observability, backups, monitoring, and autoscaling.
- [x] Define dev/test/staging/production and PR environment isolation,
  promotion, and teardown.
- [x] Map GitHub Actions identity and permissions to Terraform provisioning
  and canary/rollback controls.
- [x] Define Terraform state, secrets, and environment separation without
  embedding credentials in source or state outputs.
- [x] Mark messaging infrastructure N/A for MVP with rationale.
- [x] Identify provider-dependent decisions and production gates.
- [x] Generate `infrastructure-design.md` and `deployment-architecture.md`.
- [x] Validate mappings against Security and Resiliency Baselines.
- [x] Present the infrastructure design for review and approval.

## Planned artifacts

- `aidlc-docs/construction/centre-lock-application/infrastructure-design/infrastructure-design.md`
- `aidlc-docs/construction/centre-lock-application/infrastructure-design/deployment-architecture.md`

Infrastructure design will not create cloud resources or Terraform code; it
maps requirements and logical components into an approved design.
