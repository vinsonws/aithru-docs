# Common Frontend Constraints

This document defines common frontend constraints for all Aithru user-facing surfaces.

It is not only an `aithru-web` document. `aithru-web` is one frontend implementation. Future web consoles, admin/workbench UIs, desktop shells, local companion UIs, mobile/PWA surfaces, hosted-app shells, embedded panels, and reusable UI packages should follow the same product, state, component, security, style, technology-stack, and execution-boundary rules unless a platform-specific document explicitly overrides them.

## One-line definition

```txt
Aithru frontend = user-facing product surfaces for authoring, configuring, hosting, inspecting, approving, and monitoring Aithru workflows, agents, runs, tools, integrations, and platform-hosted apps
```

## Scope

These constraints apply to:

- `aithru-web`;
- future reusable `aithru-ui` packages;
- future `aithru-server` admin or team workbench UIs;
- future platform shells that host child applications or integration surfaces;
- future desktop or local companion UIs for personal execution;
- future mobile or PWA surfaces that expose a subset of Aithru capabilities;
- embedded panels or widgets that expose workflow, run, trace, approval, Agent, tool, or bridge/server state to users.

Package-local install commands, framework-specific examples, native platform setup, and generated API docs stay in implementation repositories.

## Constraint levels

Use these levels when interpreting frontend decisions.

| Level | Meaning |
| --- | --- |
| Must | Required across Aithru frontends unless this docs repository is updated first. |
| Default | Preferred choice for new implementation. Deviations need a short written reason in the target repository. |
| Decision required | Allowed only after a docs update, ADR, or explicit implementation design note. |

Do not treat this document as loose advice. It is the common frontend baseline for people and coding agents.

## Shared product posture

Aithru frontend surfaces are product and work surfaces, not marketing pages.

Default UI priorities:

1. Make workflow and Agent structure clear.
2. Make execution state visible.
3. Make validation and errors actionable.
4. Make human approval and permission boundaries explicit.
5. Make traces inspectable and auditable.
6. Keep long-running work comfortable to monitor.
7. Keep platform differences predictable rather than visually or behaviorally fragmented.
8. Keep host/platform chrome proportional to the user's current task.

The UI should favor clarity, consistency, recoverability, and trust over decorative effects.

## Frontend family

Aithru may have multiple frontends with different capabilities.

| Frontend type | Typical role | Shared expectation |
| --- | --- | --- |
| Web workbench | Author workflows, inspect runs, connect to bridge/server. | Browser-safe, clear mock-vs-real execution boundary. |
| Server/admin console | Manage team workspaces, users, runs, audit, connectors, and durable execution. | Strong RBAC, audit, pagination, filtering, and operational clarity. |
| Hosted app shell | Host child apps or integration surfaces inside a shared identity, authorization, theme, and audit boundary. | The active app may own the primary page chrome while platform controls stay available and recoverable. |
| Desktop/local companion | Provide a trusted local UI around a personal bridge or local host. | UI and execution host may ship together, but boundaries must remain explicit. |
| Mobile/PWA surface | Monitor runs, approve tasks, review alerts, perform lightweight edits. | Prefer focused workflows over full authoring complexity. |
| Embedded UI package | Reusable workflow, node, trace, or approval components. | Domain components must be portable and not assume one app shell. |

A new frontend should declare which type it is, which capabilities it supports, and which execution owner it talks to.

## Surface and chrome modes

Aithru frontends should choose a chrome mode according to the user's task. This prevents every surface from becoming a dense console while preserving the same security, state, and execution-boundary semantics.

| Mode | Typical surfaces | Rule |
| --- | --- | --- |
| Workbench mode | Workflow authoring, Agent inspection, run console, trace viewer. | Shared shell, navigation, context panel, and activity panel may be visible by default. |
| Admin/ops mode | Users, RBAC, apps, grants, services, audit, queues, server operation. | Dense console UI, tables, filters, sidebars, inspectors, and audit context are appropriate. |
| Native hosted app mode | Daily user-facing hosted apps or child apps inside a platform shell. | The active app owns the primary page chrome; platform identity, org, app switching, auth, and recovery controls should be available but visually secondary. |
| Recovery/boundary mode | Auth expired, permission denied, app offline, token exchange failed, bridge/server unreachable. | Platform boundary, execution target, token/app identity, and recovery actions may become explicit and actionable. |
| Embedded/compact mode | Approval widgets, mobile/PWA subsets, embedded panels, compact inspectors. | Preserve state, permission, and recovery clarity while reducing chrome. |

