# Plan HTML Summary Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines guardrails for AI-authored HTML summaries of plans. Use it when you generate a human-facing HTML preview and upload it with the AgentTeams CLI.

## Purpose

- Treat HTML as a **human-facing summary/preview only**.
- Keep the original plan Markdown/Tiptap content as the canonical source for agents, automation, status, and execution.
- Make the summary easier to scan without changing the plan's meaning, scope, blockers, owners, or verification requirements.

## Factual Safety

- Do not invent facts, statuses, blockers, owners, due dates, file paths, dependencies, acceptance criteria, or verification results.
- Do not hide uncertainty. If a section cannot be derived from the source plan, omit it or mark it as not specified.
- Do not rewrite plan commitments into stronger claims than the source supports.
- Keep unresolved risks and explicit guardrails visible.

## HTML Safety

- Do not include scripts, inline event handlers, remote analytics, tracking pixels, form submissions, authentication flows, or app-DOM assumptions.
- Avoid external assets and network dependencies unless the source plan explicitly requires them.
- Keep the document portable and readable inside a sandboxed iframe.
- Make the layout responsive for narrow and wide containers.
- Use semantic HTML where possible. Keep CSS local to the document and avoid relying on host app styles.
- Do not include noisy implementation metadata such as source hashes, cache keys, hidden prompts, model scratchpads, or internal runner logs in the visible document.

## Theme Safety (Light and Dark Mode)

- Do not rely on the host app's `body` background, theme class, or CSS custom properties for colors. The document must be self-contained.
- Declare `color-scheme: light dark` on `:root` or `html` so the browser picks the appropriate system scrollbar and form control colors.
- Define explicit background, text, border, and link colors for both light and dark modes using `prefers-color-scheme` media queries. Do not leave either mode at browser-default.
- Avoid pure `#000000` on `#ffffff` or the reverse in dark mode — use near-black and near-white (e.g., `#1a1a1a` / `#e8e8e8`) to reduce eye strain.
- Ensure information-bearing elements — links, code blocks, badges, table rows, callout boxes — remain visually distinguishable in both modes. Do not rely solely on color to convey meaning.
- Do not hardcode only a light-mode palette. Any CSS variable, class, or inline style that sets color must account for dark mode as well.
- When the host app injects a theme signal via `<html data-theme="…">`, treat attribute selectors like `[data-theme="night"]`, `[data-theme="dark"]` as the **primary** theme signal and keep `prefers-color-scheme` as a fallback. Some host themes declare `color-scheme: light` even when the OS is dark, which prevents `prefers-color-scheme: dark` from matching inside the iframe. Define color *values* only in your own CSS variables — never pull them from host CSS variables (e.g. `--p`, `--b1`); only consume the `data-theme` *signal*.
- Keep the `body` background `transparent`; let only cards, panels, and callouts paint their own `--surface`. This way the preview merges into the host modal/page background instead of looking like a boxed-in island.

## Visual Design Language

The HTML preview is a **dashboard-shaped summary** of a plan, not a styled rendering of its Markdown. The tokens, components, and layout rules below give every preview a consistent skeleton so different agents can fill it in without each reinventing visual decisions.

### Design Tokens

Define all design values as CSS custom properties on `:root`. Resolve theme in three layers: light defaults → `prefers-color-scheme: dark` fallback → host `[data-theme]` attribute selector (primary signal). The light values are the defaults; dark values are duplicated across the fallback and attribute-selector blocks because CSS cannot share value lists between selectors. Do not import color *values* from host CSS variables.

~~~css
:root {
  color-scheme: light dark;
  --max-w: 800px; /* preferred range: 720–960 */
  --font-sans: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  --font-mono: ui-monospace, SFMono-Regular, Menlo, monospace;
  --fs-caption: 11px; --fs-small: 13px; --fs-body: 15px;
  --fs-h3: 18px;
  --fs-h2: clamp(20px, 3.5vw, 22px);
  --fs-h1: clamp(24px, 5vw, 28px);
  --lh-body: 1.55; --lh-heading: 1.2;
  --s1: 4px; --s2: 8px; --s3: 12px; --s4: 16px; --s5: 24px; --s6: 32px; --s7: 48px;
  /* Light palette */
  --bg: #fbfbfa; --surface: #ffffff; --surface-2: #f4f4f2;
  --border: #e6e6e2; --border-strong: #d4d4ce;
  --text: #1a1a1a; --muted: #555; --subtle: #888;
  --accent: #2b6cb0;  --accent-soft: #e3eef9;
  --danger: #b42318;  --danger-soft: #fdeceb;
  --warn:   #b75e09;  --warn-soft:   #fdf2e3;
  --success:#1f7a3a;  --success-soft:#e6f4ea;
  --code-bg: #f4f4f2;
}

