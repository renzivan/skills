---
name: agentic-flow
description: End-to-end agentic development pipeline - brainstorm, grill, write a plan doc with zero assumptions, implement with TDD, then hand to a SEPARATE reviewer agent for verify + audit. Use when the user gives a feature/task to build and wants the full plan-then-build-then-review loop, says "agentic flow", "/agentic-flow", "do this agentically", or "plan it then build it".
---

# Agentic Flow

Drives a task from intent to merged code across two clean contexts: a **builder** (you, collaborating with the human) and a **separate reviewer** (fresh subagent). Core rule: **never assume — every unknown becomes a question to the human.**

## The pipeline

Create one TodoWrite item per phase. Do them in order. Do not skip gates.

### 1. Brainstorm + Grill

First, self-serve technical context: run `graphify query "<question>"` (and `graphify explain` / `graphify path`) to answer anything the codebase can answer — where things live, how they're wired, what models/fields exist. **Never ask the human a question graphify can answer.**
Then invoke `grill-with-docs` and spend the human's time only on intent: problem, users, edge cases, constraints — the things the graph cannot know. Walk every branch of the decision tree until you and the human agree, challenging the plan against the domain language in `CONTEXT.md` and existing ADRs, updating them inline as decisions crystallise. No code.

### 2. Plan doc — ZERO ASSUMPTIONS

Invoke `superpowers:writing-plans` to structure the plan, then render it with the `html-artifacts` skill as a single self-contained HTML file at `docs/plans/<slug>.html`. HTML, not markdown — layout carries the phases, the task checklist, and the Open Questions so the human can scan and approve fast.

- The doc MUST contain an **Open Questions** section, visually distinct.
- Any unknown, default, or "I'll assume X" → goes in Open Questions, NOT filled by guess.
- **If there are open questions, STOP and ask the human. Resolve all before writing the final doc.**
- No approval gate on the doc itself. Once open questions are answered (or there are none), write the doc and proceed straight to implementation.

### 3. Implement — ask, don't assume

Invoke `superpowers:test-driven-development`. Follow the approved doc.

- Test first, then code. Tests are part of the build, written by the builder.
- Hit ambiguity mid-build? **STOP, ask the human.** Never pick a "reasonable default" silently.
- Commit once per completed plan task or coherent unit of work (e.g. test + impl pair) — NOT per file edit or per line. Plain messages, no Claude co-author — see project CLAUDE.md.

### 4. Review — SEPARATE agent, clean context

Spawn a fresh subagent via the Agent tool (do NOT review your own work inline). The reviewer:

- Runs the app / tests to verify behavior — use the `verify` / `run` skills (and a project-specific run skill if one exists), plus `superpowers:verification-before-completion`.
- Audits the diff for bugs + quality (`code-review` skill).
- Reports findings as a structured list (severity, file:line, problem, fix). Reports back — does not fix.

Use `superpowers:requesting-code-review` to frame the handoff.

### 5. Fix loop

Builder addresses findings → re-run reviewer until clean. Each round = fresh reviewer context.

### 6. Finish

Run `graphify update .` to keep the knowledge graph current. No merge, no PR — that's the human's call, not part of this flow. Tell the human the flow is done and invite change requests.

### 7. Revisions — human wants changes after the flow

Route by size, don't restart blindly:

- **Trivial** (rename, copy tweak, small fix): builder edits directly → reviewer re-checks the diff → `graphify update .`. No brainstorm.
- **Scoped change** (new behavior on existing feature): re-enter at **step 2**, append a "Revision" section to the existing plan HTML doc, surface new Open Questions if any, then build → review → graphify.
- **Big / changes the intent**: re-enter at **step 1**. It's effectively a new task.

Pick the smallest entry point that fits. State which one you're using and why before acting.

## Roles (keep it to two contexts)

| Role                | Who                 | Context              |
| ------------------- | ------------------- | -------------------- |
| Planner + Builder   | you, with the human | shared, accumulates  |
| Reviewer / Verifier | separate subagent   | fresh, no build bias |

No dedicated "tester" agent — verify folds into the reviewer. Respect spawn-depth cap 2 (project CLAUDE.md).

## The two hard rules

1. **No assumptions.** Unknown → Open Questions → ask the human. Plan phase and build phase both.
2. **Reviewer is always a separate agent.** The builder cannot review itself; biased context misses its own gaps.
