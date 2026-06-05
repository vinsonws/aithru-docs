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

For the web/execution split, use this rule:

```txt
aithru-web owns browser authoring and inspection.
aithru-personal-bridge or aithru-server owns real execution.
```

## Module matrix

| Module | Owns | Does not own |
| --- | --- | --- |
| `aithru-core` | WorkflowSpec, formal workflow graph, node/edge semantics, runtime contracts, local runtime adapter, node SDK, trace, redaction, pause/resume, generic tool contracts, primitive nodes, human approval, HTTP example node/executor. | Agent runtime, agent internal planning, model adapters, MCP SDK/transports, product UI, server runtime, database/queue drivers, LLM providers, enterprise RBAC. |
| `@aithru/stdlib` | Optional deterministic text/json helpers and nodes, plus basic file helper APIs without sandbox policy. | Runtime kernel, default CLI registration, OAuth-heavy connectors, UI, Agent, MCP, server features. |
| `aithru-web` | Browser WorkflowSpec authoring, validation, graph editing, node palette, import/export, browser draft state, Bridge Mock, HTTP Bridge Client, trace viewing, simulated run display. | Real workflow execution, Node-only APIs, durable approval state, tool execution, backend persistence. |
| `aithru-personal-bridge` | Personal execution host, HTTP Bridge API, LocalRuntime composition, local/personal-server run records, local trace/artifact storage, local secrets/config, approval resume/rejection, cancellation, tool executor registration, localhost and private personal-server deployment. | Browser UI, WorkflowSpec/core contracts, enterprise RBAC, team workspaces, durable worker pool, public multi-tenant API, Agent/MCP implementation. |
| `aithru-agent` | Agent contracts, `AgentRuntime`, `ClassifyEngine`, `PlanRunReviewEngine`, `DeepResearchEngine`, task-local `AgentPlan`, model adapters, AgentTraceEvent taxonomy, and `@aithru/node-agent` workflow nodes. | WorkflowSpec, formal workflow graph, core scheduler, branch semantics, workflow persistence format, tool permission policy ownership, UI product shell, server/desktop product ownership, built-in retrieval/tool packages. |
| `@aithru/node-agent` | Workflow `NodeDefinition` factories for `agent.classify`, `agent.task`, and `agent.deepResearch`; model resolution injection; bridge from `AgentHost.callTool` to core `ctx.callTool`; AgentTraceEvent emission. | Agent runtime engine internals, formal workflow execution, provider-specific model ownership, core tool policy ownership. |
| `aithru-mcp` | Future MCP client adapters, MCP tool executors, MCP nodes, transport-specific integration code. | Core contracts, Agent reasoning, UI product shell. |
| `aithru-ui` | Reusable workflow designer, schema forms, trace viewer, approval panels. | Real runtime execution, credential storage, product-specific persistence. |
| `aithru-desktop` | Personal Edition local-first product shell, packaging, local project UX, local credentials, and possibly embedding or managing `aithru-personal-bridge`. | Enterprise RBAC, team audit model, server worker pool. |
| `aithru-server` | Business/Enterprise API, workers, persistence, RBAC, audit, workspace model, deployment, durable remote execution. | Core kernel, desktop-only assumptions, UI internals. |

## Feature ownership rules

| Question | Default answer |
| --- | --- |
| Should a new feature enter `aithru-core`? | Usually no. Only shared contracts and deterministic kernel primitives belong there. |
| Where should `WorkflowSpec` changes go? | `aithru-core`, but only when the formal workflow contract truly needs to change. |
| Where should formal workflow graph execution go? | `aithru-core` runtime adapters. |
| Where should a deterministic helper node go? | `@aithru/stdlib` unless it is a minimal core primitive. |
| Where should personal workflow execution bridge go? | `aithru-personal-bridge`. |
| Where should Mac mini / DGX / NAS / private personal-server execution go? | `aithru-personal-bridge` in personal-server mode. |
| Where should real desktop product packaging go? | `aithru-desktop`; it may embed or manage `aithru-personal-bridge`. |
| Where should real Agent behavior go? | `aithru-agent`. |
| Where should Agent internal planning go? | `aithru-agent`; it must not become a replacement WorkflowSpec. |
| Where should Deep Research execution go? | `aithru-agent`; it may be exposed as `agent.deepResearch`, but the reusable workflow still belongs to core. |
| Where should short intelligent routing go? | `aithru-agent` outputs a structured judgment; `aithru-core` nodes perform formal branching. |
| Where should model provider adapters for Agent go? | `aithru-agent` packages such as `@aithru/agent-model-openai-compatible`, behind `AgentModelAdapter`. |
| Where should MCP behavior go? | Future `aithru-mcp`, not Agent or Core by default. |
| Where should browser visual editing go? | `aithru-web` today, future shared `aithru-ui`. |
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
8. Is it real workflow execution for a personal/private host? If yes, it belongs to `aithru-personal-bridge`.
9. Is it multi-user durable remote execution with RBAC/audit/workspaces? If yes, it belongs to `aithru-server`.
