# Unit of Work Story Map

**Scope amendment (2026-10-26):** Authentication and sign-in of all kinds are
deferred. Only US-04 public catalog discovery and the routing-only portion of
US-08 are current MVP scope. Tenant subdomains are identifiers for routing,
not authorization. All other stories below are future work until a new
authentication scope is approved.

Every story is assigned to the single deployable unit
`centre-lock-application`. Epic groupings indicate logical work slices, not
separate services. The current sequence is public catalog discovery, tenant
subdomain routing, and delivery. The former secure-management/proxy sequence
is deferred.

| Story | Story name | Persona(s) | Unit | Logical modules | Planned slice |
|---|---|---|---|---|---|
| US-01 | Sign in to a tenant account | Tenant Admin, Tenant Member | `centre-lock-application` | Identity/access, tenant context, management UI | Deferred |
| US-02 | Invite a tenant user and assign a role | Tenant Admin | `centre-lock-application` | Identity/access, tenancy, management UI | Deferred |
| US-03 | View tenant information without edit access | Tenant Member | `centre-lock-application` | Authorization, tenant context, catalog/UI | Deferred |
| US-04 | Discover available integrations and documentation | Visitor | `centre-lock-application` | Public catalog, OpenAPI, UI | Current MVP |
| US-05 | Configure an integration for my tenant | Tenant Admin | `centre-lock-application` | Tenant configuration, catalog, secret-store boundary, UI | Deferred |
| US-06 | Maintain the platform integration catalog | Platform Operator | `centre-lock-application` | Operator authorization, catalog, operator UI | Deferred |
| US-07 | Create and manage consumer API keys | Tenant API Consumer, Tenant Admin | `centre-lock-application` | Consumer API keys, authorization, management UI | Deferred |
| US-08 | Resolve a tenant from its bespoke subdomain | Visitor | `centre-lock-application` | Tenant host routing | Current MVP: routing only |
| US-09 | Receive a completed upstream response | Tenant API Consumer | `centre-lock-application` | Proxy, connector/HTTP client | Deferred |
| US-10 | Understand platform-side proxy failures | Tenant API Consumer | `centre-lock-application` | API pipeline, proxy, safe error mapping | Deferred |
| US-11 | Record minimal request outcomes safely | Platform Operator | `centre-lock-application` | Usage recorder, structured logging, persistence | Deferred |

## Coverage check

- **Stories mapped:** 11 of 11.
- **Unassigned stories:** None.
- **Separate deployment units:** None; logical modules remain inside one
  application deployment.
- **Cross-cutting requirements:** Security, resiliency, property-based
  testing, controlled migrations, Terraform, CI, canary deployment, and
  rollback apply to the unit and are carried into per-unit design.
- **Deferred capabilities:** All authentication/sign-in, memberships/roles,
  invitations, tenant-specific configuration, catalog administration,
  consumer API keys, proxy execution, usage/billing, payment collection,
  invoicing, custom domains, and the GitHub connector.
