# Aithru Agent

`aithru-agent` is the intelligent execution layer of the Aithru ecosystem.

It is built on top of `aithru-core` public contracts where workflow integration is needed, but it must not become a second workflow system.

## One-line definition

```txt
Aithru Agent = intelligent execution inside bounded tasks or workflow nodes
```

## Core boundary

```txt
aithru-core owns formal workflows.
aithru-agent owns intelligent execution inside bounded tasks or nodes.
```

`aithru-agent` does not own `WorkflowSpec`, workflow graph semantics, branch semantics, workflow persistence, or core scheduling.

It can run an internal task-local `AgentPlan`, but that plan is not a `WorkflowSpec` and should not replace the core workflow graph.

## Current implemented baseline

| Package | Implemented role |
| --- | --- |
| `@aithru/agent-core` | Agent contracts, `AgentTask`, `AgentPlan`, `AgentEvent`, `AgentModelAdapter`, `AgentHost`, research report types, and `AgentTraceEvent` taxonomy. |
| `@aithru/agent-runtime` | `ClassifyEngine`, `PlanRunReviewEngine`, `DeepResearchEngine`, `AgentRuntime.run(...)`, `AgentRuntime.runTask(...)`, event streaming, and failure semantics. |
| `@aithru/agent-model-test` | Deterministic scripted model adapter and static structured/final helpers for tests and examples. |
| `@aithru/agent-model-openai-compatible` | OpenAI-compatible HTTP model adapter implemented with `fetch`, without provider SDKs. It emits model events and does not execute tools. |
| `@aithru/node-agent` | Workflow `NodeDefinition` factories for `agent.classify`, `agent.task`, and `agent.deepResearch`; bridges Agent tool calls to core `ctx.callTool`. |

Implemented examples include standalone classify, standalone plan/run/review, standalone deep research, direct node-agent execution, LocalRuntime workflow execution with `agent.classify`, LocalRuntime workflow execution with `agent.deepResearch`, and an opt-in OpenAI-compatible classify example that safely skips when env vars are missing.

## Formal workflow vs agent plan

| Concept | Owner | Meaning |
| --- | --- | --- |
| `WorkflowSpec` | `aithru-core` | Reusable, user-editable, versioned workflow graph. |
| Workflow node/edge | `aithru-core` | Formal execution structure and branch semantics. |
| `agent.*` node | `@aithru/node-agent`, executed by core runtime | Intelligent behavior inside a workflow node. |
| `AgentTask` | `aithru-agent` | One intelligent task. |
| `AgentPlan` | `aithru-agent` | Dynamic, task-local plan for one agent run. |

A saved workflow belongs to core. A temporary agent plan belongs to agent.

## Runtime engines

`@aithru/agent-runtime` currently provides three default engines:

| Engine | Purpose |
| --- | --- |
| `classify` | Short structured judgment, such as route/confidence/reason. |
| `plan-run-review` | Bounded task execution with plan, steps, optional tool calls, artifacts, and review. |
| `deep-research` | Bounded Deep Research V0 with task-local research planning, host-owned tool calls, structured findings/sources, report artifact, and optional review. |

All engines emit `AgentEvent` values. `AgentRuntime.run(...)` returns the full event stream; every event is also sent to `AgentHost.emit(event)` in the same order. `AgentRuntime.runTask(...)` consumes the stream and returns the final `AgentTaskOutput`, or throws `AgentTaskFailedError` if the stream contains `agent.task.failed`.

## Deep Research V0

Deep Research V0 belongs to `aithru-agent` as an intelligent execution pattern.

It can be used in two ways:

1. As a direct runtime task through `AgentRuntime.runTask("deep-research", ...)`.
2. As a formal workflow node through `agent.deepResearch` inside a core `WorkflowSpec`.

Deep Research V0 is bounded by normal run options such as `maxSteps` and `timeoutMs`, plus research-specific options such as `maxSources` and `maxSearchQueries`.

It produces `AgentResearchReport` data with sources, findings, limitations, metadata, and a `report` artifact.

It does not include real retrieval connectors or product UI by default. Hosts may provide fake local tools, provider-backed tools, or core workflow tool bridges, but the engine itself never executes tools directly.

## Agent as bounded workflow nodes

`@aithru/node-agent` exposes current Agent engines as workflow nodes:

| Node | Runtime engine | Purpose |
| --- | --- | --- |
| `agent.classify` | `classify` | Make a short structured decision for downstream deterministic branching. |
| `agent.task` | `plan-run-review` by default, configurable engine | Run a compact end-to-end intelligent task. |
| `agent.deepResearch` | `deep-research` | Run bounded Deep Research V0 and return report artifacts/metadata. |

These nodes are formal workflow nodes, but they do not own workflow semantics. Aithru Core still owns graph execution, scheduling, branch semantics, workflow validation, and workflow persistence.

## Tool-use rule

Agent code must not bypass the runtime tool layer.

```txt
Agent model event
  -> AgentRuntime
  -> AgentHost.callTool
  -> @aithru/node-agent bridge
  -> NodeExecutionContext.callTool
  -> ToolPermissionPolicy
  -> ToolExecutor
  -> trace events
```

This keeps Agent behavior compatible with permission policy, approval gates, trace recording, redaction, replay/debugging, and future personal/server execution surfaces.

## Model adapters

`aithru-agent` is provider-neutral. It depends on `AgentModelAdapter`, not provider-specific runtime concepts.

Implemented adapters:

| Package | Role |
| --- | --- |
| `@aithru/agent-model-test` | Deterministic scripted adapter for tests/examples. |
| `@aithru/agent-model-openai-compatible` | OpenAI-compatible HTTP adapter for OpenAI-style `/chat/completions` endpoints. |

`@aithru/agent-model-openai-compatible` does not execute tools. It may emit `tool_call.proposed`; actual tool execution remains the responsibility of `AgentRuntime` and `AgentHost.callTool`.

## Agent trace events

`AgentEvent` is the runtime event shape emitted by Agent engines.

`AgentTraceEvent` is an additive, provider-neutral trace-consumption view for core integrations and future UI trace viewers. It groups events with stable `kind` and `phase` fields while preserving the full original `AgentEvent` in `payload`.

`@aithru/node-agent` currently emits core `log.info` events with `AgentTraceEvent` payload and metadata:

- `agentEventType`;
- `agentTraceKind`;
- `agentTracePhase`.

This avoids requiring Aithru Core to support dynamic `agent.*` core event types while still giving future trace viewers stable filter fields.

## Non-goals / not implemented

Current `aithru-agent` does not implement MCP integration, built-in retrieval/tool packages, durable persistence, long-term memory, UI/chat, server workers, LangGraph adapter, or OpenAI Agents SDK adapter.

Those should remain optional modules or future packages rather than entering the core Agent runtime by default.
