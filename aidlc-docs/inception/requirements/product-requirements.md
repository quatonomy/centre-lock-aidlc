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

**Current scope amendment (2026-10-26):** All authentication and sign-in are
deferred, including management OIDC sessions and consumer API keys. Membership,
roles, invitations, tenant configuration, protected catalog administration,
consumer-key management, and authenticated proxy calls are out of the MVP.
Current in-scope capabilities are public read-only discovery of the
platform-published integration catalog and tenant routing based on bespoke
tenant subdomains beneath a configured shared platform base domain. A
subdomain is a routing label only; it does not authenticate callers or grant
access to tenant-private data or upstream integrations. This amendment
supersedes the earlier scope answers below wherever they include
authentication-dependent functionality.

Usage views, pricing, billing, and payment collection remain deferred.
GitHub inclusion is deferred until its use cases and permission scopes are
defined.

## Users and use cases

- **Organization and tenancy:** A tenancy represents one customer organization. The organization is the isolation boundary: a tenancy must not be able to see or access another tenancy's data or resources.
- **Tenant users (deferred):** Tenant admins may later invite users and assign admin or view-only roles. Invitation expiry, redemption, and membership enforcement are future scope; no identity or sign-in mechanism is in the current MVP.
- **Platform-managed integrations:** The platform's current catalog is published/read-only to public callers and includes OpenAPI documents. Catalog edits, allow-list changes, and publication administration are deferred until an authorized operator workflow is in scope.
- **Tenant integration management (deferred):** Tenant users may later add, remove, and modify tenant-local integration settings. Do not expose tenant-specific configuration or credentials in the current unauthenticated MVP.
- **Tenant routing:** A tenant's bespoke subdomain beneath a configured platform base domain (for example, `https://valuedCustomer.example.com/`) selects the tenant-routing context. Custom domains are deferred. Subdomains do not authenticate callers or authorize access.
- **Proxy API (deferred):** Future authenticated proxy behavior returns platform HTTP 200 for completed upstream responses and maps failures as specified below. Do not make tenant-facing upstream proxy calls in the current MVP because all authentication is out of scope.
- **Usage and billing:** Deferred beyond the MVP. Later product direction is to bill all requests, including unsuccessful and timed-out requests, at £0.01 per request in addition to the monthly plan. Plan changes must specify an effective date and affected tenancies. Payment collection and invoicing are not in the MVP.
- **Authentication and authorization (deferred):** The earlier design selected management OIDC sign-in (Google initially), tenant memberships, and roles. None is in the current MVP; consumer API-key authentication is also deferred. Do not treat subdomain routing as a security boundary.

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
- Do not forward upstream response headers. Parse a valid JSON response body as JSON; otherwise return it as text. Reject or otherwise safely handle upstream response bodies larger than 0.5 MB.
- Upstream requests time out after 30 seconds. The response must indicate
  that a timeout occurred. Platform-side failures use 400 for invalid
  requests, 401 for missing or invalid API keys, 403 for unauthorized
  requests, 502 for upstream transport failures, and 504 for timeouts, with
  a JSON platform error object.
- The response body has a top-level `connector` object containing the integration response, including its status code and body. The platform's own status code is returned as the HTTP status code.
- For local development only, expose `POST /api/integrations/proxy` with a JSON body containing the integration slug, HTTP method, path, optional query string, and optional JSON body. This unauthenticated route is available only in the Development environment, only invokes enabled catalog integrations configured with no upstream credentials, and enforces the catalog endpoint allow-list. It is not the tenant-facing proxy API.
- Seed the Star Wars API definition idempotently into the catalog in every environment. This does not make catalog-read, OpenAPI, or unauthenticated proxy routes available outside Development.
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

**Deferred:** Consumers will eventually authenticate to the platform using API keys scoped to a user within a tenant. The key lifecycle, hashing, display, revocation, and rotation requirements apply only after consumer authentication is brought back into scope.

## Operations and data

