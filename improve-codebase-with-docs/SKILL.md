---
name: improve-codebase-with-docs
description: Find deepening opportunities in a codebase, informed by the domain language in CONTEXT.md and the decisions in docs/adr/, and seed those docs from already-decided work. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, make a codebase more testable and AI-navigable, or backfill CONTEXT.md / ADRs from shipped or approved findings.
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

### 2. Present candidates as a Markdown report

Write a Markdown file to `docs/codebase-guide.md` in the repo so it's versioned alongside `docs/adr/`. Create the `docs/` directory if it doesn't exist. Use the stable filename (no timestamp) so each run overwrites the previous guide and git diffs stay clean. Tell the user the absolute path.

If a previous run's guide already exists at that path, read it first: carry decided/implemented/rejected candidates forward as a short status ledger (one line each), don't re-propose them, and let its deferrals inform this round's candidates.

Diagrams use **Mermaid fenced code blocks** (```mermaid) — GitHub, VS Code, and most Markdown viewers render them natively. Use Mermaid when relationships are graph-shaped (call graphs, dependencies, sequences); use Markdown tables when the point is a side-by-side divergence comparison. Each candidate gets a **before/after visualisation** (a two-subgraph Mermaid diagram or a comparison table). Be visual, within what Markdown can carry.

For each candidate, the same template as before, rendered as a `##` section:

- **Files** — which files/modules are involved, with line numbers
- **Problem** — why the current architecture is causing friction
- **Solution** — plain English description of what would change
- **Benefits** — explained in terms of locality and leverage, and how tests would improve
- **Before / After diagram** — Mermaid subgraphs or a comparison table, illustrating the shallowness and the deepening
- **Recommendation strength** — one of 🟢 `Strong`, 🟠 `Worth exploring`, ⚪ `Speculative`, stated on the line under the heading

End the report with a **Top recommendation** section: which candidate you'd tackle first and why.

**Use CONTEXT.md vocabulary for the domain, and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture.** If `CONTEXT.md` defines "Order," talk about "the Order intake module" — not "the FooBarHandler," and not "the Order service."

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly in the section (e.g. a blockquote callout: _"contradicts ADR-0007 — but worth reopening because…"_). Don't list every theoretical refactor an ADR forbids.

See [MD-REPORT.md](MD-REPORT.md) for the full Markdown scaffold, diagram patterns, and style guidance.

Do NOT propose interfaces yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 2b. Backfill the domain docs from decided work

Runs when the guide already carries a ledger of implemented/decided/rejected candidates but `CONTEXT.md` is missing or `docs/adr/` is empty — or whenever the user asks to seed the docs from finished work.
These are real decisions, already shipped, so recording them fabricates nothing; don't wait for the grilling loop to write them one at a time.

- **CONTEXT.md** — harvest the domain nouns the landed modules are named after. Project-specific terms only (an `Order`, a `Destination` — not general programming concepts); opinionated, with an `_Avoid_` list of banned synonyms per term. Follow [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md). Never invent a term the code doesn't use.
- **ADRs** — write one per decided candidate that clears the [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md) bar: **hard to reverse · surprising without context · a real trade-off**. Reverted/rejected candidates and deliberate deviations ("we forked X on purpose", "we kept the mixed shape") are the highest-value ADRs — they stop the next review re-suggesting the settled thing. Number sequentially from the highest existing file.
- **Cross-reference** — cite the new ADR numbers from the guide's ledger rows and from any live candidate they constrain, and fix the lede so it no longer claims the docs are absent.

Skip any decision that is easy to reverse, unsurprising, or had no real alternative — same bar as the grilling loop's ADR offer. Don't manufacture an ADR just to fill the directory.

### 3. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.

Side effects happen inline as decisions crystallize:

- **Naming a deepened module after a concept not in `CONTEXT.md`?** Add the term to `CONTEXT.md` — same discipline as `/grill-with-docs` (see [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md)). Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term during the conversation?** Update `CONTEXT.md` right there.
- **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future explorer to avoid re-suggesting the same thing — skip ephemeral reasons ("not worth it right now") and self-evident ones. See [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md).
- **Want to explore alternative interfaces for the deepened module?** See [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).
