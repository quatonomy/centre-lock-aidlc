# Requirements Clarification Questions

## Scope status

The recorded answers below capture the original requirements baseline.
**Scope amendment (2026-10-26):** All authentication and sign-in are
deferred, including management OIDC and consumer API keys. The current MVP
is public read-only catalog discovery and tenant routing from bespoke
subdomains beneath a configured shared platform base domain. See
`requirements.md` and `product-requirements.md` for the active requirements;
this amendment supersedes prior answers where they included authentication,
membership/roles, tenant-private management, or an authenticated proxy.

Your answers are recorded below, and the Security Baseline, Resiliency
Baseline, and Property-Based Testing extensions are enabled. The requested
clarifications are complete. Exact RTO/RPO values and GDPR retention/access
rules remain production-readiness decisions, not MVP blockers. Accepted
decisions remain in force; in particular, GitHub Actions and Terraform remain
the delivery standards, and tenant isolation and GDPR compliance are
constraints.

Answers are recorded in the `[Answer]:` tags below.

## Question 1: MVP capabilities
You selected an MVP requirements baseline. Which capabilities must be included
in the MVP? Also state whether the GitHub connector is included. Its
authorization method remains undecided until its use cases and permission
scopes are defined.

A) Tenant sign-in and membership/roles, tenant integration configuration,
consumer proxy, usage records and views, and plan/per-request pricing display;
defer payment collection and invoicing

B) Tenant sign-in and membership/roles, tenant integration configuration, and
consumer proxy; defer usage views, pricing, billing, and payment collection

C) Include the capabilities in A and payment collection and invoicing

X) Other (list the capabilities to include and defer after `[Answer]:` below)

[Answer]:b

## Question 2: Tenant-facing proxy status mapping
For a completed upstream response, what HTTP status should centre-lock return?
The upstream status and body will be included under `connector`.

A) Return HTTP 200 for every completed upstream response, including upstream
4xx and 5xx responses

B) Return the upstream HTTP status, while also including it and the response
body under `connector`

C) Define a mapping from upstream status classes to platform status codes;
provide the mapping

X) Other (please describe after `[Answer]:` below)

[Answer]:a

For platform-side failures such as an invalid request, rejected credential,
upstream transport failure, or timeout, specify desired status codes if they
should differ from standard 4xx, 502, and 504 responses. Also state whether
upstream response headers should be forwarded; whether response bodies should
be parsed as JSON when possible and otherwise returned as text; and any
maximum upstream response size.

## Question 3: Availability target
What availability target should apply to the production platform, measured
monthly?

A) 99.0%

B) 99.9%

C) 99.95%

D) 99.99%

X) Other (provide the target and measurement period after `[Answer]:` below)

[Answer]: x - the target is 1% the reason being during mvp build we may turn it off and on to reduce operation costs as development/testing in environments is not consistent so it does not make sense to keep them running for no reason with no customers and no testing going on, later this will be changed to option C when fit for production customer traffic

## Question 4: RTO, RPO, and disaster recovery strategy
What are your Recovery Time Objective (RTO) and Recovery Point Objective
(RPO) goals? These determine the disaster recovery strategy and infrastructure
redundancy level. Select the strategy and give exact maximum RTO/RPO values.

A) Hours — Backup & Restore strategy; data is backed up and services are
redeployed/restored on failure

B) Tens of minutes — Pilot Light strategy; data is live and services are
deployed but idle until failover

C) Minutes — Warm Standby strategy; data is live and services run at reduced
capacity until failover

D) Near real-time — Multi-site Active/Active strategy; live services run in
multiple regions

E) No cross-region disaster recovery; single-region deployment is acceptable
with multi-zone availability

X) Other (describe the strategy and exact maximum RTO/RPO after `[Answer]:`
below)

[Answer]: x - for now while building mvp it is not a concern but we will target C when fit for production traffic

## Question 5: Change management
How should production changes be governed?

A) Use an existing organizational process; provide its name or tool

B) No formal process exists; propose a lightweight change record, approval,
and rollback-note process

C) The workload is exempt from formal change management; provide the rationale

X) Other (please describe after `[Answer]:` below)

[Answer]:b - change record is applied in github in the most appropriate manner for visibility

## Question 6: Production rollback
How should a failed production deployment be rolled back?

A) Redeploy the previous version-pinned infrastructure and application artifact

B) Switch traffic back using a blue/green deployment

C) Automatically roll back a canary deployment when health metrics regress

D) Require database-aware rollback; specify the migration/data reversal
expectations

E) Use an existing organizational rollback procedure; provide its reference

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 7: Production deployment style
Which production deployment strategy is acceptable for the MVP?

A) Direct/in-place deployment

B) Rolling replacement

C) Blue/green deployment

D) Canary deployment with progressive traffic shift

X) Other (please describe after `[Answer]:` below)

[Answer]:d

## Question 8: Regional topology
Does the production workload require multi-region deployment, or is
single-region with multi-zone redundancy sufficient? Choose consistently with
your RTO/RPO decision.

A) Single-region, multi-zone; tolerate a zone failure, not a full-region
failure

B) Multi-region active-passive; fail over if a region fails

C) Multi-region active-active; continue serving across regions without
failover downtime

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 9: GDPR and log retention
You requested minimum request records and deferred broader auditability,
subject to GDPR compliance. The enabled Security Baseline calls for security
logs to be retained for at least 90 days. How should the requirements handle
these distinct records?

A) Keep minimal API usage records separate from security/application logs;
retain security/application logs for at least 90 days, and approve a
GDPR-compliant retention/access policy before production

B) Require a privacy/legal review to determine retention for both record
types before production; do not set 90 days unless that review supports it

C) Adopt another approach that meets GDPR obligations and the security logging
needs; describe retention, access, and the review deadline

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 10: Incident response
How are production incidents handled?

A) Use an existing incident response process; provide its name or reference

B) No formal process exists; propose a lightweight incident response and
Correction of Errors (COE) process

X) Other (please describe after `[Answer]:` below)

[Answer]:b - this is handled by humans for now, and documented outside the scope of github, this may change later

## Question 11: Platform-side proxy failures
You selected HTTP 200 for completed upstream responses. This question is only
about platform-side failures such as invalid requests, rejected consumer API
keys, upstream transport failures, and timeouts. Please confirm or adjust the
proposed status codes, and specify the error response shape.

A) Use 400 for invalid requests, 401 for missing or invalid API keys, 403 for
unauthorized requests, 502 for upstream transport failures, and 504 for
timeouts; return a JSON platform error object

B) Use a custom status mapping or error response shape; describe it after
`[Answer]:` below

X) Other (describe the status codes and error response shape after
`[Answer]:` below)

[Answer]:a

Specify the upstream response handling for the content included under
`connector`:
- Whether to forward upstream response headers (and which, if only some)
- Whether to parse JSON when possible and otherwise return text, or preserve
  the upstream body without parsing
- The maximum upstream response size

A) Forward no upstream response headers; parse valid JSON and otherwise
return text; impose a 10 MB maximum upstream response size

B) Forward an allowlist of upstream response headers; parse valid JSON and
otherwise return text; impose a 10 MB maximum upstream response size

C) Preserve upstream response headers and body without parsing; specify the
maximum upstream response size after `[Answer]:` below

X) Other (describe the header-forwarding policy, body representation, and
maximum response size after `[Answer]:` below)

[Answer]: A, with a 0.5 MB maximum upstream response size
