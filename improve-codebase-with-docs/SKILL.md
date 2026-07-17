---
name: improve-codebase-with-docs
description: Find deepening opportunities in a codebase, informed by the domain language in CONTEXT.md and the decisions in docs/adr/, seed those docs from already-decided work, and emit a wired refactor checklist (CONVENTIONS.md) agents actually follow. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, make a codebase more testable and AI-navigable, backfill CONTEXT.md / ADRs from shipped findings, or produce a followed CONVENTIONS.md / style-guide / refactor checklist.
---

# Improve Codebase Architecture

Surface architectural friction and propose **deepening opportunities** — refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability.

## Glossary

Use these terms exactly in every suggestion. Consistent language is the point — don't drift into "component," "service," "API," or "boundary." Full definitions in [LANGUAGE.md](LANGUAGE.md).

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the type signature.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place. (Use this, not "boundary.")
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge concentrated in one place.

Key principles (see [LANGUAGE.md](LANGUAGE.md) for the full list):

- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the test surface.**
- **One adapter = hypothetical seam. Two adapters = real seam.**

This skill is _informed_ by the project's domain model. The domain language gives names to good seams; ADRs record decisions the skill should not re-litigate.

## Process

### 1. Explore

Read the project's domain glossary and any ADRs in the area you're touching first.

Then use the Agent tool with `subagent_type=Explore` to walk the codebase. Don't follow rigid heuristics — explore organically and note where you experience friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow** — interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules leak across their seams?
- Which parts of the codebase are untested, or hard to test through their current interface?

Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

### 2. Backfill the domain docs from decided work

Before proposing anything new, record what is already decided — the standing conventions in step 3 rest on it.
Runs whenever `CONTEXT.md` is missing or `docs/adr/` is empty, or the user asks to seed the docs from finished work.
These are real decisions, already shipped, so recording them fabricates nothing; don't wait for the grilling loop to write them one at a time.

- **CONTEXT.md** — harvest the domain nouns the landed modules are named after. Project-specific terms only (an `Order`, a `Destination` — not general programming concepts); opinionated, with an `_Avoid_` list of banned synonyms per term. Follow [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md). Never invent a term the code doesn't use.
- **ADRs** — write one per decided candidate that clears the [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md) bar: **hard to reverse · surprising without context · a real trade-off**. Reverted/rejected candidates and deliberate deviations ("we forked X on purpose", "we kept the mixed shape") are the highest-value ADRs — they stop the next review re-suggesting the settled thing. Number sequentially from the highest existing file.
- Sources for "what's decided": the repo's git history, `CLAUDE.md`/`AGENTS.md`, linter config, and any prior checklist's shipped ledger.

Skip any decision that is easy to reverse, unsurprising, or had no real alternative — same bar as the grilling loop's ADR offer. Don't manufacture an ADR just to fill the directory.

### 3. Emit the checklist and wire it

The one durable, followed artifact is `CONVENTIONS.md` at the repo root (beside `CLAUDE.md`/`AGENTS.md` and `CONTEXT.md`) — a Markdown checklist, the only architecture doc this skill maintains (see [CHECKLIST-REPORT.md](CHECKLIST-REPORT.md) for the full scaffold, sections, and wiring snippets). Markdown, not HTML: it's edited every round and reviewed in PR diffs. Stable filename (no timestamp) so each run overwrites cleanly and diffs in git. Tell the user the absolute path.

If a checklist already exists, read it first: carry the shipped ledger forward (it is the authority on status), don't re-propose shipped or rejected items, and let its deferrals inform this round.

- **Three sections.**
  - *Standing conventions* (`SC*`, binding for new code) — sourced from `CLAUDE.md`/`AGENTS.md`, linter config, and the step-2 ADRs, **not invented**. Each row cites its basis.
  - *Open refactor checklist* — one card per candidate: target-state rule line + why + Do/Don't (where a snippet clarifies) + cost pill + evidence count. Order cards cheapest/highest-signal first and state that order in the lede. Use CONTEXT.md vocabulary for the domain and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture. If a candidate contradicts an ADR, flag it with a callout rather than silently proposing it.
  - *Shipped ledger* — done deepenings (`✓`/`◐`/`✗` reverted), each citing its ADR and the `SC` it became.
- **Wire it — a doc nobody loads changes nothing.** Detect what the repo uses, then wire three ways (see [CHECKLIST-REPORT.md](CHECKLIST-REPORT.md) for exact snippets): (1) a "Domain docs & conventions" pointer in every agent memory file present — `CLAUDE.md` **and/or** `AGENTS.md` — marking which docs are binding vs proposals; (2) a PreToolUse hook in `.claude/settings.json` (matcher `Edit|Write|MultiEdit`, firing on the repo's real source glob) telling the agent to read `CONVENTIONS.md` before editing source — append to existing hooks, never clobber, validate with `jq empty`; (3) optionally, lint rules for the standing conventions a linter can mechanically enforce. Match the repo's actual source root/extensions and memory-file convention — don't assume `CLAUDE.md` + `src/**/*.{ts,vue}`.
- **Standing conventions are binding; open items are proposals.** The hook must say so explicitly, so an agent applies the conventions but never auto-implements an open candidate.

Do NOT propose interfaces yet. After the checklist is written, ask the user: "Which of these would you like to explore?"

### 4. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.

Side effects happen inline as decisions crystallize:

- **Naming a deepened module after a concept not in `CONTEXT.md`?** Add the term to `CONTEXT.md` — same discipline as `/grill-with-docs` (see [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md)). Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term during the conversation?** Update `CONTEXT.md` right there.
- **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future explorer to avoid re-suggesting the same thing — skip ephemeral reasons ("not worth it right now") and self-evident ones. See [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md).
- **Want to explore alternative interfaces for the deepened module?** See [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).
- **Candidate implemented?** Update `CONVENTIONS.md`: flip its open item to a `✓` row in the shipped ledger, cite its new ADR, and — if it establishes an always/never rule for future code — add a Standing convention (`SC*`) referencing that ADR.
