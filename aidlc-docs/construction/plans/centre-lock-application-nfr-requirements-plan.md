# NFR Requirements Plan: CentreLock Application

## Current scope amendment (2026-10-26)

All authentication and sign-in are deferred, and tenant-private management
and proxy traffic are outside the current MVP. Reassess capacity and
criticality assumptions based on the narrowed public-catalog and
tenant-routing workload before production.

## Existing constraints to carry forward

Do not reopen decisions already recorded in the requirements and functional
design:

- No MVP availability SLA; service may be turned off while idle.
- Before production customer traffic, approve 99.95% monthly availability,
  exact Warm Standby RTO/RPO, and GDPR-compliant retention/access policy.
- Production topology direction is single-region, multi-zone; recovery
  strategy direction is Warm Standby.
- Upstream calls time out after 30 seconds, are not retried, and upstream
  bodies are limited to 0.5 MB; oversized responses map to 502 without
  truncation.
- Security/application logs are retained for at least 90 days, separate from
  minimal per-request usage outcomes.
- Enforce Security Baseline, Resiliency Baseline, and full PBT rules.
- Preserve the .NET modular monolith, React/TypeScript UI, PostgreSQL, EF
  Core, GitHub Actions, and Terraform decisions. Hosting and secret-manager
  provider remain deferred.

The NFR assessment will document known requirements and gates, distinguish
MVP behavior from production targets, and avoid implying unapproved
production readiness.

## Questions for owner decisions

Please answer each `[Answer]:` tag. Choose a provided option or describe a
specific alternative.

### Question 1: Initial capacity target
What expected initial load should guide MVP sizing and test scenarios?

A) No numeric MVP capacity commitment; measure baseline and load-test a
documented expected profile before production traffic (recommended while
customer demand is unknown)

B) Define an initial target now; give expected concurrent tenants and peak
consumer requests per second after `[Answer]:` below

C) Other (describe target load, growth assumptions, and when it applies after
`[Answer]:` below)

[Answer]: A) No numeric MVP capacity commitment; measure baseline and load-test a documented expected profile before production traffic

### Question 2: Platform response-time target
What response-time objective should be measured for CentreLock?

A) Set MVP measurement only, with no latency SLA; define platform overhead
percentiles and establish a numeric production target before customer traffic
(recommended while upstream latency varies)

B) Define a numeric target now; specify percentile, duration/window, and
whether it measures platform-only or end-to-end proxy latency after
`[Answer]:` below

C) Other (describe the target and measurement boundary after `[Answer]:`
below)

[Answer]: A) Set MVP measurement only, with no latency SLA; define platform overhead percentiles and establish a numeric production target before customer traffic

### Question 3: Web accessibility target
What accessibility conformance target applies to the tenant and operator UI?

A) WCAG 2.2 AA for user-facing flows (recommended)

B) WCAG 2.1 AA for user-facing flows

C) No formal conformance target for MVP; continue semantic markup and
keyboard/screen-reader testing

X) Other (describe the standard and conformance level after `[Answer]:`
below)

[Answer]: A) WCAG 2.2 AA for user-facing flows

### Question 4: Property-based testing frameworks
The full PBT extension requires a framework with domain generators,
shrinking, reproducible seeds, and runner/CI integration. Which toolchain
should be selected?

A) FsCheck with xUnit for C# domain/API logic; fast-check with Vitest for
TypeScript logic (recommended where testable domain logic exists in both)

B) FsCheck with xUnit for C#; treat TypeScript UI as not PBT-applicable unless
meaningful pure business logic is introduced

C) Specify alternative C# and/or TypeScript frameworks after `[Answer]:`
below, including any language you consider not applicable and why

[Answer]: A) FsCheck with xUnit for C# domain/API logic; fast-check with Vitest for TypeScript logic

### Question 5: Production workload classification
How should the production application and its PostgreSQL data be classified
for business criticality and outage impact?

A) High: tenant management/proxy outage blocks customer integration use and
access to tenant data (recommended)

B) Medium: customers can tolerate material but bounded service interruption

C) Critical: outage has immediate severe business, financial, or regulatory
impact

X) Other (describe the classification and impact after `[Answer]:` below)

[Answer]: A) High: tenant management/proxy outage blocks customer integration use and access to tenant data

## Carried-forward mandatory topics

- **Security:** Apply all Security Baseline rules as blocking constraints,
  including encryption, access control, request validation, headers,
  rate-limiting, supply-chain controls, safe errors, security alerting, and
  centralized structured logging.
- **Resilience:** Carry forward the approved availability/recovery direction,
  single-region multi-zone topology, canary rollout, previous-version
  rollback, GitHub-visible change record, and human-managed incident response
  documented outside GitHub.
- **Observability:** Define health/readiness checks, resource and dependency
  monitoring, authentication/authorization alerts, log integrity/retention,
  dashboards, and service-level measurement needed for the production gate.
- **Recovery:** Define backup/restore and validation requirements without
  inventing exact retention or RTO/RPO; those require owner approval before
  production traffic.
- **PBT:** Map testable properties to the selected frameworks, use
  domain-specific bounded generators, preserve shrinking and seed
  reproducibility, and complement property tests with examples, API tests,
  PostgreSQL tests, and Playwright UI tests.

The Resiliency Baseline's DR/chaos testing approach is a user decision for
NFR Design; it will be asked there rather than silently selected here.

## NFR assessment checklist

- [x] Analyze answers and resolve material ambiguity.
- [x] Document measurable NFRs, assumptions, scope, and production gates in
  `nfr-requirements.md`.
- [x] Document stack and testing framework decisions with rationale in
  `tech-stack-decisions.md`.
- [x] Map security, resiliency, and PBT constraints to verification evidence.
- [x] Identify requirements with no numeric target and record their decision
  owner and deadline/gate rather than inventing values.
- [x] Present NFR requirements for user review and approval.

## Planned artifacts

- `aidlc-docs/construction/centre-lock-application/nfr-requirements/nfr-requirements.md`
- `aidlc-docs/construction/centre-lock-application/nfr-requirements/tech-stack-decisions.md`

**Status:** The owner selected no numeric MVP capacity/latency SLA, WCAG 2.2
AA, FsCheck/xUnit and fast-check/Vitest, and High production criticality.
NFR artifacts are generated and await review.

Capacity response defers numeric sizing until a measured expected profile is
load-tested before production traffic. Latency response defers a numeric
objective until platform-overhead baselines are measured; a numeric
production target is required before customer traffic. No custom framework
or accessibility target was requested.
