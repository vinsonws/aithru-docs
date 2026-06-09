# Aithru Docs

`aithru-docs` is the design source of truth for the Aithru ecosystem.

Aithru is a local-first and enterprise-ready AI WorkflowOS. This repository keeps the product vision, architecture boundaries, module ownership, roadmap, and architecture decisions that guide implementation across `aithru-core`, `aithru-platform`, the new server-hosted `aithru-workbench`, `aithru-agent`, and future optional integration packages.

## Current baseline

| Repository | Current role | Boundary |
| --- | --- | --- |
| `vinsonws/aithru-core` | Deterministic workflow kernel, local runtime adapter, node SDK, primitive nodes, human approval, HTTP node and tool executor, trace, redaction, pause/resume, and generic tool contracts. | Owns formal `WorkflowSpec` and workflow execution semantics. Does not include Agent runtime, MCP SDK, UI, platform auth, server app runtime, database/queue adapters, or LLM providers by default. |
| `vinsonws/aithru-platform` | Product platform, portal, user/auth/permission boundary, app shell, and subsystem entry point. | Owns identity, authorization, grants, portal/app registration, and platform shell. Does not own workflow runtime semantics or Aithru Core contracts. |
| `vinsonws/aithru-workbench` | New target: platform-hosted workflow workbench that combines the former web authoring surface and server-side execution bridge into one simpler subsystem. | Owns workflow UI, run API, run/event storage, approval endpoints, runtime composition with `aithru-core`, and platform-context consumption. Does not own platform authentication/RBAC or core workflow semantics. |
| `vinsonws/aithru-agent` | Intelligent execution layer with Agent contracts, model adapters, classify/plan-run-review/deep-research engines, AgentTraceEvent taxonomy, and optional `@aithru/node-agent` workflow nodes. | Owns intelligent execution inside bounded tasks or nodes. Does not own `WorkflowSpec`, workflow graph semantics, core scheduler behavior, platform auth, or tool permission policy ownership. |
| `vinsonws/aithru-web` | Legacy/superseded browser-only workflow authoring surface. | No longer the primary product direction. Useful as a source of UI concepts and components to migrate into `aithru-workbench`. |
| `vinsonws/aithru-personal-bridge` | Legacy/superseded personal execution bridge. | No longer the primary product direction for the current server-hosted platform path. Useful as a source of `RuntimeHost`, event bus, and run API patterns to migrate into `aithru-workbench`. |
| `vinsonws/aithru-docs` | Ecosystem-level architecture, ADRs, specs, roadmaps, and implementation prompts. | Does not replace package READMEs or generated API docs; it explains cross-repo design decisions and long-term direction. |

Core principle:

```txt
Workflow is deterministic. Integrations are nodes or tools.
```

Simplified product boundary:

```txt
aithru-platform owns auth, permission, portal, and subsystem entry.
aithru-workbench owns workflow UI and server-side workflow execution APIs.
aithru-core owns WorkflowSpec, runtime contracts, and execution semantics.
```

Core/Agent boundary:

```txt
aithru-core owns formal workflows.
aithru-agent owns intelligent execution inside bounded tasks or nodes.
```

Execution boundary:

```txt
Browser UI presents, edits, inspects, and approves.
Server-side runtime hosts execute workflows, persist runs/events, and register tools.
```

For the current platform-hosted path, that server-side runtime host is `aithru-workbench`, not a separate `aithru-personal-bridge` process.

## Start here

1. [Vision](./docs/00-vision.md)
2. [Ecosystem Overview](./docs/01-ecosystem-overview.md)
3. [Module Boundaries](./docs/02-module-boundaries.md)
4. [Aithru Core](./docs/03-aithru-core.md)
5. [Aithru Workbench](./docs/10-aithru-workbench.md)
6. [Aithru Agent](./docs/05-aithru-agent.md)
7. [Runtime, Trace, Approval, Tooling](./docs/06-runtime-trace-approval-tooling.md)
8. [Roadmap](./docs/07-roadmap.md)
9. [Development Workflow](./docs/08-development-workflow.md)
10. [Core and Agent Boundary](./docs/09-core-agent-boundary.md)
11. [Legacy Aithru Web](./docs/04-aithru-web.md)
12. [Legacy Aithru Personal Bridge](./docs/10-aithru-personal-bridge.md)
13. [Frontend Constraints](./docs/11-frontend-constraints.md)
14. [Frontend Visual Design](./docs/12-frontend-visual-design.md)

Architecture decisions:

- [ADR-0001: Keep docs as the ecosystem source of truth](./adr/ADR-0001-docs-as-source-of-truth.md)
- [ADR-0002: Keep core, web, and agent boundaries explicit](./adr/ADR-0002-core-agent-web-boundaries.md)
- [ADR-0003: Browser UI does not execute the real runtime](./adr/ADR-0003-browser-ui-does-not-execute-runtime.md)
- [ADR-0004: Core owns workflows, Agent owns intelligent execution](./adr/ADR-0004-core-owns-workflows-agent-owns-intelligence.md)
- [ADR-0005: Introduce aithru-personal-bridge as the personal execution host](./adr/ADR-0005-aithru-personal-bridge.md)
- [ADR-0006: Agent engines may be exposed as workflow nodes](./adr/ADR-0006-agent-engines-as-workflow-nodes.md)
- [ADR-0007: Consolidate Web and Personal Bridge into Aithru Workbench](./adr/ADR-0007-workbench-consolidation.md)

Reusable implementation prompt:

- [Codex docs sync prompt](./prompts/codex-docs-sync-prompt.md)

## Documentation rules

- Design changes that affect more than one repository should start here.
- Package-level usage details stay in each implementation repository.
- Cross-repo boundaries, product direction, and architecture decisions stay in this repository.
- New concrete integrations should be optional packages by default, not additions to the core kernel.
- Agent and MCP integrations must respect `ToolPermissionPolicy`, trace, redaction, approval, and `NodeExecutionContext.callTool`.
- For the current platform-hosted product path, workflow UI and server-side workflow execution should be composed in `aithru-workbench` to reduce cross-repository complexity.
- Personal/local bridge and standalone browser workbench ideas may be revisited later for desktop or personal edition, but they are not the primary implementation path now.
