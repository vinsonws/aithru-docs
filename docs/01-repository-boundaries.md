# Repository Boundaries

## One-Line Boundaries

```txt
aithru-platform owns platform security, app hosting, grants, tokens, audit, and SDK integration.
aithru-flowe owns workflow expression, validation, execution semantics, trace, and local runtime state.
aithru-agent owns intelligent harness execution, tools, workspace, artifacts, memory, approvals, subagents, and Agent event streams.
```

## Platform

`aithru-platform` is the security and integration authority.

It owns:

- global user identity and organization membership;
- bootstrap, login, refresh, restore, logout, and organization switching;
- platform roles and organization-scoped admin permissions;
- app registry, portal app list, integration metadata, and hosted app entry;
- app user context grants for `app`, `org`, and `group` contexts;
- group-context role grants;
- app-level permission and role catalogs supplied by subsystems;
- service clients, service policies, service tokens, token exchange, delegation,
  and audit;
- frontend shell modes: Native Hosted App and Admin Center;
- subsystem SDKs and language-neutral contracts.

It does not own:

- Flowe workflow graph semantics or execution internals;
- Agent run/tool execution internals;
- subsystem business resources such as documents, workflow definitions, files,
  agent workspaces, or domain ACLs;
- child app UI internals.

Platform rule:

```txt
Subsystems define permissions.
Platform stores grants.
Subsystems enforce decisions with SDKs and authz APIs.
```

## Flowe

`aithru-flowe` is the current workflow kernel.

It owns:

- `apiVersion: aithru.flow/v1alpha1` workflow shape;
- node, port, config, runtime hint, and mapping contracts;
- strict validation and migration tooling;
- DAG execution, retries, timeouts, concurrency, cache, and parallel execution;
- built-in nodes such as input, output, template, file, HTTP, JSON transform,
  markdown, human approval, external task, and mock LLM;
- run traces, packet inspection, replay, diff, and local artifact storage;
- HTTP runtime endpoints, local server state, workflow registry, versioning,
  published workflow execution, webhooks, queue, worker loop, and resume;
- the thin server-served `/console`.

It does not own:

- platform identity, app grants, service policies, or audit truth;
- Agent skills, model adapters, memory, subagents, or sandbox policy;
- production visual editor, database-backed runtime, distributed workers, MCP
  marketplace, plugin system, or full expression language.

## Agent

`aithru-agent` is the current AI harness backend.

It owns:

- Agent threads, messages, runs, runtime todos, skills, workspaces, tools,
  artifacts, memory, approvals, subagents, and snapshots;
- Aithru-owned stream events and trace projection;
- Pydantic AI-backed harness execution behind Aithru contracts;
- capability routing for local tools, sandbox, workspace, memory, artifacts,
  todos, task/subagent delegation, and approval pause/resume;
- in-memory and SQLite persistence;
- worker execution and queued run handling.

It does not own:

- formal workflow definitions or Flowe graph semantics;
- platform authentication, app registry, app grants, hosted app shell, service
  credentials, or audit authority;
- unrestricted shell, browser, filesystem, external network, database, or
  service-token access for model code.

Agent rule:

```txt
Model output may propose tool calls.
Real actions must pass through the Aithru capability boundary.
```

## Dependency Direction

```txt
Platform is the trust and hosting boundary.
Flowe and Agent may integrate with Platform through explicit APIs, tokens, SDKs, and hosted-app contracts.
Agent may call workflow capability APIs when they exist, but must not import or reinterpret Flowe internals as Agent plans.
Flowe must stay a deterministic workflow kernel and must not depend on Agent harness packages.
Browser frontends must talk to backends through typed clients, postMessage contracts, IPC, or HTTP APIs.
```
