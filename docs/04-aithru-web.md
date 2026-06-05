# Aithru Web

`aithru-web` is the current browser-safe workflow authoring and inspection surface for Aithru Core workflows.

Current implementation baseline: `0.1.0` package version, with v0.6-style HTTP Bridge Client groundwork in the app.

## One-line definition

```txt
Aithru Web = browser-safe WorkflowSpec editor + node palette + validation + graph editing + trace viewer + BridgeClient UI
```

## Current responsibilities

`aithru-web` currently provides:

- editable `WorkflowSpec` JSON as the browser source of truth;
- validation through `@aithru/spec`;
- browser-safe node registry panel;
- node insertion from the palette;
- visual graph preview and minimal graph editing;
- selected-node controls for name, position, config, edge, and deletion edits;
- import/export controls for local workflow JSON;
- versioned browser `localStorage` draft persistence;
- validation warnings for browser-only limitations;
- Bridge Mock for in-browser future-shaped run records and events;
- HTTP Bridge Client for calling an external execution host;
- trace viewer over bridge/mock events.

## Browser-safe imports

The current web app imports browser-safe package entry points only:

- `@aithru/spec`
- `@aithru/runtime-core`
- `@aithru/node-sdk`
- `@aithru/nodes-core`
- `@aithru/nodes-human`
- `@aithru/nodes-http`
- `@aithru/stdlib/text`
- `@aithru/stdlib/json`

It intentionally does not import:

- `@aithru/runtime-local`
- `@aithru/tool-http`
- `@aithru/stdlib/file`
- Node-only filesystem helpers
- concrete runtime implementations

## Execution boundary

The browser must not own real workflow execution.

Current rule:

```txt
aithru-web edits and validates workflows.
aithru-personal-bridge or aithru-server executes workflows.
```

This preserves a clean split between:

| Concern | Owner |
| --- | --- |
| Workflow JSON source of truth | `aithru-web` |
| Browser-safe validation | `aithru-web` + `@aithru/spec` |
| Node palette metadata | `aithru-web` + browser-safe node packages |
| HTTP Bridge Client | `aithru-web` |
| Personal/local runtime execution | `aithru-personal-bridge` |
| Business/Enterprise durable execution | future `aithru-server` |
| Real HTTP execution | tool executor in runtime composition, not browser UI |
| Durable human approval | personal bridge or server runtime, not browser simulation |

## Bridge targets

The Run Panel can use these execution surfaces:

| Target | Meaning |
| --- | --- |
| Bridge Mock | In-browser, in-memory mock records and events. Useful for UI development only. |
| `aithru-personal-bridge` | Real personal execution host for localhost or private personal-server deployments. |
| future `aithru-server` | Multi-user, durable, RBAC/audit, team execution host. |

## Current warnings

The app should keep warning users when a node is present but not truly executable in the browser:

- `core.httpRequest`: metadata only in the browser; execution requires `aithru-personal-bridge` or `aithru-server`.
- `core.humanApproval`: simulation/inspection only in the browser; durable pause/resume requires `aithru-personal-bridge` or `aithru-server`.

## Future direction

`aithru-web` may evolve in two possible ways:

1. Stay as a product-specific local development app.
2. Extract reusable UI pieces into a future `aithru-ui` package.

A future `aithru-ui` package could own:

- WorkflowDesigner;
- NodePalette;
- NodeConfigPanel;
- WorkflowJsonEditor;
- ValidationPanel;
- TraceViewer;
- ApprovalCard.

`aithru-web` would then become a thin product shell using those components.

## Non-goals

`aithru-web` should not become the real runtime, credential store, tool executor, durable approval system, or bridge/server implementation.

It should also avoid importing Node-only packages into the browser bundle.