/* Fallback when the host does not signal a theme. */
@media (prefers-color-scheme: dark) {
  :root {
    --bg:#141414; --surface:#1c1c1c; --surface-2:#232323;
    --border:#2e2e2e; --border-strong:#3d3d3d;
    --text:#e8e8e8; --muted:#a8a8a8; --subtle:#808080;
    --accent:#82b8e6;  --accent-soft:#1d2c3a;
    --danger:#f0857a;  --danger-soft:#3a1b18;
    --warn:  #e0a464;  --warn-soft:  #3a2a14;
    --success:#7dc795; --success-soft:#16301d;
    --code-bg:#1c1c1c;
  }
}

/* Primary signal: host theme attribute (DaisyUI dark theme names). */
:root[data-theme="night"],    :root[data-theme="dark"],
:root[data-theme="dim"],      :root[data-theme="sunset"],
:root[data-theme="black"],    :root[data-theme="luxury"],
:root[data-theme="business"], :root[data-theme="coffee"],
:root[data-theme="forest"],   :root[data-theme="halloween"],
:root[data-theme="dracula"],  :root[data-theme="abyss"] {
  color-scheme: dark;
  --bg:#141414; --surface:#1c1c1c; --surface-2:#232323;
  --border:#2e2e2e; --border-strong:#3d3d3d;
  --text:#e8e8e8; --muted:#a8a8a8; --subtle:#808080;
  --accent:#82b8e6;  --accent-soft:#1d2c3a;
  --danger:#f0857a;  --danger-soft:#3a1b18;
  --warn:  #e0a464;  --warn-soft:  #3a2a14;
  --success:#7dc795; --success-soft:#16301d;
  --code-bg:#1c1c1c;
}

body {
  background: transparent;
  color: var(--text);
  font: var(--fs-body)/var(--lh-body) var(--font-sans);
  max-width: var(--max-w);
  margin: 0 auto;
  padding: var(--s5);
}
~~~

### Document Shell

The tokens above are CSS only — they must live inside a **complete, standalone HTML document**. Wrap every preview in this shell; it is the one part that is identical across all previews. Fill `<style>` with the Design Tokens block above plus only the component CSS you actually use, and fill `<body>` with the Hero plus the components you pick from *Component Patterns*.

~~~html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    /* 1. Design Tokens — paste the :root + prefers-color-scheme + [data-theme] + body block verbatim. */
    /* 2. Component CSS — only the patterns this preview actually uses. */
  </style>
</head>
<body>
  <header><p class="eyebrow">…</p><h1>…</h1><p class="lead">…</p></header>
  <!-- metric grid / flow / cards / compare / DoD — pick only the components the plan needs -->
</body>
</html>
~~~

- `<meta charset="utf-8">` is **mandatory** — previews routinely contain non-ASCII content, and a missing charset corrupts it.
- `<meta name="viewport">` is required for the responsive rules in *Layout Rules* (`clamp()` headings, ≤640px collapse) to take effect.
- Set `lang` to the plan's language.

> **Token sync:** the Design Tokens block above is duplicated verbatim in `plan-preview-lite-guide.md` → *Reference Skeleton* so each guide is self-contained when loaded on its own. The two copies must stay **byte-identical** — when you change a token value or theme rule here, make the identical edit there.

### Component Patterns

One canonical markup per pattern — combine them; do not invent variants.

- **Hero** — `<header><p class="eyebrow">…</p><h1>…</h1><p class="lead">…</p></header>`
- **Metric grid** — `<section class="metrics"><div class="metric"><span class="label">Tasks</span><span class="value">12</span></div>…</section>` (4 or 6 cells, top of page)
- **Task / Wave flow** — `<ol class="flow"><li class="node">…</li>…</ol>` with `→` between nodes; collapses to a vertical stack at ≤640px
- **Compare grid** — `<section class="compare"><div class="pro">✓ …</div><div class="con">✕ …</div></section>` using `--success-soft` / `--danger-soft`
- **Card / callout** — `<aside class="card">…</aside>` (`--surface` bg, 1px `--border`, radius 8)
- **Status badge** — `<span class="badge badge-accent">…</span>`; four variants only: `accent` / `danger` / `warn` / `success`
- **Code / command block** — `<pre><code>$ agentteams …</code></pre>` (`--code-bg`, `var(--font-mono)`, `overflow-x: auto`; optional `$ ` prompt prefix)
- **Checklist with progress** — `<section class="dod"><header>DoD <span>0/5 · 0%</span><div class="bar"><i style="width:0%"></i></div></header><ul>…</ul></section>`
- **File list** — `<ul class="files"><li><code>path/to/file.ts</code> <span class="tag">M</span></li>…</ul>` (word-break on narrow widths)
- **Risk / guardrail card** — `<aside class="risk"><strong>label</strong> <span>one-line body</span></aside>` (`--warn-soft` bg)

