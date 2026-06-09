# Agent Product Boundary

This document defines the product boundary between Aithru Workbench and Aithru Agent.

It complements:

- [Module Boundaries](./02-module-boundaries.md)
- [Aithru Agent](./05-aithru-agent.md)
- [Runtime, Trace, Approval, Tooling](./06-runtime-trace-approval-tooling.md)
- [Core and Agent Boundary](./09-core-agent-boundary.md)
- [Frontend Constraints](./11-frontend-constraints.md)

## One-line rule

```txt
There is one formal workflow system: WorkflowSpec in Aithru Core.
Agent may have workflow-like runtime plans, but those plans are not product workflow definitions.
```

## Why this document exists

Aithru Agent naturally looks workflow-like:

```txt
AgentTask
  -> AgentPlan
  -> AgentStep
  -> tool call
  -> artifact
  -> review
  -> resume / rerun
```

That shape is useful for intelligent execution, trace inspection, and debugging. However, if Agent plans become a second user-editable workflow system, Aithru would have two competing workflow concepts:

- formal `WorkflowSpec` graphs edited and run through Workbench;
- dynamic Agent plans edited and saved through Agent surfaces.

That would confuse product boundaries, UI ownership, trace semantics, approval behavior, versioning, scheduling, and future platform/server execution.

The boundary in this document keeps Agent powerful without turning it into a second workflow workbench.

## Product concepts

| Concept | Owner | Meaning | Product rule |
| --- | --- | --- | --- |
| `WorkflowSpec` | `aithru-core`, surfaced by `aithru-workbench` | Formal deterministic workflow definition with nodes, edges, validation, branch semantics, versioning, and execution semantics. | The only formal workflow definition in Aithru. |
| Workbench Workflow | `aithru-workbench` | Product surface for authoring, saving, validating, running, approving, and inspecting `WorkflowSpec`. | Use graph/workflow language here. |
| Agent Chat | Agent product surface or platform-hosted Agent subsystem | Conversational entry point that creates and continues intelligent tasks. | User-facing interaction, not a workflow editor. |
| Agent Skill | Agent product surface or product-specific host | Reusable intelligent capability template: instruction, engine, model, tools, context policy, output schema, review policy, and approval policy. | Not a DAG workflow. Do not call it a workflow unless promoted to `WorkflowSpec`. |
| `AgentTask` | `aithru-agent` | One bounded intelligent task with goal, input, context, constraints, and expected artifacts. | Can be created from chat, API, a skill, or an `agent.*` node. |
| `AgentPlan` | `aithru-agent` | Runtime task-local plan produced or consumed by an Agent engine. | Workflow-like runtime plan, not stored workflow definition. |
| `AgentRun` / Agent events | `aithru-agent`, stored by a host product when needed | Observable execution of one Agent task, including plan, steps, tool calls, artifacts, review, approval pauses, and failures. | Inspectable trace/run data. Not an editable workflow graph. |
| `agent.*` workflow node | `@aithru/node-agent`, executed by a core runtime | A formal workflow node that invokes Agent runtime inside a bounded step. | The outer graph belongs to Workbench/Core; Agent owns only the node's intelligent behavior. |

## Layer model

```txt
Aithru Platform
  owns identity, authorization, grants, app shell, subsystem entry

Aithru Workbench
  owns formal WorkflowSpec product UI, run APIs, run/event storage,
  workflow approval endpoints, and runtime composition with Aithru Core

Aithru Agent product surface
  owns chat, skills, intelligent task runs, artifacts, Agent approvals,
  model/tool/memory configuration, and Agent-specific inspection

Aithru Agent packages
  own AgentTask, AgentPlan, AgentRuntime, engines, model adapters,
  AgentTraceEvent, and optional workflow node integration

Aithru Core
  owns WorkflowSpec, graph validation, runtime contracts,
  tool contracts, trace/redaction/pause/resume primitives
```

## Workbench vs Agent

### Workbench is the workflow workbench

Workbench should own surfaces such as:

- workflow list;
- workflow graph editor;
- node palette;
- node config panels;
- workflow validation;
- workflow import/export;
- workflow run console;
- workflow run/event storage;
- workflow approval resume/reject/cancel;
- runtime capabilities for registered workflow nodes and tools.

Use words such as:

```txt
workflow
node
edge
graph
branch
run workflow
validate workflow
open in Workbench
```

