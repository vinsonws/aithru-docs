# Roadmap

This roadmap is organized around module boundaries rather than a single monolithic repository.

## Phase 0: Current baseline

Status: implemented across `aithru-core`, `aithru-web`, and `aithru-agent`; `aithru-personal-bridge` remains the next execution-host step.

### `aithru-core`

- v0.15.0-rc.0 release candidate baseline.
- API freeze for current package-root public APIs.
- Deterministic workflow spec and validation.
- Local runtime adapter.
- Node SDK.
- Core, human, and HTTP nodes.
- HTTP tool executor.
- Trace, redaction, pause/resume, and generic tool contracts.
- Optional stdlib for deterministic text/json helpers and nodes.

### `aithru-web`

- Browser-safe workflow editor.
- JSON source-of-truth editing.
- Node palette.
- Minimal graph editing.
- Validation panel.
- Local browser draft persistence.
- Bridge Mock.
- HTTP Bridge Client.
- Trace viewer foundation.

### `aithru-agent`

- `@aithru/agent-core`:
  - Agent contracts.
  - `AgentEvent`.
  - `AgentTraceEvent` taxonomy.
  - `agentTraceEventFromAgentEvent(...)`.
  - research report types.
- `@aithru/agent-runtime`:
  - `ClassifyEngine`.
  - `PlanRunReviewEngine`.
  - `DeepResearchEngine` V0.
  - `AgentRuntime.run(...)`.
  - `AgentRuntime.runTask(...)`.
  - runtime failure semantics.
- `@aithru/agent-model-test`:
  - deterministic scripted model adapter.
- `@aithru/agent-model-openai-compatible`:
  - OpenAI-compatible HTTP model adapter.
  - no provider SDK.
  - does not execute tools.
- `@aithru/node-agent`:
  - `agent.classify`.
  - `agent.task`.
  - `agent.deepResearch`.
  - model resolution injection.
  - tool bridge to core `ctx.callTool`.
  - AgentTraceEvent bridge through core `log.info` events.
- Examples:
  - standalone classify.
  - standalone plan-run-review.
  - standalone deep research.
  - node-agent-basic.
  - workflow-node-agent.
  - workflow-node-agent-deep-research.
  - optional OpenAI-compatible classify example.
- Bounded agent execution only; `AgentPlan` is not `WorkflowSpec`.

## Phase 1: Documentation source of truth

Goal: make `aithru-docs` the cross-repository design hub.

Deliverables:

- Vision and ecosystem overview.
- Module boundaries.
- Core and Web baseline documents.
- Personal Bridge document.
- Agent design document.
- Runtime/trace/tooling document.
- ADRs for docs, boundaries, browser runtime split, core/agent split, and personal bridge.
- Reusable prompt for keeping docs and code aligned.

## Phase 2: Aithru Personal Bridge MVP

Goal: create the trusted personal execution host that connects `aithru-web` HTTP Bridge Client to real Aithru Core runtime execution.

### v0.1: Loopback bridge

Suggested deliverables:

- `aithru-personal-bridge` TypeScript Node project.
- Bind to `127.0.0.1` by default.
- Implement HTTP Bridge API:
  - `POST /runs`
  - `GET /runs/:runId`
  - `GET /runs/:runId/events`
  - `POST /runs/:runId/resume`
  - `POST /runs/:runId/cancel`
- Validate `WorkflowSpec` with `@aithru/spec`.
- Compose `@aithru/runtime-local` with core/human/http node packages.
- Register `@aithru/tool-http` only inside the bridge runtime host.
- Store runs/events in memory for v0.1.
- Support human approval pause/resume/reject.
- Confirm `aithru-web` HTTP Bridge mode can start and inspect a simple run.

### v0.2: Personal-server mode

Suggested deliverables:

- Configurable bind host.
- Token auth for non-loopback access.
- CORS allowlist.
- Local trace/artifact directory.
- Tailscale/LAN/HTTPS reverse proxy deployment notes.
- Clear warning not to expose an unauthenticated bridge to the public internet.

## Phase 3: Extract or formalize UI boundary

Goal: decide whether `aithru-web` remains a product app or whether reusable pieces become `aithru-ui`.

Possible deliverables:

- WorkflowDesigner component boundary.
- NodePalette component boundary.
- NodeConfigPanel schema-driven design.
- TraceViewer fixtures.
- ApprovalCard UI.
- Browser-safe import rules.

## Phase 4: Aithru Web Agent trace viewer

Goal: add first-class web UX for inspecting Agent execution without moving Agent runtime ownership into `aithru-web`.

Suggested deliverables:

- Agent trace event timeline in `aithru-web`.
- Agent run detail panel.
- Mapping from `AgentTraceEvent` to UI-friendly views.
- Example fixtures for standalone and workflow-node Agent runs.
- Browser-safe trace inspection only; no runtime execution.

## Phase 5: Richer Deep Research tools

Goal: extend `DeepResearchEngine` with additional research affordances while keeping Agent execution bounded inside `aithru-agent`.

Suggested deliverables:

- More source and citation handling inside the Agent runtime.
- Research tool coordination that still goes through `AgentHost.callTool`.
- Browser/web research tools only if explicitly added as optional integrations later.
- Durable persistence remains out of scope.
- Memory remains out of scope.
- UI/chat remains out of scope.

## Phase 6: MCP optional integration

Goal: reintroduce MCP as an optional package, not as a core dependency.

Suggested deliverables:

- MCP tool executor.
- MCP node definitions.
- stdio/http/sse client adapters as appropriate.
- Permission and approval tests.
- Trace compatibility tests.
- Optional package wiring only; MCP is not implemented yet.

## Phase 7: Desktop Personal Edition

Goal: create the local-first product shell.

Suggested deliverables:

- Local project/workspace model.
- Local runtime composition.
- Local trace browser.
- Local credential storage.
- Embed, manage, or package `aithru-personal-bridge`.
- Plugin installation model.
- Templates.

## Phase 8: Server Business Edition

Goal: create team-grade execution and governance.

Suggested deliverables:

- API server.
- Worker runtime.
- Durable run store.
- Postgres trace store.
- Queue integration.
- RBAC.
- Audit log.
- Workspace/team model.
- Private node/tool registry.

## Release discipline

- Core changes should be conservative after v0.15 RC.
- New concrete integrations should default to optional packages.
- Cross-repo boundary changes should update this repository first.
- Implementation repositories should update package-level README files when behavior changes.
- Personal execution work should start in `aithru-personal-bridge` before being embedded by a desktop product shell.
