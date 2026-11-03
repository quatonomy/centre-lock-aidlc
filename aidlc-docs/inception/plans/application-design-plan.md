# Application Design Plan

## Current scope amendment (2026-10-26)

The original application-design questions and answers captured
authentication-dependent management and proxy behavior. All authentication
and sign-in are deferred. Current implementation scope is public catalog
discovery and tenant routing from bespoke subdomains only; see the amended
requirements and application design.

## Design context

Design the approved MVP requirements and stories within the accepted
architecture:

- .NET modular monolith with explicit module contracts and PostgreSQL/EF Core.
- React/TypeScript management UI; initial product architecture builds and
  serves the UI with the application. Reconcile this with the current local
  Nginx UI container without changing the accepted production direction.
- Identity/access, tenancy, catalog, connector, proxy, and usage boundaries
  remain logical modules, not independent services.
- Tenant scope is enforced at the application boundary and in data access;
  tenant-supplied identifiers are not authorization.
- Keep upstream credentials behind a provider-neutral interface until the
  hosting provider and its secret manager are selected.
- Keep the Development-only catalog/proxy routes separate from tenant-facing
  authenticated routes.

The design will define component responsibilities, high-level method
signatures, service orchestration, and dependencies. Detailed business rules
remain for Functional Design.

## Design questions

Please fill in each `[Answer]:` tag. Select one option or describe a custom
decision. Recommendations are defaults for owner review, not assumed
decisions.

### Question 1: Initial tenant and administrator provisioning
How is the first tenant administrator established?

A) A platform operator provisions the tenant and its first admin (recommended
for controlled B2B onboarding)

B) Self-service registration creates the tenant and first admin

C) Support both operator provisioning and self-service registration

X) Other (describe the onboarding and first-admin flow after `[Answer]:`
below)

[Answer]: A) A platform operator provisions the tenant and its first admin

### Question 2: Consumer API-key management permissions
Who may create, revoke, or replace tenant API keys scoped to a user?

A) Each user manages their own keys; tenant admins may also revoke keys
within their tenant (recommended balance of ownership and tenant control)

B) Each user manages only their own keys; tenant admins cannot manage member
keys

C) Tenant admins manage keys for tenant users; users cannot manage keys
themselves

X) Other (describe create/revoke permissions and tenant boundaries after
`[Answer]:` below)

[Answer]: A) Each user manages their own keys; tenant admins may also revoke keys within their tenant

### Question 3: Platform catalog administration interface
Which interface should platform operators use to manage catalog definitions,
endpoint allow lists, and OpenAPI documentation?

A) A protected operator management UI (recommended for the management
workflow)

B) A protected platform administration API only

C) Both an operator UI and a protected administration API

X) Other (describe the interface and authorization boundary after
`[Answer]:` below)

[Answer]: A) A protected operator management UI

## Design execution checklist

- [x] Analyze answers for ambiguity and resolve material gaps.
- [x] Obtain explicit approval of this design plan.
- [x] Define component boundaries and responsibilities in `components.md`.
- [x] Define high-level component methods and contracts in
  `component-methods.md`; leave detailed business rules to Functional Design.
- [x] Define service responsibilities and orchestration in `services.md`.
- [x] Define dependencies, communication patterns, and data flow in
  `component-dependency.md`.
- [x] Consolidate the design in `application-design.md`.
- [x] Check consistency with the accepted architecture, requirements, user
  stories, Security Baseline, Resiliency Baseline, and PBT extension.
- [x] Present the design artifacts for user review and approval.

## Planned artifacts

- `aidlc-docs/inception/application-design/components.md`
- `aidlc-docs/inception/application-design/component-methods.md`
- `aidlc-docs/inception/application-design/services.md`
- `aidlc-docs/inception/application-design/component-dependency.md`
- `aidlc-docs/inception/application-design/application-design.md`

## Approval

The design approach must be explicitly approved after the questions above
are answered and before the design artifacts are generated. The generated
application design requires a separate user approval before Units Generation.

**Status:** Design artifacts are generated and awaiting user review and
approval. The over-limit response behavior and detailed error-object schema
are presented as proposals, not settled requirements.