### Agent is the intelligent capability surface

Agent product surfaces should own surfaces such as:

- chat;
- skills;
- intelligent task runs;
- Agent artifacts;
- Agent approval cards;
- Agent tool permission summaries;
- Agent trace inspection;
- model configuration;
- memory/context policy;
- reusable intelligent task templates.

Use words such as:

```txt
chat
skill
task
plan
step
artifact
review
research
classify
agent run
save as skill
create workflow draft
open in Workbench
```

Avoid using these labels for Agent product surfaces unless the object is a real `WorkflowSpec`:

```txt
workflow editor
workflow graph
sub-workflow
agent workflow definition
save AgentPlan as workflow
```

## Integration patterns

### 1. Workbench calls Agent as a workflow node

```txt
core.manualTrigger
  -> agent.classify
  -> core.if
  -> agent.deepResearch
  -> core.humanApproval
  -> core.httpRequest
```

In this pattern:

- the outer graph is a formal `WorkflowSpec`;
- Workbench owns editing, validation, run storage, and workflow-level approval;
- Core owns graph execution, branch semantics, and scheduler behavior;
- Agent owns intelligent behavior inside `agent.*` nodes;
- Agent tool calls must flow through the host bridge into core tool contracts;
- Agent events should be mapped into `AgentTraceEvent` for trace consumers.

### 2. Agent Chat or Skill calls a Workbench workflow

Agent may invoke a saved workflow as a tool-like capability:

```txt
Agent Chat / Skill
  -> propose tool call: workbench.runWorkflow
  -> Workbench runs WorkflowSpec
  -> Agent receives result, artifact reference, or trace summary
```

In this pattern:

- Agent does not interpret or schedule the workflow graph itself;
- Workbench remains the execution owner for formal workflows;
- Agent may explain, summarize, or continue from the workflow result;
- approval, permission, and trace boundaries must remain visible.

### 3. Agent creates a workflow draft

Agent may help users author a workflow:

```txt
User request in Agent Chat
  -> Agent proposes a WorkflowSpec draft
  -> user opens the draft in Workbench
  -> Workbench validates, edits, saves, versions, and runs it
```

In this pattern:

- Agent is an authoring assistant;
- Workbench is still the workflow product surface;
- the draft becomes formal only when represented as `WorkflowSpec` and accepted by Workbench validation.

### 4. Agent direct run

Agent may run standalone bounded tasks:

```txt
Agent Skill / Chat / API
  -> AgentTask
  -> AgentRuntime engine
  -> AgentRun events
  -> artifacts / review / approval
```

In this pattern:

- the run is intelligent execution, not workflow execution;
- the runtime plan may be displayed for observability;
- the plan must not become a second editable graph surface;
- repeated reusable automation should be promoted to Workbench workflow when it needs workflow semantics.

## Promotion rule

Promote an Agent pattern into a formal Workbench workflow when it becomes:

- reusable across users, teams, or products;
- user-editable as a stable graph;
- versioned or published;
- scheduled or triggered by external systems;
- branch-heavy or deterministic after the intelligent step;
- dependent on durable approval, audit, RBAC, or workflow run storage;
- composed of multiple deterministic tools/nodes where Agent is only one step.

Keep it as an Agent Skill or Agent Task when it is:

- primarily conversational;
- one-off or exploratory;
- LLM-driven planning, judgment, writing, research, review, or rerun logic;
- best expressed as instruction + tools + output schema rather than nodes + edges;
- useful as a reusable intelligent capability but not as a deterministic DAG.

Boundary test:

```txt
Is this a reusable, user-editable, versioned graph with deterministic control flow?
```

If yes, it belongs in Workbench/Core as `WorkflowSpec`.

```txt
Is this an intelligent task, skill, research loop, review loop, or conversational operation whose plan is produced or adjusted at runtime?
```

If yes, it belongs in Agent.

## Agent Skill

`AgentSkill` is the recommended product concept for reusable intelligent capability.

It is not currently a required core package contract. Product hosts may implement it as an application-level contract first.

A skill may contain:

- identity: id, name, description;
- engine: `classify`, `plan-run-review`, or `deep-research`;
- instruction and task template;
- input and output schemas;
- context and memory policy;
- allowed tools and risk policy;
- max steps, timeout, max sources, and max search queries;
- review criteria;
- expected artifacts.

