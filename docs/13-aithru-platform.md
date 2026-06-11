# Aithru Platform

`aithru-platform` is the current product platform and control plane for the Aithru ecosystem.

It is not just an admin console. It is the platform layer that gives all hosted subsystems a shared identity, organization, authorization, app registry, service registry, token, delegation, audit, and portal boundary.

## Current design baseline

The current platform design has shifted from "platform owns every business resource" to a healthier subsystem-oriented model:

```txt
Subsystems define permissions and roles.
Platform stores grants.
Subsystems enforce permission decisions through SDK and platform authz APIs.
Subsystems still own their own business resources and resource-level ACLs.
```

This means the platform should not try to register every file, knowledge document, workflow run, meeting, or agent task as a platform resource. Those belong to the subsystem that owns the domain.

The platform owns the durable cross-subsystem security and integration layer:

```txt
identity
organization membership
groups and departments
platform admin roles
subsystem app registry
subsystem permission catalog
subsystem role catalog
grants to users, groups, departments, organizations, and app contexts
effective app permission computation
authz decision API
token issuing and validation contracts
delegated authorization
service-to-service policy
audit
portal app visibility
hosted app entry
```

## Product shape

The platform frontend has two faces:

```txt
Daily user experience:
  native-feel hosted apps
  platform chrome stays subtle
  users feel they are using Knowledge / Files / Agent / Workflow / Meeting directly

Admin / operator experience:
  Admin Center
  dense platform control plane
  members, groups, departments, apps, roles, grants, authz simulator, delegation, services, audit
```

Daily apps should not be wrapped in a heavy backend-admin frame. Routes such as `/knowledge`, `/files`, `/agent`, `/workflow`, and `/meeting` should feel like first-class native product pages. The platform appears through low-noise account, org switcher, app switcher, status, recovery, and boundary inspection affordances.

Admin Center is a first-class internal platform app under `/admin/*`. It can use a persistent sidebar, dense tables, filters, drawers, inspectors, and audit views.

## Backend authority rule

The browser frontend must never become the authority for security.

```txt
Frontend can display, request, inspect, and recover.
Backend enforces identity, grants, tokens, delegation, service policy, and audit truth.
Subsystem backend enforces its own resource-level authorization.
```

Do not put durable permission decisions, refresh token security, service token issuing, delegated authorization, secret storage, or audit truth into the browser.

## Permission and role model

A subsystem uploads or registers a manifest that defines:

```txt
app key
app display metadata
permissions
roles
role-permission mappings
service client metadata
optional integration metadata
```

The platform persists this as a catalog. The catalog describes what can be granted; it is not itself the grant.

Actual authorization is represented by grants:

```txt
user -> app access / app role / app permission
group -> app access / app role / app permission
department -> app access / app role / app permission
organization/default -> app access / app role / app permission
service client -> service policy / service permission
```

A user may belong to multiple organizations. Inside each organization, the user may belong to multiple groups or departments. Effective permissions are computed from the selected organization context plus all applicable direct and inherited grants.

Recommended mental model:

```txt
effective user app permissions
  = direct user grants
  + group grants
  + department grants
  + organization/default grants
  + delegated/exchanged constraints when token type requires them
```

Avoid storing a duplicated permission snapshot for every `user x group x subsystem` combination as the source of truth. The platform should store normalized grants and compute or cache effective permissions with versioning.

## Group and organization context

The important design point is context:

```txt
same global user
  may be member of org A and org B
  may have different group memberships in each org
  may have different subsystem access in each org
  may receive different effective permissions for the same app in each org
```

Therefore, all app grants must be organization-scoped unless a future design explicitly introduces global/system-wide grants.

Every access token used for user work should carry enough organization context for subsystems to reject ambiguous calls. A subsystem should not infer organization from URL or client-side state when a validated token claim is required.

## Resource-level authorization boundary

The platform currently centralizes app-level and context-level grants, not resource ACLs.

Example:

```txt
Platform can answer:
  Can user U in org O use app knowledge?
  Does user U in org O have knowledge.document.create?
  Does user U inherit knowledge.admin from group G?

Knowledge subsystem answers:
  Can user U read document D?
  Can user U edit collection C?
  Can delegated workflow W access this specific embedding index?
```

