# User Story Generation Plan

## Current scope amendment (2026-10-26)

The original story generation captured an authenticated tenant-management
and consumer-proxy MVP. That scope is superseded: all authentication and
sign-in are deferred. US-04 is public catalog discovery and US-08 is narrowed
to tenant-subdomain routing only; see the current `stories.md`.

## Purpose and scope

Create user-centered stories for the approved MVP requirements in
`aidlc-docs/inception/requirements/requirements.md`. Stories cover tenant
sign-in and membership/roles, tenant integration configuration, and the
tenant-facing consumer proxy. Do not turn deferred billing, payment
collection, custom domains, or GitHub connector functionality into MVP
stories. Keep Development-only routes distinct from tenant-facing behavior.

## Story approach options

- **User journey-based (recommended):** Follows an actor's end-to-end flow and
  makes cross-screen and API outcomes visible. Good for acceptance testing;
  requires careful slicing to keep each story small.
- **Feature-based:** Groups by capability such as sign-in, integration
  management, and proxy. Easy to map to system capabilities; can obscure
  end-to-end journeys across features.
- **Persona-based:** Groups all stories by actor. Highlights different
  permissions; may duplicate shared workflows.
- **Domain-based:** Groups by identity, tenancy, catalog, and proxy domains.
  Aligns to architecture; can drift toward implementation components instead
  of user value.
- **Epic-based:** Establishes broad outcomes with smaller stories beneath.
  Useful for navigation and scope boundaries; epics themselves are not
  implementation-sized.
- **Hybrid:** Journey-based stories grouped under feature epics, with persona
  labels for ownership. Balances user flow and scope traceability.

## Questions for story generation

Please fill each `[Answer]:` tag. A recommended default is provided where
appropriate; you may select another option or describe a custom preference.

### Question 1: Story organization
Which organization approach should be used?

A) Hybrid: user-journey stories grouped under feature epics, with persona
labels (recommended)

B) User-journey-based without feature epics

C) Feature-based

D) Persona-based

E) Domain-based

X) Other (describe the approach after `[Answer]:` below)

[Answer]: A) Hybrid: user-journey stories grouped under feature epics, with persona labels

### Question 2: Acceptance-criteria format
Which acceptance-criteria format should stories use?

A) Given/When/Then scenarios for each criterion (recommended; explicit and
directly testable)

B) Concise observable-outcome bullets

C) Both: Given/When/Then for primary and security-sensitive paths, bullets
for simple behavior

X) Other (describe the format after `[Answer]:` below)

[Answer]: A) Given/When/Then scenarios for each criterion

### Question 3: Story granularity
What is the desired story size?

A) Small, independently verifiable user outcomes, splitting workflows into
separate stories where needed (recommended)

B) Larger end-to-end workflows, even if they span multiple capabilities

C) Other (describe the preferred size and split rule after `[Answer]:` below)

[Answer]: A) Small, independently verifiable user outcomes, splitting workflows into separate stories where needed

## Generation checklist

- [x] Analyze all answers for ambiguity and resolve any material gaps.
- [x] Obtain explicit approval of this story-generation approach.
- [x] Create `aidlc-docs/inception/user-stories/personas.md` for the MVP actors.
- [x] Create `aidlc-docs/inception/user-stories/stories.md` with stories that
  satisfy INVEST as appropriate.
- [x] Add testable acceptance criteria to each story.
- [x] Map each story to its persona and approved requirement(s).
- [x] Exclude deferred capabilities and implementation-only tasks.
- [x] Review story coverage against the requirements and identify any
  requirement with no story or acceptance criterion.
- [x] Present stories and personas for user review and approval.

## Planned artifacts

- `aidlc-docs/inception/user-stories/personas.md`
- `aidlc-docs/inception/user-stories/stories.md`

## Approval

Story approach approval is required after the questions above are answered
and before story generation begins. Generated stories and personas require a
separate user approval before the next AI-DLC stage.

**Status:** Story and persona artifacts are generated and awaiting user
review and approval. Design-stage decisions and production-readiness gates
are called out in the stories rather than inferred here.
