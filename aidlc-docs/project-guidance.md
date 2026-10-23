# Project context guide

- `centre-lock/SPEC.md` is the concise, implementation-focused product and
  codebase summary intended for routine coding tasks.
- Detailed requirements and unresolved questions are in
  `aidlc-docs/inception/requirements/`. Do not treat examples or open GitHub
  issues as accepted requirements.
- Accepted architecture decisions are in `aidlc-docs/architecture/decisions/`;
  architecture boundaries are in `aidlc-docs/architecture/overview.md`.
- Engineering standards are in `aidlc-docs/engineering-standards/`. Consult
  the relevant coding, testing, security, and CI guidance for changes.
- The current implementation and its gaps are described in
  `aidlc-docs/inception/reverse-engineering/`. Confirm details in source before
  changing behavior.
- Keep changes focused, preserve existing behavior unless requirements say
  otherwise, add tests at affected boundaries, and do not invent answers to
  unresolved security, tenant, API-contract, or operational decisions.
