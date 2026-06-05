# ADR-0006: Agent engines may be exposed as workflow nodes

Status: Accepted

Date: 2026-06-05

## Context

Aithru separates formal workflow execution from intelligent Agent execution.

`aithru-core` owns `WorkflowSpec`, graph semantics, validation, scheduling, and workflow runtime behavior.

`aithru-agent` owns bounded intelligent execution through Agent tasks, Agent plans, model adapters, Agent runtime engines, artifacts, review, and Agent trace events.

The implemented Agent runtime includes `ClassifyEngine`, `PlanRunReviewEngine`, and `DeepResearchEngine` V0.

## Decision

Agent runtime engines may be exposed as Aithru workflow nodes through `@aithru/node-agent`.

The current nodes are:

- `agent.classify`
- `agent.task`
- `agent.deepResearch`

These are workflow nodes, but they do not own workflow semantics. Formal workflow execution remains in `aithru-core`.

## Rules

- `AgentPlan` remains task-local and is not `WorkflowSpec`.
- Model resolution is injected by the host application.
- Agent nodes call Agent runtime engines.
- External actions requested by Agent nodes go through the core node context bridge.
- Agent events are mapped to `AgentTraceEvent` for trace consumers.

## Consequences

- There is still only one formal workflow system.
- Agent engines can run standalone or inside workflows.
- Future trace viewers can consume stable Agent trace metadata.
- Richer Agent-specific core event types can be considered later, but are not required now.
