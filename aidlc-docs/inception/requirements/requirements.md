# Requirements Analysis

## Intent analysis

- **User intent:** Establish the product and operational requirements for the
  centre-lock MVP using the completed verification and clarification answers.
- **Request type:** Brownfield product definition.
- **Scope:** MVP requirements and production-readiness gates; not implementation
  planning or code changes.
- **Complexity:** Moderate. The platform handles multi-tenant identity,
  credentials, and third-party API traffic.

The detailed product baseline, including future product direction, is in
[`product-requirements.md`](product-requirements.md). The answered
clarification record is in
[`requirements-clarification-questions.md`](requirements-clarification-questions.md).

## MVP scope

**Scope amendment (2026-10-26):** Authentication and sign-in of every kind,
including management OIDC sessions and consumer API keys, are deferred
entirely. Tenant membership, roles, invitations, tenant-specific integration
configuration, protected catalog administration, consumer key lifecycle, and
authenticated proxy calls are not part of the current MVP. The in-scope work
is public platform-catalog discovery and tenant routing by a bespoke
subdomain beneath the platform's configured base domain. A subdomain
identifies a tenant for routing only; it is not proof of identity,
authorization, or data access. Do not expose tenant-specific or sensitive
data, or make upstream proxy calls, without a separately approved
authentication scope.

Usage views, pricing, billing, payment collection, and invoicing remain
deferred. The GitHub connector remains deferred until its use cases and
permission scopes are defined. This amendment supersedes earlier MVP scope
statements that included sign-in, membership/roles, consumer keys, or an
authenticated tenant proxy.

## Functional requirements

- **FR-01 Tenant access (deferred):** OIDC management sign-in and
  tenant-membership authorization were part of the former baseline. All
  authentication and sign-in are out of scope for the current MVP.
- **FR-02 Membership (deferred):** Tenant admins invite users and assign
  admin or view-only roles. Invitations expire after 7 days, can be revoked
  or resent, and require a verified matching identity at redemption. No
  membership, role, invitation, or sign-in behavior is in the current MVP.
- **FR-03 Integration catalog:** Platform operators manage available
  integrations, endpoint allow lists, and OpenAPI documentation. **MVP:**
  expose only non-tenant-specific published catalog entries and their
  documentation for public read access. Catalog mutation, tenant selection,
  and tenant configuration are deferred until authorized management access
  is in scope.
- **FR-04 Tenant configuration (deferred):** Support tenant-specific
  upstream credentials and settings only after authentication and
  authorization are approved. Never expose credentials; select the
  hosting-provider secret manager with the hosting provider.
- **FR-05 Consumer tenant resolution:** The future consumer API requires a
  tenant subdomain and tenant-scoped API key to identify the same tenant.
  Consumer API-key authentication and proxy access are deferred entirely
  from the current MVP.
- **FR-06 Proxy completed responses (deferred):** Return platform HTTP 200
  for every completed upstream response, including upstream 4xx/5xx. Include
  the upstream status and response body under a top-level `connector` object.
- **FR-07 Proxy failures (deferred):** Return 400 for invalid requests, 401
  for missing or invalid API keys, 403 for unauthorized requests, 502 for
  upstream transport failures, and 504 for timeouts. Return a JSON platform
  error object.
- **FR-08 Upstream response handling (deferred):** Do not forward upstream
  response headers. Parse valid JSON bodies as JSON and otherwise return
  text. Limit upstream response bodies to 0.5 MB.
- **FR-09 Proxy limits (deferred):** Set an upstream timeout of 30 seconds,
  indicate timeouts in the response, and do not retry upstream requests.
- **FR-10 Consumer keys (deferred):** Create tenant-scoped API keys through
  the UI, scoped to a user within a tenant. Users manage their own keys;
  tenant admins may revoke keys within their tenant. Show the usable key only
  at creation, store only a one-way hash, and allow multiple active keys per
  user to support overlap during replacement. Rotate keys every 12 months
  using a configurable interval.
- **FR-11 Initial integration:** Seed the Star Wars API as a trial integration
  in every environment. GitHub authorization and MVP inclusion remain
  deferred.
- **FR-12 Tenant subdomain routing (MVP):** Resolve a tenant routing context
  from its bespoke subdomain beneath the configured platform base domain.
  This is host routing only; it does not authenticate callers or authorize
  tenant data access or upstream proxy calls. Reject unknown, malformed,
  reserved, or unconfigured subdomains. Custom domains are deferred.

## Non-functional requirements

- **NFR-01 Tenant isolation:** A tenant must not see or access another
  tenant's data or resources. Tenant subdomains are routing labels, not
  authorization; do not serve tenant-private resources until authorization
  is approved and implemented.
- **NFR-02 Security baseline:** Enforce the enabled Security Baseline as a
  blocking cross-cutting constraint. Do not log credentials or API keys.
- **NFR-03 Privacy and logs:** Keep minimal request usage records separate
  from security/application logs. Retain security/application logs for at
  least 90 days. Approve GDPR-compliant retention and access rules before
  production.
- **NFR-04 MVP availability:** No availability SLA applies during MVP; the
  service may be turned off when idle.
- **NFR-05 Production reliability gate:** Before production customer traffic,
  approve a 99.95% monthly availability target and exact Warm Standby RTO/RPO
  values. Production topology is single-region, multi-zone; exact recovery
  values are deferred during MVP.
- **NFR-06 Change and deployment:** Use canary deployment with progressive
  traffic shift. On failure, redeploy the previous version-pinned
  infrastructure and application artifact. Maintain a lightweight GitHub-
  visible change record, approval, and rollback note.
- **NFR-07 Incident response:** Production incident response is human-managed
  for now and documented outside GitHub.
- **NFR-08 Testing:** Enforce the Property-Based Testing extension in full,
  alongside example-based tests.
- **NFR-09 Resiliency:** Apply the Resiliency Baseline as directional,
  design-time guidance; it is not a production-readiness certification.

## Constraints and decisions

- Tenant isolation and GDPR compliance are mandatory constraints.
- GitHub Actions and Terraform remain the delivery standards.
- Usage, billing, and pricing are deferred; future product direction is
  £0.01 per request plus the monthly plan.
- Requirements do not infer scope from open GitHub issues.
- Tenant-specific subdomains beneath a shared configured platform base domain
  are in scope. Custom domains are deferred. DNS, TLS/wildcard certificate,
  reserved-hostname, and provider-specific routing details remain deployment
  decisions; they do not change the application rule that a subdomain only
  selects a tenant-routing context.

## Deferred decisions and production gates

- **Tenant subdomain provisioning:** How tenant subdomain names are allocated,
  reserved, and mapped to tenant records while authenticated management and
  operator provisioning are deferred. Define a safe interim provisioning
  mechanism before onboarding tenants.
- **Platform domain and routing:** Select the shared base domain, wildcard
  DNS/TLS approach, reserved hostnames, and provider-specific edge routing.
- Before production customer traffic: approve exact RTO/RPO values and the
  99.95% monthly availability target; approve GDPR-compliant retention/access
  rules for usage and security/application logs.
- Future product/design decisions: GitHub use cases and scopes, catalog
  administration workflow, exact platform error-object schema, and
  presentation of deferred usage and billing.
- The detailed requirements record tracks additional product questions and
  their scope in [`product-requirements.md`](product-requirements.md).