`/api/authz/check` should deny or return a clear unsupported decision when a request tries to push subsystem-owned resource authorization into the platform before the platform supports such a model.

## Subsystem SDK contract

Subsystems should integrate through SDKs instead of hand-writing platform glue.

Baseline SDK responsibilities:

```txt
manifest registration
service instance registration and heartbeat
JWKS fetch/cache
JWT verification
current actor extraction
org_id requirement for user/exchanged/delegated tokens
local scope checks
remote authz/check client
delegated token exchange client
service token helpers
on-behalf-of helpers
basic audit client
```

Current SDK priority:

```txt
Java SDK: basically usable for Spring Boot subsystems
TypeScript SDK: needed for Node/TS subsystems
Go SDK: should follow the same contracts, not invent a parallel auth model
contracts: shared schemas and protocol definitions should be language-neutral
```

SDKs should not hide the platform/subsystem boundary. They should make the correct boundary easy to follow.

## Service-to-service and delegation model

There are three different access shapes and they should remain separate:

```txt
service_access:
  subsystem A calls subsystem B as a service
  allowed by service policy
  no broad arbitrary user impersonation

exchanged_access / on-behalf-of:
  subsystem A calls subsystem B while preserving current user and org context
  allowed only when service policy and user permission both allow it

delegated_access:
  workflow / agent / scheduled task acts through a durable delegation grant
  token is short-lived and constrained by grant, org, target app, delegate app, and current user permission
```

Do not store long-lived user tokens in subsystems. Durable authorization should be represented as a platform delegation grant, and executable workers should exchange it for short-lived delegated tokens.

## Frontend implementation baseline

The platform frontend baseline is:

```txt
React
TypeScript
Vite
React Router
Tailwind CSS + CSS variables
shadcn/ui + Radix UI
lucide-react
TanStack Query
TanStack Table
React Hook Form + Zod
Recharts
```

V1 should use Vite SPA because the Spring Boot backend owns HTTP APIs, no SSR is required yet, and there is no BFF/session server layer in the frontend.

The platform frontend should implement:

```txt
first-run bootstrap
login
in-memory auth session
organization switcher
portal app list
native hosted app routes
app switcher
admin shell
admin overview
applications registry
directory and RBAC pages
grants and AuthZ simulator
delegation center
service ops
audit explorer
boundary inspector
central API client and typed hooks
semantic CSS token theme
```

Out of scope for platform frontend v1:

```txt
real Files / Knowledge / Agent / Workflow business logic
workflow editor implementation
agent runtime
OAuth/OIDC consent UI
mandatory Wujie dependency
durable token storage in browser localStorage/sessionStorage
```

## Current phase priority

The current implementation priority should be:

1. Platform backend foundation and bootstrap.
2. Authentication core and org-context token lifecycle.
3. Directory, group, department, and org membership management.
4. Platform admin authorization.
5. App registry and subsystem manifest registration.
6. App/user/group/org context grants and `/api/authz/check`.
7. Delegated authorization.
8. SDK baseline.
9. Service-to-service integration.
10. Demo subsystem end-to-end validation.
11. Frontend portal and Admin Center hardening.

Security experience enhancements such as MFA, passkeys, account recovery, invitations, and new-device UI are intentionally deferred until the platform integration story is validated.

## Design risks and rules

### Do not over-centralize resources

If the platform starts owning every subsystem resource, it will become a bottleneck and all apps will be forced into one authorization model too early.

Keep the line:

```txt
Platform: app-level and context-level authorization.
Subsystem: domain resource authorization.
```

### Do not duplicate grants as source of truth

Do not persist "every user's permission in every group in every subsystem" as the canonical model. Store normalized grants. Compute or cache effective permission results with invalidation/versioning.

### Do not make frontend auth authoritative

Browser checks are UX hints only. Backend and subsystem SDK enforcement are mandatory.

### Do not let SDKs diverge by language

Java, TypeScript, and Go SDKs must share protocol contracts. Language ergonomics may differ, but claims, authz decisions, manifest schema, service policy semantics, and delegation semantics should stay aligned.

### Do not make Admin Center the daily product shell

Daily users should enter native-feel hosted apps. Admin Center is for operators and permission management, not the default visual wrapper for every subsystem.
