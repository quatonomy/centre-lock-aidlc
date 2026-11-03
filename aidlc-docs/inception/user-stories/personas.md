# MVP Personas

**Scope amendment (2026-10-26):** The only current public-facing personas
are Visitors browsing the public catalog and visitors using tenant subdomains
for routing. Tenant Admin, Tenant Member, Tenant API Consumer, and Platform
Operator descriptions below are future-state personas because authentication
and sign-in are deferred.

## Visitor

- **Role:** An unauthenticated person browsing public platform information.
- **Goals:** Discover published integrations and documentation; use a
  tenant-specific hostname only to reach the appropriate public routing
  context.
- **Constraints:** A hostname does not authenticate the visitor or grant
  access to tenant-private data or upstream integrations.

## Tenant Admin

- **Role:** An authorized administrator for one customer tenancy.
- **Goals:** Sign in, invite tenant users, assign admin or view-only roles,
  select available integrations, and manage tenant-local configuration.
- **Needs:** Clear tenant scope, secure credential setup, and confidence that
  tenant settings and users cannot affect another organization.
- **Constraints:** Cannot create or edit platform catalog definitions or
  endpoint allow lists.

## Tenant Member

- **Role:** A tenant user with view-only permissions.
- **Goals:** Sign in and view the integrations and documentation available to
  their tenancy.
- **Needs:** Access to permitted tenant information without edit or
  administrative capabilities.
- **Constraints:** Cannot invite users, change roles, or modify tenant
  integration settings.

## Tenant API Consumer

- **Role:** A consumer application or developer making API requests for a
  tenant.
- **Goals:** Call a configured integration through the tenant-specific
  centre-lock endpoint.
- **Needs:** Tenant-scoped API credentials, predictable response behavior,
  and errors that distinguish platform failures from completed upstream
  responses.
- **Constraints (future state):** The tenant subdomain and API key must
  identify the same tenant; only catalog-allowed endpoints may be called.

## Platform Operator

- **Role:** A trusted operator responsible for the platform-managed
  integration catalog and operational controls.
- **Goals:** Maintain approved integrations, endpoint allow lists, and their
  documentation.
- **Needs:** Platform-level controls distinct from tenant administration and
  tenant data.
- **Constraints:** The exact catalog administration workflow and operator
  role model remain for design; tenant users must not gain catalog write
  access.

## Project Owner

- **Role:** Product and delivery decision-maker for the MVP.
- **Goals:** Review product scope, approve design and delivery decisions, and
  ensure production readiness gates are satisfied.
- **Needs:** Traceable requirements, testable stories, and explicit
  distinction between MVP scope and deferred decisions.
- **Constraints:** Production customer traffic must not begin before the
  availability, recovery, and GDPR retention/access gates are addressed.