- The platform does not retry upstream requests; retry behavior belongs to the customer workflow.
- Log one minimal usage record per API request, recording whether it succeeded or failed; do not log credentials or API keys. Keep these records separate from security/application logs. Retain security/application logs for at least 90 days. Approve GDPR-compliant retention and access rules before production.
- No availability SLA applies during MVP; the service may be turned off when idle. Before production customer traffic, establish a 99.95% monthly availability target.
- Production recovery direction is Warm Standby with single-region, multi-zone deployment. Exact maximum RTO/RPO values are deferred during MVP and must be approved before production customer traffic.
- Production deployments use canary rollout with progressive traffic shift. On failure, redeploy the previous version-pinned infrastructure and application artifact.
- Use a lightweight production change record, approval, and rollback-note process visible in GitHub. Human operators handle production incidents; incident-response documentation is maintained outside GitHub for now.

## Constraints and decisions

 - A tenant cannot see or access another tenant's data or resources.
 - The platform operator manages the central integration catalog and endpoint allow lists, not tenants.
 - **Future scope:** Authenticated tenant API calls are proxied to upstream integrations.
 - **Future scope:** Tenant API subdomains and tenant-scoped API keys must identify the same tenant.
 - The platform returns HTTP 200 for completed upstream responses; the upstream status code is included in the response body.
 - Pricing tiers and thresholds must be editable.
 - Per-request API charges are additional to the monthly plan price.
 - Upstream credentials are tenant-specific secrets and are not exposed through the UI or API.
 - The platform operator provides and manages connectors for supported third-party services; each tenant configures its own account credentials for a connector.
 - **Deferred:** Consumer API keys are tenant-scoped; only one-way hashes are stored.
 - The platform does not retry upstream requests.
 - **Deferred:** Management UI authentication uses SSO/federated identity with OpenID Connect; Google was the initial provider candidate.
 - **Deferred:** Initial tenant roles are admin (can edit) and user (view-only); the role model must be extendable.
 - Tenant-specific subdomains beneath a shared configured platform base domain are used for routing in the MVP; custom domains are deferred. A subdomain is not an access control.
 - Star Wars API is the seeded trial integration; GitHub inclusion and authorization are deferred until its use cases and scopes are defined.
 - The proxy timeout is 30 seconds and must be indicated in the response.
 - The response body contains a `connector` object with the integration response; the platform status is the HTTP status and the upstream status is in the body.
 - Platform-side proxy failures use 400 (invalid request), 401 (missing/invalid API key), 403 (unauthorized), 502 (upstream transport failure), and 504 (timeout), with a JSON platform error object.
 - Future product direction bills all API requests at £0.01 each; usage and billing are deferred beyond the MVP.
 - **Deferred:** Consumer API keys are created through the UI, scoped per user per tenant, and rotated every 12 months using a configurable interval.
 - Keep minimal request usage records separate from security/application logs; retain security/application logs for at least 90 days and approve GDPR-compliant retention/access rules before production.
 - No MVP availability SLA; permit the service to be turned off when idle. Approve a 99.95% monthly availability target and exact Warm Standby RTO/RPO values before production customer traffic.
 - Use canary deployment, previous-version rollback, and a GitHub-visible change record for production. Incident response is human-managed and documented outside GitHub for now.

## Open questions

- **Integration credentials:** What authorization flow and credential types should a future GitHub connector use?
- **Catalog administration:** Which platform-operator role and UI/API workflow may create, update, disable, and publish catalog definitions?
- **Tenant credentials:** Which hosting-provider secret manager will encrypt and retrieve tenant-specific API keys and bearer tokens?
- **API contract:** What are the exact response property names for the platform error object? (The status mapping and JSON error-object form are defined; the exact schema remains unspecified.)
- **Pricing and metering:** How should deferred usage, charges, and invoices be presented?
- **Operations:** Approve exact Warm Standby RTO/RPO values and the 99.95% monthly availability target before production customer traffic. Define the human-managed incident-response documentation outside GitHub.
- **Data and logs:** Approve GDPR-compliant retention and access rules for minimal usage records and security/application logs before production.
- **Product positioning:** What is the platform's unique selling proposition compared with alternatives?
