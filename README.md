# Aithru Docs

`aithru-docs` is the cross-repository architecture note set for the current
Aithru implementation.

It describes what is running now across:

- `aithru-platform`
- `aithru-flowe`
- `aithru-agent`

This repository intentionally does not keep a roadmap. Implementation progress,
milestones, and package-level API details belong in the implementation
repositories.

## Current Model

```txt
Aithru Platform = identity, organizations, grants, app hosting, tokens, audit, SDKs
Aithru Flowe    = Go workflow expression and local-first runtime kernel
Aithru Agent    = Python/FastAPI/Pydantic AI harness for tool-using intelligent work
```

Platform authorization rule:

```txt
Subsystems define permissions and roles.
Platform stores grants.
Subsystems enforce decisions through SDKs and platform authz APIs.
```

Runtime ownership rule:

```txt
Browser UI presents, configures, hosts, inspects, and approves.
Backends execute, persist, authorize, issue tokens, and audit.
```

## Start here

1. [Current State](./docs/00-current-state.md)
2. [Repository Boundaries](./docs/01-repository-boundaries.md)
3. [Runtime and Security Contracts](./docs/02-runtime-and-security-contracts.md)
4. [Frontend Constraints](./docs/03-frontend-constraints.md)
5. [Development Workflow](./docs/04-development-workflow.md)

Architecture decisions:

- [ADR-0001: Keep docs focused on current cross-repo boundaries](./adr/ADR-0001-docs-as-source-of-truth.md)

## Documentation rules

- Keep this repository small. Prefer one current-state document over multiple
  overlapping design notes.
- Do not document retired repository names as current owners.
- Do not add roadmaps here.
- Put install commands, API endpoint details, package usage, and local examples
  in the implementation repository that owns them.
- Update this repository when repository ownership, dependency direction,
  security boundaries, runtime contracts, or cross-frontend constraints change.
