# Checklist Report Format (`docs/style-guide.html`)

The checklist is the **followed** artifact: a single self-contained HTML file at `docs/style-guide.html` that an agent reads before editing code.
`docs/codebase-guide.md` stays the **detailed backing** (before/after diagrams + file:line evidence per candidate); the checklist is its scannable, wired front end.

Emit the checklist when the review has produced candidates and at least some decided/shipped work exists to bank.
Same stable filename each run (no timestamp) so it overwrites cleanly and diffs in git.

## Why HTML, not Markdown

The checklist is wired to a PreToolUse hook (below) and read as a rendered page — status glyphs, cost pills, and a scan table carry it.
Self-contained: inline CSS, no CDN, no external assets. It renders by double-click and diffs in git.

## Three sections

### 1. Standing conventions (binding for new code)

A compact table of the rules new code must follow — `SC1`, `SC2`, … Each row: rule (one line, "do X / never Y"), basis, status.

- **Source them, don't invent them.** Pull from what's already true and decided: the repo's `CLAUDE.md`, any linter config, and the ADRs written in step 2b. An ADR-backed decision becomes a standing convention; cite the ADR in the basis column.
- Status glyph: `✓` done/dominant · `◐` partial (rule active, migration open) · link the migration to an open checklist item.
- These are what the hook enforces. Keep the list tight — only rules a reviewer would actually reject a diff over.

### 2. Refactor checklist — open

One card per open candidate (same numbers as `codebase-guide.md`). Each card:

- **Heading** — `N · title` + `○ open` tag + optional `ADR-NNNN` tag + a cost pill (`low`/`mid`/`high`).
- **Rule line** — the *target state* as an imperative (what the code should look like after), accented left-border.
- **Why** — one to three sentences, the friction + evidence.
- **Do / Don't** — a two-column code comparison for the candidates where a snippet clarifies; omit for the self-evident ones.
- **Meta line** — evidence count ("23 unwrap sites", "~11 re-scans") and/or the deletion-test verdict.

Order the cards by tackle-order (cheapest/highest-signal first), and state that order in the section lede.

### 3. Shipped ledger

A table of the done deepenings — kept so they aren't re-proposed. Columns: `#`, deepening, status (`✓`/`◐`/`✗` reverted), decision (one line; cite the ADR and which `SC` it became).

## Header, callout, legend

- **Header** — repo name + "Refactor Checklist & Style Guide", one-line subtitle.
- **Callout** (info box, top) — the doc map, in four lines: *this file* = checklist + binding standing conventions; `codebase-guide.md` = detailed backing; `CONTEXT.md` = binding vocabulary; `docs/adr/` = decisions. State plainly: open items are proposals — pick, grill, then they land as ✓.
- **Legend** — the status glyphs and cost pills.

## Self-contained styling

Inline `<style>` only. Reuse this token set (light, editorial, matches the sibling docs):

```
--bg:#faf9f5; --fg:#141413; --muted:#6b6862; --accent:#d97757; --line:#e8e5dd;
--success:#4a7c59; --warning:#c28b2d; --danger:#b54848; --info:#4a6b8a;
spacing 4/8/16/24/32/64 · radius 4/8/16 · max-width 1100px
```

Cards: white on `--bg`, 1px `--line`, subtle shadow. Rule line: `--accent` left border. Status glyphs coloured by state. `.grid-2` collapses to one column under 700px. Keep it responsive; wide code blocks scroll in their own `overflow-x:auto` `<pre>`.

## Wiring (this is what makes it followed)

A document nobody loads changes nothing. After writing the checklist, wire it three ways so agents actually read and obey it. First **detect what the repo uses** — don't assume:

- Agent memory file: `CLAUDE.md`, `AGENTS.md`, or both (also `.cursor/rules/*`, `.github/copilot-instructions.md` if present). Update **every** one that exists; if none exists, create the one matching the primary harness (`CLAUDE.md` for Claude Code, `AGENTS.md` otherwise).
- Settings for hooks: `.claude/settings.json` (project) — the hook mechanism is Claude Code-specific.
- Linter config: `.eslintrc*` / `eslint.config.*`, `ruff.toml`, etc. — for the standing conventions a linter can mechanically enforce.
- The repo's source root + extensions (`src/**/*.{ts,vue}`, `app/**/*.rb`, …) — the hook glob must match reality.

### 1. Agent memory pointer (portable — every harness reads this)

Add a short "Domain docs & style guide" section to each memory file found (`CLAUDE.md` **and** `AGENTS.md` when both exist — keep them in sync). This is the only file auto-loaded every session, and the only enforcement non–Claude-Code agents get. State which docs are **binding** (style-guide standing conventions, `CONTEXT.md`, `docs/adr/`) and which are **proposals** (open checklist + `codebase-guide.md` — reference, never auto-implement). Point at `docs/style-guide.html`.

### 2. PreToolUse hook (Claude Code — the forcing function)

Merge into `.claude/settings.json` (create `hooks.PreToolUse` if absent; **append**, never clobber existing hooks — many repos already have a Bash/graphify hook). Matcher `Edit|Write|MultiEdit`; the `jq` command fires only on the repo's source glob (adjust `/src/.*\.(ts|vue)$` to the real root/extensions):

```json
{
  "matcher": "Edit|Write|MultiEdit",
  "hooks": [
    {
      "type": "command",
      "command": "jq -c 'if (.tool_input.file_path // \"\") | test(\"/src/.*\\\\.(ts|vue)$\") then {hookSpecificOutput:{hookEventName:\"PreToolUse\",additionalContext:\"STYLE GUIDE (MANDATORY): editing a source file. If you have not read docs/style-guide.html this session, STOP and read it now. Its Standing conventions (SC*) are binding for new code; CONTEXT.md vocabulary and docs/adr decisions are also binding. The open checklist items and docs/codebase-guide.md are proposals — do NOT auto-implement. Violations of standing conventions require rework.\"}} else empty end'"
    }
  ]
}
```

Validate with `jq empty .claude/settings.json` after editing, and dry-run the matcher on a sample path before trusting it.

### 3. Lint enforcement (optional — mechanical rules)

For standing conventions a linter can catch, add the rule so drift is caught automatically, not just by reminder (the strongest form of "followed"). Examples: a no-restricted-imports ban, an import-alias rule, a banned-syntax rule, a dependency-version pin. Cite the lint rule in the `SC` row's basis column. Skip conventions that aren't mechanically checkable — the hook + memory pointer carry those.

## Keeping it in sync (grilling loop)

When a candidate is implemented in step 3:

1. Flip its card from `○ open` to a `✓` row in the shipped ledger.
2. Write its ADR (if it clears the bar) and cite it.
3. If it establishes a new always/never rule for future code, add a Standing convention (`SC*`) citing that ADR.

The checklist is the living surface; `codebase-guide.md` keeps the full history and diagrams.
