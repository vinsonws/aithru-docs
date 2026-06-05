# Frontend Visual Design

This document defines the default visual design direction for Aithru frontends.

It complements [Common Frontend Constraints](./11-frontend-constraints.md). The constraints document defines the frontend stack and style-system rules; this document defines the recommended product look, color palette, tokens, and usage rules.

## One-line definition

```txt
Aithru visual design = Slate-based technical workbench UI with Indigo brand actions and Cyan intelligence highlights
```

## Style name

Default style name:

```txt
Aithru Slate Intelligence
```

This style is intended for workflow authoring, Agent monitoring, run inspection, trace debugging, approval surfaces, and admin/workbench screens.

## Design posture

Aithru should feel like:

```txt
clear, calm, technical, trustworthy, dense-enough workbench UI
```

The product should look closer to an engineering console, workflow workbench, or AI operations dashboard than to a consumer landing page.

Default visual goals:

1. Calm base surfaces for long sessions.
2. Clear hierarchy for dense workflow and trace information.
3. Strong but restrained brand color.
4. Color used primarily for action, state, risk, and execution feedback.
5. Subtle intelligence/AI feeling without decorative gradients everywhere.
6. Good light mode for configuration and reading.
7. Excellent dark mode for logs, traces, consoles, and execution monitoring.

## Primary palette

| Role | Color family | Default hex | Usage |
| --- | --- | --- | --- |
| Base | Slate | `#f8fafc` / `#020617` | Page backgrounds in light/dark mode. |
| Surface | White / Slate | `#ffffff` / `#0f172a` | Cards, panels, dialogs, sidebars. |
| Border | Slate | `#e2e8f0` / `#1e293b` | Region separation and low-noise structure. |
| Primary | Indigo | `#4f46e5` / `#818cf8` | Main actions, selected navigation, focused workflow nodes. |
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

## Light theme tokens

Use light mode for configuration, workflow authoring, admin tables, and general reading.

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

Light mode should avoid a pure-white page background. Use a slightly cool Slate background with white cards so panels remain visible without heavy shadows.

## Dark theme tokens

Use dark mode for trace inspection, run consoles, long monitoring sessions, and developer-focused screens.

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

- Use `background` for the full page.
- Avoid pure white in light mode page backgrounds.
- Avoid pure black in dark mode page backgrounds.
- Keep the page calm enough for long-running trace and workflow sessions.

### Cards and panels

- Use `card` for primary content panels.
- Prefer thin borders over heavy shadows.
- Use subtle background changes to separate regions.
- Keep radius consistent across cards, buttons, dialogs, inputs, and panels.
- Use shadow sparingly, mostly for floating panels, dialogs, menus, and overlays.

### Navigation

- Active navigation should use Primary/Indigo.
- Secondary navigation should use muted foreground and border contrast.
- Avoid decorative gradients in navigation.
- Workspace, project, profile, and execution target should remain visible in workbench shells.

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
- Use monospace for run IDs, node IDs, trace event IDs, logs, JSON, and code snippets.

### Density

Aithru is a workbench, so it should be denser than a marketing site but not cramped.

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

## Recommended component variants

Primitive and common components should expose variants rather than requiring pages to assemble raw Tailwind class strings.

Recommended variants:

```txt
Button: primary, secondary, outline, ghost, destructive
Badge: neutral, primary, accent, success, warning, destructive
Card: default, muted, elevated, selected, interactive
Alert: info, success, warning, destructive
Panel: default, sidebar, activity, floating
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
- using red/orange/yellow for decoration instead of state;
- mixing Ant Design, Material UI, Chakra, or another full component library into the same app;
- making dark mode the only polished mode.

## Alternative palettes

The default is Aithru Slate Intelligence. These alternatives are allowed only with a design note.

| Palette | Colors | When to use | Risk |
| --- | --- | --- | --- |
| Slate + Indigo + Cyan | Current default | Main workbench, Agent, workflow, trace. | Lowest risk. |
| Zinc + Blue + Emerald | More traditional SaaS | Admin-heavy surfaces where familiarity matters. | Less brand recognition. |
| Neutral + Violet + Sky | More AI-product feeling | Lighter assistant/Agent-focused surfaces. | Can become decorative or trendy. |

If there is no strong reason, use the default palette.

## Implementation checklist

Before a frontend style implementation is accepted, check:

- [ ] Uses Aithru Slate Intelligence as the default palette.
- [ ] Defines light and dark theme tokens.
- [ ] Maps Tailwind to semantic CSS variables.
- [ ] Avoids hard-coded feature-level hex colors.
- [ ] Uses Indigo for primary actions and selection.
- [ ] Uses Cyan for running/AI/live activity.
- [ ] Uses Emerald/Amber/Rose only for semantic state.
- [ ] Uses thin borders and subtle surfaces instead of heavy shadows.
- [ ] Keeps graph color primarily tied to selection, validation, and execution state.
- [ ] Provides readable trace/log/console views.
- [ ] Keeps both light and dark modes polished.
