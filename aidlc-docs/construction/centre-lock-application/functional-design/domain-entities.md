# Domain Entities: CentreLock Application

## Current scope amendment (2026-10-26)

Authentication and sign-in, memberships, consumer API keys, tenant-private
configuration, and proxy requests are deferred. Tenant slugs/subdomains in
this future-state model may identify routing context only in the current
MVP; they do not authorize access.

The entities below are conceptual. Persistence mappings, primary-key types,
indexes, concurrency tokens, and provider-specific storage are settled during
NFR/Infrastructure Design and implementation.

## Tenant

Represents one customer organization.

| Attribute | Meaning |
|---|---|
| TenantId | Stable identity |
| Name/Slug | Display name and unique routing identifier |
| Status | Provisioning/active/suspended lifecycle state |
| CreatedAt | Creation timestamp |

**Relationships:** Owns memberships, invitations, tenant integration
configurations, API keys, and usage records. Tenant status is checked before
management and consumer operations.

## User and Membership

**User** represents an authenticated OIDC identity. Keep external subject
identity distinct from mutable display/email attributes.

**Membership** joins one User to one Tenant and assigns an extensible role.
Initial role values are Admin and Member (view-only).

Relationship: `Tenant 1..* Membership *..1 User`. Enforce one effective
membership per user/tenant unless a future role model explicitly allows
otherwise. A membership is the authorization basis for tenant operations.

## Invitation

| Attribute | Meaning |
|---|---|
| InvitationId | Stable identity |
| TenantId | Owning tenant |
| NormalizedEmail | Invitee identity constraint |
| AssignedRole | Admin or Member |
| TokenHash | One-way token verifier; never store a usable link token |
| IssuedAt / ExpiresAt | 7-day validity window |
| Status | Pending, Consumed, Revoked, or Expired |
| ConsumedAt / RevokedAt | Optional lifecycle timestamps |

Resend replaces the token hash and resets the expiry to 7 days from resend;
tenant and role remain unchanged. Redemption requires a verified OIDC email
matching `NormalizedEmail` and atomically transitions Pending to Consumed
while creating Membership.

## Catalog Integration and Allowed Endpoint

**Catalog Integration** is platform-owned:

- `IntegrationId`, stable slug, display metadata, upstream base address,
  OpenAPI document/reference, publication status, and catalog version.
- Authentication scheme/configuration metadata may describe how the
  connector uses secrets, but contains no tenant credential values.

**Allowed Endpoint** belongs to a Catalog Integration and describes an
allowed HTTP method and path template. Runtime matching uses normalized
method/path semantics defined and tested in the connector design.

Only published integrations and their active endpoint entries are available
to tenant configuration and proxying. Tenant users cannot mutate either
entity.

## Tenant Integration Configuration

Represents a tenant's configuration of one catalog integration.

| Attribute | Meaning |
|---|---|
| TenantIntegrationId | Stable identity |
| TenantId | Owning tenant |
| IntegrationId | Selected catalog integration |
| LocalSettings | Validated tenant-specific display/operational settings |
| SecretReference | Optional opaque reference; not credential material |
| CreatedAt / UpdatedAt | Lifecycle timestamps |

Enforce uniqueness on `(TenantId, IntegrationId)` for the MVP. Removal
disables future proxy use and coordinates cleanup of its secret reference.
The tenant cannot change catalog-owned fields or endpoint allow lists.

## Consumer API Key

| Attribute | Meaning |
|---|---|
| ApiKeyId | Stable non-secret identity |
| TenantId / OwnerUserId | Scope and owner |
| KeyHash | One-way hash of high-entropy key |
| CreatedAt / ExpiresAt | Lifecycle metadata; expiry follows configured rotation policy |
| RevokedAt | Optional revocation timestamp |
| LastUsedAt | Optional non-secret operational metadata |

Multiple active keys per user are allowed. Owner may create, replace, and
revoke own keys. A Tenant Admin may revoke keys in the same tenant but cannot
retrieve raw key material.

## Proxy Request and Result

These may be transient value objects rather than persisted domain entities.

**ProxyRequest** includes validated tenant subdomain, presented key (secret
handling boundary only), integration slug/identity, HTTP method, normalized
path, query, and bounded request body.

**UpstreamResult** includes upstream status, parsed JSON-or-text body,
completion/transport/timeout outcome, and measured response size. Do not
forward upstream response headers.

**ProxyResult** distinguishes completed upstream result (platform HTTP 200,
`connector` payload) from platform failure (400/401/403/502/504 with JSON
error envelope).

No raw API key or credential may be retained in these value objects beyond
the minimal execution scope needed to validate/forward the request.

## Usage Outcome and Security Event

**UsageOutcome** records one minimal per-request success/failure result with
timestamp and correlation ID. It should avoid unnecessary user-identifying
data and contain no credentials or key values.

**SecurityEvent** and **ApplicationLogEvent** are separate records/sinks from
usage outcomes. Include structured timestamp, correlation/request ID,
severity/event code, and safe context. Security/application logs have a
minimum 90-day retention target; access and wider retention policies await
the pre-production GDPR review.

## Relationship summary

```text
Tenant 1 --- * Membership * --- 1 User
Tenant 1 --- * Invitation
Tenant 1 --- * TenantIntegrationConfiguration * --- 1 CatalogIntegration
CatalogIntegration 1 --- * AllowedEndpoint
Tenant 1 --- * ConsumerApiKey * --- 1 User
Tenant 1 --- * UsageOutcome
TenantIntegrationConfiguration --- 0..1 SecretReference
```

Enforce tenant ownership on every relationship lookup and mutation; do not
rely solely on globally unique identifiers to provide isolation.
