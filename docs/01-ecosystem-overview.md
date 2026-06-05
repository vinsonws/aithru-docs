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
| `aithru-web` | browser-safe editor / bridge client surface | WorkflowSpec editor, node palette, validation panel, graph editing, draft persistence, Bridge Mock, HTTP Bridge Client, and trace viewer. |
| `aithru-personal-bridge` | planned personal execution host | Trusted personal execution host for Aithru-Web; runs LocalRuntime on localhost or a private personal server such as a Mac mini, DGX-like workstation, NAS, home server, or private small server. |
| `aithru-docs` | design hub | Cross-repository architecture, ADRs, specs, and roadmap. |

## Future modules

| Module | Role | Depends on |
| --- | --- | --- |
| `aithru-agent` | Optional Agent runtime and harness layer. | `aithru-core`; optional model, MCP, and tool packages. |
| `aithru-mcp` | Optional MCP integration package. | `aithru-core`; optional MCP SDK/transports. |
| `aithru-ui` | Reusable workflow designer and trace viewer library. | `aithru-core` browser-safe types and metadata. |
| `aithru-desktop` | Personal Edition local-first product shell; may embed, manage, package, or later replace `aithru-personal-bridge`. | core, UI, stdlib, personal bridge, optional Agent/MCP packages. |
| `aithru-server` | Business/Enterprise product shell. | core, UI, stdlib, optional Agent/MCP packages, database and worker infrastructure. |

## Deployment modes

| Mode | Execution host | Notes |
| --- | --- | --- |
| Browser-only demo | `aithru-web` Bridge Mock | Simulated, in-memory, not real execution. |
| Personal loopback | `aithru-personal-bridge` on `127.0.0.1` | Browser and execution bridge are usually on the same machine. |
| Personal server | `aithru-personal-bridge` on Mac mini, DGX-like workstation, NAS, or private server | Browser may be remote; access should use LAN, Tailscale/VPN, or HTTPS reverse proxy plus token auth. |
| Business/Enterprise server | future `aithru-server` | Multi-user, durable, RBAC, audit, worker pool, persistence. |

## Dependency direction

```txt
aithru-core
  <- aithru-web
  <- aithru-personal-bridge
  <- aithru-agent
  <- aithru-mcp
  <- aithru-ui
  <- aithru-desktop
  <- aithru-server
```

`aithru-web` currently imports only browser-safe package entry points from the core workspace. It intentionally avoids `@aithru/runtime-local`, `@aithru/tool-http`, and file-system helpers.

`aithru-personal-bridge` is allowed to import Node/runtime-side packages such as `@aithru/runtime-local`, `@aithru/tool-http`, `@aithru/stdlib/file`, tool executors, trace stores, and file/artifact helpers because it runs in a trusted personal execution host rather than in the browser.

`aithru-agent` should be designed as an optional runtime/harness layer. It should reuse core contracts instead of introducing a parallel execution model.

## Runtime ownership

| Capability | Owner |
| --- | --- |
| `WorkflowSpec` schema and validation | `aithru-core` / `@aithru/spec` |
| Node and runtime contracts | `aithru-core` / `@aithru/runtime-core` |
| Local single-process DAG execution | `aithru-core` / `@aithru/runtime-local` |
| Browser editing and validation | `aithru-web` today; future shared `aithru-ui` |
| Personal/local execution host | `aithru-personal-bridge` |
| Personal trace/artifact storage | `aithru-personal-bridge` |
| Desktop-local execution bridge and product shell | future `aithru-desktop`, possibly embedding, managing, packaging, or later replacing `aithru-personal-bridge` |
| Real Agent loop and model adapters | future `aithru-agent` |
| MCP transport and tool integration | future `aithru-mcp` |
| Durable team execution | future `aithru-server` |

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
