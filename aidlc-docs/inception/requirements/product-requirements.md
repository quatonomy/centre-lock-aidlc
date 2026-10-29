# centre-lock specification

This is the detailed product-requirements record. It reflects answers in
`requirement-verification-questions.md`; unresolved follow-ups are tracked in
`requirements-clarification-questions.md`. The concise implementation summary
is `centre-lock/SPEC.md`.

## Purpose and goals

- Provide a B2B integrations platform through which customers manage and consume integrations.
- Provide a UI for a customer's team to manage the integrations available to its tenancy.
- Proxy consumer API requests to the configured upstream integrations.

## MVP scope

The MVP includes tenant sign-in and membership/roles, tenant integration
configuration, and the consumer proxy. Usage views, pricing, billing, and
payment collection are deferred. GitHub inclusion is also deferred until its
use cases and permission scopes are defined. The current implementation does
not yet include these MVP capabilities.

## Users and use cases

- **Organization and tenancy:** A tenancy represents one customer organization. The organization is the isolation boundary: a tenancy must not be able to see or access another tenancy's data or resources.
- **Tenant users:** Tenant admins invite users and assign admin or view-only user roles. Roles must be extendable. Usage views and plans are deferred beyond the MVP; later product direction allows all tenant users to view usage, while only admins view plan and charge details.
- **Platform-managed integrations:** The platform operator manages the central catalog of integrations and their endpoint allow lists. A tenant can view the integrations available to it and select from that list; it cannot create catalog entries or change allow lists.
- **Tenant integration management:** Through the UI, a tenant can add, remove, and modify its configured integrations, and view their OpenAPI documentation. Tenants may modify credentials and tenant-local display or operational settings, but not catalog definitions or endpoint allow lists.
- **API consumers:** A tenant's consumers access its configured integrations through a tenant-specific subdomain, for example `https://valuedCustomer.example.com/registeredintegration/api/health`. Tenant subdomains are in scope for the MVP; custom domains are deferred. Both subdomain and tenant-scoped API key must identify the same tenant.
- **Proxy API:** For every completed upstream response, the tenant-facing API returns platform HTTP 200. The response body contains a `connector` object holding the integration response, including the upstream status code and response body. Platform-side failure mapping remains unresolved.
- **Usage and billing:** Deferred beyond the MVP. Later product direction is to bill all requests, including unsuccessful and timed-out requests, at £0.01 per request in addition to the monthly plan. Plan changes must specify an effective date and affected tenancies. Payment collection and invoicing are not in the MVP.
- **Current MVP scope update (from source `SPEC.md`):** The current MVP excludes all authentication (management OIDC and consumer API keys) and focuses on public read-only catalog discovery and bespoke-subdomain tenant-route resolution; a subdomain is only a routing label and grants no access. Membership, roles, invitations, tenant configuration, catalog administration, consumer keys, and proxy execution are deferred. The items below describe the intended later direction.
- **Authentication and authorization:** Management UI sign-in uses OIDC. Google is required for initial personal testing; platform operators may configure additional providers later. Tenant admins invite users and assign admin or view-only user roles. The role model must be extendable.

## Pricing

Usage views, pricing, and billing are deferred beyond the MVP. These values
are future product direction, not a requirement to implement billing in the
MVP.

Monthly tenancy plans are based on the number of users:

| Plan | Users | Monthly price |
|---|---:|---:|
| Small | Fewer than 10 | Free |
| Medium | 10–25 | £15 |
| Large | 26 or more (unlimited) | £50 |

Plan thresholds and prices must be editable rather than hard-coded. Changes
must specify an effective date and affected tenancies. API calls are £0.01
per request in addition to the monthly plan price. All requests count as
billable.

## Initial integrations

- Star Wars API, seeded into the catalog in every environment as a trial
  integration.
- GitHub is a product candidate; MVP inclusion and authorization are deferred
  until its use cases and permission scopes are defined.

The exact authentication and credential setup for each connector remain integration-specific.

## Integration behavior

- The platform acts as a proxy between tenant API consumers and upstream integrations.
- Store platform-managed integration definitions, endpoint allow lists, and associated OpenAPI documentation as catalog data so approved standard HTTP integrations can be onboarded without a provider-specific code change or application release. Tenants can see which integrations are available, but cannot change the catalog or otherwise control which endpoints are permitted.
- The initial generic HTTP connector supports no upstream authentication, tenant-specific API keys sent in a catalog-configured header, and tenant-specific bearer tokens. Keep the model extensible for delegated OAuth 2.0 flows, which follow the upstream credential requirements above. Upstream secret retrieval remains tenant-scoped and credentials are never stored in the public catalog definition.
- Each integration has associated OpenAPI documentation.
- For every completed upstream response, the tenant-facing platform returns HTTP 200 and includes the upstream status code and body in its response body.
- Upstream errors are returned in the platform response body.
- Upstream requests time out after 30 seconds. The response must indicate
  that a timeout occurred; platform-side error mapping and timeout-body
  details remain to be defined.
