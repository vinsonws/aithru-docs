# Current State

This document is the current-state index for Aithru. It replaces older planning
documents that described `aithru-core`, `aithru-web`, `aithru-workbench`, or
`aithru-personal-bridge` as active system owners.

## Active Repositories

| Repository | Current role | Implementation shape |
| --- | --- | --- |
| `aithru-platform` | Platform foundation and product shell. Owns identity, organizations, app registry, app/context grants, token issuing, delegated authorization, service policy, audit, portal app list, hosted app shell, Admin Center, subsystem SDKs, examples, and mock host. | Java 21/Spring Boot backend; React/TypeScript/Vite frontend; Java, TypeScript, Go, Python server SDKs; browser hosted-app SDK. |
| `aithru-flowe` | Workflow expression and execution kernel. Owns the `aithru.flow/v1alpha1` spec, validation, node registry, DAG runner, trace/debug tooling, local artifacts, secret references, server runtime, server-served console, local queue/resume, workflow registry, and webhooks. | Go CLI and HTTP server. Local-first persistence under `.flowe`; no production visual editor or database runtime. |
| `aithru-agent` | AI harness backend for long-running, tool-using, permission-aware intelligent work. Owns Agent threads, messages, runs, todos, skills, tools, workspace files, artifacts, memory, approvals, subagents, event streams, trace projection, workers, and Pydantic AI-backed execution. | Python 3.12/FastAPI backend with in-memory and SQLite stores, SSE streams, worker CLI, and Pydantic AI harness path. |
| `aithru-docs` | Cross-repository boundary notes. Owns concise current-state docs and ADRs for decisions that affect more than one repository. | Markdown only. Not a roadmap and not a replacement for implementation READMEs. |

## Retired Names

These names should not appear as current implementation owners in this docs
repository:

- `aithru-core`
- `aithru-web`
- `aithru-workbench`
- `aithru-personal-bridge`

If a future repository is reintroduced, add it here only after it is active
again and has its own implementation README.

## Current Product Shape

```txt
Platform
  -> authenticates users
  -> holds organization context
  -> stores app/context grants
  -> issues user, hosted, service, exchanged, and delegated tokens
  -> hosts daily apps and Admin Center
  -> provides subsystem SDK contracts

Flowe
  -> validates workflow specs
  -> executes local-first workflow runs
  -> records trace, packets, artifacts, and run state
  -> exposes CLI and a thin HTTP/server console

Agent
  -> accepts threads, messages, skills, and run requests
  -> routes model-proposed actions through Aithru tools
  -> persists run events, snapshots, artifacts, workspace, approvals, and memory
  -> may later be surfaced as a Platform hosted app
```

## Non-Goals For This Docs Repository

- No roadmap.
- No phase-completion history.
- No package-level install guide.
- No endpoint inventory unless the endpoint is part of a cross-repository
  contract.
- No speculative module document for a repository that is not currently active.
