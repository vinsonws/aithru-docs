# Core and Agent Boundary

This document fixes the boundary between `aithru-core` and `aithru-agent`.

The goal is to prevent future confusion between the deterministic workflow system and the intelligent agent execution layer.

## One-line rule

```txt
aithru-core owns formal workflows.
aithru-agent owns intelligent execution inside bounded tasks or nodes.
```

## What belongs to core

`aithru-core` owns the formal workflow system.

This includes:

- `WorkflowSpec`;
- workflow nodes and edges;
- graph validation;
- branch and condition semantics;
- workflow runtime adapters;
- run status;
- execution events;
- trace contracts;
- pause/resume contracts;
- human approval contracts;
- redaction policy;
- generic tool execution contracts;
- tool permission policy.

A formal workflow is user-editable, reusable, versionable, visualizable, and executed by an Aithru runtime adapter.

## What belongs to agent

`aithru-agent` owns intelligent execution.

This includes:

- `AgentTask`;
- `AgentPlan`;
- `AgentStep`;
- `AgentRun`;
- model adapters;
- planning;
- deep research loops;
- short classification or routing decisions;
- review and rerun logic;
- agent artifacts;
- agent event mapping to core trace concepts.

An agent plan is task-local. It helps the agent complete one task. It is not a formal `WorkflowSpec`.

## Formal workflow vs agent plan

| Concept | Owner | Purpose | Persistence expectation |
| --- | --- | --- | --- |
| `WorkflowSpec` | `aithru-core` | Defines a reusable workflow graph. | Stored, edited, versioned, visualized. |
| Workflow node/edge | `aithru-core` | Defines formal execution structure. | Stored as workflow definition. |
| Agent node | `aithru-agent` package, executed by core runtime as a node. | Adds intelligent behavior inside a workflow step. | Stored as a workflow node config. |
| `AgentPlan` | `aithru-agent` | Dynamic plan for one agent task. | Recorded in trace/result, not treated as workflow definition. |
| `AgentStep` | `aithru-agent` | Internal step during one agent task. | Recorded for observability. |

## Correct integration pattern

Agent behavior may be exposed as workflow nodes:

```txt
core.manualTrigger
  -> agent.classify
  -> core.if
  -> agent.deepResearch
  -> core.humanApproval
  -> core.httpRequest
```

In this pattern:

- `aithru-core` owns the graph and branch execution.
- `aithru-agent` owns the intelligent behavior inside `agent.*` nodes.
- tool execution still goes through core tool contracts.

## Decision and routing nodes

For short intelligent decisions, use an agent node to produce structured output, then use core nodes for formal branching.

Example:

```txt
agent.classify outputs { route: "deep_research", confidence: 0.87 }
core.if uses route to choose the next branch
```

The agent gives a judgment. The core workflow performs the branch.

## Deep research nodes

Deep research belongs to `aithru-agent` as an intelligent task pattern.

It can be used in two ways:

1. As a direct agent task through `agentRuntime.runTask`.
2. As a workflow node such as `agent.deepResearch` inside a core workflow.

The reusable workflow that contains the research node still belongs to `aithru-core`.

## What agent must not do

`aithru-agent` must not own or redefine:

- `WorkflowSpec`;
- graph execution semantics;
- formal workflow versioning;
- core scheduler behavior;
- branch semantics;
- workflow persistence format;
- tool permission policy;
- approval contract;
- trace contract.

If an agent needs to run a formal workflow, it should request a core runtime to run that workflow. It should not execute workflow graphs itself.

## Practical test

When deciding where a feature belongs, ask:

```txt
Is this a reusable, user-editable, versioned workflow graph?
```

If yes, it belongs to `aithru-core`.

Then ask:

```txt
Is this LLM-driven planning, judgment, research, writing, review, or rerun logic inside a task?
```

If yes, it belongs to `aithru-agent`.
