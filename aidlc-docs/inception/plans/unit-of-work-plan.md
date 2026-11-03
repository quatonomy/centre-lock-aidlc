# Unit of Work Plan

## Current scope amendment (2026-10-26)

The original dependency plan included identity, roles, consumer keys, and an
authenticated proxy. These are deferred. The current work unit is narrowed
to public catalog discovery and tenant-subdomain routing only; see the
amended unit-of-work and story map.

## Decomposition basis

The accepted architecture is one .NET modular monolith with a shared
PostgreSQL database and a management UI delivered with the application. Under
the Units Generation definition, this is one deployable unit of work,
`CentreLock Application`, containing logical modules. Do not create
independently deployable services or split stories into separate deployments.

Logical modules from the approved design:

- API host and shared request pipeline
- Identity/access and tenant provisioning
- Tenancy context
- Platform integration catalog and operator UI
- Tenant integration configuration
- Provider-neutral secret-store boundary
- Consumer API-key lifecycle
- Consumer proxy
- Minimal usage records and structured security/application logging
- Tenant management UI
- PostgreSQL/EF Core persistence and migrations
- Terraform and GitHub Actions delivery

## Context review by decomposition category

- **Story grouping:** All 11 stories belong to the same product/application
  deployment unit. The story epics remain useful as implementation slices,
  not service boundaries.
- **Dependencies:** All modules share application contracts and PostgreSQL.
  The proxy depends on tenant context, key validation, catalog allow lists,
  tenant configuration, and secret retrieval. No cross-service messaging or
  independent deployment is required.
- **Team alignment:** The accepted delivery decision has the project owner
  review all pull requests. No separate team ownership boundary is currently
  defined.
- **Technical considerations:** The API, UI, database, and Terraform delivery
  remain coordinated as one product. The hosting provider is unselected;
  provider-specific secret-manager and infrastructure work stays behind
  interfaces/configuration until selection.
- **Business domains:** Identity/tenancy, catalog/configuration,
  consumer credentials/proxy, and operational records are logical domains
  within the monolith.
- **Code organization:** Brownfield project; greenfield-only repository
  layout preference questions do not apply.

## Question: Internal implementation sequence
The deployable unit is fixed by the accepted architecture. Which sequence
should organize development inside that unit?

A) Build secure identity/tenant/data foundations first, then implement
catalog/configuration and consumer key/proxy capabilities as end-to-end
vertical slices (recommended)

B) Implement the tenant access, integration management, and consumer proxy
epics as end-to-end vertical slices, adding each shared foundation with its
first dependent story

C) Complete shared backend/domain modules first, then integrate the
management UI, then finish delivery/infrastructure

X) Other (describe the internal sequence and dependency rule after
`[Answer]:` below)

[Answer]: A) Build secure identity/tenant/data foundations first, then implement catalog/configuration and consumer key/proxy capabilities as end-to-end vertical slices

## Generation checklist

- [x] Analyze the answer for ambiguity and resolve any material gaps.
- [x] Obtain approval of this unit-of-work plan.
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work.md` with
  the single deployable unit and its logical modules.
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md`
  with the dependency matrix and integration strategy.
- [x] Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md`
  mapping every story to the unit.
- [x] Validate unit boundaries and dependencies against the accepted
  architecture and application design.
- [x] Verify that every approved user story is assigned.
- [x] Present the unit artifacts for user review and approval.

## Approval

Units are generated only after the internal sequencing answer is recorded
and this plan is approved. Unit artifacts require separate user approval
before Construction begins.

**Status:** The unit definition, dependency map, and story map are generated
and awaiting user review and approval.
