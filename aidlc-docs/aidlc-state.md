# AI-DLC State Tracking

## Project Information
- **Project**: centre-lock
- **Project Type**: Brownfield
- **Start Date**: 2026-10-10T10:42:21Z
- **Current Stage**: CLOSED - project abandoned during CONSTRUCTION (2026-11-03)
- **Workspace Root**: `centre-lock/` (application; AIDLC context is stored in sibling `aidlc/` and `aidlc-docs/`)

## Workspace State
- **Existing Code**: Yes
- **Programming Languages**: C#, TypeScript, JavaScript
- **Build Systems**: .NET SDK / MSBuild, npm
- **Project Structure**: ASP.NET Core API, React UI, PostgreSQL, tests, Docker Compose
- **Reverse Engineering Needed**: Yes; initial artifacts created because none existed

## Stage Progress
### INCEPTION
- [x] Workspace Detection
- [x] Reverse Engineering - artifacts reviewed and approved
- [x] Requirements Analysis - approved
- [x] User Stories - approved
- [x] Workflow Planning - approved
- [x] Application Design - approved
- [x] Units Generation - approved

### CONSTRUCTION
- [x] Functional Design - approved
- [x] NFR Requirements - approved
- [x] NFR Design - approved
- [x] Infrastructure Design - approved
- [ ] Code Generation - ABANDONED at Part 2 (public catalog delivered; tenant host routing US-08 not built)
- [ ] Build and Test - not started; will not be executed

### OPERATIONS
- [ ] Operations - placeholder

## Execution Plan Summary
- **Stages to Execute**: User Stories, Application Design, Units Generation, Functional Design, NFR Requirements, NFR Design, Infrastructure Design, Code Generation, Build and Test
- **Stages to Skip**: None of the applicable design stages; Operations remains a placeholder
- **Review Artifact**: `aidlc-docs/inception/plans/execution-plan.md`

## Current Status
- **Lifecycle Phase**: CLOSED (abandoned by the user on 2026-11-03)
- **Current Stage**: None. The project stopped during CONSTRUCTION - Code Generation.
- **Next Stage**: None. See the Closure section below.
- **Final Status**: Abandoned. Delivered: public read-only catalog (US-04) and a GCP CI/CD pipeline with per-PR previews and staging. Not delivered: US-08 tenant host routing, production environment, all deferred authentication work.
- **Status before closure**: Scope amended on 2026-10-26: all authentication and sign-in are deferred, including OIDC and consumer API keys. Current MVP scope is public catalog discovery (US-04) and subdomain-based tenant routing only (US-08). Subdomains do not authorize access to tenant-private data or upstream proxy execution. Existing OIDC/membership code is experimental scaffolding, not a current deliverable. Tenant management, catalog writes, tenant configuration, keys, and proxy execution are deferred. Custom domains, the platform base domain, wildcard DNS/TLS, reserved hostnames, and provider-specific routing remain to be decided.
- **External Context**: Eight open GitHub issues retrieved and captured in `aidlc-docs/inception/reverse-engineering/github-issue-context.md`; issues are future-context, not accepted requirements.

## Extension Configuration
| Extension | Enabled | Decided At |
|---|---|---|
| Security Baseline | Yes | Requirements Analysis |
| Resiliency Baseline | Yes | Requirements Analysis |
| Property-Based Testing | Yes | Requirements Analysis (full enforcement) |

## Code Location Rules
- **Application Code**: Workspace root (never in `aidlc-docs/`)
- **AI-DLC Documentation**: `aidlc-docs/` only

## Closure (2026-11-03)
The user chose to end this AI-DLC cycle and abandon the project. No stage was approved after Infrastructure Design; Code Generation stopped part-way.

**Delivered (application repo, branch `devops-setup`)**
- Public read-only integration catalog and OpenAPI routes in every environment (US-04), plus the Development/flag-gated proxy.
- GCP non-production infrastructure as code (`state`, `bootstrap`, `shared`, `environment` Terraform roots): Cloud Run, Cloud SQL with a lease controller, Workload Identity Federation, per-PR preview and staging.
- GitHub pipelines: `ci.yml` (containers, Terraform validate, preview deploy plus full Bruno API suite), `cd.yml` (shared infrastructure and staging), `destroy-preview.yml`.

**Not delivered / known open items**
- US-08 tenant subdomain routing; base domain and wildcard DNS/TLS were never decided.
- Production environment, identity and stability-test promotion. Policy that was decided: full tests on CI containers, preview and staging; production runs stability tests only (`stability` Bruno tag).
- Preview-database sweeper for abandoned PRs, budget alerts and a Cloud Run spend cap.
- The last lease-controller fix (`a03c460`) and the `/renew` returning HTTP 500 in the CD `deploy-infrastructure` job were unresolved: `shared` had to be re-applied by hand.
- Authentication (OIDC, memberships, API keys) was deferred and not built; existing code is experimental scaffolding.
- Unpushed app commits at closure: see `git log origin/devops-setup..HEAD` in `centre-lock/`.

**Cloud clean-up if the GCP project is no longer wanted** (nothing is torn down by this document)
- Close or delete open previews, then `terraform destroy` in `environment/` for each remaining `pr-*`/`staging` state.
- `terraform destroy` in `shared/` (Cloud SQL, lease controller, scheduler), then `bootstrap/`, then `state/` last. The Cloud SQL instance is the only meaningful recurring cost (storage while stopped, about $0.035/hr while running).
