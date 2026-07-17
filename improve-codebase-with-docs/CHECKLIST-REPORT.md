# Checklist Format (`CONVENTIONS.md`)

The checklist is the one durable, followed artifact: a Markdown file named `CONVENTIONS.md` at the **repo root**, beside `CLAUDE.md`/`AGENTS.md` and `CONTEXT.md`. It is the only architecture doc the skill maintains — candidates live in it as open items, not in a separate guide.

**Markdown, not HTML.** The doc is edited every round (flip `○`→`✓`, add `SC` rows) and reviewed in PR diffs, and agents read it as text either way — so keep it plain. Status glyphs + scan tables carry it. Stable filename (no timestamp) so each run overwrites cleanly and diffs in git.

Write one full sentence per line, so diffs between runs stay readable.

## Header

- H1 title (`<repo> — Conventions & Refactor Checklist`) + a one-line subtitle.
- **Callout** — a blockquote up top mapping the docs, in three lines: *this file* = the durable checklist + binding standing conventions; `CONTEXT.md` = binding vocabulary; `docs/adr/` = decisions. State plainly: open items are proposals — pick, grill, then they land as `✓`.
- **Legend** — one line: `✓` done · `◐` partial · `○` open · cost `low`/`mid`/`high`.

## Three sections

### 1. Standing conventions (binding for new code)

A Markdown table — `SC1`, `SC2`, … Columns: `#`, Rule (one line, "do X / never Y"), Basis, Status.

- **Source them, don't invent them.** Pull from what's already true and decided: the repo's `CLAUDE.md`/`AGENTS.md`, linter config, and the ADRs from step 2. An ADR-backed decision becomes a standing convention; cite the ADR in the Basis column.
- Status glyph: `✓` done/dominant · `◐` partial (rule active, migration open) — link the migration to an open item (e.g. "◐ see #27").
- Keep it tight — only rules a reviewer would actually reject a diff over.

### 2. Refactor checklist — open

One `###` subsection per open candidate (stable candidate numbers across rounds). Each:

- **Rule line** — the *target state* as an imperative (a blockquote), what the code should look like after.
- **Why** — one to three sentences: the friction + evidence.
- **Do / Don't** — a fenced code block for the candidates where a snippet clarifies; omit for the self-evident ones.
- **Evidence / cost** — the status + cost + a count on one line (e.g. `○ open · mid` and "23 unwrap sites").

Order the subsections by tackle-order (cheapest/highest-signal first), and state that order in the section lede.
Use `CONTEXT.md` vocabulary for the domain and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture.
If a candidate contradicts an ADR, flag it with a callout rather than silently proposing it.

### 3. Shipped ledger

A Markdown table of the done deepenings — kept so they aren't re-proposed. Columns: `#`, Deepening, Status (`✓`/`◐`/`✗` reverted), Decision (one line; cite the ADR and which `SC` it became).

## Wiring (this is what makes it followed)

A document nobody loads changes nothing. After writing `CONVENTIONS.md`, wire it three ways so agents actually read and obey it. First **detect what the repo uses** — don't assume:

- Agent memory file: `CLAUDE.md`, `AGENTS.md`, or both (also `.cursor/rules/*`, `.github/copilot-instructions.md` if present). Update **every** one that exists; if none, create the one matching the primary harness.
- Settings for hooks: `.claude/settings.json` (project) — the hook mechanism is Claude Code-specific.
- Linter config: `.eslintrc*` / `eslint.config.*`, `ruff.toml`, etc.
- The repo's source root + extensions (`src/**/*.{ts,vue}`, `app/**/*.rb`, …) — the hook glob must match reality.

### 1. Agent memory pointer (portable — every harness reads this)

Add a short "Domain docs & conventions" section to each memory file found (`CLAUDE.md` **and** `AGENTS.md` when both exist — keep them in sync). This is the only file auto-loaded every session, and the only enforcement non–Claude-Code agents get. State which docs are **binding** (the `CONVENTIONS.md` standing conventions, `CONTEXT.md`, `docs/adr/`) and which are **proposals** (the open checklist items — reference, never auto-implement). Point at `CONVENTIONS.md`.

### 2. PreToolUse hook (Claude Code — the forcing function)

Merge into `.claude/settings.json` (create `hooks.PreToolUse` if absent; **append**, never clobber existing hooks — many repos already have a Bash/graphify hook). Matcher `Edit|Write|MultiEdit`; the `jq` command fires only on the repo's source glob (adjust `/src/.*\.(ts|vue)$` to the real root/extensions):

```json
{
  "matcher": "Edit|Write|MultiEdit",
  "hooks": [
    {
      "type": "command",
      "command": "jq -c 'if (.tool_input.file_path // \"\") | test(\"/src/.*\\\\.(ts|vue)$\") then {hookSpecificOutput:{hookEventName:\"PreToolUse\",additionalContext:\"CONVENTIONS (MANDATORY): editing a source file. If you have not read CONVENTIONS.md this session, STOP and read it now. Its Standing conventions (SC*) are binding for new code; CONTEXT.md vocabulary and docs/adr decisions are also binding. The open checklist items in CONVENTIONS.md are proposals — do NOT auto-implement. Violations of standing conventions require rework.\"}} else empty end'"
    }
  ]
}
```

Validate with `jq empty .claude/settings.json` after editing, and dry-run the matcher on a sample path before trusting it.

### 3. Lint enforcement (optional — mechanical rules)

For standing conventions a linter can catch, add the rule so drift is caught automatically, not just by reminder (the strongest form of "followed"). Examples: a no-restricted-imports ban, an import-alias rule, a banned-syntax rule, a dependency-version pin. Cite the lint rule in the `SC` row's Basis column. Skip conventions that aren't mechanically checkable — the hook + memory pointer carry those.

## Keeping it in sync (grilling loop)

When a candidate is implemented in step 4:

1. Flip its subsection from `○ open` to a `✓` row in the shipped ledger.
2. Write its ADR (if it clears the bar) and cite it.
3. If it establishes a new always/never rule for future code, add a Standing convention (`SC*`) citing that ADR.

`CONVENTIONS.md` is the living surface and the full record; git history holds the diffs.
