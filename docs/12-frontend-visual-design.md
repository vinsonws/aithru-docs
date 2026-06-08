# Frontend Visual Design

This document defines the default visual design direction for Aithru frontends.

It complements [Common Frontend Constraints](./11-frontend-constraints.md). The constraints document defines the frontend stack, surface/chrome modes, and style-system rules; this document defines the recommended product look, color palette, tokens, visual modes, and usage rules.

## One-line definition

```txt
Aithru visual design = calm technical product family with Slate base surfaces, Indigo brand actions, Cyan intelligence highlights, and mode-aware chrome density
```

## Style name

Default style name:

```txt
Aithru Slate Intelligence
```

This style is the shared visual language for the Aithru frontend family. It is intentionally flexible enough to support workflow authoring, Agent monitoring, run inspection, trace debugging, approval surfaces, admin/workbench screens, local companion UIs, embedded panels, and hosted daily app surfaces.

## Design posture

Aithru should feel like:

```txt
clear, calm, technical, trustworthy product UI; dense where workbench or admin tasks require it
```

Aithru should not feel like a marketing site, a decorative AI toy, or a generic admin template. Workbench, trace, approval, and admin surfaces may feel closer to an engineering console, workflow workbench, or AI operations dashboard. Hosted daily app surfaces may feel more native and app-like, with platform chrome visually reduced, as long as state, permission, recovery, and boundary clarity are preserved.

Default visual goals:

1. Calm base surfaces for long sessions.
2. Clear hierarchy for dense workflow, trace, admin, and integration information.
3. Strong but restrained brand color.
4. Color used primarily for action, state, risk, and execution feedback.
5. Subtle intelligence/AI feeling without decorative gradients everywhere.
6. Good light mode for configuration, reading, and daily app usage.
7. Excellent dark mode for logs, traces, consoles, and execution monitoring.
8. Chrome density that matches the user's task instead of forcing every surface into a console layout.

## Visual modes

Aithru uses one shared visual language with multiple visual modes. A mode changes chrome density, navigation visibility, and page rhythm; it must not change security, permission, execution, approval, or redaction semantics.

### Workbench visual mode

Use for:

- workflow authoring;
- Agent monitoring;
- run inspection;
- trace debugging;
- approval review;
- bridge/server connection inspection;
- developer-facing configuration.

Characteristics:

- visible shell and navigation are acceptable;
- context panels, activity panels, inspectors, and validation panels are expected;
- information density can be higher than ordinary SaaS pages;
- execution target, run status, trace state, and approval boundaries should be prominent;
- the UI should feel precise, inspectable, and comfortable for long sessions.

### Admin/ops visual mode

Use for:

- organization, member, department, group, and workspace management;
- RBAC, role assignments, grants, and permission catalogs;
- application registry, service registry, service policy, queues, and audit;
- durable server/team operation.

Characteristics:

- persistent admin navigation is allowed;
- tables, filters, detail drawers, diff/inspect panels, and audit trails are appropriate;
- permission denied, degraded, and unavailable states should be explicit;
- destructive and security-sensitive actions should be guarded with confirmations;
- the UI may look more like a control plane than a daily end-user app.

### Native hosted app visual mode

Use for:

- child apps hosted inside a platform shell;
- daily user-facing business apps;
- integration surfaces where the active app should feel like a first-class native app.

Characteristics:

- the active app owns the primary page header, navigation, toolbar, empty states, and page rhythm;
- platform chrome should be minimal and visually secondary during normal use;
- app switching, organization/account context, theme, notifications, and admin entry may live in an account drawer, command palette, compact switcher, or other low-noise affordance;
- platform integration details such as app key, origin, integration mode, token type, message bus, and service target should stay hidden during normal use;
- those details should become visible in recovery, permission, boundary, or debug/inspector states;
- the page should not look like an admin console containing an app unless the user intentionally entered an admin or inspector flow.

### Recovery/boundary visual mode

Use for:

- auth expired;
- permission denied;
- approval unavailable;
- bridge/server/local host unavailable;
- child app offline;
- origin or integration mismatch;
- token exchange failure;
- message bus failure.

Characteristics:

- platform identity, execution target, app identity, and boundary information may become explicit;
- show enough context to recover safely without exposing secrets;
- prefer clear recovery actions over generic error toasts;
- keep raw details inspectable for operators when safe.

### Embedded/compact visual mode

Use for:

- mobile/PWA subsets;
- embedded approval widgets;
- compact trace or status panels;
- reusable UI packages inside another shell.

Characteristics:

