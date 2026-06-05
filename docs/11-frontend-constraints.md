# Frontend Constraints

This document defines cross-repository frontend constraints for Aithru browser surfaces.

It complements [Aithru Web](./04-aithru-web.md): `aithru-web` remains the browser-safe workflow authoring and inspection surface, while real execution belongs to `aithru-personal-bridge` or future `aithru-server`.

## One-line definition

```txt
Aithru frontend = workbench UI for workflow authoring, inspection, trace review, bridge/server interaction, and human approval
```

## Scope

These constraints apply to:

- `aithru-web`;
- future reusable `aithru-ui` packages;
- future `aithru-server` or admin/workbench browser surfaces;
- any package that exposes Aithru workflow, run, trace, approval, Agent, or bridge state to users.

Package-local install commands, package-specific examples, and generated API docs stay in implementation repositories.

## Product posture

Aithru frontend surfaces are workbench products, not marketing pages.

Default UI priorities:

1. Make workflow structure clear.
2. Make execution state visible.
3. Make validation and errors actionable.
4. Make human approval and permission boundaries explicit.
5. Make traces inspectable and auditable.
6. Keep long-running work comfortable to monitor.

The UI should favor clarity, consistency, and recoverability over decorative effects.

## Non-negotiable browser boundary

Browser UI must not own real workflow execution.

```txt
aithru-web edits and validates workflows.
aithru-personal-bridge or aithru-server executes workflows.
```

Frontend code must not import Node-only runtime or filesystem capabilities into browser bundles.

Browser-facing code may import browser-safe contracts, specs, metadata, validators, and bridge clients. It must not directly own:

- real runtime scheduling;
- local filesystem access;
- secret storage;
- durable approval state;
- concrete tool execution;
- server-side RBAC, audit, queue, or persistence semantics.

If a feature needs these capabilities, expose them through an explicit bridge/server API and make the boundary visible in the UI.

## Technical baseline

Current `aithru-web` baseline is React + Vite + TypeScript.

Cross-repo frontend rules:

- Use TypeScript for all frontend application and shared UI code.
- Do not introduce another UI framework for core workbench surfaces without updating this docs repository first.
- Do not introduce a second component system inside the same app without a documented migration plan.
- Keep browser-safe shared packages separate from Node/server-only packages.
- Treat all API and bridge responses as typed contracts, not ad hoc JSON blobs.

Framework-specific choices such as Vite, Next.js, routing libraries, table libraries, and chart libraries belong in implementation repositories, but they must respect the constraints in this document.

## App shell

Workbench pages should share a consistent shell.

```txt
AppShell
├── TopBar        global context, workspace/project selector, search, user actions
├── Sidebar       primary module navigation
├── MainArea      current page or editor
├── RightPanel    selected item details, config, validation, context
└── BottomPanel   run console, trace stream, logs, approval status when useful
```

Default rules:

- Workflow editing, run inspection, settings, task lists, and trace views should use the shared AppShell.
- Public docs, auth screens, or simple landing pages may use a lighter shell.
- The shell should preserve enough context that users know which workflow, bridge target, run, and environment they are looking at.
- Long-running execution views should not hide status behind transient toast messages only.

## Page taxonomy

Use a small set of page types instead of inventing a new layout per feature.

| Page type | Purpose | Required states |
| --- | --- | --- |
| Dashboard | Overview of recent workflows, runs, bridge/server status, and important actions. | loading, empty, degraded/offline |
| List page | Workflows, runs, agents, connectors, tools, approvals, or settings records. | loading, empty, error, pagination/filter state |
| Detail page | Single workflow, run, node, tool, bridge target, or approval item. | loading, not found, permission/availability, error |
| Editor page | Workflow graph/JSON authoring and validation. | dirty, valid, invalid, saving/exporting/importing |
| Run console | Live or historical execution view. | queued, running, waiting approval, success, failed, cancelled, timeout |
| Settings page | Local bridge, model, tool, permission, and UI configuration. | loading, saved, unsaved, validation error |

A new page type should be added only when these patterns are insufficient.

## Component layers

Keep UI components layered by responsibility.

