# Aithru Agent

`aithru-agent` is the Aithru-native AI harness layer.

It should be understood as Aithru's own DeepAgents-like intelligent execution environment: a platform-hosted harness for chat, skills, long-running agent runs, tool calls, workspace files, subagents, sandboxed execution, approvals, artifacts, memory, and traceable intelligent work.

It is built on top of Aithru platform identity and Aithru core/workbench capabilities. It must not become a second formal workflow system.

## One-line definition

```txt
Aithru Agent = platform-hosted AI harness for skill-driven, tool-using, permission-aware intelligent work
```

## Mental model

Use this comparison when reasoning about boundaries:

```txt
LangGraph / low-level agent runtime  -> DeepAgents / high-level agent harness
Aithru Core + Workbench capabilities -> Aithru Agent / high-level AI harness
```

The analogy is not exact. Aithru Core is a deterministic workflow kernel, not LangGraph. Aithru Workbench is the formal workflow product surface. Aithru Agent should still borrow the DeepAgents-style product shape: skills, todos, workspace, tools, subagents, memory, sandbox, approvals, and artifacts.

## Core boundary

```txt
aithru-core owns formal workflows.
aithru-workbench owns formal workflow product UI and server-side workflow execution APIs.
aithru-agent owns AI harness behavior and dynamic intelligent execution.
```

`aithru-agent` does not own:

- `WorkflowSpec`;
- workflow graph semantics;
- node/edge branch semantics;
- workflow scheduling;
- workflow persistence format;
- core tool permission policy ownership;
- platform identity, grants, or app shell ownership.

Agent may produce a runtime plan or todo list, but that plan is not a `WorkflowSpec` and must not become a second graph editor.

## New product center

The product center is no longer a small list of engines such as `classify`, `plan-run-review`, and `deep-research`.

The product center is the harness:

```txt
Agent Thread
  -> messages
  -> selected Skill
  -> Agent Run
  -> runtime todos / plan timeline
  -> tool calls
  -> workspace files
  -> subagents
  -> sandbox/interpreter calls
  -> artifacts
  -> review
  -> approvals
  -> trace/event stream
```

Engines are implementation strategies inside the harness. They are not the main product boundary.

## Primary concepts

| Concept | Meaning | Product rule |
| --- | --- | --- |
| Agent Thread | Conversation and task context under an org/user. | Main conversational entry. Not a workflow. |
| Agent Skill | Reusable intelligent capability: instructions, when-to-use notes, allowed tools, subagents, memory policy, sandbox policy, approval policy, and output expectations. | A real skill, not a DAG. May be referenced by Workbench `agent.*` nodes. |
| Agent Run | One execution of a chat request, skill, API task, delegated task, or Workbench agent node. | Auditable intelligent run. |
| Agent Todo / runtime plan | Model/harness-maintained task breakdown. | Runtime observability state, not editable workflow graph. |
| Agent Workspace | Thread/run-scoped virtual file/artifact workspace. | Stores intermediate files, patches, reports, structured outputs. |
| Agent Tool | Capability the harness may request. | Must route through Aithru permission, policy, approval, trace, and redaction. |
| Subagent | Harness-spawned specialized worker such as researcher, coder, reviewer, analyst, or workflow designer. | Internal agent delegation, not Workbench node graph. |
| Sandbox / Interpreter | Controlled execution environment for scripts, code, shell-like work, or analysis. | Never direct model access; always policy-gated. |
| Memory | Optional thread/workspace/project/org memory. | Product host controls scope, retention, and authorization. |
| Artifact | Output such as report, markdown, JSON, patch, file, decision, or workflow draft. | Agent output object; may be opened in Workbench if it is a WorkflowSpec draft. |
| Approval | Human or policy decision for risky agent action. | Distinct from workflow human approval, but must remain audit-friendly. |

## Aithru capability router

Aithru Agent should not execute real capabilities directly from the model loop.

All real actions should pass through an Aithru capability router:

```txt
model proposes tool call
  -> Agent Harness validates skill/tool policy
  -> Aithru Capability Router
  -> platform/core/workbench permission checks
  -> approval if required
  -> concrete capability backend
  -> trace + artifact + redaction
```

Capability backends may include:

| Backend | Purpose | Rule |
| --- | --- | --- |
| core tool executor | Reuse Aithru Core tool contracts. | Must preserve `ToolPermissionPolicy`, trace, approval, and redaction. |
| core node adapter | Expose selected single-node capabilities as tools. | Only for safe node-as-capability cases; must not run workflow graphs. |
| workbench workflow adapter | Run saved `WorkflowSpec` through Workbench APIs. | Agent must not interpret or schedule the graph itself. |
| subsystem API adapter | Call other platform apps through token exchange/delegation. | Must respect platform connection policy and scopes. |
| sandbox adapter | Execute scripts/code in controlled environments. | Must be approval/policy gated. |
| workspace adapter | Read/write Agent workspace files. | Must respect workspace policy and retention. |
| memory adapter | Read/write memory. | Must respect org/user/project/thread scope. |
| MCP adapter | Future optional MCP capability bridge. | Must stay behind Aithru policy and audit boundaries. |

