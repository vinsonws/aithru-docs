# Vision

Aithru is a local-first and enterprise-ready AI WorkflowOS.

The project keeps one deterministic workflow foundation and grows new capabilities through optional modules. The current `aithru-core` baseline already provides the portable workflow kernel. `aithru-web` proves that the same spec and node metadata can be edited in a browser without bringing runtime-only dependencies into the UI.

## Product shape

```txt
Personal Edition
  local-first runtime + local files + local credentials + optional integrations

Business Edition
  server runtime + database + workers + RBAC + audit log + shared credentials
```

Both editions should share the same core contracts and differ mainly in composition, persistence, deployment, and permission model.

## Principles

- Keep workflow execution inspectable and portable.
- Treat integrations as nodes or tools.
- Keep product shells as composition layers.
- Register optional packages explicitly.
- Make every run observable through structured events.
- Keep `aithru-core` small and stable.
- Add Agent, MCP, UI, desktop, and server features as horizontal modules.

## North-star flow

For deterministic workflow automation:

```txt
Design workflow
  -> validate WorkflowSpec
  -> run workflow
  -> inspect events and trace
  -> pause for approval when needed
  -> resume or retry
  -> reuse as a template
```

For agent-driven work:

```txt
Task
  -> plan
  -> execute bounded steps
  -> call approved tools
  -> produce artifacts
  -> review
  -> rerun or finalize
```

## Non-goals

Aithru should not become a single monolithic default package that includes the UI, server, Agent runtime, MCP transport layer, every connector, every LLM provider, and enterprise deployment logic.

The ecosystem should grow through clear module boundaries instead.
