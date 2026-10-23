# Security and data-handling standards

Consider security implications for every change that handles identity,
tenant context, credentials, external input, API traffic, logs, persistence,
or deployment configuration. Follow the requirements in
[`centre-lock/SPEC.md`](../../centre-lock/SPEC.md), the detailed requirements
in [`../inception/requirements/product-requirements.md`](../inception/requirements/product-requirements.md),
and the boundaries in the
[architecture overview](../architecture/overview.md); this document does not
create security guarantees beyond those sources.

## Change review

For each relevant change, check:

- **Authentication and authorization:** Protected actions are authenticated
  and authorized on the server. UI visibility is not an authorization
  control. Preserve the specified admin/user permissions.
- **Tenant isolation:** Derive tenant scope from trusted authenticated or
  routing context. Do not treat caller-supplied tenant IDs as proof of
  access. Check reads, writes, list/search paths, and failure behavior for
  cross-tenant access.
- **Input and upstream boundaries:** Validate untrusted inputs against their
  contract. Enforce centrally managed endpoint allow lists and prevent
  caller-controlled paths or payloads from bypassing connector/proxy
  boundaries.
- **Credentials and keys:** Keep upstream credentials tenant-specific and
  inaccessible to consumers; never return them after setup or expose them in
  logs. Store consumer API keys as one-way hashes. Do not commit secrets or
  put them in images, responses, test fixtures, or build artifacts.
- **Logging and data:** Log only the minimum specified. Do not log credentials,
  API keys, or sensitive upstream payloads. Follow defined retention and
  access rules; where GDPR handling is still unspecified, do not invent a
  policy and identify the unresolved requirement.
- **Dependencies and configuration:** Avoid unnecessary dependencies. Keep
  configuration separate from secrets and use the selected environment's
  secret-management mechanism when one is defined.
- **Errors and external services:** Do not turn security or integration
  failures into success-shaped responses. Avoid leaking secrets or sensitive
  internal details through errors.

Add or update tests for authorization, tenant isolation, validation, and
failure cases affected by the change. Consult
[`testing.md`](testing.md) for test-layer expectations. Include dependency
and security scanning in CI and keep it effective as dependency manifests or
build paths change.

If a proposed change requires a decision about authentication, tenant
resolution, credential handling, data retention, or another material
security property that is not settled by the specification or accepted
decisions, resolve that requirement before implementing it.