## Workbench integration

### Workbench calls Agent

Workbench may use Agent through workflow nodes such as:

```txt
agent.skill
agent.task
```

The node should reference an Agent Skill and map workflow input/output to the harness.

Recommended future node config:

```ts
type AgentSkillNodeConfig = {
  skillId: string;
  inputMapping?: Record<string, string>;
  outputMapping?: Record<string, string>;
  workspaceMode?: "ephemeral" | "workflow_run";
  approvalMode?: "inherit_workflow" | "agent_policy" | "both";
  toolPolicyOverride?: unknown;
};
```

The Workbench graph remains the formal workflow. The Agent harness owns only the intelligent behavior inside the node.

### Agent calls Workbench

Agent may call Workbench workflows as tools:

```txt
Agent Harness
  -> tool: workbench.runWorkflow
  -> Workbench executes WorkflowSpec
  -> Agent receives result, artifact reference, or trace summary
```

Agent must not import Workbench internals or schedule workflow graphs itself.

### Agent creates workflow drafts

Agent may generate a `WorkflowSpec` draft artifact and offer:

```txt
Open in Workbench
Validate in Workbench
Download WorkflowSpec JSON
```

The draft becomes a formal workflow only when Workbench validates and saves it.

## Existing implemented baseline

The current repository already implements useful primitives:

| Package | Current role | Future positioning |
| --- | --- | --- |
| `@aithru/agent-core` | `AgentTask`, `AgentPlan`, events, model adapter, host, artifacts, approval, trace types. | Keep as primitive contract layer; extend toward thread/skill/run/todo/workspace/tool/subagent contracts. |
| `@aithru/agent-runtime` | `ClassifyEngine`, `PlanRunReviewEngine`, `DeepResearchEngine`, `AgentRuntime`. | Keep as compatibility/runtime primitive layer; harness should sit above it. |
| `@aithru/agent-model-test` | Deterministic scripted adapter. | Keep for tests and harness examples. |
| `@aithru/agent-model-openai-compatible` | OpenAI-compatible model adapter. | Keep as model provider adapter. |
| `@aithru/node-agent` | Workflow node factories for `agent.classify`, `agent.task`, `agent.deepResearch`. | Evolve toward skill/harness invocation nodes. |

The existing primitives should not be treated as the final Agent product design.

## Target package direction

Future package shape should move toward:

```txt
packages/
  agent-core/            # shared harness contracts
  agent-harness/         # DeepAgents-like main harness
  agent-skills/          # skill loading, validation, composition
  agent-workspace/       # virtual workspace and artifact APIs
  agent-tools/           # Aithru capability router and tool adapters
  agent-subagents/       # subagent delegation contracts
  agent-sandbox/         # sandbox/interpreter interfaces
  agent-memory/          # memory provider contracts
  agent-model-*/         # model adapters
  node-agent/            # Workbench/Core node integration

apps/
  agent-server/          # Platform subsystem backend
  agent-web/             # Platform hosted app frontend
```

This package direction is a design target, not an immediate implementation requirement.

## Platform-hosted product surface

Agent is expected to be a Platform sub-application with its own manifest, permissions, roles, hosted app frontend, and server connection.

Recommended navigation:

```txt
Chat
Skills
Workspace
Runs
Artifacts
Approvals
Tools
Memory
Settings
```

The UI should resemble a high-capability AI harness such as Codex, Claude Code, or DeepAgents-style systems, not a workflow graph editor.

## Tool-use rule

Agent code must not bypass the Aithru capability layer.

```txt
Agent model event
  -> Agent Harness
  -> Aithru Capability Router
  -> Aithru Core / Workbench / Platform capability boundary
  -> ToolPermissionPolicy / authz / approval
  -> concrete executor
  -> trace events / artifacts / redaction
```

Model adapters may propose tool calls. They must not execute tools.

## Non-goals

Aithru Agent should not:

- own `WorkflowSpec`;
- implement a second workflow graph editor;
- let models execute shell/filesystem/network actions directly;
- bypass platform org/user permissions;
- bypass Aithru Core tool policy, trace, redaction, or approval;
- make Workbench depend on Agent internals;
- make Core depend on Agent packages.

## Review checklist

Before accepting Agent design or implementation changes, check:

- [ ] Does the change strengthen the AI harness model rather than adding a second workflow system?
- [ ] Are Skills real reusable agent capabilities, not DAG workflows?
- [ ] Are Todos/runtime plans observable runtime state, not editable graphs?
- [ ] Do all tool calls pass through Aithru capability routing?
- [ ] Are sandbox, filesystem, code execution, and external calls policy-gated?
- [ ] Are Workbench workflows invoked only through explicit APIs/tools?
- [ ] Can Workbench call Agent only through explicit `agent.*` node integration?
- [ ] Are platform identity, org, grants, token exchange, audit, and redaction respected?
