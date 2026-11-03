# NFR Design Plan: CentreLock Application

## Current scope amendment (2026-10-26)

Authentication and sign-in, consumer keys, tenant-private management, and
proxy execution are deferred. This original NFR design plan remains
future-state where it assumes those workloads; current NFR implementation
must match the public catalog and tenant-routing scope.

## Design basis

Design provider-neutral logical patterns for the approved high-criticality
modular monolith. Carry forward:

- Security Baseline rules as blocking constraints.
- Resiliency Baseline, including metrics/logs/health, multi-zone production,
  scaling, dependency isolation, backups, failover, and resiliency testing.
- Full PBT using FsCheck/xUnit and fast-check/Vitest.
- No MVP SLA; service may be off while idle. Before production traffic,
  approve 99.95% monthly availability, exact RTO/RPO, GDPR retention/access,
  measured capacity and latency targets.
- Production region topology selected as single-region, multi-zone.
- Recovery strategy direction selected as Warm Standby, with exact RTO/RPO
  deferred.
- 30-second upstream timeout, 0.5 MB body limit, no automatic retries,
  canary rollout, previous-version rollback, and manual incident response.
- Hosting provider and secret manager are unselected; use logical interfaces
  and state any provider-dependent design gates.

## Required owner decisions

### Question 1: Warm Standby scope and regional failure
Warm Standby was selected as the production recovery strategy, while the
regional topology is single-region/multi-zone and explicitly does not survive
a full-region failure. Which interpretation is intended?

A) Warm Standby is an in-region reduced-capacity standby for instance/zone
failure; full-region failure is out of scope, consistent with the selected
single-region topology (recommended interpretation of the accepted answers)

B) Warm Standby must survive a full-region failure; revise the production
topology to multi-region active-passive

C) Keep the regional topology undecided until exact RTO/RPO and hosting
provider are selected

X) Other (describe the intended DR scope after `[Answer]:` below)

[Answer]: A) Warm Standby is an in-region reduced-capacity standby for instance/zone failure; full-region failure is out of scope

### Question 2: Resiliency and DR testing approach
How should failover, restore, and resiliency mechanisms be validated?

A) Use an existing DR testing/game-day/chaos practice; provide its reference
after `[Answer]:` below

B) Propose a DR test schedule and controlled chaos experiment plan for
adoption before production

C) Defer execution to the Operations phase; document test scenarios and
acceptance criteria now (recommended while provider and operating model are
unselected)

X) Other (describe the approach, owner, and timing after `[Answer]:` below)

[Answer]: C) Defer resiliency/DR test execution to Operations and document scenarios and acceptance criteria now. Clarification: no existing practice reference was identified.

## Planned NFR design

- Define logical observability components for metrics, structured logs,
  traces where applicable, dashboards, alerting, and request correlation.
- Define shallow/deep health checks and routing integration without naming a
  cloud-specific product.
- Define capacity/scaling controls, minimum/maximum limits, quota monitoring,
  and a load-test gate; defer numerical sizing to measured traffic.
- Define upstream dependency timeouts, circuit-breaker/bulkhead behavior,
  graceful failure, and ensure there are no automatic request retries.
- Define encrypted backups, restore validation, retention decision gate,
  and the chosen recovery scope.
- Define deployment canary verification, rollback boundaries, and
  migration-safe change process.
- Map Security Baseline, Resiliency Baseline, and PBT rules to logical
  components and evidence.
- Generate `nfr-design-patterns.md` and `logical-components.md`.

## Checklist

- [x] Analyze answers and resolve any ambiguity.
- [x] Obtain approval of this design plan.
- [x] Generate `nfr-design-patterns.md`.
- [x] Generate `logical-components.md`.
- [x] Validate no provider-specific selection is implied.
- [x] Present NFR design artifacts for approval.

**Status:** NFR design artifacts are generated and awaiting user review.