```txt
src/
├── components/
│   ├── ui/             # primitive reusable UI components
│   ├── layout/         # AppShell, Sidebar, TopBar, panels
│   ├── common/         # EmptyState, ErrorState, LoadingState, ConfirmDialog
│   └── domain/         # shared Aithru domain components
├── features/
│   ├── workflows/
│   ├── runs/
│   ├── bridge/
│   ├── approvals/
│   ├── agents/
│   ├── tools/
│   └── settings/
├── lib/
│   ├── api/
│   ├── bridge/
│   ├── config/
│   ├── validators/
│   └── utils/
└── types/
```

Rules:

- `components/ui` must stay domain-neutral.
- `components/layout` must not own feature-specific fetching or mutation logic.
- `components/common` should provide consistent loading, empty, error, and confirmation patterns.
- `components/domain` may know Aithru concepts such as workflow, run, trace, node, approval, tool, and bridge target.
- `features/*` owns feature-specific composition, state coordination, and screen-level behavior.
- Route/page files should compose features; they should not become large business-logic containers.

## Reusable domain components

Prefer shared domain components for recurring Aithru concepts.

Recommended shared components include:

- `WorkflowStatusBadge`
- `RunStatusBadge`
- `BridgeTargetBadge`
- `ValidationPanel`
- `TraceViewer`
- `RunLogViewer`
- `ApprovalCard`
- `NodePalette`
- `NodeConfigPanel`
- `WorkflowJsonEditor`
- `ToolPermissionSummary`
- `EmptyState`
- `ErrorState`
- `ConfirmDialog`

Do not create one-off status badges, log viewers, approval cards, or validation panels per page unless there is a documented reason.

## Styling constraints

Style rules should make the UI consistent and easy for coding agents to extend.

- Use a single styling system per app.
- Centralize color, spacing, radius, typography, and shadow decisions as design tokens or theme variables.
- Avoid hard-coded hex colors inside feature components.
- Avoid inline styles except for measured geometry, canvas/graph positioning, or dynamic values that cannot be represented cleanly in the styling system.
- Use consistent iconography across modules.
- Keep animation subtle and functional: progress, expansion, focus, transition, or spatial orientation.
- Do not use animation to hide slow, failed, or uncertain states.
- Prefer information density suitable for a workbench, but avoid hiding primary actions or status behind hover-only controls.

## State model

Execution-related UI should reuse a common state vocabulary.

```txt
idle
queued
running
waiting_approval
success
failed
cancelled
timeout
degraded
offline
```

State display rules:

| State | UI expectation |
| --- | --- |
| `idle` | Neutral, no implied progress. |
| `queued` | Clearly waiting to start. |
| `running` | Live progress, stream, spinner, or current step. |
| `waiting_approval` | Prominent pending human decision with resume/cancel context. |
| `success` | Completion summary and outputs/artifacts when available. |
| `failed` | Error summary plus inspectable details. |
| `cancelled` | Explicitly user/system cancelled, not confused with failure. |
| `timeout` | Timeout reason and retry path if available. |
| `degraded` | Some capability is unavailable but the page may still function. |
| `offline` | Bridge/server unavailable; make read-only or recovery behavior clear. |

Every async surface should handle:

- loading;
- empty;
- error;
- permission or capability unavailable;
- retry or recovery path where appropriate.

## Workflow editor constraints

The workflow editor must preserve a clear source of truth.

- Editable `WorkflowSpec` remains the canonical workflow representation.
- Graph and form views must round-trip to the same workflow data model.
- Node config forms should be schema-driven when possible.
- Validation should be visible before run/export actions.
- Import/export should preserve version information.
- Unsaved local changes must be visible to the user.
- Browser-only mock execution must be labeled as mock/simulation.
- Real run actions must show the active bridge/server target.

Graph editing should avoid hidden side effects. Adding, deleting, connecting, or reconfiguring nodes should update the canonical workflow data predictably.

## Run and trace constraints

Run UI must be optimized for inspection and debugging.

- Run events should be append-oriented and inspectable.
- Trace entries should keep timestamp, event type, node/task identity, status, and relevant redacted metadata.
- Raw details should be available for debugging when safe.
- Error messages should be actionable and should not only appear in toast notifications.
- Approval pauses must show what is waiting, who/what can resume it, and what the possible decisions are.
- Historical runs should be distinguishable from live runs.
- Partial output should be visibly partial.