Rules:

- A surface that deviates from the default workbench shell should declare its chrome mode in an implementation design note, route file, story, or package README.
- A lighter chrome mode must not hide permission, approval, execution-target, or recovery boundaries when they matter.
- Admin and debug affordances should not dominate daily hosted-app pages unless the user intentionally enters an admin, recovery, or inspector surface.
- The same shared state vocabulary, status components, semantic tokens, and typed clients should be reused across modes.

## Execution ownership boundary

Frontend code must not own Aithru execution semantics.

```txt
Frontend surfaces present, author, configure, inspect, and approve.
Execution hosts run, persist, schedule, resume, and enforce policy.
```

Execution owners may include:

- `aithru-personal-bridge` for personal/local execution;
- future `aithru-server` for durable team execution;
- package-level mocks for UI development and demos;
- other explicitly documented execution hosts.

Rules:

- UI code must not duplicate workflow scheduler semantics.
- UI code must not bypass `ToolPermissionPolicy`, approval, redaction, or trace ownership.
- UI code must make the active execution target visible before real execution actions.
- Mock/simulation execution must be labeled as mock/simulation.
- If a desktop product bundles a local execution host, the UI layer still talks to that host through an explicit IPC/local API boundary instead of directly owning execution internals.
- If a web product calls a bridge/server, the bridge/server owns real execution, persistence, secrets, and resume behavior.

## Platform-specific boundaries

Common constraints apply everywhere, but each platform has additional boundaries.

### Browser-based frontends

Browser-based frontends must not import Node-only runtime or filesystem capabilities into browser bundles.

Browser-facing code may import browser-safe contracts, specs, metadata, validators, UI packages, and clients. It must not directly own:

- real runtime scheduling;
- local filesystem access;
- secret storage;
- durable approval state;
- concrete tool execution;
- server-side RBAC, audit, queue, or persistence semantics.

If a browser feature needs these capabilities, expose them through an explicit bridge/server API and make the boundary visible in the UI.

### Desktop or local companion frontends

Desktop/local frontends may access stronger local capabilities only through documented application boundaries.

- Prefer IPC/local HTTP/loopback APIs between UI and local execution host.
- Keep secrets, filesystem access, tool execution, and runtime state outside primitive UI components.
- Make the current local host, profile, workspace, and execution mode visible.
- Do not rely on desktop privileges to skip permission prompts, approval, or redaction.

### Mobile/PWA frontends

Mobile/PWA frontends should expose a capability subset rather than a cramped copy of the full workbench.

- Prioritize monitoring, approval, alerts, summaries, and lightweight configuration.
- Avoid complex graph editing unless the interaction model is explicitly designed for small screens.
- Preserve the same execution state vocabulary and approval semantics as other frontends.

## Technology stack constraints

Implementation-specific details stay in implementation repositories, but the Aithru frontend family should not fragment without a reason.

### Must

- Use typed contracts for workflow, run, trace, approval, tool, Agent, workspace, user, and execution-target data.
- Treat API, IPC, and bridge/server responses as contracts, not ad hoc JSON blobs.
- Use TypeScript for web apps, desktop webview apps, shared UI packages, shared client packages, and frontend build tooling unless a native platform requires another language.
- Keep browser-safe shared packages separate from Node/server-only packages.
- Keep reusable domain components portable enough to support future shared `aithru-ui` packages.
- Do not introduce a second component system inside the same app without a documented migration plan.
- Do not introduce another primary frontend framework for core workbench surfaces without updating this docs repository first.

### Default stack for web and desktop-webview frontends

