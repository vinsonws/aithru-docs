# Aithru Agent

`aithru-agent` is the proposed optional Agent runtime and harness layer for the Aithru ecosystem.

It should be built on top of `aithru-core`, not inside it.

## One-line definition

```txt
Aithru Agent = optional Agent harness that uses core workflow, trace, approval, and tool contracts
```

## Core boundary

The most important boundary is:

```txt
aithru-core owns formal workflows.
aithru-agent owns intelligent execution inside bounded tasks or nodes.
```

`aithru-agent` must not become a second workflow system.

It can run an internal task plan, but that plan is not a `WorkflowSpec` and should not replace the core workflow graph.

## Relationship to core

```txt
aithru-agent depends on aithru-core.
aithru-core does not depend on aithru-agent.
```

`aithru-core` defines the shared world:

- WorkflowSpec
- NodeDefinition
- runtime contracts
- event contracts
- trace contracts
- pause/resume contracts
- redaction policy
- generic tool execution contracts
- tool permission contracts

`aithru-agent` runs agentic work inside those boundaries.

## Formal workflow vs agent plan

| Concept | Owner | Meaning |
| --- | --- | --- |
| `WorkflowSpec` | `aithru-core` | Reusable, user-editable, versioned workflow graph. |
| Workflow node/edge | `aithru-core` | Formal execution structure and branch semantics. |
| `agent.*` node | `aithru-agent` package, executed by core runtime | Intelligent behavior inside a workflow node. |
| `AgentTask` | `aithru-agent` | One intelligent task. |
| `AgentPlan` | `aithru-agent` | Dynamic, task-local plan for one agent run. |
| `AgentStep` | `aithru-agent` | Internal execution step inside one agent task. |

A saved workflow belongs to core. A temporary agent plan belongs to agent.

## Recommended first shape

The first useful shape is a plan/run/review harness:

```txt
Task
  -> planner
  -> executor
  -> tool calls through core contracts
  -> artifacts
  -> reviewer
  -> rerun when needed
```

This is an internal agent execution pattern, not a replacement for the formal workflow graph.

## Agent as a bounded workflow node

The safest integration model is to expose Agent behavior as one or more bounded workflow nodes.

Example node families:

| Node | Purpose |
| --- | --- |
| `agent.classify` | Make a short structured decision for downstream workflow branching. |
| `agent.plan` | Produce a structured task-local agent plan. |
| `agent.execute` | Execute bounded steps using approved tools. |
| `agent.review` | Review outputs and decide pass/fail or rerun. |
| `agent.deepResearch` | Run a bounded deep research task and produce structured artifacts. |
| `agent.task` | Run a compact end-to-end agent task for simple cases. |

These nodes should emit normal Aithru execution events and use core tool contracts.

## Core branching, agent judgment

Agent nodes may make intelligent judgments, but core nodes should own formal branch execution.

Recommended pattern:

```txt
agent.classify
  -> core.if
  -> branch A / branch B
```

For example, `agent.classify` can output:

```json
{
  "route": "deep_research",
  "confidence": 0.87,
  "reason": "The task requires multi-source analysis."
}
```

Then `core.if` or another deterministic workflow node performs the formal route selection.

## Deep research

Deep research belongs to `aithru-agent` as an intelligent execution pattern.

It may appear as:

1. a direct `agentRuntime.runTask(...)` call; or
2. a workflow node such as `agent.deepResearch` inside a formal core workflow.

The reusable workflow that contains a deep research node still belongs to `aithru-core`.

## Tool-use rules

Agent code must not directly bypass the runtime tool layer.

Required rule:

```txt
Agent tool use -> NodeExecutionContext.callTool -> ToolPermissionPolicy -> ToolExecutor -> trace events
```

This keeps Agent behavior compatible with:

- permission policy;
- approval gates;
- trace recording;
- redaction;
- replay/debugging;
- future desktop/server execution surfaces.

## Model adapter layer

`aithru-agent` should not bind itself to one provider.

Suggested adapter shape:

```ts
interface AgentModelAdapter {
  generate(input: AgentModelInput): AsyncIterable<AgentModelEvent>;
}
```

The event stream should allow future adapters to represent:

- text deltas;
- structured output;
- tool-call proposals;
- tool-call arguments;
- reasoning metadata when available;
- final result;
- model/provider errors.

Potential adapters:

- OpenAI-compatible APIs;
- Claude;
- DeepSeek;
- Qwen;
- local vLLM;
- test/scripted adapter.

## Runtime constraints

Every agent run should support bounded execution:

| Constraint | Purpose |
| --- | --- |
| `maxSteps` | Prevent unbounded loops. |
| `timeoutMs` | Bound wall-clock runtime. |
| `allowedTools` | Restrict tools by task or node. |
| `approvalPolicy` | Pause before risky work. |
| `redactionPolicy` | Protect persisted traces. |
| `budget` | Optional token/cost boundary. |

## State and trace

The Agent layer should not hide what happened inside a single opaque text blob.

It should record:

- task input;
- plan;
- step decisions;
- model events;
- tool-call proposals;
- tool-call results;
- artifacts;
- review result;
- rerun history.

When possible, these should map to Aithru core events or extension events that remain compatible with the trace viewer.

## Initial package proposal

```txt
aithru-agent/
  packages/
    agent-core/        Agent contracts, event types, run options
    agent-runtime/     plan/run/review runtime
    agent-node/        workflow node definitions
    model-openai/      optional provider adapter
    model-test/        deterministic scripted/test adapter
  examples/
    plan-run-review.ts
    agent-node-workflow.json
```

For the first version, prefer a small deterministic test adapter plus one real provider adapter.

## Non-goals for the first version

- No visual workflow builder.
- No formal `WorkflowSpec` replacement.
- No workflow scheduler replacement.
- No server worker pool.
- No durable distributed queue.
- No MCP transport management unless delegated to a separate MCP package.
- No direct filesystem, shell, browser, or GitHub access outside the core tool permission system.
