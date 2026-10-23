# Requirements Verification Questions

Reverse engineering is approved. The existing product requirements and accepted
architecture decisions are treated as the baseline; these questions address
material gaps before requirements are consolidated. Open GitHub issues remain
future context and are not requirements unless you choose to include them.

Fill in each `[Answer]:` tag. For an option that needs more detail, include it
after the selected letter. Choose the final "Other" option and describe your
preference if none fits.

## Question 1: Requirements scope
What scope should this requirements baseline cover?

A) The full product vision in the detailed requirements record; leave implementation sequencing to Workflow Planning

B) Only the MVP; identify which product capabilities are included in the MVP

C) A specific near-term increment; identify its intended outcome and boundaries

X) Other (please describe after `[Answer]:` below)

[Answer]:b

## Question 2: User onboarding and roles
How should tenant users be invited, assigned to a tenancy, and given their initial role?

A) Tenant admins invite users and assign admin or view-only user roles

B) Platform operators provision tenant users and assign their roles

C) Support both tenant-admin invitations and platform-operator provisioning

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 3: Consumer tenant resolution
How should a consumer request identify its tenant while ensuring tenant scope is trusted?

A) Resolve the tenant from the tenant subdomain, then validate the presented tenant-scoped API key and its user

B) Resolve the tenant from the tenant-scoped API key; use the subdomain only as a consistency check

C) Require both the tenant subdomain and API key to identify the same tenant

X) Other (please describe after `[Answer]:` below)

[Answer]:c

## Question 4: Tenant integration settings
Which settings may a tenant modify on an integration it has selected?

A) Only its own upstream credentials and account-specific configuration

B) Credentials and tenant-local display or operational settings, but not catalog definitions or endpoint allow lists

C) The settings differ by integration; describe the allowed categories and who defines them

X) Other (please describe after `[Answer]:` below)

[Answer]:b

## Question 5: GitHub connector authorization
Which GitHub authorization method should the initial GitHub integration support?

A) Delegated OAuth authorization for each tenant

B) Tenant-provided GitHub token, stored as a tenant-specific secret

C) Both methods, with the required permissions and supported GitHub account types specified

D) Defer the method until the GitHub use cases and permission scopes are defined

X) Other (please describe after `[Answer]:` below)

[Answer]:d

## Question 6: Tenant secret storage
What should determine the mechanism for encrypting and retrieving tenant-specific upstream credentials?

A) Use a hosting-provider secret manager selected with the hosting provider

B) Use a provider-neutral secret-management service such as Vault

C) Keep the mechanism undecided until hosting and operational constraints are evaluated; preserve a provider-neutral application interface

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 7: Consumer proxy response contract
For the tenant-facing proxy API, how should responses represent upstream results and failures?

A) Return the upstream status as the platform HTTP status and pass through the upstream response body

B) Return the platform's mapped status and use the `connector` object for every upstream response, including upstream error responses

C) Use the `connector` object for completed upstream responses, with a separately defined platform error shape for transport failures and timeouts

X) Other (please describe after `[Answer]:` below)

[Answer]:b

If applicable, specify treatment of upstream headers, malformed or non-JSON bodies, response-size limits, and the HTTP status/body for timeouts and other failures. The Development proxy's already-specified 404/403/502/504 behavior remains distinct from the tenant-facing API unless you direct otherwise.

## Question 8: Per-request price
What is the intended charge for each billable API request, replacing the ambiguous “£0.1p” in the original product specification?

A) £0.10 per request (10 pence)

B) £0.01 per request (1 penny)

C) £0.001 per request (0.1 pence)

X) Other (please specify the currency and amount after `[Answer]:` below)

[Answer]:b

## Question 9: Plan pricing administration
Who may edit plan prices and user thresholds, and how should a change affect existing tenancies?

A) Platform operators edit plans; changes apply to existing tenancies immediately

B) Platform operators edit plans; changes apply only to new or renewing billing periods

C) Platform operators edit plans; each change specifies when and which tenancies it affects

X) Other (please describe after `[Answer]:` below)

[Answer]:c

## Question 10: Usage and billing presentation
What should tenant users be able to see about usage and charges in the initial product?

A) All tenant users can view request usage; only admins can view plan and charge details

B) All tenant users can view request usage and charges; only admins can manage billing

C) Usage and charges are both visible to all tenant users, with no billing-management workflow in the initial scope

X) Other (please describe after `[Answer]:` below)

[Answer]:a

If invoices, exports, or payment collection are in scope, describe the required behavior.

## Question 11: Request logs, auditability, and privacy
What retention and access rules should apply to minimal API request records and administrative audit events?

A) Define retention and access rules now; state the required periods and authorized roles

B) Keep only the minimum success/failure usage record needed for usage reporting; defer retention and broader auditability pending a privacy review

C) Include an auditable administrative event history as well as request usage; define event types, access, and retention

X) Other (please describe after `[Answer]:` below)

[Answer]: carry out B but with the guard rail that we must adhere to GDPR, this does not need implementation now potentially, but needs to be solved before we're in breach of it

## Question 12: Service and recovery targets
What reliability and operational targets should the requirements establish before production?

A) Set availability, recovery time (RTO), and recovery point (RPO) targets now; specify the targets

B) Define monitoring and alerting needs now, but defer numerical availability and recovery targets

C) Defer service targets and monitoring details until the hosting provider and operating model are evaluated

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 13: Security Baseline extension
Should security extension rules be enforced for this project?

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)

B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 14: Resiliency Baseline extension
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of **directional, design-time best practices**, derived from the **AWS Well-Architected Framework (Reliability Pillar)** and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability — covering 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does **not** make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a **starting point** that scaffolds good resiliency decisions early — it is not a substitute for a formal **AWS Well-Architected Review** of the built system.

Treat the output as a well-grounded **first draft** of your resiliency posture to build on and validate — not a finished, production-certified result.

A) Yes — apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live)

B) No — skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability)

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 15: Property-Based Testing extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)

B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)

C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)

X) Other (please describe after `[Answer]:` below)

[Answer]:a

## Question 16: Management identity providers
How should OpenID Connect identity providers beyond the required initial Google sign-in be added?

A) Google is the only provider initially; platform operators can configure additional providers later

B) Google is required initially, and additional named providers should also be supported at launch; list them

C) Tenant administrators may configure their own identity providers

X) Other (please describe after `[Answer]:` below)

[Answer]:a
