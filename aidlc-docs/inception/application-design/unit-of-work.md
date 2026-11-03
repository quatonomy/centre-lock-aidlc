# Unit of Work: CentreLock Application

## Current scope amendment (2026-10-26)

The current MVP excludes every form of authentication and sign-in. In scope
are public read-only catalog discovery and host-based routing using bespoke
tenant subdomains beneath a shared configured base domain. Subdomains do not
authorize private tenant data or proxy calls. Authentication-dependent
responsibilities and stories below are deferred until a new scope decision.

## Unit definition

- **Unit ID:** `centre-lock-application`
- **Unit name:** CentreLock Application
- **Deployment model:** One .NET modular monolith with its management UI and
  shared PostgreSQL database.
- **Unit count:** One. The accepted architecture and Units Generation
  definition treat the application as one deployable unit containing logical
  modules, not independently deployed services.
- **Scope:** Public catalog discovery and tenant-subdomain routing, including
  required validation, security, test, and delivery work.

## Responsibilities

- Provide public read-only discovery of platform-published catalog entries
  and OpenAPI documentation.
- Resolve tenant hostnames beneath the configured base domain to routing
  context without treating the hostname as authentication or authorization.
- Do not expose tenant-private configuration, credentials, or upstream
  proxy execution while authentication is out of scope.
- Enforce the specified response behavior, timeout, response-size limit, and
  no-retry policy.
- Persist tenant and catalog data through module-owned PostgreSQL/EF Core
  repositories and reviewed migrations.
- Record minimal request outcomes separately from structured
  security/application logs.
- Provide automated tests, GitHub Actions delivery, and Terraform
  infrastructure work while retaining provider-neutral boundaries.

## Logical modules

1. **API host and shared pipeline** - composition, route groups, validation,
   authentication/authorization middleware, safe exception mapping,
   security headers, rate limiting, request size limits, and correlation.
2. **Identity/access and tenant provisioning** - OIDC integration,
   membership/roles, operator authorization, tenant bootstrap.
3. **Tenant context** - trusted tenant resolution for management and consumer
   requests.
4. **Integration catalog** - platform definitions, allow lists, OpenAPI, and
   protected operator management.
5. **Tenant integration configuration** - tenant-selected integrations and
   local settings, with references to secrets.
6. **Secret-store adapter** - provider-neutral contract and later
   hosting-provider implementation.
7. **Consumer API keys** - one-way hashes, user scope, key lifecycle, and
   owner/admin revocation permissions.
8. **Consumer proxy** - key/subdomain agreement, allow-list enforcement,
   outbound requests, response mapping, timeout, and payload-size limits.
9. **Usage and security/application logging** - minimal outcome persistence
   and separate structured logs.
10. **Management/operator UI** - tenant and operator journeys served in the
    accepted production application shape.
11. **Persistence and delivery** - module-owned EF Core data access,
    controlled migrations, GitHub Actions and Terraform.

## Internal implementation sequence

The amended current-scope sequence is:

1. Publish and serve the non-tenant-specific catalog read-only, including
   OpenAPI documents, without exposing credentials.
2. Implement and test tenant subdomain resolution and reject unknown,
   malformed, reserved, or unconfigured hosts.
3. Keep tenant-private management and proxy routes unavailable.
4. Complete delivery and testing for these in-scope capabilities.

Defer OIDC, memberships/roles, invitations, protected catalog mutation,
tenant configuration, consumer API keys, and tenant proxy execution until
authentication and authorization are explicitly brought back into scope.

The sequence is one application delivery path. It does not create separate
service deployments or permit security controls to be deferred until the end.

## Boundaries and exclusions

- Modules own their persistence; other modules call explicit contracts.
- The application deploys as one OCI-compatible artifact. PostgreSQL is a
  shared service; tenant isolation is enforced by application/data-access
  boundaries and database controls where safe.
- The selected hosting provider and secret manager are not known. Do not
  implement provider-specific services until that decision.
- Usage views, billing, payment collection, invoicing, custom domains, and
  GitHub connector functionality are excluded from this MVP unit.
- Exact Warm Standby RTO/RPO values and GDPR-compliant retention/access rules
  remain production-readiness gates.

## Unit acceptance criteria

- All user stories are mapped to this unit in `unit-of-work-story-map.md`.
- Component dependencies follow `unit-of-work-dependency.md` and the approved
  application design.
- No logical module is treated as an independent deployable service.
- Catalog public-data boundaries and subdomain routing have automated test
  coverage. Tenant isolation and proxy authorization remain future gates.
- Security Baseline and full Property-Based Testing requirements are
  addressed in per-unit design and implementation plans.
- Production availability/recovery and GDPR gates are explicit and are not
  represented as already satisfied.
