# Component Dependencies and Data Flows

## Current scope amendment (2026-10-26)

OIDC, membership, consumer API keys, tenant-private management, and proxy
execution are deferred. Current dependencies are limited to public catalog
read access and tenant host routing. The diagrams below that show
authentication or authenticated proxy flows describe future-state behavior.

## Dependency rules

- UI components call application endpoints; they do not access persistence or
  secrets directly.
- Identity and tenancy establish trusted principal/context before tenant
  data access.
- Tenant integration configuration depends on the catalog for published
  integration metadata and on the secret-store interface for credentials.
- The consumer proxy depends on tenant resolution, API-key validation,
  catalog allow-list validation, tenant integration configuration,
  secret-store interface, and outbound HTTP client.
- Catalog administration uses its owning application service and
  platform-operator authorization.
- Usage recording and security/application logging are separate contracts;
  both receive correlation context but neither receives secret values.
- PostgreSQL access belongs to each module's own persistence adapter.
  Cross-module coordination uses service contracts, not direct table access.

## Dependency matrix

| Caller | Depends on | Purpose |
|---|---|---|
| API Host | Identity, tenancy, catalog, tenant config, API keys, proxy, logging | Composition, routing, shared middleware |
| Tenant UI | Authenticated management API | Tenant membership and integration workflows |
| Operator UI | Protected operator API | Tenant bootstrap and catalog maintenance |
| Identity/Access | OIDC provider, membership repository | Authenticate principals and authorize roles |
| Tenant Provisioning | Identity/Access, tenancy repository | Create tenant and first admin |
| Catalog Service | Catalog repository, validation | Publish definitions, endpoint allow lists, OpenAPI |
| Tenant Integration Service | Catalog contract, secret-store interface, own repository | Store tenant configuration and secret references |
| API-Key Service | Tenant context, key repository | Create/validate/revoke tenant user keys |
| Proxy Service | Tenant context, API-key, catalog, tenant config, secret-store, HTTP client | Authenticate and forward allow-listed calls |
| Usage Recorder | Usage repository | Persist minimal request outcome |
| Security/Application Logger | Central logging sink | Structured security and operational events |
| Module persistence adapters | PostgreSQL via EF Core | Store only each module's owned data |
| Terraform/CI | Hosting provider, registry, GitHub Actions | Provision and deliver portable application artifact |

## Management sign-in and tenant administration

```mermaid
sequenceDiagram
    actor User
    participant UI as Management UI
    participant API as API Host
    participant IdP as OIDC Provider
    participant Access as Identity/Access
    participant Module as Tenant Module
    participant DB as PostgreSQL
    User->>UI: Start sign-in
    UI->>API: OIDC authorization request
    API->>IdP: Redirect / validate callback
    IdP-->>API: Authenticated identity
    API->>Access: Resolve principal, membership, role
    Access->>DB: Read membership
    DB-->>Access: Tenant membership
    Access-->>API: Authorized tenant context
    API-->>UI: Authenticated session
    UI->>API: Tenant-scoped operation
    API->>Module: Invoke authorized application contract
    Module->>DB: Read/write owned tenant data
    DB-->>Module: Result
    Module-->>API: Response without secret values
    API-->>UI: Tenant-scoped result
```

## Consumer proxy request

```mermaid
sequenceDiagram
    actor Consumer
    participant API as Consumer API
    participant Key as API-Key/Tenant Resolver
    participant Catalog as Catalog
    participant Config as Tenant Integration
    participant Secret as Secret Store
    participant Proxy as Proxy/HTTP Client
    participant Upstream
    participant Record as Usage and Logs
    Consumer->>API: Request via tenant subdomain + API key
    API->>Key: Validate key and subdomain agreement
    Key-->>API: Tenant/user context or reject
    API->>Catalog: Validate integration and endpoint allow list
    API->>Config: Resolve tenant integration settings
    Config-->>API: Settings and secret reference
    API->>Secret: Retrieve tenant-scoped upstream credential
    Secret-->>API: Secret value (proxy boundary only)
    API->>Proxy: Send one request, 30s timeout, 0.5MB cap
    Proxy->>Upstream: Allow-listed request
    Upstream-->>Proxy: Status, headers, body
    Proxy-->>API: Normalized result or platform failure
    API->>Record: Minimal outcome + correlated safe logs
    API-->>Consumer: HTTP 200 connector result or mapped platform error
```

## Deployment boundary

The application is deployed as a modular monolith and an OCI-compatible
artifact. Terraform provisions provider-specific resources only after a
hosting provider is selected. GitHub Actions builds, validates, and promotes
the same immutable artifact; canary behavior and previous-version rollback
must be designed with the selected host. Database migrations are controlled
deployment steps. The MVP has no availability SLA; 99.95% monthly
availability and exact Warm Standby RTO/RPO are production-readiness gates.