| Concern | Default choice | Notes |
| --- | --- | --- |
| Language | TypeScript | Required for shared web/desktop UI and clients. |
| UI framework | React | Default for workbench, admin, desktop webview, and reusable UI packages. |
| Client workbench build | Vite | Default for browser-safe client workbench apps such as `aithru-web`. |
| Server-backed web app | Next.js | Use when the product needs server-side routing, auth/session integration, or team/admin console behavior. |
| Styling | Tailwind CSS + CSS variables | Tokens must be centralized. Do not scatter hard-coded colors. |
| Primitive UI | shadcn/ui + Radix UI | Default for web/desktop webview primitive components. |
| Icons | lucide-react | Keep icon style consistent across modules. |
| Forms | React Hook Form + Zod | Non-trivial forms should have schema validation. |
| Server/bridge state | TanStack Query | Prefer explicit query/mutation hooks over ad hoc component-level fetching. |
| Tables | TanStack Table | Default for data-heavy lists, admin grids, runs, traces, and audit views. |
| Workflow graph | React Flow or documented equivalent | Use one graph engine per app; shared graph components should hide engine details. |
| JSON/code editing | One canonical editor component per app | Monaco or CodeMirror may be chosen per implementation; do not mix casually. |
| Charts | Recharts by default | Use ECharts only for complex interactive analytics that justify the extra weight. |

### Shared UI package constraints

A future `aithru-ui` package should be:

- React + TypeScript first;
- framework-neutral within the React ecosystem, meaning it must not require Next.js APIs;
- styling-token driven;
- independent of one app shell unless a component is explicitly a shell component;
- independent of a specific bridge/server implementation;
- safe to consume from web workbench, admin console, hosted app shells, and desktop-webview frontends.

### Decision required

These choices require a docs update, ADR, or explicit implementation design note before adoption in a core Aithru frontend:

- Vue, Svelte, Angular, Solid, or another primary UI framework;
- Material UI, Ant Design, Chakra UI, Bootstrap, Mantine, or another full component system;
- CSS-in-JS as the primary styling system;
- a second table, form, query, graph, or chart library in the same app;
- a visual design system that does not use the common Aithru tokens;
- native mobile UI that changes domain vocabulary or execution state semantics;
- direct API calls from feature components instead of centralized clients/hooks.

## Style system constraints

Aithru should feel like one product family even when it has multiple frontends.

### Visual direction

Default visual posture:

```txt
clear, calm, technical, trustworthy product UI; dense where workbench or admin tasks require it
```

Use restrained decoration. Workbench, run, trace, and admin surfaces should feel closer to an engineering console or workflow workbench than to a consumer landing page. Daily hosted-app surfaces may use lighter chrome when the active app needs to feel native.

### Must

- Use one styling system per app.
- Centralize product-level tokens for color, typography, spacing, radius, density, shadow/elevation, border, and z-index.
- Use semantic tokens such as `background`, `foreground`, `muted`, `border`, `accent`, `success`, `warning`, `destructive`, and `ring` instead of feature-specific raw colors.
- Avoid hard-coded hex colors inside feature components.
- Avoid inline styles except for measured geometry, graph/canvas positioning, native shell integration, or dynamic values that cannot be represented cleanly in the styling system.
- Use consistent iconography across modules and frontends.
- Status must not be communicated by color alone.
- Do not use animation to hide slow, failed, or uncertain states.

### Default token posture

- Base surfaces should be neutral and low-noise.
- Accent color should highlight primary actions, active navigation, selected nodes, and important focus states.
- Success/warning/destructive colors should be reserved for state and risk, not decoration.
- Borders and subtle backgrounds should separate workbench regions instead of heavy shadows.
- Radius should be consistent across cards, panels, buttons, dialogs, and inputs.
- Spacing should follow a predictable scale; avoid arbitrary one-off spacing in feature components.
- Dense pages should still preserve hierarchy through section headers, grouping, sticky context, and progressive disclosure.
- Lighter chrome surfaces should still expose recoverable paths to app switching, account/org context, and boundary diagnostics.

### Tailwind and shadcn rules for web/desktop-webview

