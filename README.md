# Skills

Personal collection of agent skills.

## agentic-flow requirements

`agentic-flow` orchestrates other skills. Beyond Claude Code built-ins (`code-review`, `verify`, `run`), these must be installed:

- [graphify](https://github.com/safishamsi/graphify) — codebase knowledge graph (step 1 context, step 6 update)
- `grill-with-docs` (this repo) — brainstorm + grill, CONTEXT.md/ADR updates (step 1)
- `html-artifacts` — render plan as self-contained HTML (step 2)
- [superpowers](https://github.com/obra/superpowers) plugin — `writing-plans` (step 2), `test-driven-development` (step 3), `verification-before-completion` + `requesting-code-review` (step 4)

## Good to have

- [claude-mem](https://github.com/thedotmack/claude-mem) — gives Claude a long-term memory. Normally Claude forgets everything when a session ends; claude-mem automatically records what you worked on and lets future sessions look it up, so you can ask things like "what did I change in auth last month" without re-explaining anything. Technically: lifecycle hooks capture session work into SQLite + vector search. Alternative to built-in memory when automatic recall across sessions/projects is needed. _Note: burns a lot of tokens — not recommended for Pro users._
- [ccstatusline](https://github.com/sirmalloc/ccstatusline) — customizable status line for the bottom of the Claude Code terminal. Shows model, thinking level, context used, session usage, reset timer, account, memory, git branch, and more at a glance. Highly configurable widgets, colors, and layout.

## Attribution

- `grill-with-docs/` — from [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs)
- `improve-codebase-with-docs/` — based on [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture)
- `html-artifacts/` — by [margibs](https://github.com/margibs)
