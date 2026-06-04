# Aithru Core

`aithru-core` is the current deterministic workflow kernel and local runtime baseline.

Current implementation baseline: `v0.15.0-rc.0`.

## One-line definition

```txt
Aithru Core = portable WorkflowSpec + runtime contracts + local runtime adapter + node SDK + core nodes + tool contracts + trace/pause/redaction foundations
```

## Current package groups

| Package | Role |
| --- | --- |
| `@aithru/spec` | WorkflowSpec schema, validation, graph rules, DAG helpers. |
| `@aithru/runtime-core` | Framework-neutral runtime contracts, events, node contracts, pause/resume, trace, redaction, secrets, storage, and generic tool execution contracts. |
| `@aithru/runtime-local` | Local single-process runtime adapter. |
| `@aithru/node-sdk` | Helpers for authoring custom nodes. |
| `@aithru/nodes-core` | Primitive deterministic nodes: manual trigger, constant, transform, if. |
| `@aithru/nodes-human` | Human approval node. |
| `@aithru/nodes-http` | HTTP request node definition. |
| `@aithru/tool-http` | HTTP tool executor implementation for the HTTP node. |
| `@aithru/stdlib` | Official optional standard library for deterministic text/json helpers and nodes, plus basic file helper APIs. |

## Runtime shape

```txt
Workflow JSON
  -> validateWorkflowSpec
  -> validate graph
  -> topological scheduling
  -> node execution
  -> tool calls through ToolExecutor
  -> execution events
  -> optional trace persistence
  -> completed / paused / failed result
```

## Core owns

- WorkflowSpec and graph validation.
- Runtime contracts.
- NodeDefinition and node registry contracts.
- Execution events.
- Run and event persistence contracts.
- Trace replay helpers.
- Redaction policy helpers.
- Pause/resume and human approval contracts.
- Generic tool execution contracts.
- Local single-process runtime adapter.
- Minimal deterministic primitive nodes.
- HTTP node and executor as a small example integration.

## Core does not own

- Real Agent runtime.
- MCP SDK transports.
- UI framework.
- Server runtime.
- Database and queue adapters.
- LLM provider SDKs.
- Enterprise RBAC and audit implementation.
- Durable cross-process approval resume.

## Important boundaries

`@aithru/runtime-core` must stay framework-neutral. It should not import React, LangGraph, Temporal, MCP SDKs, LLM providers, databases, queues, or concrete persistence backends.

`@aithru/runtime-local` executes validated workflows and receives registries, policies, trace stores, and tool executors from the app layer. It should not hardcode concrete integrations.

Node packages may define behavior and call `NodeExecutionContext.callTool`, but concrete IO belongs behind tool executors or app-owned composition.

## Why Agent and MCP are outside core

Agent and MCP features were explored earlier, but they make the default core larger and less stable.

The v0.15 RC direction is:

```txt
Default core: stable, small, embeddable, deterministic.
Agent / MCP: future optional integration packages.
```

This keeps `aithru-core` useful for both personal and business editions without forcing every consumer to install Agent, MCP, or LLM dependencies.

## Verification baseline

Core changes should continue to pass the existing repository verification suite:

```bash
./scripts/verify.sh
```

The core README describes this suite as including typecheck, build, tests, package import smoke checks, CLI example runs, local runtime examples, custom node runner, custom tool executor, and human approval examples.