- reduce chrome and navigation;
- preserve state labels, permission clarity, and recovery paths;
- avoid copying the full desktop workbench into a cramped surface;
- provide a path to inspect details elsewhere when the compact view summarizes sensitive or complex state.

## Primary palette

| Role | Color family | Default hex | Usage |
| --- | --- | --- | --- |
| Base | Slate | `#f8fafc` / `#020617` | Page backgrounds in light/dark mode. |
| Surface | White / Slate | `#ffffff` / `#0f172a` | Cards, panels, dialogs, sidebars, drawers. |
| Border | Slate | `#e2e8f0` / `#1e293b` | Region separation and low-noise structure. |
| Primary | Indigo | `#4f46e5` / `#818cf8` | Main actions, selected navigation, focused workflow nodes, platform-level selection. |
| Intelligence accent | Cyan | `#06b6d4` / `#22d3ee` | Running states, streaming, AI/Agent activity, live trace highlights. |
| Success | Emerald | `#10b981` / `#34d399` | Completed runs, passing validation, successful actions. |
| Warning | Amber | `#f59e0b` / `#fbbf24` | Waiting approval, caution, degraded state. |
| Destructive | Rose | `#f43f5e` / `#fb7185` | Failure, deletion, blocked execution, dangerous actions. |

Default recommendation:

```txt
Base: Slate
Primary: Indigo
AI Accent: Cyan
Success: Emerald
Warning: Amber
Destructive: Rose
```

Do not use every color as decoration. Color should communicate product meaning.

Hosted daily apps may introduce an app-specific accent only when it maps to semantic tokens and does not replace shared state colors. For example, an app may style its own primary action with an app accent, but failure, warning, success, running, and approval states should still follow the shared state color rules.

## Light theme tokens

Use light mode for configuration, workflow authoring, admin tables, daily hosted apps, and general reading.

```css
:root {
  --background: #f8fafc;
  --foreground: #0f172a;

  --card: #ffffff;
  --card-foreground: #0f172a;

  --muted: #f1f5f9;
  --muted-foreground: #64748b;

  --border: #e2e8f0;
  --input: #cbd5e1;

  --primary: #4f46e5;
  --primary-foreground: #ffffff;

  --accent: #06b6d4;
  --accent-foreground: #ecfeff;

  --success: #10b981;
  --warning: #f59e0b;
  --destructive: #f43f5e;

  --ring: #6366f1;
  --radius: 0.625rem;
}
```

Light mode should avoid a pure-white page background in workbench and admin surfaces. Hosted app surfaces may use white or near-white app canvases when the active app needs a native product feel, but panels, floating controls, and recovery surfaces should still use shared tokens.

## Dark theme tokens

Use dark mode for trace inspection, run consoles, long monitoring sessions, and developer-focused screens. Other surfaces should still remain usable in dark mode when the product supports theme switching.

```css
.dark {
  --background: #020617;
  --foreground: #e5e7eb;

  --card: #0f172a;
  --card-foreground: #f8fafc;

  --muted: #111827;
  --muted-foreground: #94a3b8;

  --border: #1e293b;
  --input: #334155;

  --primary: #818cf8;
  --primary-foreground: #0f172a;

  --accent: #22d3ee;
  --accent-foreground: #083344;

  --success: #34d399;
  --warning: #fbbf24;
  --destructive: #fb7185;

  --ring: #818cf8;
  --radius: 0.625rem;
}
```

Dark mode should not use pure black as the main background. Deep Slate keeps the interface more comfortable and more premium during long trace/log sessions.

## Tailwind mapping

Aithru frontends should map Tailwind colors to semantic CSS variables.

```ts
theme: {
  extend: {
    colors: {
      background: "var(--background)",
      foreground: "var(--foreground)",
      card: "var(--card)",
      "card-foreground": "var(--card-foreground)",
      muted: "var(--muted)",
      "muted-foreground": "var(--muted-foreground)",
      border: "var(--border)",
      input: "var(--input)",
      primary: "var(--primary)",
      "primary-foreground": "var(--primary-foreground)",
      accent: "var(--accent)",
      "accent-foreground": "var(--accent-foreground)",
      success: "var(--success)",
      warning: "var(--warning)",
      destructive: "var(--destructive)",
      ring: "var(--ring)",
    },
    borderRadius: {
      lg: "var(--radius)",
      md: "calc(var(--radius) - 2px)",
      sm: "calc(var(--radius) - 4px)",
    },
  },
}
```

Feature modules should consume semantic classes such as `bg-background`, `bg-card`, `text-foreground`, `border-border`, `text-muted-foreground`, `bg-primary`, and `text-destructive`.

