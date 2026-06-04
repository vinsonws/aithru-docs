# ADR-0002: Keep core, web, and agent boundaries explicit

Status: Accepted

Date: 2026-06-04

## Context

`aithru-core` has reached a v0.15.0-rc.0 baseline as a deterministic workflow kernel. It includes workflow validation, runtime contracts, a local runtime adapter, node SDK, trace, redaction, pause/resume, generic tool execution contracts, and a small set of node/tool packages.

`aithru-web` is a browser-safe workflow authoring and inspection surface. It imports browser-safe package entry points and deliberately avoids Node-only runtime packages.

The next proposed major module is `aithru-agent`, a full Agent harness/runtime.

## Decision

Keep the dependency direction explicit:

```txt
aithru-web   -> aithru-core
aithru-agent -> aithru-core
aithru-core  -> no web or agent dependency
```

`aithru-core` defines shared contracts. `aithru-web` edits and validates workflows. `aithru-agent` runs bounded agentic work on top of the core contracts.

Agent/MCP functionality must remain optional integration work, not part of the default core package surface.

## Consequences

Positive:

- `aithru-core` stays small, stable, and embeddable.
- Browser UI can stay browser-safe.
- Agent work can evolve quickly without destabilizing the workflow kernel.
- Desktop and server products can compose the pieces they need.

Tradeoffs:

- Some shared Agent event concepts may later need careful extension design.
- The Agent package must do more integration work instead of relying on hidden core behavior.

## Boundary rules

- `aithru-core` must not depend on UI frameworks, LLM provider SDKs, MCP SDKs, durable queues, product database adapters, or product shells.
- `aithru-web` must not import Node-only runtime packages into the browser bundle.
- `aithru-agent` must route tool use through core tool contracts and permission policies.
