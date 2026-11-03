# Application Design

## Current MVP scope amendment (2026-10-26)

All authentication and sign-in are deferred, including OIDC management
sessions and consumer API keys. Current scope is public read-only catalog
discovery and tenant routing based on bespoke subdomains beneath a shared
configured platform base domain. A subdomain selects routing context only; it
does not authorize tenant-private data or upstream proxy access. The approved
design below describes future-state modules where they depend on authentication
or protected tenant operations; it does not authorize implementing those
features in the current MVP.

## Summary

Design the MVP as a single ASP.NET Core modular monolith with a React/TypeScript
management UI and PostgreSQL. Keep identity/access, tenancy, catalog, tenant
integration configuration, consumer API keys, proxy, and usage/logging as
logical modules with explicit application contracts and owned persistence.
Do not introduce separately deployed services.

Platform operators provision the initial tenant and admin and manage catalog
definitions through a protected operator UI. Tenant users manage their own
consumer API keys; a Tenant Admin may revoke keys within their tenant without
retrieving usable secrets. Tenant admins manage integration configuration;
view-only members have server-enforced read access.

The consumer proxy validates that tenant subdomain and user-scoped API key
resolve to the same tenant, checks the catalog allow list, and retrieves
upstream credentials through a provider-neutral secret-store interface.
Provider-specific secret storage waits for hosting selection.

## Component design

See [components.md](components.md) for responsibilities and
[component-methods.md](component-methods.md) for illustrative high-level
contracts. See [services.md](services.md) for orchestration and
[component-dependency.md](component-dependency.md) for boundaries and flows.

## Key request contracts

- Completed upstream responses return platform HTTP 200, with the upstream
  status and body under `connector`.
- Do not forward upstream response headers. Parse valid JSON; otherwise return
  text. Enforce a 0.5 MB upstream response limit.
- Platform failures use 400/401/403/502/504 and a JSON error object. Timeout
  is 30 seconds, and proxy requests are not retried.
- Usage outcome records are minimal and separate from structured
  security/application logs.

## Security and resiliency boundaries

- Derive tenant scope from validated identity or matching subdomain/key;
  authorize all operations server-side and fail closed.
- Store only one-way hashes of consumer API keys. Display usable keys only at
  creation. Keep upstream credentials in the hosting-provider secret manager
  behind a scoped application interface.
- Validate request types, sizes, and formats; enforce allow lists, rate
  limits, safe errors, security headers, and structured redacted logs.
- Keep security/application logs for at least 90 days. GDPR retention/access
  rules require approval before production.
- No availability SLA applies during MVP. Approve 99.95% monthly availability
  and Warm Standby RTO/RPO values before production customer traffic.

## Proposals requiring review

- Return a platform-side 502 JSON error rather than truncate an upstream
  response that exceeds 0.5 MB. Exact error-object fields are for design
  review.
- The requirements do not define the exact over-limit error code or payload.
- The accepted ADR packages the UI with the application; reconcile the
  separate local Nginx UI container during implementation planning.

## Scope exclusions and gates

Usage views, pricing, billing, payment collection, invoicing, custom domains,
and the GitHub connector are outside MVP scope. Exact RTO/RPO values and
GDPR-compliant retention/access policy are required before production
customer traffic.