Do not scatter raw color classes such as `text-blue-500`, `bg-red-100`, or hard-coded hex colors inside feature components unless the usage is part of a documented visualization scale.

## Surface design rules

### Page background

- Use `background` for full-page workbench, admin, trace, and recovery surfaces.
- Avoid pure white in light mode workbench/admin page backgrounds.
- Avoid pure black in dark mode page backgrounds.
- Keep the page calm enough for long-running trace and workflow sessions.
- Hosted app canvases may use lighter native-app backgrounds when the active app owns the main experience.

### Cards and panels

- Use `card` for primary content panels.
- Prefer thin borders over heavy shadows.
- Use subtle background changes to separate regions.
- Keep radius consistent across cards, buttons, dialogs, inputs, and panels.
- Use shadow sparingly, mostly for floating panels, dialogs, menus, drawers, and overlays.
- In native hosted app mode, avoid wrapping the entire child app in a heavy card or obvious container unless it is a recovery or inspector state.

### Navigation

- Active workbench/admin navigation should use Primary/Indigo.
- Secondary navigation should use muted foreground and border contrast.
- Avoid decorative gradients in navigation.
- Workspace, project, profile, and execution target should remain visible in workbench shells.
- In native hosted app mode, platform navigation may collapse into a command palette, app switcher, account drawer, or compact affordance so the active app can feel native.

### Typography

Default type direction:

```txt
UI text: Inter or system-ui
Code/logs/IDs: JetBrains Mono or ui-monospace
```

Rules:

- Use normal font weight for dense data.
- Use medium/semi-bold for headings, labels, and active states.
- Avoid oversized marketing-style headings inside workbench pages.
- Hosted app surfaces may use more spacious product-style headings when the page is not a dense workbench or admin surface.
- Use monospace for run IDs, node IDs, trace event IDs, logs, JSON, code snippets, app keys, service IDs, and token/debug identifiers.

### Density

Aithru density should match the visual mode.

- Workbench and admin/ops surfaces should be denser than a marketing site but not cramped.
- Hosted app surfaces may use a lighter, more native app rhythm for ordinary user tasks.
- Prefer compact controls in tables, traces, logs, and side panels.
- Preserve readable spacing in forms and approval decisions.
- Use grouping, sticky headers, panels, and progressive disclosure to manage dense screens.
- Do not solve density by making everything tiny.

## State color rules

Color must communicate state consistently across frontends.

| State | Color role | Visual treatment |
| --- | --- | --- |
| `idle` | muted | Neutral text/badge, no implied progress. |
| `queued` | muted + foreground | Waiting indicator, low urgency. |
| `running` | accent / Cyan | Live pulse, border, spinner, or active trace marker. |
| `waiting_approval` | warning / Amber | Prominent but not destructive. |
| `success` | success / Emerald | Completion badge, validation passed. |
| `failed` | destructive / Rose | Error summary and inspectable details. |
| `cancelled` | muted | Clearly cancelled, not failed. |
| `timeout` | warning | Retry/recovery path should be visible. |
| `degraded` | warning | Capability issue but not necessarily failure. |
| `offline` | muted + warning when needed | Bridge/server/local host unavailable. |

Status UI must not rely on color alone. Use labels, icons, descriptions, or structured sections.

## Workflow graph visual design

Workflow graph/editor is a core Aithru surface and should have dedicated visual rules.

| Element | Default visual treatment |
| --- | --- |
| Canvas background | Slightly different from page background. |
| Grid | Border color at low opacity. |
| Default node | Card background, thin border, compact metadata. |
| Selected node | Indigo ring/border. |
| Running node | Cyan border, live indicator, optional subtle glow. |
| Successful node | Emerald status badge, not a fully green card. |
| Failed node | Rose status badge and error affordance. |
| Waiting approval node | Amber status badge and approval affordance. |
| Edges | Muted by default; Cyan when active/running. |
| Invalid edge/node | Rose marker with validation details. |
| Disabled/unavailable node | Muted style with explanation. |

Rules:

- Do not make graph nodes randomly colorful by node type.
- Use color primarily for execution state, validation state, and selection.
- Node type can be communicated through icons, labels, subtle badges, or grouping.
- Active execution paths may use Cyan, but the glow should be subtle.
- Failed paths should be easy to locate without turning the whole graph into an error wall.

## Trace and console visual design

Trace and console views should feel precise and inspectable.

