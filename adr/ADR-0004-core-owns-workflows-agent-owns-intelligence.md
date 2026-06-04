# ADR-0004: Core owns workflows, Agent owns intelligent execution

Status: Accepted

Date: 2026-06-04

## Context

Aithru is adding an optional `aithru-agent` layer on top of `aithru-core`.

There is a risk that `aithru-agent` could gradually become a second workflow engine if its internal plans, steps, routing, and long-running task logic are not clearly separated from `aithru-core` workflows.

That would create two competing definitions of workflow:

- formal Aithru workflows in `WorkflowSpec`;
- dynamic Agent plans inside `aithru-agent`.

This would make product behavior, UI editing, tracing, approval, and future server/desktop execution harder to reason about.

## Decision

`aithru-core` owns formal workflows.

`aithru-agent` owns intelligent execution inside bounded tasks or workflow nodes.

This means:

- `WorkflowSpec` belongs to `aithru-core`.
- Workflow graph nodes, edges, conditions, branch semantics, and runtime scheduling belong to `aithru-core`.
- `AgentTask`, `AgentPlan`, `AgentStep`, `AgentRun`, model adapters, deep research loops, classification, review, and rerun logic belong to `aithru-agent`.
- `AgentPlan` is task-local and should not be treated as a stored workflow definition.
- Agent behavior may be exposed as `agent.*` workflow nodes, but those nodes are still executed by a core runtime adapter as part of a formal workflow.

## Correct integration pattern

```txt
core.manualTrigger
  -> agent.classify
  -> core.if
  -> agent.deepResearch
  -> core.humanApproval
  -> core.httpRequest
```

In this pattern:

- core owns the formal graph;
- agent owns the intelligent behavior inside `agent.*` nodes;
- tools still go through core tool contracts and permission policy;
- approval and trace remain core-compatible.

## Consequences

Positive:

- There is only one formal workflow system.
- Aithru Web and future UI components can keep editing `WorkflowSpec` as the source of truth.
- Agent can stay powerful without taking over workflow semantics.
- Deep research and short classification can be added as intelligent nodes.
- Server and desktop runtimes can compose Agent nodes without learning a second workflow definition.

Tradeoffs:

- Some Agent plans may look like workflows, but they must remain task-local execution plans.
- If a repeated Agent plan becomes reusable and user-editable, it should be promoted into a formal core workflow.
- Agent direct calls are useful, but reusable product automation should eventually be represented as core workflows.

## Boundary test

Ask:

```txt
Is this a reusable, user-editable, versioned workflow graph?
```

If yes, it belongs to `aithru-core`.

Ask:

```txt
Is this LLM-driven planning, judgment, research, writing, review, or rerun logic inside a bounded task?
```

If yes, it belongs to `aithru-agent`.
