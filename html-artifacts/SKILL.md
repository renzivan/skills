---
name: html-artifacts
description: Use when generating a deliverable where layout, comparison, interactivity, or visual hierarchy carries information — code reviews, design systems, planning docs, prototypes, status reports, slide decks, diagrams, custom editors, research explainers. Produces a single self-contained HTML file instead of Markdown. Triggers on "give me an HTML report", "make this an artifact", "side-by-side comparison", "interactive prototype", "design system page", "slide deck", "incident report", "PR writeup", "make it visual", or whenever the user asks for a doc-shaped output that benefits from spatial layout. Based on Thariq S. — *The Unreasonable Effectiveness of HTML*.
---

# HTML Artifacts

Default to a self-contained `.html` file when the output benefits from layout, comparison, interaction, or visual hierarchy. Markdown collapses dimensions HTML preserves.

## When to use

Use HTML over Markdown when ANY of these apply:

- The reader needs to **compare** options side-by-side (trade-off tables, design alternatives, code approaches).
- The output has **spatial structure** (dependency graphs, flowcharts, timelines, annotated PRs, margin notes).
- It's a **deliverable that ships in HTML anyway** (design systems, slide decks, dashboards, status pages, component sheets).
- It needs **interaction** to be understood (animation tuners, clickable prototypes, manipulable concept demos).
- It's a **custom editor** whose value is the round-trip (drag-to-triage, flag editor, prompt tuner — must include an export button).

Stick with Markdown when the output is genuinely linear prose: a chat reply, a single answer, a short note, code-only.

## The nine recipes

| Use case                         | Pattern                                                                                                                                     |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Exploration / planning**       | Three columns, one per option. Trade-offs as inline pill tags. Risk matrix at the bottom.                                                   |
| **Code review**                  | File diff in main column. Margin notes with severity tags (info/warn/block). Sticky file index sidebar.                                     |
| **PR writeup**                   | Author voice intro → motivation → file-by-file rationale with code snippets → testing notes.                                                |
| **Code understanding**           | Module dependency graph (inline SVG). Highlight critical path. Click-to-expand details per node.                                            |
| **Design system**                | Token tables (color swatches with copyable hex), typography scale, spacing ruler, component variants grid, elevation samples.               |
| **Component variants**           | Every state × size in one grid. Labeled. Real components, not screenshots.                                                                  |
| **Animation prototype**          | Live sample + sliders for duration / easing / delay. Reset button.                                                                          |
| **Interaction prototype**        | Multi-screen flow, click to advance, breadcrumb of current state.                                                                           |
| **SVG illustration / flowchart** | Inline `<svg>`. Hoverable nodes. Annotations as foreignObject or sibling divs.                                                              |
| **Slide deck**                   | `<section>` per slide, ~20 lines of JS for next/prev + arrow keys. No framework.                                                            |
| **Research explainer**           | Collapsible `<details>` sections, tabbed code samples, glossary in margin column.                                                           |
| **Concept explainer**            | Embed an interactive demo of the concept itself (e.g. consistent-hashing ring with addable nodes).                                          |
| **Status report**                | KPI cards at top, charts (inline SVG), table of work items, optimized for Monday-morning scan.                                              |
| **Incident report**              | Minute-by-minute timeline, severity color band, action checklist, postmortem section.                                                       |
| **Custom editor**                | Task-specific UI (drag, toggle, edit). **Always include an Export button** that emits JSON / Markdown the user can paste back to the agent. |

## Build rules

1. **One file. No build step. No `node_modules`.** Inline CSS in `<style>`, inline JS in `<script>`. External resources only if essential and CDN-hosted.
2. **Inline SVG, not images.** SVG is the agent's pen — editable, scalable, themeable.
3. **Self-contained means runnable from `file://`.** Open in browser, works.
4. **Side-by-side beats stacked** when the reader is comparing. Use CSS grid with named columns.
5. **Show, don't describe** for motion, timing, interaction. If you'd write "the animation eases out over 300ms," ship a slider instead.
6. **Export buttons close the loop.** Any editor-style artifact must export its state as JSON or Markdown to a textarea the user can copy.
7. **Accessibility basics:** semantic tags (`<header>`, `<main>`, `<section>`, `<nav>`), labeled controls, sufficient contrast, keyboard navigation for slide decks.

## Visual style defaults

Match the gallery's restrained editorial feel. Don't ship default-Tailwind.

```css
:root {
  /* warm earthy palette */
  --bg: #faf9f5; /* ivory */
  --fg: #141413; /* slate */
  --muted: #6b6862;
  --accent: #d97757; /* clay */
  --line: #e8e5dd;
  --success: #4a7c59;
  --warning: #c28b2d;
  --danger: #b54848;
  --info: #4a6b8a;

  /* 8-point spacing */
  --s-1: 4px;
  --s-2: 8px;
  --s-3: 16px;
  --s-4: 24px;
  --s-5: 32px;
  --s-6: 64px;

  /* type scale */
  --t-display: 48px;
  --t-h1: 32px;
  --t-h2: 24px;
  --t-h3: 18px;
  --t-body: 15px;
  --t-caption: 12px;

  /* radius + elevation */
  --r-sm: 4px;
  --r-md: 8px;
  --r-lg: 16px;
  --r-xl: 20px;
  --shadow-1: 0 1px 2px rgba(20, 20, 19, 0.06);
  --shadow-2: 0 4px 12px rgba(20, 20, 19, 0.08);
  --shadow-3: 0 12px 32px rgba(20, 20, 19, 0.12);
}
body {
  margin: 0;
  background: var(--bg);
  color: var(--fg);
  font:
    var(--t-body)/1.55 -apple-system,
    ui-sans-serif,
    system-ui,
    "Inter",
    sans-serif;
  font-feature-settings: "ss01", "cv11";
}
```

Sans-serif system stack, line-height 1.55 for body / 1.1–1.2 for headings, generous whitespace, soft shadows, rounded corners (4–20px). Avoid pure black, pure white, neon, gradients-by-default.

## Skeleton

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{{artifact title}}</title>
    <style>
      /* tokens + layout (see Visual style defaults) */
    </style>
  </head>
  <body>
    <header>
      <h1>{{title}}</h1>
      <p class="muted">{{subtitle}}</p>
    </header>
    <main>
      <!-- task-specific content: grid of options / SVG diagram / slides / etc -->
    </main>
    <script>
      /* only if interaction is required */
    </script>
  </body>
</html>
```

## Anti-patterns

- Producing a Markdown document that _describes_ a layout instead of building it.
- Linking to external CSS frameworks for a one-off artifact.
- Multi-file output (HTML + separate CSS + separate JS) — defeats portability.
- Static screenshots of components instead of rendering the components.
- Editors with no export — the user has nowhere to put the result.
- Default browser styling — looks unfinished, undermines the artifact's authority.

## Reference

- Page: https://thariqs.github.io/html-effectiveness/
- Local report: `html-effectiveness-report.md` (project root, when available)
- Demo index: 20 files numbered `01-…html` through `20-…html` covering exploration, code review, design, prototyping, illustrations, decks, research, reports, editors.