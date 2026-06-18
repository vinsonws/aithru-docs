# Runtime and Security Contracts

## Browser Authority

Browser code is never the security authority.

```txt
Frontend can display, request, inspect, recover, and route user intent.
Backend services enforce identity, grants, tokens, service policy, durable state, and audit.
Subsystem backends enforce their own resource-level authorization.
```

Do not store access tokens, refresh tokens, service client secrets, delegated
tokens, or bearer tokens in ordinary browser storage.

## Platform Token And App Contract

Platform-owned token concepts stay distinct:

| Token shape | Purpose | Boundary |
| --- | --- | --- |
| user access | Browser user session in one organization context. | Short-lived; access token stays in memory. |
| refresh | Session restore and rotation. | Stored in `AITHRU_REFRESH` HttpOnly cookie, not readable by JavaScript. |
| hosted access | Browser-hosted app backend calls. | Issued only when a hosted child app requests allowed backend scopes. |
| service access | Service-to-service call. | Requires service policy and service credentials; never issued directly from ordinary browser UI. |
| exchanged access | On-behalf-of current user call. | Preserves user and org context through service policy. |
| delegated access | Background/workflow/agent work through a durable delegation grant. | Short-lived and constrained by grant, org, target app, delegate app, and current permission. |

Hosted app message flow:

```txt
Platform shell
  -> sends startup auth context, theme, org, and app metadata
  -> validates child postMessage origin against registered app origin
  -> issues hosted tokens only for explicit child backend-scope requests
  -> redacts tokens from trace/debug output
```

## Flowe Runtime Contract

Flowe owns workflow execution semantics.

Important guarantees:

- workflow files are YAML or JSON contracts, not UI-only state;
- validation must reject unsupported graph features explicitly;
- trace output is required for normal and failed runs;
- packet/mapping failures must be inspectable;
- local artifacts and run state live under the Flowe runtime boundary;
- queue/resume behavior is local-first and file-backed today;
- `/console` is a thin server-served tool over the same runtime API, not a
  separate frontend product.

Flowe should not silently grow into platform RBAC, a production distributed
runtime, or a plugin marketplace without a new cross-repo decision.

## Agent Runtime Contract

Agent owns intelligent execution state, but not unrestricted capabilities.

```txt
model / Pydantic AI
  -> Aithru Tool Bridge
  -> Capability Router
  -> policy / scope / approval
  -> local tool or future workflow capability API
  -> AgentStreamEvent / trace / artifact
```

Rules:

- Pydantic AI and `pydantic-ai-harness` are implementation details, not public
  Aithru API contracts.
- Skills may provide instructions and tool policy, but tool exposure remains
  constrained by run context.
- Workspace, artifact, todo, memory, sandbox, and subagent tools must bind
  changes to the current run/thread/org/user boundary.
- Risky actions pause for approval and resume through persisted approval state.
- Event streams must be replayable enough to drive chat, run timeline, trace,
  workspace, artifact, approval, and debug projections.
- Sensitive payloads must be redacted before replay, SSE, debug, or audit
  exposure.

## Resource Authorization Split

Platform can answer app/context questions:

```txt
Can user U in org O enter app A?
Does user U in org O have app permission P?
Does user U inherit app role R from a group context?
```

Subsystems answer domain-resource questions:

```txt
Can user U read document D?
Can workflow run W use secret S?
Can Agent run R write workspace file F?
Can Flowe run Q resume external task T?
```

Do not push subsystem resource ACLs into Platform as durable source of truth
unless a new shared resource model is explicitly designed.
