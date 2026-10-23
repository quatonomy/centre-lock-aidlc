# Operations

**Purpose**: Placeholder for future operational phases (deployment, monitoring, maintenance)

**Status**: This phase is currently a placeholder and will be expanded in future versions.

## Future Scope

The Operations phase will eventually include:
- Deployment planning and execution
- Monitoring and observability setup
- Incident response procedures
- Maintenance and support workflows
- Production readiness checklists

## Current State

All build and test activities have been moved to the CONSTRUCTION phase.
The AI-DLC workflow currently ends after the Build and Test phase in CONSTRUCTION.

## Completion: Review the Copilot Context Summary

Whenever an Operations stage is completed, review the context produced or
updated across Inception, Construction, and Operations against the concise
`centre-lock/SPEC.md`.

Treat detailed AI-DLC artifacts as authoritative and `centre-lock/SPEC.md` as
a small, implementation-oriented summary for future Copilot coding tasks.
Consider whether durable changes to product behavior, architecture,
implementation status, or engineering conventions are important enough to
add to the summary.

Do not copy detailed requirements, plans, operational procedures, or history
into the summary. Leaving it unchanged is valid and expected. Keep its length
stable for routine work; expand it only rarely, for example when a substantial
feature materially changes the context needed for future coding tasks. Never
present unresolved decisions or planned behavior as implemented facts.

Record that the review occurred and whether `centre-lock/SPEC.md` changed in
the Operations completion notes or `aidlc-docs/audit.md`. If changed, verify
that the content is accurate, concise, and consistent with the detailed
artifacts.
