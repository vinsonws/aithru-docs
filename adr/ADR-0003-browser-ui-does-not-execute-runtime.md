# ADR-0003: Browser UI does not execute the real runtime

Status: Accepted

Date: 2026-06-04

## Context

`aithru-web` is a browser-based workflow authoring and inspection app. It uses browser-safe Aithru packages for `WorkflowSpec` validation and node metadata.

The real local runtime currently lives in `@aithru/runtime-local`, and concrete tool execution can require Node-only capabilities such as file access, HTTP executor composition, secrets, and future local/desktop permissions.

## Decision

The browser UI should not execute the real runtime directly.

Current rule:

```txt
aithru-web edits and validates WorkflowSpec.
aithru-personal-bridge or aithru-server executes WorkflowSpec.
```

The current run panel may simulate execution status for local development, but it should not be treated as authoritative runtime behavior.

## Consequences

Positive:

- Browser bundle stays safe and simple.
- Node-only runtime code does not leak into the UI layer.
- `aithru-personal-bridge`, future desktop, and future server products can share the same UI components while owning execution.
- Tool permission, secrets, approval, and durable traces remain runtime-owned concerns.

Tradeoffs:

- A standalone browser app cannot perform real tool calls without a bridge/server.
- The simulated run panel must be clearly labeled to avoid confusion.

## Future integration path

`aithru-personal-bridge`, a future desktop shell, or a future server shell may expose:

- run workflow;
- resume paused run;
- read trace;
- approve/reject human approval request;
- list available runtime nodes/tools;
- manage credentials or secret references.

The browser UI can call those APIs, but should still avoid owning runtime execution logic directly.