- The response body has a top-level `connector` object containing the integration response, including its status code and body. The platform's own status code is returned as the HTTP status code.
- For local development only, expose `POST /api/integrations/proxy` with a JSON body containing the integration slug, HTTP method, path, optional query string, and optional JSON body. This unauthenticated route is available only in the Development environment (or when explicitly enabled via `Integrations:EnableProxy`), only invokes enabled catalog integrations configured with no upstream credentials, and enforces the catalog endpoint allow-list. It is not the tenant-facing proxy API.
- Seed the Star Wars API definition idempotently into the catalog in every environment. As of the current implementation, public read-only catalog-list and OpenAPI routes are available in every environment; the unauthenticated proxy remains Development-only unless `Integrations:EnableProxy` is enabled (non-production testing).
- The development proxy returns HTTP 200 for a completed upstream response and includes the upstream status and parsed JSON-or-text body under `connector`. It returns 404 for an unavailable integration, 403 for credentialed integrations or disallowed endpoints, 502 for upstream transport failures, and 504 with an explicit error for an upstream timeout.
- The platform does not retry upstream requests. Consumers are responsible for any retry workflow.

## Credential handling

### Upstream integration credentials

The platform provides a centrally managed connector for each supported third-party service; tenants do not create bespoke connectors. Each tenant authorizes or configures its own account for a connector.

- Use the provider's delegated authorization flow, such as OAuth, where available. The tenant grants access to the platform, which stores the resulting access and refresh tokens encrypted.
- If a provider does not support delegated authorization, collect the credential required by that provider, such as an API key, and store it encrypted. Do not collect a customer's third-party account password.
- Credentials are tenant-specific, accessible only to the proxy when making upstream requests, and never returned through the UI or API after setup.
- Support disconnecting an account and replacing or revoking its credentials. Follow the provider's revocation flow when available.
- The exact credential types and authorization steps are defined for each supported integration.

### Consumer API credentials

Consumers authenticate to the platform using API keys scoped to a user within a tenant. Store only a one-way hash of each key, not the usable secret; show the key to the customer only when it is created. Keys are created through the UI and must be revocable and replaceable. Keys must be rotated every 12 months; the rotation interval must be configurable so it can be changed later.

## Operations and data

- The platform does not retry upstream requests; retry behavior belongs to the customer workflow.
- Log one record per API request, recording whether it succeeded or failed. Keep logged data to a minimum and do not log credentials or API keys. GDPR-related retention, access, and data-use requirements remain to be specified.

## Constraints and decisions

 - A tenant cannot see or access another tenant's data or resources.
 - The platform operator manages the central integration catalog and endpoint allow lists, not tenants.
 - Tenant API calls are proxied to upstream integrations.
 - Tenant API subdomains and tenant-scoped API keys must identify the same tenant.
 - The platform returns HTTP 200 for completed upstream responses; the upstream status code is included in the response body.
 - Pricing tiers and thresholds must be editable.
 - Per-request API charges are additional to the monthly plan price.
 - Upstream credentials are tenant-specific secrets and are not exposed through the UI or API.
 - The platform operator provides and manages connectors for supported third-party services; each tenant configures its own account credentials for a connector.
 - Consumer API keys are tenant-scoped; only one-way hashes are stored.
 - The platform does not retry upstream requests.
 - Management UI authentication uses SSO/federated identity with OpenID Connect; Google sign-in is required for initial personal testing.
 - Initial tenant roles are admin (can edit) and user (view-only); the role model must be extendable.
 - Tenant subdomains are used for the MVP; custom domains are deferred.
 - Star Wars API is the seeded trial integration; GitHub inclusion and authorization are deferred until its use cases and scopes are defined.
 - The proxy timeout is 30 seconds and must be indicated in the response.
 - The response body contains a `connector` object with the integration response; the platform status is the HTTP status and the upstream status is in the body.
 - Future product direction bills all API requests at £0.01 each; usage and billing are deferred beyond the MVP.
 - Consumer API keys are created through the UI, scoped per user per tenant, and rotated every 12 months using a configurable interval.
 - Log one minimal success/error record per request; approve GDPR-compliant retention/access rules before production.

## Open questions

- **Integration credentials:** What authorization flow and credential types should a future GitHub connector use?
- **Catalog administration:** Which platform-operator role and UI/API workflow may create, update, disable, and publish catalog definitions?
- **Tenant credentials:** Which hosting-provider secret manager will encrypt and retrieve tenant-specific API keys and bearer tokens?
- **API contract:** What are the exact response property names and representations for upstream headers, errors, and platform errors? What HTTP status code and body represent a timeout? What response size limit applies?
- **Pricing and metering:** How should deferred usage, charges, and invoices be presented?
- **Operations:** What availability, RTO/RPO, regional topology, change-management, rollback, deployment, and incident-response targets/processes apply before production?
- **Data and logs:** What GDPR-compliant retention, access, and permitted-use rules must be approved before production?
- **Product positioning:** What is the platform's unique selling proposition compared with alternatives?