The UI must not imply that a run is complete before the execution owner has reported completion.

## API and bridge client constraints

Frontend code should access backends through explicit clients.

- Components should not assemble raw API URLs inline.
- Bridge/server clients should live under `lib/api`, `lib/bridge`, or the implementation repository's equivalent.
- Request and response types should be shared or derived from source contracts when practical.
- Mutations must have success and failure feedback.
- Long-running actions must return or subscribe to inspectable run state rather than hiding execution behind a single spinner.
- Offline, timeout, and unreachable bridge/server states must be represented in UI.
- Do not treat local mock responses as production bridge/server behavior.

## Forms and destructive actions

Forms should be predictable, typed, and recoverable.

- Use schema validation for non-trivial forms.
- Show field-level validation errors when possible.
- Disable or guard submit actions while a mutation is in progress.
- Preserve user input after recoverable failures.
- Destructive actions require explicit confirmation.
- Dangerous workflow/run/tool actions should include enough context in the confirmation dialog to prevent accidental execution.
- Approval decisions should be auditable and should not be triggered by ambiguous UI controls.

## Security and privacy constraints

Frontend UI must respect Aithru's permission, trace, redaction, and approval model.

- Never display secrets by default.
- Redact sensitive values in traces, logs, API inspectors, and error panels.
- Do not persist secrets in browser storage as ordinary preferences or drafts.
- Make tool permission boundaries visible before execution.
- Make bridge/server target identity visible when executing real actions.
- Do not add browser-side shortcuts that bypass `ToolPermissionPolicy`, approval, redaction, or execution ownership.

## Accessibility and usability baseline

Workbench UI should remain usable during long sessions.

- Primary actions should be reachable by keyboard.
- Dialogs, drawers, menus, and tabs should have correct focus behavior.
- Status should not be communicated by color alone.
- Important error and approval text should be readable without relying on tooltips.
- Tables and logs should support scanning, filtering, or search when they can grow large.
- Dense screens should still preserve clear visual hierarchy.

## Agent-assisted development constraints

Coding agents working on frontend changes should follow these rules.

Before implementation:

1. Read `README.md` in `aithru-docs`.
2. Read this document.
3. Read [Aithru Web](./04-aithru-web.md) for browser/runtime boundaries.
4. Read the target implementation repository README.
5. Produce a small implementation plan when the change affects layout, state, API, or component ownership.

During implementation, do not:

- create a second button/input/dialog/status component family without justification;
- import Node-only packages into browser code;
- hide execution errors behind toasts only;
- add new execution states without updating the shared state vocabulary;
- persist secrets in localStorage/sessionStorage;
- bypass bridge/server ownership for real execution;
- put feature fetching/mutation logic into primitive UI components;
- copy large feature components instead of extracting shared domain components.

After implementation, check:

- typecheck passes;
- build passes;
- loading, empty, error, and unavailable states are handled;
- browser-safe import rules still hold;
- workflow/run/approval state naming is consistent;
- docs are updated if a cross-repo frontend boundary changed.

## PR checklist

Use this checklist for meaningful frontend changes:

- [ ] Does the page use the shared shell or document why it does not?
- [ ] Are reusable UI/domain components reused instead of duplicated?
- [ ] Are loading, empty, error, and unavailable states handled?
- [ ] Are execution and approval states visible and named consistently?
- [ ] Are bridge/server targets clear for real execution actions?
- [ ] Are browser-safe import boundaries respected?
- [ ] Are secrets and sensitive trace values redacted?
- [ ] Are destructive actions confirmed?
- [ ] Are API/bridge calls typed and centralized?
- [ ] Does the change preserve the distinction between mock/browser simulation and real execution?
- [ ] Do `aithru-web` verification commands pass when relevant?

## Non-goals

This document does not choose every package-level frontend library.

It does not replace implementation repository README files, component stories, generated API docs, or visual design specs. It defines the cross-repo constraints that keep Aithru frontend surfaces coherent as the ecosystem grows.