Rules:

- A skill may be invoked from Chat.
- A skill may be referenced by an `agent.task` workflow node.
- A skill may generate artifacts.
- A skill may require approval before risky tool calls.
- A skill may create a `WorkflowSpec` draft, but Workbench must own validation and saving.
- A skill is not a workflow graph and must not own branch semantics or scheduling.

## UI implications

### Agent product navigation

Recommended top-level Agent navigation:

```txt
Chat
Skills
Runs
Artifacts
Approvals
Tools
Settings
```

Agent UI may include run and trace inspection, but those surfaces should support the primary intelligent product experience. They should not become another workflow graph editor.

### Workbench product navigation

Recommended top-level Workbench navigation:

```txt
Workflows
Runs
Approvals
Nodes
Tools
Settings
```

Overlap is acceptable if the object is clear:

| Label | In Workbench | In Agent |
| --- | --- | --- |
| Runs | Workflow runs | Agent task runs |
| Approvals | Workflow pause approvals | Agent tool/skill approvals |
| Tools | Runtime tool executors and workflow capabilities | Tools available to Agent tasks/skills |
| Artifacts | Workflow outputs and stored artifacts | Agent task outputs, reports, patches, decisions |

### Agent plan display

Agent UI may show plans and steps, but the visual language should communicate runtime observability:

- plan timeline;
- step list;
- tool call cards;
- artifact cards;
- review summary;
- trace event stream;
- approval cards.

Avoid editable graph affordances for Agent plans unless the user is explicitly converting the behavior into a Workbench workflow draft.

### Workbench node display for Agent nodes

When Workbench displays an `agent.*` node:

- the node config should expose skill/engine/model/tool policy fields;
- the run detail may show `AgentTraceEvent` details for that node;
- the node should not expand into a nested editable `AgentPlan` graph;
- a user may open the related Agent run details for debugging if the host product supports it.

## Storage and execution ownership

Agent packages do not need to own durable product storage by default.

Product hosts may store:

- chat threads;
- skills;
- Agent runs;
- Agent events;
- artifacts;
- approval records;
- model settings;
- tool policies;
- memory/context references.

Rules:

- Formal workflow persistence remains `WorkflowSpec` persistence and belongs to Workbench/Core hosts.
- Formal workflow run/event persistence belongs to Workbench/server hosts.
- Agent product storage must not redefine workflow persistence.
- Browser UI must not own execution semantics, real tool execution, secret storage, or durable approval state.
- Sensitive trace values, tool arguments, model inputs, and secrets must be redacted according to host policy before long-term storage or display.

## Implementation guidance

Before implementing Agent product surfaces, decide and document:

1. Is this a product app, platform subsystem, local companion, or demo/lab?
2. Does it store chat, skills, runs, artifacts, or approvals?
3. Which host owns real model calls and tool execution?
4. How does it expose mock vs real execution?
5. How does it call Workbench workflows without owning their execution?
6. How does Workbench reference Agent skills or engines in `agent.*` nodes?
7. What is the promotion path from Agent Skill to Workbench Workflow?
8. What redaction and approval policy applies to Agent traces and artifacts?

Default recommendation:

```txt
Start with Agent Chat + Agent Skills + Agent Runs.
Do not build an Agent graph editor.
When users need stable graph orchestration, generate or open a WorkflowSpec in Workbench.
```

## PR checklist

- [ ] Does the feature preserve one formal workflow system: `WorkflowSpec`?
- [ ] Does the feature avoid treating `AgentPlan` as a stored workflow definition?
- [ ] If the feature is reusable and graph-like, is it represented as a Workbench/Core workflow instead of an Agent plan?
- [ ] If the feature is an intelligent reusable capability, is it modeled as an Agent Skill rather than a workflow graph?
- [ ] Are Workbench runs and Agent runs clearly distinguished in UI copy and routes?
- [ ] Are Workbench approvals and Agent approvals clearly distinguished?
- [ ] Does Agent call Workbench workflows through an explicit API/tool boundary instead of interpreting workflow graphs itself?
- [ ] Does Workbench call Agent through `agent.*` nodes or another explicit integration boundary?
- [ ] Are real model calls, tool execution, secrets, and durable approvals kept outside browser UI code?
- [ ] Are traces, tool arguments, model inputs, and artifacts redacted according to host policy?
