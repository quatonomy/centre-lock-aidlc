# Active GitHub Issue Context

Open repository issues were retrieved from `quatonomy/centre-lock` on 2026-10-10 after GitHub CLI authentication became available. These issues describe possible future objectives and questions. They are context for Requirements Analysis, not accepted requirements, decisions, or authorization to implement work.

## Open Issues

### #10 - config/ai driven

> its strategic to be config/data driven for new integrations, we need to keep on top of integration endpoints, ensuring we can connect to all of them as required. Ai is likely needed to periodically review and update bruno and other tests. an AI workflow to add integrations as a whole is likely needed, saves creating them features in web and can be left internally to GitHub users.

URL: https://github.com/quatonomy/centre-lock/issues/10

**Context**: Reinforces investigation of centrally managed, data-driven catalog onboarding. It raises possible recurring AI-supported endpoint/test maintenance and an internal GitHub-user workflow for adding integrations. The automation, permissions, review gates, test update scope, and meaning of "all" endpoints are undecided.

### #9 - DTO

> should we manage dto's too? this would be easier for teams if we could somehow allow them to easier understand our objects? otherwise they will always be creating their own?

URL: https://github.com/quatonomy/centre-lock/issues/9

**Context**: Raises whether the platform should publish or manage DTOs to help customer teams understand integration objects. No contract format, ownership model, transformation behavior, or support commitment is decided.

### #8 - audibility

> need code and info solution for this

URL: https://github.com/quatonomy/centre-lock/issues/8

**Context**: Signals an unresolved auditability need, but provides no definition of auditable events, data, access, retention, or implementation.

### #7 - cd

> on merge to main, run a new cd pipeline, this should:
> 1. run on docker instance
> 2. run on stage environment
> 3. run on production environment
>
> We need to work out what out platform should be, consider options with copilot and possibly document here.

URL: https://github.com/quatonomy/centre-lock/issues/7

**Context**: Requests a merge-to-main delivery pipeline through Docker, staging, and production, while explicitly leaving hosting-platform selection for evaluation. Align with accepted ADR 0002 rather than treating the issue as a provider decision.

### #6 - Ai structure

> Once we start building specific workflows for very specific agents/skills needed, we will need to rearrange the structure from monolith files to a more industry standard for larger scalable codebases.

URL: https://github.com/quatonomy/centre-lock/issues/6

**Context**: Anticipates restructuring agent/skill guidance as it grows. The repository's current AI guidance layout and the intended target structure are not fully defined.

### #5 - test coverage

> unit/feature/integration coverage, probably in a pipeline.

URL: https://github.com/quatonomy/centre-lock/issues/5

**Context**: Requests coverage across unit, feature, and integration tests, likely in CI. The repository coding standard separately states a target for meaningful production behavior, but metric, exclusions, report publication, and enforcement threshold remain to be determined.

### #4 - DORA

> community package to capture?

URL: https://github.com/quatonomy/centre-lock/issues/4

**Context**: Ambiguous question about capturing DORA metrics through a community package. Metrics, package, data handling, and purpose are unspecified.

### #3 - Observability

> Setup monitoring/alerting.

URL: https://github.com/quatonomy/centre-lock/issues/3

**Context**: Raises monitoring and alerting as future work. Signals, service-level objectives, tooling, routing, ownership, and hosting constraints are open.

### #2 - Full Deployment Setup

> - Find platform and deploy
> - dev-production environments

URL: https://github.com/quatonomy/centre-lock/issues/2

**Context**: Calls for platform selection and deployment across development and production environments. Selection, environment topology, and operational requirements remain unresolved; accepted ADR 0002 provides current delivery principles.

## Cross-Cutting Themes

- Make integration onboarding data-driven and consider internally governed AI support for catalog and test maintenance.
- Clarify customer-facing integration contracts and discoverability, including whether DTOs are a product capability.
- Establish the auditability, monitoring, alerting, and data-handling requirements before choosing implementation details.
- Evaluate hosting and delivery topology against the accepted portable-container and staged-release decisions.
- Improve measurable test coverage without confusing source coverage with endpoint/behavioral coverage.
- Evolve AI-agent and skill organization only when workflow growth justifies a defined structure.

## Analysis Boundary

These issues supplement the goals and open questions in `aidlc-docs/inception/requirements/product-requirements.md`. Their wording does not by itself settle requirements, security properties, architecture, provider selection, tooling, or prioritization. Resolve material choices during Requirements Analysis with the product owner.
