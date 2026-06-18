# ADR-0001: Keep docs focused on current cross-repo boundaries

Status: Accepted

Date: 2026-06-18

## Context

Aithru is currently implemented across `aithru-platform`, `aithru-flowe`, and
`aithru-agent`. Earlier docs also described older or speculative owners such as
`aithru-core`, `aithru-web`, `aithru-workbench`, and
`aithru-personal-bridge`. Those documents drifted from the active codebase and
made the docs harder to trust.

Implementation repositories already contain package-level READMEs, API notes,
run commands, and design details. This repository should explain only the
cross-repository facts that those READMEs should not duplicate.

## Decision

Maintain `aithru-docs` as a small current-state boundary index for:

- active repository roles;
- dependency direction;
- platform/runtime/security contracts;
- shared frontend constraints;
- ADRs for decisions that affect more than one repository.

Do not keep a roadmap in this repository. Do not document a retired repository
name as a current owner. Do not copy package-level usage details out of the
implementation repositories.

## Consequences

Positive:

- The docs are easier to audit against the active repositories.
- Coding agents can read a short boundary set before editing code.
- Implementation READMEs remain the authority for package-local commands,
  endpoints, and examples.
- Retired designs are less likely to be mistaken for current work.

Tradeoffs:

- Long-term plans need another home when they are needed.
- Historical rationale is thinner because obsolete documents are removed instead
  of preserved as active guidance.

## Follow-up rules

- Any boundary change affecting multiple repositories should update
  `aithru-docs`.
- Any implementation change that only affects one repository should stay in that
  repository README/API docs.
- ADRs should be added only for decisions that are hard to reverse or affect
  dependency direction, security, runtime contracts, or shared frontend
  constraints.
