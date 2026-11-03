# Frontend Functional Design

## Current scope amendment (2026-10-26)

All sign-in, authenticated management screens, and consumer API-key
authentication are deferred. Current UI scope is public catalog discovery.
The tenant and operator screens below are future-state designs and must not
be implemented until authentication/authorization returns to scope.

## Shared UI behavior

- Management UI supports OIDC sign-in and communicates only with
  authenticated application APIs.
- The API remains the authorization authority. Hiding a control in the UI
  does not replace server-side permission checks.
- Tenant screens operate within an authenticated tenant context. Do not
  accept a user-editable tenant ID as scope.
- Operator UI is a protected surface, separate from Tenant Admin role
  permissions.
- Forms show safe validation errors; never repopulate or display saved
  credentials/API-key values after initial creation.
- Loading, empty, success, validation, permission-denied, and service-error
  states are explicit. Do not show stack traces or internal server details.

## Component hierarchy

```text
ApplicationShell
├── AuthenticationRoutes
│   ├── SignInPage
│   └── OidcCallbackPage
├── TenantRoutes [authenticated tenant membership]
│   ├── TenantDashboardPage
│   ├── MembersPage [Tenant Admin: invite/role/revoke]
│   ├── IntegrationsPage
│   │   ├── IntegrationCatalogList
│   │   ├── IntegrationDetails
│   │   ├── OpenApiDocumentationView
│   │   └── TenantIntegrationSettingsForm [Tenant Admin]
│   └── ApiKeysPage
│       ├── OwnApiKeyList
│       ├── CreateApiKeyDialog
│       ├── OneTimeApiKeySecretView
│       └── RevokeApiKeyAction [owner or Tenant Admin in tenant]
└── OperatorRoutes [platform operator]
    ├── TenantProvisioningPage
    └── CatalogManagementPage
        ├── IntegrationDefinitionForm
        ├── AllowedEndpointsEditor
        ├── OpenApiDocumentEditor
        └── PublishIntegrationAction
```

## Component behavior

### Sign-in and session

- Start OIDC through the configured provider; Google is required initially.
- Show a generic authentication failure if authentication is invalid or
  membership is not available.
- After sign-in, render only routes available to the resolved role.
- On sign-out, clear local session state and navigate to the signed-out view.

### Tenant member management

- Members list is tenant-scoped and available only to authenticated
  membership.
- Tenant Admin may invite a user and choose Admin or view-only Member.
- Display invitation state and expiry; allow resend/revoke only for pending
  invitations.
- Resend refreshes the invitation link and 7-day expiry. The old link must
  not be shown as usable.
- Never display invitation tokens.
- View-only members cannot access mutation actions; API denial is shown
  safely if access is denied.

### Integration browsing and configuration

- List only published integrations available to the tenant.
- Integration details link to the associated OpenAPI documentation.
- Tenant Admin may add one configuration per catalog integration, edit
  tenant-local settings, or remove the tenant configuration.
- Form fields are driven by approved tenant-local settings; catalog
  definitions and endpoint allow lists are read-only.
- Credential entry is write-only. After save, show configured/not-configured
  state, never the stored value. Replacing credentials explicitly requires
  the user to enter a new value.

### Consumer key lifecycle

- Show the signed-in user's own key metadata; do not show secret values.
- Permit create/replace and owner revocation. Tenant Admins may also revoke
  keys in their tenant.
- On creation/replacement, display the usable key once with an explicit
  copy-now warning; closing the view does not allow retrieving it later.
- Allow overlapping active keys. Do not revoke a previous key automatically
  when a replacement is created.
- Confirm revocation and refresh the key status after success.

### Operator surfaces

- Tenant provisioning requires an operator route and creates the first
  tenant Admin.
- Catalog management supports integration metadata, endpoint allow lists,
  OpenAPI documentation, and publish state.
- Every operator API call requires server-side operator authorization.
- Catalog save/publish failures preserve safe user input where possible but
  never expose internal error details.

## API interaction and failure behavior

- Use the authenticated API client for management routes and preserve
  correlation IDs where supplied.
- Render 400 validation feedback without echoing secrets.
- On 401, clear invalid session/key-dependent UI state and require
  re-authentication where appropriate.
- On 403, show a permission-denied state; do not retry with a different
  tenant context.
- Consumer-facing proxy response handling belongs to the consuming
  application, not the management UI.
- The protected operator UI has no access to API key or upstream secret
  plaintext after creation/configuration.

## Accessibility and state

- Forms have associated labels, keyboard-operable controls, and announced
  validation/status messages.
- Destructive actions (remove integration, revoke invitation/key) require
  clear confirmation.
- Navigation and data views preserve tenant scope from the authenticated
  session and do not infer authorization from cached client state.
