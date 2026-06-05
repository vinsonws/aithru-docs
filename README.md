# Aithru Docs

`aithru-docs` is the design source of truth for the Aithru ecosystem.

Aithru is a local-first and enterprise-ready AI WorkflowOS. This repository keeps the product vision, architecture boundaries, module ownership, roadmap, and architecture decisions that guide implementation across `aithru-core`, `aithru-web`, `aithru-personal-bridge`, and future integration packages.

## Current baseline

| Repository | Current role | Boundary |
| --- | --- | --- |
| `vinsonws/aithru-core` | Deterministic workflow kernel, local runtime adapter, node SDK, primitive nodes, human approval, HTTP node and tool executor, trace, redaction, pause/resume, and generic tool contracts. | Owns formal WorkflowSpec and workflow execution semantics. Does not include real Agent runtime, MCP SDK, UI, server runtime, database/queue adapters, or LLM providers by default. |
| `vinsonws/aithru-web` | Browser-safe React/Vite workflow authoring and inspection surface for Aithru Core workflows. | Edits and validates `WorkflowSpec`; browser run panel is simulation/inspection only. Real execution belongs to `aithru-personal-bridge` or future `aithru-server`. |
| `vinsonws/aithru-personal-bridge` | Planned trusted personal execution host for Aithru-Web. | Runs LocalRuntime on localhost or a private personal server such as Mac mini, DGX-like workstation, NAS, or home server. It is single-user/private by default, not an enterprise server. |
| `vinsonws/aithru-docs` | Ecosystem-level architecture, ADRs, specs, roadmaps, and implementation prompts. | Does not replace package READMEs or generated API docs; it explains cross-repo design decisions and long-term direction. |

Core principle:

```txt
Workflow is deterministic. Integrations are nodes or tools.
```

Core/Agent boundary:

```txt
aithru-core owns formal workflows.
aithru-agent owns intelligent execution inside bounded tasks or nodes.
```

Execution boundary:

```txt
aithru-web edits and validates WorkflowSpec.
aithru-personal-bridge or aithru-server executes WorkflowSpec.
```

## Start here

1. [Vision](./docs/00-vision.md)
2. [Ecosystem Overview](./docs/01-ecosystem-overview.md)
3. [Module Boundaries](./docs/02-module-boundaries.md)
4. [Aithru Core](./docs/03-aithru-core.md)
5. [Aithru Web](./docs/04-aithru-web.md)
6. [Aithru Agent](./docs/05-aithru-agent.md)
7. [Runtime, Trace, Approval, Tooling](./docs/06-runtime-trace-approval-tooling.md)
8. [Roadmap](./docs/07-roadmap.md)
9. [Development Workflow](./docs/08-development-workflow.md)
10. [Core and Agent Boundary](./docs/09-core-agent-boundary.md)
11. [Aithru Personal Bridge](./docs/10-aithru-personal-bridge.md)

Architecture decisions:

- [ADR-0001: Keep docs as the ecosystem source of truth](./adr/ADR-0001-docs-as-source-of-truth.md)
- [ADR-0002: Keep core, web, and agent boundaries explicit](./adr/ADR-0002-core-agent-web-boundaries.md)
- [ADR-0003: Browser UI does not execute the real runtime](./adr/ADR-0003-browser-ui-does-not-execute-runtime.md)
- [ADR-0004: Core owns workflows, Agent owns intelligent execution](./adr/ADR-0004-core-owns-workflows-agent-owns-intelligence.md)
- [ADR-0005: Introduce aithru-personal-bridge as the personal execution host](./adr/ADR-0005-aithru-personal-bridge.md)

Reusable implementation prompt:

- [Codex docs sync prompt](./prompts/codex-docs-sync-prompt.md)

## Documentation rules

- Design changes that affect more than one repository should start here.
- Package-level usage details stay in each implementation repository.
- Cross-repo boundaries, product direction, and architecture decisions stay in this repository.
- New concrete integrations should be optional packages by default, not additions to the core kernel.
- Agent and MCP integrations must respect `ToolPermissionPolicy`, trace, redaction, approval, and `NodeExecutionContext.callTool`.
- Personal execution is owned by `aithru-personal-bridge`; durable team execution is owned by future `aithru-server`.
