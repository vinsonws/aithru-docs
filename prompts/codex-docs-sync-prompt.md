# Codex Docs Sync Prompt

Use this prompt when asking a coding agent to update `aithru-docs` after implementation changes in `aithru-core`, `aithru-web`, or future Aithru repositories.

```txt
You are working on the Aithru ecosystem.

Read these files first:

1. README.md
2. docs/01-ecosystem-overview.md
3. docs/02-module-boundaries.md
4. docs/03-aithru-core.md
5. docs/04-aithru-web.md
6. docs/05-aithru-agent.md
7. docs/06-runtime-trace-approval-tooling.md
8. docs/07-roadmap.md
9. docs/08-development-workflow.md
10. adr/*.md

Then inspect the implementation repository changes you were given.

Your task:

- Identify whether the implementation changed any cross-repo architecture boundary.
- Update the relevant docs in aithru-docs.
- Add or update an ADR if the change affects dependency direction, module ownership, runtime execution, trace, approval, tool policy, Agent integration, MCP integration, or the Web/runtime split.
- Do not put package-local API usage details here unless they affect cross-repo design.
- Keep docs concise and actionable.
- Preserve the rule that core defines contracts, optional modules implement capabilities, and product shells compose modules.

When finished, report:

- files changed;
- design decisions updated;
- any implementation docs that still need follow-up in the target repository;
- verification commands run, if any.
```