- Use dark mode as the preferred trace/log viewing mode when practical.
- Use monospace for raw logs, JSON, IDs, and event payloads.
- Use structured rows for trace events: timestamp, event type, node/task/agent, status, summary, expandable details.
- Use Cyan for active streaming/running events.
- Use Amber for waiting approval/degraded events.
- Use Rose for failure/error events.
- Do not hide errors in toast messages only.
- Do not overuse terminal-green; Aithru is a workbench, not a retro terminal theme.

## Approval visual design

Approval surfaces should look trustworthy and explicit.

- Waiting approval should use Amber.
- Dangerous approval decisions should use Destructive/Rose.
- Approval cards must show what is being approved, why it is needed, what tool/target is involved, and what happens after approval.
- Primary approval action should be visually clear, but not dangerously easy to click by accident.
- Reject/cancel actions should be visually distinct from destructive system actions.

## Hosted app visual design

Hosted app surfaces are allowed to feel less like a platform shell and more like native applications, but they still live inside Aithru's identity, authorization, trace, redaction, and recovery boundary.

Rules:

- Do not show a permanent admin sidebar in ordinary hosted-app pages.
- Do not make the hosting platform chrome visually dominate the child app during normal use.
- Do not surround the child app with a heavy iframe-looking frame unless the surface is in recovery, boundary, or inspector mode.
- The active app may own its own logo, header, navigation, toolbar, and empty states.
- Platform app switching, organization switching, account settings, theme, and admin entry should remain reachable through low-noise controls.
- Permission denied, auth expired, app offline, origin mismatch, and token exchange failure states should reveal enough platform context to recover.
- Secrets, refresh tokens, service client secrets, bearer tokens, and sensitive trace values must remain redacted.

## Recommended component variants

Primitive and common components should expose variants rather than requiring pages to assemble raw Tailwind class strings.

Recommended variants:

```txt
Button: primary, secondary, outline, ghost, destructive
Badge: neutral, primary, accent, success, warning, destructive
Card: default, muted, elevated, selected, interactive
Alert: info, success, warning, destructive
Panel: default, sidebar, activity, floating, inspector
StatusBadge: idle, queued, running, waiting_approval, success, failed, cancelled, timeout, degraded, offline
```

Feature pages should prefer these variants over one-off class combinations.

## Things to avoid

Avoid these directions unless a specific product surface justifies them:

- heavy gradients as a default background;
- bright neon cyberpunk UI;
- pure black terminal-only design;
- large glassmorphism panels everywhere;
- colorful graph nodes by default;
- large shadows on every card;
- marketing landing-page spacing in dense workbench screens;
- permanent admin-console chrome around ordinary hosted app pages;
- iframe-looking borders around hosted apps during normal use;
- using red/orange/yellow for decoration instead of state;
- mixing Ant Design, Material UI, Chakra, or another full component library into the same app;
- making dark mode the only polished mode.

## Alternative palettes

The default is Aithru Slate Intelligence. These alternatives are allowed only with a design note.

| Palette | Colors | When to use | Risk |
| --- | --- | --- | --- |
| Slate + Indigo + Cyan | Current default | Main workbench, Agent, workflow, trace, admin/ops, and shared platform chrome. | Lowest risk. |
| Zinc + Blue + Emerald | More traditional SaaS | Admin-heavy surfaces where familiarity matters. | Less brand recognition. |
| Neutral + Violet + Sky | More AI-product feeling | Lighter assistant/Agent-focused or hosted-app surfaces. | Can become decorative or trendy. |

If there is no strong reason, use the default palette. App-specific accents are allowed only as semantic token extensions with a short design note.

## Implementation checklist

Before a frontend style implementation is accepted, check:

- [ ] Uses Aithru Slate Intelligence as the default palette.
- [ ] Declares the visual mode for meaningful surface-level work: workbench, admin/ops, native hosted app, recovery/boundary, or embedded/compact.
- [ ] Defines light and dark theme tokens.
- [ ] Maps Tailwind to semantic CSS variables.
- [ ] Avoids hard-coded feature-level hex colors.
- [ ] Uses Indigo for platform-level primary actions and selection.
- [ ] Uses Cyan for running/AI/live activity.
- [ ] Uses Emerald/Amber/Rose only for semantic state.
- [ ] Uses thin borders and subtle surfaces instead of heavy shadows.
- [ ] Keeps graph color primarily tied to selection, validation, and execution state.
- [ ] Provides readable trace/log/console views.
- [ ] Keeps both light and dark modes polished.
- [ ] Avoids making daily hosted app pages look like admin consoles unless the user is in admin, recovery, or inspector mode.