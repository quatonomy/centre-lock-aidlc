# Component Method Contracts

## Current scope amendment (2026-10-26)

All application authentication and sign-in are deferred. Contracts below
that rely on OIDC, membership, consumer API keys, or authenticated proxy
requests are future-state proposals, not current MVP implementation
requirements. Bespoke tenant subdomains select routing context only.

The signatures below are illustrative application-level contracts, not
compile-ready types. Functional Design will specify detailed validation,
transaction, error, and business-rule behavior.

## Identity and tenancy

```text
IIdentityService
  AuthenticateOidcAsync(OidcCallback) -> AuthenticatedPrincipal
  GetTenantMembershipAsync(PrincipalId, TenantId) -> TenantMembership?
  RequirePlatformOperatorAsync(PrincipalId) -> AuthorizationResult

ITenantProvisioningService
  ProvisionTenantAsync(OperatorPrincipal, TenantProvisioningRequest)
    -> TenantProvisioningResult
  ResolveManagementTenantAsync(AuthenticatedPrincipal, TenantSelector)
    -> TenantContext
  ResolveConsumerTenantAsync(TenantSubdomain, PresentedApiKey)
    -> ConsumerTenantContext
```

`ProvisionTenantAsync` creates a tenant and its initial Tenant Admin only for
an authorized platform operator. Consumer resolution validates that the
subdomain and key identify the same tenant before any integration lookup or
upstream call.

## Integration catalog and tenant configuration

```text
IIntegrationCatalog
  ListPublishedAsync() -> IReadOnlyList<IntegrationSummary>
  GetPublishedAsync(IntegrationId) -> IntegrationDefinition?
  GetAllowedEndpointAsync(IntegrationId, HttpMethod, Path)
    -> AllowedEndpoint?
  CreateOrUpdateAsync(OperatorPrincipal, CatalogDefinition)
    -> IntegrationDefinition
  PublishAsync(OperatorPrincipal, IntegrationId) -> Result

ITenantIntegrationService
  ListAsync(TenantContext) -> IReadOnlyList<TenantIntegration>
  AddAsync(TenantContext, IntegrationId, TenantIntegrationSettings)
    -> TenantIntegration
  UpdateAsync(TenantContext, TenantIntegrationId, TenantIntegrationSettings)
    -> TenantIntegration
  RemoveAsync(TenantContext, TenantIntegrationId) -> Result
```

Catalog write methods require platform-operator authorization. Tenant
configuration methods require a Tenant Admin and always derive tenant scope
from trusted context.

## Secret store

```text
IUpstreamSecretStore
  StoreAsync(TenantId, TenantIntegrationId, SecretValue) -> SecretReference
  RetrieveAsync(TenantId, TenantIntegrationId, SecretReference)
    -> SecretValue?
  DeleteAsync(TenantId, TenantIntegrationId, SecretReference) -> Result
```

Implement the interface with the selected hosting provider's secret manager
after hosting selection. `SecretValue` must not be serialized into ordinary
application responses, logs, or catalog data.

## Consumer API keys

```text
IConsumerApiKeyService
  CreateAsync(TenantContext, OwnerUserId) -> CreatedApiKey
  ValidateAsync(PresentedApiKey) -> ValidatedApiKey?
  RevokeOwnAsync(TenantContext, OwnerUserId, ApiKeyId) -> Result
  RevokeByAdminAsync(TenantContext, AdminUserId, ApiKeyId) -> Result
  ReplaceAsync(TenantContext, OwnerUserId, ApiKeyId) -> CreatedApiKey
```

`CreatedApiKey` returns the usable key only once. Persistent records contain a
one-way hash and non-secret metadata. Owner and admin operations verify
tenant membership and key ownership/tenant scope.

## Proxy and request recording

```text
IConsumerProxyService
  ExecuteAsync(ConsumerTenantContext, ProxyRequest) -> ProxyResult

IUpstreamHttpClient
  SendAsync(UpstreamRequest, Timeout, MaxResponseBytes)
    -> UpstreamResult

IRequestOutcomeRecorder
  RecordAsync(RequestOutcome) -> Result

ISecurityEventLogger
  WriteAsync(SecurityEvent) -> Result
```

`ExecuteAsync` composes catalog allow-list validation, tenant integration
lookup, scoped secret retrieval, outbound request, response processing, and
minimal outcome recording. Security/application logs and usage records use
separate sinks or record types.

## UI-facing application services

```text
ITenantManagementService
  GetDashboardAsync(TenantContext) -> TenantDashboard
  GetIntegrationDocumentationAsync(TenantContext, IntegrationId)
    -> OpenApiDocument

IOperatorCatalogService
  GetCatalogAsync(OperatorPrincipal) -> IReadOnlyList<CatalogEntry>
  SaveCatalogEntryAsync(OperatorPrincipal, CatalogDefinition) -> Result
```

The UI invokes application endpoints; it does not access PostgreSQL or secret
storage directly.