For React web and desktop-webview frontends:

- Tailwind CSS is the default styling layer.
- shadcn/ui + Radix UI are the default primitive component foundation.
- `components/ui` should be treated as the primitive layer and should stay domain-neutral.
- Do not create custom primitives that duplicate shadcn/Radix behavior unless the base primitive is insufficient.
- Do not mix another large component library into the same app for convenience.
- Prefer variant-driven components over copy-pasted class strings for buttons, badges, cards, alerts, and status indicators.
- Keep theme tokens in the app-level style/theme files; feature modules should consume tokens, not define them.

## Shared shell model

Workbench-class frontends should share a recognizable shell model even when the exact layout differs by platform.

```txt
WorkbenchShell
├── GlobalContext      workspace/project/profile, bridge/server target, environment
├── Navigation         primary module navigation
├── MainSurface        current page, editor, dashboard, or console
├── ContextPanel       selected item details, config, validation, help, or metadata
└── ActivityPanel      run console, trace stream, logs, approvals, alerts when useful
```

Default rules:

- Workflow editing, run inspection, settings, task lists, approvals, and trace views should use the shared shell model.
- Admin/ops surfaces may use persistent navigation, filters, tables, and inspectors when they improve operational clarity.
- Native hosted app surfaces may let the active app own primary chrome, but platform context, authorization recovery, and boundary diagnostics must remain accessible.
- Mobile or embedded surfaces may collapse shell regions, but they should not remove execution context or status clarity.
- Public docs, auth screens, or simple landing pages may use a lighter shell.
- The shell should preserve enough context that users know which workflow, workspace, bridge/server target, run, app, organization, and environment they are looking at when those concepts are relevant.
- Long-running execution views should not hide status behind transient toast messages only.

## Page taxonomy

Use a small set of page types instead of inventing a new layout per feature.

| Page type | Purpose | Required states |
| --- | --- | --- |
| Dashboard | Overview of recent workflows, runs, bridge/server status, and important actions. | loading, empty, degraded/offline |
| List page | Workflows, runs, agents, connectors, tools, approvals, users, or settings records. | loading, empty, error, pagination/filter state |
| Detail page | Single workflow, run, node, tool, bridge target, user, workspace, app, or approval item. | loading, not found, permission/availability, error |
| Editor page | Workflow graph/JSON/form authoring and validation. | dirty, valid, invalid, saving/exporting/importing |
| Run console | Live or historical execution view. | queued, running, waiting approval, success, failed, cancelled, timeout |
| Hosted app page | A child app or integration surface hosted inside a shared platform boundary. | loading, ready, permission denied, auth expired, degraded/offline, origin/integration unavailable |
| Approval surface | Human decision point on desktop, web, mobile, or embedded UI. | pending, accepted, rejected, expired/cancelled, unavailable |
| Settings page | Local bridge, server, model, tool, permission, account, and UI configuration. | loading, saved, unsaved, validation error |
| Admin/ops page | Team, RBAC, audit, connector, queue, or server operation. | loading, empty, degraded, permission denied, error |

A new page type should be added only when these patterns are insufficient.

## Component layers

Keep UI components layered by responsibility.

```txt
src/
├── components/
│   ├── ui/             # primitive reusable UI components
│   ├── layout/         # shell, navigation, panels, responsive layout
│   ├── common/         # EmptyState, ErrorState, LoadingState, ConfirmDialog
│   └── domain/         # shared Aithru domain components
├── features/
│   ├── workflows/
│   ├── runs/
│   ├── bridge-or-server/
│   ├── approvals/
│   ├── agents/
│   ├── tools/
│   ├── users-and-workspaces/
│   └── settings/
├── lib/
│   ├── api-or-ipc/
│   ├── clients/
│   ├── config/
│   ├── validators/
│   └── utils/
└── types/
```

Rules:

