# Skills

Personal collection of agent skills.

## agentic-flow requirements

`agentic-flow` orchestrates other skills. Beyond Claude Code built-ins (`code-review`, `verify`, `run`), these must be installed:

- [graphify](https://github.com/safishamsi/graphify) — codebase knowledge graph (step 1 context, step 6 update)
- `grill-with-docs` (this repo) — brainstorm + grill, CONTEXT.md/ADR updates (step 1)
- `html-artifacts` — render plan as self-contained HTML (step 2)
- [superpowers](https://github.com/obra/superpowers) plugin — `writing-plans` (step 2), `test-driven-development` (step 3), `verification-before-completion` + `requesting-code-review` (step 4)

## Attribution

- `grill-with-docs/` — from [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs)
- `improve-codebase-architecture/` — from [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture)
