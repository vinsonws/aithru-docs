# ADR-0005: Introduce aithru-personal-bridge as the personal execution host

Status: Accepted

Date: 2026-06-05

## Context

`aithru-web` is browser-safe. It edits, validates, and displays `WorkflowSpec` data, but it must not execute the real runtime directly.

`aithru-core` owns `WorkflowSpec`, runtime contracts, `@aithru/runtime-local`, node packages, trace contracts, approval contracts, and tool execution contracts.

Users may want real personal execution in two shapes:

- loopback execution on the same machine as the browser;
- personal-server execution on a private Mac mini, DGX-like workstation, NAS, home server, or small private server while the browser runs elsewhere.

A full `aithru-server` is too heavy for this personal/single-user deployment because it implies business features such as workspaces, RBAC, audit, durable worker pools, database-backed run stores, and multi-user governance.

## Decision

Create and document `aithru-personal-bridge` as the trusted personal execution host.

It supports:

- loopback mode for local development and future desktop embedding;
- personal-server mode for private single-user servers;
- the Execution Bridge API consumed by `aithru-web`;
- runtime composition using `aithru-core` packages;
- real workflow execution outside the browser;
- local traces, artifacts, approval resume/rejection, and tool executor registration.

`aithru-personal-bridge` is not the enterprise server. Durable team execution remains the responsibility of future `aithru-server`.

## Consequences

Positive consequences:

- Keeps the browser bundle safe and free of Node-only runtime code.
- Gives `aithru-web` a real execution target for its HTTP Bridge Client.
- Allows personal Mac mini / DGX / NAS / private server deployments without prematurely building enterprise server features.
- Keeps `aithru-core` thin and product-neutral.
- Creates a clean prototype that future `aithru-desktop` may embed or manage.

Tradeoffs:

- Adds another repository/module before the desktop shell exists.
- Personal-server mode requires careful token auth, CORS, bind address, and network exposure rules.
- Local traces and artifacts may contain sensitive information and require explicit redaction/retention policy.

## Future direction

`aithru-personal-bridge` may later be:

- embedded by `aithru-desktop`;
- managed as a background process by desktop or personal-server installers;
- kept as a standalone developer/personal-server bridge;
- superseded by `aithru-server` for team and enterprise deployment.

It should not become a multi-tenant platform, UI shell, Agent runtime, or MCP runtime by default.
