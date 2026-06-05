# Roadmap

This roadmap is organized around module boundaries rather than a single monolithic repository.

## Phase 0: Current baseline

Status: mostly done in `aithru-core` and `aithru-web`.

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

## Phase 4: Aithru Agent MVP

Goal: build the first optional Agent runtime/harness on top of core contracts.

Suggested MVP:

- Agent contracts package.
- Test/scripted model adapter.
- One real model adapter.
- Plan/run/review runtime.
- Agent workflow node definitions.
- Tool-use path through core `callTool` and `ToolPermissionPolicy`.
- Agent trace events mapped to Aithru run trace concepts.
- Example workflow and programmatic runner.

## Phase 5: MCP optional integration

Goal: reintroduce MCP as an optional package, not as a core dependency.

Suggested deliverables:

- MCP tool executor.
- MCP node definitions.
- stdio/http/sse client adapters as appropriate.
- Permission and approval tests.
- Trace compatibility tests.

## Phase 6: Desktop Personal Edition

Goal: create the local-first product shell.

Suggested deliverables:

- Local project/workspace model.
- Local runtime composition.
- Local trace browser.
- Local credential storage.
- Embed, manage, or package `aithru-personal-bridge`.
- Plugin installation model.
- Templates.

## Phase 7: Server Business Edition

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