- `components/ui` must stay domain-neutral.
- `components/layout` must not own feature-specific fetching or mutation logic.
- `components/common` should provide consistent loading, empty, error, and confirmation patterns.
- `components/domain` may know Aithru concepts such as workflow, run, trace, node, approval, tool, Agent, workspace, app, organization, and bridge/server target.
- `features/*` owns feature-specific composition, state coordination, and screen-level behavior.
- Route/page/window files should compose features; they should not become large business-logic containers.
- Reusable components should not assume that the only consumer is `aithru-web`.

## Reusable domain components

Prefer shared domain components for recurring Aithru concepts.

Recommended shared components include:

- `WorkflowStatusBadge`
- `RunStatusBadge`
- `ExecutionTargetBadge`
- `BridgeTargetBadge`
- `ServerTargetBadge`
- `ValidationPanel`
- `TraceViewer`
- `RunLogViewer`
- `ApprovalCard`
- `NodePalette`
- `NodeConfigPanel`
- `WorkflowJsonEditor`
- `ToolPermissionSummary`
- `AgentRunSummary`
- `WorkspaceSwitcher`
- `EmptyState`
- `ErrorState`
- `ConfirmDialog`

Do not create one-off status badges, log viewers, approval cards, validation panels, or execution-target indicators per frontend unless there is a documented reason.

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
| `degraded` | Some capability is unavailable but the surface may still function. |
| `offline` | Bridge/server/local host unavailable; make read-only or recovery behavior clear. |

Every async surface should handle:

- loading;
- empty;
- error;
- permission or capability unavailable;
- retry or recovery path where appropriate.

## Workflow authoring constraints

Workflow authoring surfaces must preserve a clear source of truth.

- Editable `WorkflowSpec` remains the canonical workflow representation.
- Graph, JSON, and form views must round-trip to the same workflow data model.
- Node config forms should be schema-driven when possible.
- Validation should be visible before run/export/deploy actions.
- Import/export should preserve version information.
- Unsaved local changes must be visible to the user.
- Mock execution must be labeled as mock/simulation.
- Real run actions must show the active execution target.

Graph editing should avoid hidden side effects. Adding, deleting, connecting, or reconfiguring nodes should update the canonical workflow data predictably.

## Run and trace constraints

Run UI must be optimized for inspection and debugging across frontends.

- Run events should be append-oriented and inspectable.
- Trace entries should keep timestamp, event type, node/task/agent identity, status, and relevant redacted metadata.
- Raw details should be available for debugging when safe and appropriate for the surface.
- Error messages should be actionable and should not only appear in toast notifications.
- Approval pauses must show what is waiting, who/what can resume it, and what the possible decisions are.
- Historical runs should be distinguishable from live runs.
- Partial output should be visibly partial.
- Mobile or compact surfaces may summarize traces, but they should provide a path to inspect details elsewhere.

The UI must not imply that a run is complete before the execution owner has reported completion.

## API, IPC, and bridge/server client constraints

Frontend code should access execution owners and backends through explicit clients.

- Components should not assemble raw API, IPC, or bridge URLs inline.
- Clients should live under `lib/api`, `lib/ipc`, `lib/bridge`, `lib/server`, or the implementation repository's equivalent.
- Request and response types should be shared or derived from source contracts when practical.
- Mutations must have success and failure feedback.
- Long-running actions must return or subscribe to inspectable run state rather than hiding execution behind a single spinner.
- Offline, timeout, permission denied, and unreachable target states must be represented in UI.
- Do not treat local mock responses as production bridge/server behavior.

## Forms and destructive actions

Forms should be predictable, typed, and recoverable.

- Use schema validation for non-trivial forms.
- Show field-level validation errors when possible.
- Disable or guard submit actions while a mutation is in progress.
- Preserve user input after recoverable failures.
- Destructive actions require explicit confirmation.
- Dangerous workflow/run/tool/admin actions should include enough context in the confirmation dialog to prevent accidental execution.
- Approval decisions should be auditable and should not be triggered by ambiguous UI controls.

## Security and privacy constraints

Frontend UI must respect Aithru's permission, trace, redaction, and approval model.

