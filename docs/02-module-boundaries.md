# Module Boundaries

Aithru grows horizontally. New capabilities should become optional modules unless they are truly shared contracts required by every product and integration.

## Boundary principle

```txt
Core defines contracts.
Optional modules implement capabilities.
Product shells compose modules.
```

For the core/agent split, use the stricter rule:

```txt
aithru-core owns formal workflows.
aithru-agent owns intelligent execution inside bounded tasks or nodes.
```

## Module matrix

| Module | Owns | Does not own |
| --- | --- | --- |
| `aithru-core` | WorkflowSpec, formal workflow graph, node/edge semantics, runtime contracts, local runtime adapter, node SDK, trace, redaction, pause/resume, generic tool contracts, primitive nodes, human approval, HTTP example node/executor. | Real Agent runtime, agent internal planning, model adapters, MCP SDK/transports, product UI, server runtime, database/queue drivers, LLM providers, enterprise RBAC. |
| `@aithru/stdlib` | Optional deterministic text/json helpers and nodes, plus basic file helper APIs without sandbox policy. | Runtime kernel, default CLI registration, OAuth-heavy connectors, UI, Agent, MCP, server features. |
| `aithru-web` | Browser WorkflowSpec authoring, validation, graph editing, node palette, import/export, browser draft state, simulated run display. | Real workflow execution, Node-only APIs, durable approval state, tool execution, backend persistence. |
| `aithru-agent` | Agent harness, model adapters, bounded Agent execution, task-local AgentPlan, deep research, classification/judgment, plan/run/review flow, agent trace integration, optional tool-use engines. | WorkflowSpec, formal workflow graph, core scheduler, branch semantics, workflow persistence format, tool permission policy ownership, UI product shell, server/desktop product ownership. |
| `aithru-mcp` | MCP client adapters, MCP tool executors, MCP nodes, transport-specific integration code. | Core contracts, Agent reasoning, UI product shell. |
| `aithru-ui` | Reusable workflow designer, schema forms, trace viewer, approval panels. | Real runtime execution, credential storage, product-specific persistence. |
| `aithru-desktop` | Personal Edition app shell, local composition, local traces, local credentials, local bridge. | Enterprise RBAC, team audit model, server worker pool. |
| `aithru-server` | Business/Enterprise API, workers, persistence, RBAC, audit, workspace model, deployment. | Core kernel, desktop-only assumptions, UI internals. |

## Feature ownership rules

| Question | Default answer |
| --- | --- |
| Should a new feature enter `aithru-core`? | Usually no. Only shared contracts and deterministic kernel primitives belong there. |
| Where should `WorkflowSpec` changes go? | `aithru-core`, but only when the formal workflow contract truly needs to change. |
| Where should formal workflow graph execution go? | `aithru-core` runtime adapters. |
| Where should a deterministic helper node go? | `@aithru/stdlib` unless it is a minimal core primitive. |
| Where should real Agent behavior go? | `aithru-agent`. |
| Where should Agent internal planning go? | `aithru-agent`; it must not become a replacement WorkflowSpec. |
| Where should deep research go? | `aithru-agent`, usually exposed as an `agent.deepResearch` workflow node. |
| Where should short intelligent routing go? | `aithru-agent` outputs a structured judgment; `aithru-core` nodes perform formal branching. |
| Where should MCP behavior go? | `aithru-mcp`. |
| Where should browser visual editing go? | `aithru-web` today, future shared `aithru-ui`. |
| Where should real desktop execution bridge go? | `aithru-desktop`. |
| Where should durable team execution go? | `aithru-server`. |
| Where should enterprise RBAC/audit go? | `aithru-server`. |

## Review checklist

Before adding a package or feature, answer:

1. Is this a shared contract or a concrete integration?
2. Does it need to be available in every product surface?
3. Does it introduce UI, database, queue, model provider, transport, or product-shell dependencies?
4. Can it be registered explicitly by an app instead of built into the core?
5. Does it respect trace, redaction, approval, and tool permission contracts?
6. Is it a reusable, user-editable, versioned workflow graph? If yes, it belongs to core.
7. Is it LLM-driven planning, judgment, research, writing, review, or rerun logic inside a bounded task? If yes, it belongs to agent.
