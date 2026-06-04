# Development Workflow

Aithru should use a design-first workflow for cross-repository changes.

## Default flow

```txt
Design in aithru-docs
  -> implement in the relevant repository
  -> verify behavior
  -> update package README/API docs
  -> update docs if the design changed
```

## When to start in docs

Start in `aithru-docs` when a change affects:

- module boundaries;
- public contracts;
- package ownership;
- runtime execution model;
- trace, redaction, approval, or tool policy;
- Agent/MCP integration model;
- UI/runtime split;
- desktop/server product line direction.

Small package-local bug fixes do not need an ADR.

## Repository-specific docs

Use package README files for:

- installation commands;
- import examples;
- minimal usage examples;
- package-local boundary notes;
- package-specific verification commands.

Use `aithru-docs` for:

- cross-repo design;
- roadmap;
- ADRs;
- module ownership;
- product direction;
- prompts for implementation agents.

## Verification expectations

For `aithru-core`, keep using the repository verification script before finishing runtime changes:

```bash
./scripts/verify.sh
```

For `aithru-web`, keep checking:

```bash
corepack pnpm --filter aithru-web typecheck
corepack pnpm --filter aithru-web build
```

Future repositories should define their own verification commands and link them from this documentation hub.

## Agent-assisted implementation workflow

A recommended loop for Codex or another coding agent:

```txt
1. Read README.md in aithru-docs.
2. Read the relevant docs/* and adr/* documents.
3. Read the target repository README and package README files.
4. Produce a small implementation plan.
5. Make changes in the target repository.
6. Run verification commands.
7. Update docs if implementation diverged from the design.
8. Summarize changed files and verification result.
```

## Documentation update checklist

After a meaningful change, check:

- Does README still point to the right documents?
- Did module ownership change?
- Did dependency direction change?
- Did execution semantics change?
- Did browser-safe import rules change?
- Did Agent or MCP behavior change?
- Did roadmap status change?
- Does an ADR need to be added or updated?