### Layout Rules

- Card / callout *types* per page: 3 or fewer.
- Color emphasis only when it carries status meaning. No decorative gradients, drop shadows, or emojis (status glyphs `✓ ✕ → ⚠` are fine when they convey meaning).
- Content max width follows `--max-w`; container is centered.
- Use only two font weights: 400 and 600.
- **Body background stays `transparent`** — only cards, panels, and callouts paint their own `--surface`. This lets the preview blend into the host modal / page background instead of looking like a boxed-in island.
- Mobile: the main grids (metrics, flow, compare, file list, risk grid) collapse to a single column at ≤640px. `h1` / `h2` use `clamp()` to scale down. Page padding shrinks (e.g. `--s4`).
- Apply `min-width: 0` to grid / flex children so they cannot exceed the parent's width.
- One `<h1>` per document. Section headings are `<h2>`.

### Differentiation from Markdown

The HTML preview must **not** be a one-to-one rendering of the source Markdown. Markdown remains the canonical plan body; HTML is a dashboard-shaped summary of the *decision surface*.

*Anti-patterns (do not do)*:

- Mapping every Markdown section to `<h2>` + card.
- Copying every bullet from the source body into the preview.
- Information form limited to text + lists.
- A "whole-body HTML page" that mirrors the Markdown end to end.
- **Meta label boxes / footers** — `plan id`, "AI-curated · summary/preview only", "source: …" pointers. The host UI already supplies this context; rendering it inside the preview is noise.

*Required forms (use at least one)*:

- **Top metric grid** — Tasks / Files / Commits / Waves / Effort / Tier shown as label + large number.
- **Task / Wave flow diagram** — nodes + arrows visualizing dependencies (impossible to express in Markdown).
- **Compare grid** — Have ↔ Not, Before ↔ After (impossible to express in Markdown).
- **Progress visualization** — DoD checklists rendered with a `0/N · 0%` counter and a progress bar.
- **Information compression** — keep only the decision surface (critical path, key guardrails, 3–5 DoD items). Do not move every bullet from the source body.
- **Delegate the body to Markdown** — references, step-by-step detail, and full prose belong in the Markdown plan, not the HTML preview. Do not draw a "see Markdown" pointer box; a short, dense page is self-explanatory.

## Upload Workflow

1. Read the source plan and any referenced runbook context.
2. Generate a local HTML file as a summary of the plan, not a replacement for it.
3. Inspect the HTML for factual drift, unsafe content, and accidental hidden metadata.
4. Upload only after review:

~~~bash
agentteams plan upload-html --id {planId} --file .agentteams/cli/temp/plan-summary.html
~~~

For pipeline-style automation, stdin upload is also supported:

~~~bash
cat .agentteams/cli/temp/plan-summary.html | agentteams plan upload-html --id {planId} --stdin
~~~

## Pre-Upload Checklist

- The HTML summary matches the source plan's scope and current state.
- Missing information is omitted or marked as not specified.
- No scripts, trackers, forms, auth flows, or remote dependencies were added.
- The document remains useful if rendered in a sandboxed iframe.
- The output is a complete standalone document built on the *Document Shell* (doctype + `<meta charset="utf-8">` + viewport + `<style>`), not a bare fragment; the Design Tokens match `plan-preview-lite-guide.md` byte-for-byte.
- The source plan Markdown/Tiptap remains the canonical execution document.
- Light and dark modes are both explicitly styled; the document does not depend on the host app's theme.
- The Visual Design Language tokens, component patterns, and layout rules are honored.
- The preview is a dashboard-shaped summary, not a Markdown one-to-one rendering — at least one of {metric grid, flow diagram, compare grid, progress visualization} is present.
- The `body` background is `transparent` and the preview renders in dark mode under host dark themes such as `[data-theme="night"]`.