- Never display secrets by default.
- Redact sensitive values in traces, logs, API inspectors, IPC inspectors, and error panels.
- Do not persist secrets in ordinary browser storage, local UI preferences, or frontend drafts.
- Make tool permission boundaries visible before execution.
- Make execution target identity visible when executing real actions.
- Do not add UI-side shortcuts that bypass `ToolPermissionPolicy`, approval, redaction, RBAC, audit, or execution ownership.
- Admin/team frontends must make permission denied states explicit rather than silently hiding failed actions.

## Accessibility and usability baseline

Workbench UI should remain usable during long sessions.

- Primary actions should be reachable by keyboard where the platform supports it.
- Dialogs, drawers, menus, tabs, and native windows should have correct focus behavior.
- Status should not be communicated by color alone.
- Important error and approval text should be readable without relying on tooltips.
- Tables and logs should support scanning, filtering, or search when they can grow large.
- Dense screens should still preserve clear visual hierarchy.
- Mobile surfaces should preserve approval clarity and error readability even when summarizing details.

## Agent-assisted development constraints

Coding agents working on frontend changes should follow these rules.

Before implementation:

1. Read `README.md` in `aithru-docs`.
2. Read this document.
3. Read the platform-specific frontend document when one exists, such as [Aithru Web](./04-aithru-web.md).
4. Read the target implementation repository README.
5. Produce a small implementation plan when the change affects layout, state, API/IPC, execution target, security, styling, technology stack, component ownership, or chrome mode.

During implementation, do not:

- assume `aithru-web` is the only frontend;
- create a second button/input/dialog/status component family without justification;
- import Node-only packages into browser code;
- hide execution errors behind toasts only;
- add new execution states without updating the shared state vocabulary;
- persist secrets in localStorage/sessionStorage or ordinary UI preferences;
- bypass bridge/server/local-host ownership for real execution;
- put feature fetching/mutation logic into primitive UI components;
- copy large feature components instead of extracting shared domain components;
- make shared components depend on one app shell unless that is the component's explicit purpose;
- introduce a non-default styling system, component library, or primary frontend framework without a decision note;
- force daily hosted-app surfaces into admin/ops chrome unless the user is intentionally in an admin, recovery, or inspector flow.

After implementation, check:

- typecheck passes;
- build passes;
- loading, empty, error, permission, and unavailable states are handled;
- platform-specific import/runtime rules still hold;
- workflow/run/approval state naming is consistent;
- execution target identity is visible where real actions occur;
- chosen stack and styling choices match this document;
- the chosen surface/chrome mode is appropriate for the user task;
- docs are updated if a cross-frontend boundary changed.

## PR checklist

Use this checklist for meaningful frontend changes:

- [ ] Is the target frontend type clear: web, admin/server, hosted app shell, desktop/local, mobile/PWA, embedded, or shared UI?
- [ ] Is the surface/chrome mode clear: workbench, admin/ops, native hosted app, recovery/boundary, or embedded/compact?
- [ ] Does the page/surface use the shared shell model when it is workbench-class, or document why a lighter mode is appropriate?
- [ ] Does the implementation follow the default stack or document a justified deviation?
- [ ] Does the implementation use the shared style tokens and one styling system?
- [ ] Are reusable UI/domain components reused instead of duplicated?
- [ ] Are loading, empty, error, permission, and unavailable states handled?
- [ ] Are execution and approval states visible and named consistently?
- [ ] Are execution targets clear for real actions?
- [ ] Are platform-specific boundaries respected, especially browser-safe imports for browser surfaces?
- [ ] Are secrets and sensitive trace values redacted?
- [ ] Are destructive actions confirmed?
- [ ] Are API/IPC/bridge/server calls typed and centralized?
- [ ] Does the change preserve the distinction between mock/simulation and real execution?
- [ ] Do relevant implementation-repository verification commands pass?

## Non-goals

This document does not replace implementation repository README files, component stories, generated API docs, platform-specific frontend specs, or visual design specs.

It defines the common cross-frontend constraints that keep Aithru user-facing surfaces coherent as the ecosystem grows. It intentionally sets default technology and style-system choices for core Aithru frontend work so that future frontends do not fragment accidentally.