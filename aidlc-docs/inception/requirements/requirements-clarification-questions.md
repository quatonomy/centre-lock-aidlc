# Requirements Clarification Questions

Thank you. I recorded your answers and enabled the Security Baseline,
Resiliency Baseline, and Property-Based Testing extensions. The following
details remain open and are needed before the requirements can be finalized.
Accepted decisions remain in force; in particular, GitHub Actions and
Terraform remain the delivery standards, and tenant isolation and GDPR
compliance are constraints.

Fill in each `[Answer]:` tag. Include requested detail after your selected
letter, or choose the final "Other" option and describe your preference.

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

[Answer]:

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

[Answer]:

## Question 5: Change management
How should production changes be governed?

A) Use an existing organizational process; provide its name or tool

B) No formal process exists; propose a lightweight change record, approval,
and rollback-note process

C) The workload is exempt from formal change management; provide the rationale

X) Other (please describe after `[Answer]:` below)

[Answer]:

## Question 6: Production rollback
How should a failed production deployment be rolled back?

A) Redeploy the previous version-pinned infrastructure and application artifact

B) Switch traffic back using a blue/green deployment

C) Automatically roll back a canary deployment when health metrics regress

D) Require database-aware rollback; specify the migration/data reversal
expectations

E) Use an existing organizational rollback procedure; provide its reference

X) Other (please describe after `[Answer]:` below)

[Answer]:

## Question 7: Production deployment style
Which production deployment strategy is acceptable for the MVP?

A) Direct/in-place deployment

B) Rolling replacement

C) Blue/green deployment

D) Canary deployment with progressive traffic shift

X) Other (please describe after `[Answer]:` below)

[Answer]:

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

[Answer]:

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

[Answer]:

## Question 10: Incident response
How are production incidents handled?

A) Use an existing incident response process; provide its name or reference

B) No formal process exists; propose a lightweight incident response and
Correction of Errors (COE) process

X) Other (please describe after `[Answer]:` below)

[Answer]:
