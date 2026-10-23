# ADR 0003: Data-driven integration catalog

- **Status:** Accepted
- **Date:** 2026-10-07

## Context

The first provider-specific connector implementation hard-coded a provider and
its resource list into application code. That would make approved customer
integrations wait for a code change, review, and release. The platform needs
to onboard standard HTTP integrations by managing catalog data while retaining
the centrally managed endpoint allow-list and tenant isolation requirements.

## Decision

- Store platform-managed integration definitions, endpoint method/path
  allow-lists, and OpenAPI documents in PostgreSQL.
- Use a generic HTTP proxy that resolves an enabled catalog definition at
  runtime. It forwards only methods and paths matched by the definition and
  does not allow callers to select an upstream host.
- Support no authentication, tenant-specific API keys sent in a configured
  header, and tenant-specific bearer tokens in the initial generic
  authentication model. Keep credential retrieval behind an interface; do not
  store credentials in the catalog definition.
- Keep catalog definitions and endpoint allow-lists platform-managed.
  Standard HTTP integrations should be onboarded through catalog data rather
  than provider-specific source code. Add provider-specific code only when a
  protocol cannot be safely represented by the generic connector.
- Do not expose catalog-management or proxy endpoints until platform-admin
  authorization, tenant resolution, consumer API-key validation, and tenant
  credential storage are implemented. These security boundaries are not
  implied by the data-driven catalog decision.

## Consequences

- Adding a catalog definition does not require provider-specific connector
  code, but the authenticated catalog-management workflow must be completed
  before operators can manage definitions through the application.
- Database schema changes still require reviewed EF Core migrations; ordinary
  catalog changes do not.
- OAuth 2.0 provider flows must follow the delegated-authorization
  requirements and are not part of the initial common-auth schemes. Tenant
  credential storage/encryption, catalog-admin authorization, and the public
  proxy response contract remain separate implementation work.
