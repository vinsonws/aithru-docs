# Frontend Constraints

This document is the single frontend constraint document for the current Aithru
docs set. It folds the old common frontend and visual design documents into one
shorter source.

## Current Frontend Surfaces

| Surface | Current state | Constraint summary |
| --- | --- | --- |
| `aithru-platform/frontend` | Active React/Vite frontend. | Native Hosted App mode for ordinary users plus Admin/Ops mode under `/admin/*`. |
| `aithru-flowe /console` | Active thin server-served HTML/CSS/JS console. | Debugging and local runtime operation only; no frontend build system. |
| `aithru-agent` frontend | Not active yet. | When added, it should be a Platform hosted app, not a standalone global shell or workflow graph editor. |

## Required Platform Frontend Stack

The active platform frontend uses and should continue using:

- React 19;
- TypeScript;
- Vite 6;
- React Router;
- Tailwind CSS 3 with CSS variables;
- shadcn/ui and Radix UI primitives;
- lucide-react icons;
- TanStack Query for server state;
- TanStack Table for admin lists;
- React Hook Form and Zod for non-trivial forms;
- semantic API modules under feature folders.

Do not introduce another primary framework, component system, styling system,
query library, table library, or form library without a decision note.

## Product Modes

### Native Hosted App Mode

Use for normal daily app routes such as `/apps` and `/:appKey`.

Rules:

- The active child app should feel native, not framed by a heavy admin console.
- Platform controls stay low-noise: account, org switcher, app switcher, theme,
  recovery, and admin entry.
- App key, origin, token, message bus, and integration details stay hidden in
  normal use.
- Boundary details become visible in recovery, permission, inspector, and debug
  states.

### Admin/Ops Mode

Use for `/admin/*`.

Rules:

- Dense tables, filters, sidebars, inspectors, drawers, and audit context are
  appropriate.
- Each page must handle loading, empty, error, permission denied, and degraded
  states.
- Destructive and security-sensitive actions need explicit confirmation.
- Service secrets, bearer tokens, refresh tokens, and internal token values must
  never be displayed.

### Recovery/Boundary Mode

Use when auth expires, app loading fails, origin validation fails, hosted token
exchange fails, permission is denied, or a child app is offline.

Rules:

- Show app key, org, integration mode, origin, and recovery actions when useful.
- Keep secrets redacted.
- Prefer actionable recovery panels over generic toasts.

## Security Rules

- Access tokens stay in React state/memory only.
- Refresh tokens live in the `AITHRU_REFRESH` HttpOnly cookie and are restored
  through `/api/auth/restore`.
- Browser storage may keep only non-sensitive preferences or debug toggles, such
  as last organization, recent apps, or bridge trace enablement.
- No access token, refresh token, hosted token, service client secret, delegated
  token, or bearer credential may be stored in `localStorage`, `sessionStorage`,
  IndexedDB, `window.name`, URL parameters, logs, or visible UI.
- Ordinary browser UI must not call internal service-token or token-exchange
  endpoints with service client credentials.
- Hosted iframe `postMessage` handlers must validate `event.origin` against the
  registered app origin.
- Hosted tokens are issued only when the child app explicitly requests backend
  API scopes.

## Visual Direction

The current visual direction is:

```txt
Aithru Slate Intelligence:
calm technical product UI, net-white light canvas, warm-grey layering,
Indigo primary actions, Cyan intelligence highlights, explicit state colors.
```

Light theme rules:

- `--background` is net white: `#ffffff`.
- Cards, dialogs, and page canvas may all be white; hierarchy comes from
  shadow, border, spacing, and semantic accents.
- `--muted` uses low-saturation warm grey around `#f7f8f9`.
- Avoid large cool-blue grey fills that make the product look hazy.
- Brand color appears through restrained Indigo to Cyan gradients, icon chips,
  focus states, and small highlights, not as full-page decoration.

Dark theme rules:

- Keep deep slate surfaces.
- Preserve the same semantic color roles.
- Do not make dark mode pure black unless a local component has a clear reason.

Semantic color roles:

| Role | Color family | Usage |
| --- | --- | --- |
| Primary | Indigo | Main actions, selected navigation, focused platform state. |
| Accent | Cyan | AI/Agent activity, live/running state, trace highlights. |
| Success | Emerald | Completed runs and successful actions. |
| Warning | Amber | Approval waiting, degraded state, caution. |
| Destructive | Rose | Failure, deletion, blocked execution, dangerous actions. |

Feature components should consume semantic classes such as `bg-background`,
`bg-card`, `bg-muted`, `text-foreground`, `text-muted-foreground`,
`border-border`, `bg-primary`, `text-primary`, `text-success`,
`text-warning`, and `text-destructive`.

Avoid:

- hard-coded feature-level hex colors or one-off `text-blue-*` / `bg-red-*`
  color systems;
- marketing hero spacing in Admin Center;
- heavy gradients, glassmorphism, neon/cyberpunk styling, or decorative effects
  that reduce readability;
- status communicated by color alone.

## Component And State Rules

- Centralize API access in typed feature API modules.
- Use shared loading, empty, error, permission, status, confirmation, redaction,
  and table components instead of one-off variants.
- Status UI must include text or icon labels, not color alone.
- Copyable values that may be sensitive must use explicit redaction behavior.
- Feature pages must not assemble raw API URLs inline when a typed client module
  exists.
- The same platform vocabulary should be used across portal, Admin Center,
  hosted app recovery, SDK docs, and future Agent hosted UI.
