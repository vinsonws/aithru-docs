# Ecosystem Overview

Aithru is organized as a small deterministic core plus horizontal modules.

The important dependency rule is:

```txt
Product shells and integrations depend on core.
Core does not depend on product shells or integrations.
```

## Current repositories

| Repository | Status | Role |
| --- | --- | --- |
| `aithru-core` | v0.15.0-rc.0 baseline | Deterministic workflow kernel, local runtime adapter, node SDK, node packages, tool executor contracts, trace, redaction, pause/resume, and HTTP example executor. |
| `aithru-web` | v0.1/v0.2 local dev surface | Browser-safe WorkflowSpec editor, node palette, validation panel, graph preview, draft persistence, and simulated run panel. |
| `aithru-docs` | design hub | Cross-repository architecture, ADRs, specs, and roadmap. |

## Future modules

| Module | Role | Depends on |
| --- | --- | --- |
| `aithru-agent` | Optional Agent runtime and harness layer. | `aithru-core`; optional model, MCP, and tool packages. |
| `aithru-mcp` | Optional MCP integration package. | `aithru-core`; optional MCP SDK/transports. |
| `aithru-ui` | Reusable workflow designer and trace viewer library. | `aithru-core` browser-safe types and metadata. |
| `aithru-desktop` | Personal Edition local-first product shell. | core, UI, stdlib, optional Agent/MCP packages. |
| `aithru-server` | Business/Enterprise product shell. | core, UI, stdlib, optional Agent/MCP packages, database and worker infrastructure. |

## Dependency direction

```txt
aithru-core
  <- aithru-web
  <- aithru-agent
  <- aithru-mcp
  <- aithru-ui
  <- aithru-desktop
  <- aithru-server
```

`aithru-web` currently imports only browser-safe package entry points from the core workspace. It intentionally avoids `@aithru/runtime-local`, `@aithru/tool-http`, and file-system helpers.

`aithru-agent` should be designed as an optional runtime/harness layer. It should reuse core contracts instead of introducing a parallel execution model.

## Runtime ownership

| Capability | Owner |
| --- | --- |
| `WorkflowSpec` schema and validation | `aithru-core` / `@aithru/spec` |
| Node and runtime contracts | `aithru-core` / `@aithru/runtime-core` |
| Local single-process DAG execution | `aithru-core` / `@aithru/runtime-local` |
| Browser editing and validation | `aithru-web` today; future shared `aithru-ui` |
| Real Agent loop and model adapters | future `aithru-agent` |
| MCP transport and tool integration | future `aithru-mcp` |
| Desktop-local execution bridge | future `aithru-desktop` |
| Durable workers and team execution | future `aithru-server` |

## Composition rule

Apps decide what to register:

- node packages;
- tool executors;
- stdlib nodes;
- trace stores;
- redaction policy;
- approval policy;
- optional Agent or MCP integrations.

This keeps the core portable while still allowing rich products to be built on top.
