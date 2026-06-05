# Aithru Personal Bridge

`aithru-personal-bridge` is the trusted personal execution host for Aithru workflows.

## One-line definition

```txt
Aithru Personal Bridge = private single-user execution bridge for Aithru-Web, backed by Aithru Core LocalRuntime.
```

## Why it exists

Aithru has three pieces that should stay separate:

```txt
aithru-web
  browser authoring, validation, trace display, approval UI

aithru-core
  WorkflowSpec, LocalRuntime, node/runtime/tool contracts

aithru-personal-bridge
  trusted Node host that connects Web to real runtime execution
```

The browser UI must not execute `LocalRuntime` directly. Aithru-Web already has a browser-safe HTTP Bridge Client, and Aithru Core already has `@aithru/runtime-local`. The missing piece is a trusted Node host that owns execution, tools, traces, approvals, local files, and secrets.

## Deployment modes

### Loopback mode

- Binds to `127.0.0.1` by default.
- Browser and bridge are usually on the same machine.
- Suitable for local development and future desktop embedding.

### Personal-server mode

- Runs on a Mac mini, DGX-like workstation, NAS, home server, or private VPS.
- Browser may run on another laptop, tablet, or phone.
- Access should use LAN, Tailscale, VPN, or HTTPS reverse proxy.
- Still private and single-user by default.

## Responsibilities

`aithru-personal-bridge` owns:

- the Execution Bridge API:
  - `POST /runs`
  - `GET /runs/:runId`
  - `GET /runs/:runId/events`
  - `POST /runs/:runId/resume`
  - `POST /runs/:runId/cancel`
- `WorkflowSpec` validation before execution;
- runtime registry creation;
- node package registration;
- tool executor registration;
- workflow execution with `@aithru/runtime-local`;
- run event collection;
- trace event exposure to Aithru-Web;
- local trace/artifact storage;
- human approval resume/rejection;
- cancellation semantics;
- local security policy.

## Non-goals

`aithru-personal-bridge` does not own:

- browser UI;
- Aithru Core contracts;
- WorkflowSpec definition;
- enterprise RBAC;
- team workspaces;
- durable distributed worker pools;
- public multi-tenant APIs;
- Agent runtime by default;
- MCP runtime by default.

## Security model

The bridge can execute workflows and access runtime-side resources, so it must be treated as trusted local infrastructure.

Default security posture:

- bind to `127.0.0.1` by default;
- require token auth for personal-server mode;
- configure a strict CORS allowlist;
- prefer Tailscale, VPN, LAN, or HTTPS reverse proxy for remote browser access;
- never expose an unauthenticated bridge to the public internet;
- assume logs, traces, artifacts, and node outputs can contain sensitive information;
- keep redaction and retention policy explicit before exposing traces to remote browsers.

## Relation to other modules

| Module | Relationship |
| --- | --- |
| `aithru-web` | Uses the HTTP Bridge Client to submit runs, view events, and send approval decisions. |
| `aithru-core` | Provides WorkflowSpec, LocalRuntime, node/runtime/tool contracts, and runtime packages. |
| `aithru-desktop` | May later embed, manage, package, or replace the personal bridge for a polished desktop product. |
| `aithru-server` | Owns durable multi-user remote execution, RBAC, audit, workspaces, persistence, and worker pools. |
| `aithru-agent` | May later register optional Agent nodes or execution capabilities through the same tool/trace/approval contracts. |
| `aithru-mcp` | May later register optional MCP nodes and tool executors through the bridge runtime composition. |

## v0.1 target

The first implementation should be intentionally small:

- TypeScript Node project.
- Loopback bind only.
- In-memory run records and events.
- `LocalRuntime` composition with core/human/http node packages.
- HTTP tool executor only inside the bridge runtime host.
- Approval pause/resume/reject mapping.
- Aithru-Web HTTP Bridge compatibility.

## v0.2 target

The second implementation should add personal-server safety:

- configurable bind host;
- token auth;
- CORS allowlist;
- local trace/artifact directory;
- Tailscale/LAN/HTTPS deployment guide;
- explicit warnings for public exposure.
