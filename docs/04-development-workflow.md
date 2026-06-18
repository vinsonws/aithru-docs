# Development Workflow

## Default Flow

```txt
Read this docs repository for cross-repo boundaries.
Read the target repository README and local design docs for implementation details.
Make the smallest coherent change in the target repository.
Run that repository's verification commands.
Update this docs repository only if a cross-repo boundary changed.
```

## When To Update This Repository

Update `aithru-docs` when a change affects:

- active repository ownership;
- dependency direction between Platform, Flowe, Agent, and future subsystems;
- platform security, token, grant, SDK, or hosted-app contracts;
- Flowe workflow/runtime semantics that another repository must rely on;
- Agent capability, approval, tool, event, or trace contracts;
- frontend stack, shell mode, security, or visual constraints shared across
  current or future frontends.

Do not update this repository for:

- package install commands;
- endpoint inventory that only affects one repository;
- phase notes, milestones, or roadmaps;
- implementation details already covered by the target repository README.

## Repository Verification Commands

Use the target repository's current README as the final authority. Current
representative checks are:

```bash
# aithru-platform backend
cd ../aithru-platform/backend
mvn test

# aithru-platform frontend
cd ../aithru-platform/frontend
npm run typecheck
npm run build

# aithru-flowe
cd ../aithru-flowe
go test ./...

# aithru-agent backend
cd ../aithru-agent/backend
uv run pytest
uv run python examples/file_report_agent.py
```

## Docs Review Checklist

Before finishing a meaningful docs change:

- Does `README.md` link only to files that exist?
- Are only active repositories described as current owners?
- Are retired names kept out of current ownership tables?
- Is there a roadmap or future phase list that should be removed?
- Did frontend constraints reflect the active platform frontend implementation?
- Did security rules preserve the backend/browser boundary?
- Did the change avoid duplicating details that belong in an implementation
  repository README?
