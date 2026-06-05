# Runtime, Trace, Approval, Tooling

This document summarizes the execution model shared by current `aithru-core`, `aithru-agent`, `aithru-personal-bridge`, and future optional modules.

## Runtime model

The current core runtime is deterministic DAG execution.

```txt
WorkflowSpec
  -> schema validation
  -> graph validation
  -> topological scheduling
  -> node execution
  -> branch decisions
  -> terminal outputs
  -> completed / paused / failed result
```

`@aithru/runtime-local` is the current local single-process runtime adapter. It schedules validated workflows and delegates concrete actions to injected tool executors.

`aithru-personal-bridge` owns the first cross-process personal execution host around this runtime. It can run on the same machine as the browser or on a private personal server, but it is still personal/single-user by default. Durable multi-user workers belong to future `aithru-server`.

`@aithru/agent-runtime` is not a workflow runtime. It runs bounded Agent tasks through engines such as `classify`, `plan-run-review`, and `deep-research`. Those engines can be called standalone or wrapped as workflow nodes through `@aithru/node-agent`.

## Event and trace model

Aithru runs should be observable through structured events.

Current core concepts include:

- execution events;
- run records;
- event stores;
- trace stores;
- local file traces;
- trace replay helpers;
- redaction policy.

The local file trace shape is:

```txt
.aithru/
  runs/
    <runId>/
      run.json
      events.jsonl
```

`aithru-personal-bridge` may persist personal run records, local traces, and local artifacts using this shape or a future local storage adapter. `aithru-web` consumes trace data through the Bridge API and must treat traces as read-only display data.

## Agent trace integration

`aithru-agent` emits runtime-level `AgentEvent` values from Agent engines.

`@aithru/agent-core` also defines `AgentTraceEvent`, a provider-neutral trace-consumption view with stable fields:

- `kind`;
- `phase`;
- `agentEventType`;
- optional task, step, tool, artifact, error, and summary fields;
- `payload`, which preserves the original `AgentEvent`.

`agentTraceEventFromAgentEvent(...)` maps runtime events into this trace view.

`@aithru/node-agent` currently bridges Agent events into core execution events as `log.info` events. The core event payload is the `AgentTraceEvent`, and metadata includes `agentEventType`, `agentTraceKind`, and `agentTracePhase`.

This avoids requiring Aithru Core to support dynamic `agent.*` core event types while still giving future trace viewers stable fields for filtering and display.

## Redaction model

Runtime return values may remain full fidelity for the immediate caller, while persisted trace records can be redacted before writing.

This allows local or server products to keep useful debugging data without leaking configured sensitive paths into long-lived trace storage.

Future modules should reuse the same redaction policy concepts instead of creating unrelated trace sanitization systems.

In personal-server mode, `aithru-personal-bridge` must assume trace and artifact data may contain sensitive local information. It should keep redaction and retention policy explicit before exposing traces to remote browsers.

## Pause and approval model

Human approval is represented as a structured pause/resume flow.

```txt
node returns pause
  -> runtime emits approval request event
  -> runtime emits run paused event
  -> caller receives paused result
  -> caller resumes with approval response
  -> runtime continues or rejects
```

Current `aithru-core` supports same-process local resume. Cross-process personal resume belongs to `aithru-personal-bridge`. Durable multi-user resume belongs to future `aithru-server`.

## Tool execution model

Nodes do not execute concrete external actions directly. They call tools through runtime contracts.

```txt
NodeExecutionContext.callTool
  -> ToolPermissionPolicy
  -> tool lifecycle events
  -> ToolExecutor
  -> ToolExecutionResult or ToolExecutionError
```

This model is important because it gives every product surface one consistent place to enforce:

- allow/deny policy;
- risk level checks;
- approval before execution;
- secret access events;
- redaction;
- audit traces;
- runtime-specific tool execution.

`aithru-personal-bridge` is the right place for personal tool executor registration because it is a trusted Node host. It can register HTTP, local, search/read, or other optional tool executors while keeping those capabilities out of the browser bundle.

## Why Agent must use the same tool model

Agent code can produce dynamic tool calls, but those calls should still go through the same tool execution pipeline.

Required workflow-node direction:

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

This keeps deterministic workflows and Agent workflows compatible with the same inspection, approval, trace, and redaction surfaces.

Model adapters may propose tool calls, but they must not execute tools. Agent runtime engines and Agent workflow nodes also must not bypass `AgentHost.callTool` or `NodeExecutionContext.callTool`.

## Future runtime extensions

| Extension | Owner |
| --- | --- |
| Personal execution host | `aithru-personal-bridge` |
| Personal trace/artifact storage | `aithru-personal-bridge` |
| Personal-server token/CORS/network policy | `aithru-personal-bridge` |
| Desktop embedding and packaging | future `aithru-desktop` |
| Durable team execution | future `aithru-server` |
| Durable run store | `aithru-server` or product-specific adapter |
| Postgres-backed trace store | `aithru-server` |
| Local encrypted credential UX | future `aithru-desktop`; execution-side secret access in `aithru-personal-bridge` |
| Vault/KMS integration | `aithru-server` |
| Browser run timeline UI | `aithru-web` or future `aithru-ui` |
| Agent runtime event stream | `aithru-agent` |
| Agent trace viewer consumption | `aithru-web` / future `aithru-ui` |
| MCP transport lifecycle | future `aithru-mcp` |
