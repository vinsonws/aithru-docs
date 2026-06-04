# ADR-0001: Keep docs as the ecosystem source of truth

Status: Accepted

Date: 2026-06-04

## Context

Aithru is now split across multiple repositories. `aithru-core` already contains implementation-level README files and design notes. `aithru-web` contains a browser workflow editor. Future work is expected to add Agent, MCP, desktop, server, and UI modules.

Keeping all design decisions only inside implementation repositories would make cross-repo boundaries difficult to maintain.

## Decision

Create and maintain `aithru-docs` as the cross-repository source of truth for:

- product vision;
- ecosystem overview;
- module boundaries;
- roadmap;
- architecture decisions;
- cross-repo development workflow;
- prompts used by coding agents.

Implementation repositories still keep their own README files, package usage guides, and API-specific documentation.

## Consequences

Positive:

- Cross-repo design decisions have one stable home.
- Coding agents can read a smaller, purpose-built documentation hub before editing code.
- `aithru-core` can stay focused on implementation and package-local docs.
- Future modules can be added without losing architectural context.

Tradeoffs:

- Cross-repo changes need an extra documentation step.
- Docs can drift if implementation changes are not reflected here.

## Follow-up rules

- Any boundary change affecting multiple repositories should update `aithru-docs`.
- Any implementation change that only affects one package can stay in that package README/API docs.
- ADRs should be added for decisions that are hard to reverse or affect dependency direction.
