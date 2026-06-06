# Plan Preview — Lite Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines the **lite** HTML preview used for `MINIMAL` and `STANDARD` complexity plans. It is the small-plan counterpart to the rich dashboard preview in `plan-preview-guide.md`. A FULL plan uses the rich preview; a MINIMAL/STANDARD plan uses this lite one. The preview is mandatory at every tier — there is no escape hatch.

## When to Use This

- The plan's stored `complexity` is `MINIMAL` or `STANDARD`.
- The decision surface is small enough that a dashboard (metric grids, wave-flow diagrams, DoD progress bars) would be overkill and would invent structure the plan does not have.
- If the plan is `FULL`, stop and use `plan-preview-guide.md` instead.

## Purpose & Safety

The lite preview follows the **same factual, HTML, and theme safety rules** as the rich preview — re-read those sections in `plan-preview-guide.md`. In short:

- HTML is a human-facing **summary**, never a replacement for the canonical Markdown/Tiptap plan body.
- Do not invent facts, statuses, owners, file paths, or verification results. Omit or mark anything not derivable from the source.
- No scripts, inline handlers, trackers, forms, auth flows, or remote assets. Keep it portable inside a sandboxed iframe.
- No noisy metadata (source hashes, cache keys, plan ids, "AI-curated" footers). The host UI already supplies that context.
- Both light and dark mode must be explicitly styled; the `body` background stays `transparent`.

## Structure — Four Blocks, In Order

A lite preview is a single narrow column with exactly these blocks. Omit a block only if the source plan genuinely has nothing for it.

1. **Title** — one `<h1>` with the plan title, an eyebrow line for `type · complexity`, and an optional compact chip row (`.meta`) carrying `type` / `complexity` / `priority` / `status`. The chip row is an inline pill strip, **not** a metric grid.
2. **TL;DR** — the plan's one-to-two sentence summary plus its deliverables as a short list.
3. **Changes** — what this plan will change: the files/modules or concrete steps. A short list, not the full task breakdown.
4. **Verification** — how the work is proven done: the acceptance criteria / tests / checks. A short list.

Keep the whole document scannable in a few seconds. If you find yourself adding a fifth section, a metric grid, or a flow diagram, the plan is probably FULL — reconsider the tier.

## Design Tokens — Reuse the Rich Preview's CSS

Do **not** invent a new palette. Use the exact `:root` design tokens, the three-layer theme resolution (light defaults → `prefers-color-scheme: dark` → host `[data-theme]` attribute), and the `body` rules from `plan-preview-guide.md` → *Design Tokens*. These are inlined verbatim in the *Reference Skeleton* below, so the lite preview is theme-correct on its own and visually consistent with the rich one — it just uses fewer components. The full theme layer is mandatory: without it, a transparent `body` over a host dark theme renders default black text and is unreadable.

From the shared token set, a lite preview typically uses only:

- **Hero** — `<header class="hero"><p class="eyebrow">type · COMPLEXITY</p><h1>…</h1><p class="lead">TL;DR sentence</p>…</header>`
- **Meta chip row** — `<ul class="meta"><li class="chip">Type <b>FEATURE</b></li>…</ul>` — a compact inline pill strip for `type` / `complexity` / `priority` / `status`. Put `chip-accent` on the single chip that carries the most signal (usually priority). This is the lite counterpart to the rich preview's metric grid: same top-of-page anchoring role, far lighter. It is **not** a metric grid — no large numbers, no boxed cells.
- **Card / callout** — `<aside class="card">…</aside>` for the Changes and Verification blocks (`--surface` bg, 1px `--border`, radius 10). The card title is a small uppercase label (`.card > h2`), not a big heading — this is what gives the lite preview a dashboard feel instead of a Markdown-rendering feel.
- **File list** — `<ul class="files"><li><code>path/to/file.ts</code> <span class="tag tag-m">M</span> <span class="desc">— what changes</span></li>…</ul>` for the Changes block when it is file-oriented. Tag variants: `tag-m` (modified), `tag-a` (added), `tag-d` (deleted).
- **Checklist** — `<ul class="checks"><li>…</li></ul>` for the Verification block; each item renders with a `✓` marker. A plain `<ul>` is fine when the content is not a pass/fail checklist.

Avoid the metric grid, wave-flow, compare grid, and DoD progress bar — those are rich-preview forms for FULL plans. The hero chip row is the only structured anchor a lite preview needs.

## Reference Skeleton

A minimal, self-contained shape. The shared design tokens and the three-layer theme resolution are inlined below — identical to `plan-preview-guide.md` → *Design Tokens*; keep the values in sync. Copying this skeleton as-is already yields correct light and dark rendering:

~~~html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    /* Shared design tokens — identical to plan-preview-guide.md. Keep values in sync. */
    :root {
      color-scheme: light dark;
      --max-w: 800px;
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
    h1 { font-size: var(--fs-h1); line-height: var(--lh-heading); margin: 0 0 var(--s2); }
    .eyebrow { color: var(--subtle); font-size: var(--fs-caption); text-transform: uppercase; letter-spacing: .04em; margin: 0 0 var(--s1); }
    .lead { color: var(--muted); margin: 0 0 var(--s3); }
    /* Hero meta — a compact inline chip row, NOT a metric grid. type / complexity / priority / status only. */
    .meta { list-style: none; display: flex; flex-wrap: wrap; gap: var(--s2); margin: var(--s3) 0 var(--s5); padding: 0; }
    .chip { display: inline-flex; align-items: baseline; gap: var(--s1); font-size: var(--fs-caption); text-transform: uppercase; letter-spacing: .03em; color: var(--subtle); background: var(--surface-2); border: 1px solid var(--border); border-radius: 999px; padding: var(--s1) var(--s3); }
    .chip b { font-weight: 600; color: var(--text); }
    .chip-accent { background: var(--accent-soft); border-color: transparent; }
    .chip-accent b { color: var(--accent); }
    .card { background: var(--surface); border: 1px solid var(--border); border-radius: 10px; padding: var(--s4); margin: 0 0 var(--s3); }
    .card > h2 { font-size: var(--fs-caption); font-weight: 600; text-transform: uppercase; letter-spacing: .06em; color: var(--subtle); margin: 0 0 var(--s3); }
    .files, .checks { list-style: none; margin: 0; padding: 0; display: grid; gap: var(--s2); }
    .files li { display: flex; flex-wrap: wrap; gap: var(--s2); align-items: baseline; font-size: var(--fs-small); }
    .files .desc { color: var(--muted); }
    .tag { font-size: var(--fs-caption); font-weight: 600; border-radius: 4px; padding: 0 var(--s1); flex: none; }
    .tag-m { color: var(--accent);  background: var(--accent-soft); }
    .tag-a { color: var(--success); background: var(--success-soft); }
    .tag-d { color: var(--danger);  background: var(--danger-soft); }
    .checks li { display: flex; gap: var(--s2); }
    .checks li::before { content: "✓"; color: var(--success); font-weight: 600; flex: none; }
    code { font-family: var(--font-mono); background: var(--code-bg); padding: 0 var(--s1); border-radius: 4px; word-break: break-all; }
    @media (max-width: 640px) { body { padding: var(--s4); } }
  </style>
</head>
<body>
  <header class="hero">
    <p class="eyebrow">FEATURE · STANDARD</p>
    <h1>Plan title</h1>
    <p class="lead">One-to-two sentence TL;DR of what this plan achieves.</p>
    <ul class="meta">
      <li class="chip">Type <b>FEATURE</b></li>
      <li class="chip">Complexity <b>STANDARD</b></li>
      <li class="chip chip-accent">Priority <b>Medium</b></li>
    </ul>
  </header>

  <aside class="card">
    <h2>Changes</h2>
    <ul class="files">
      <li><code>path/to/file.ts</code> <span class="tag tag-m">M</span> <span class="desc">— what changes</span></li>
      <li><code>path/to/new.ts</code> <span class="tag tag-a">A</span> <span class="desc">— what is added</span></li>
    </ul>
  </aside>

  <aside class="card">
    <h2>Verification</h2>
    <ul class="checks">
      <li>Acceptance criterion / test / check that proves the work is done</li>
    </ul>
  </aside>
</body>
</html>
~~~

## Upload Workflow

Identical to the rich preview — the lite HTML is uploaded in the same `plan create` / preview-affecting `plan update` command via `--html-file` or `--html-stdin`. See `plan-preview-guide.md` → *Upload Workflow* for the standalone `plan upload-html` form.

## Pre-Upload Checklist

- The plan's `complexity` is `MINIMAL` or `STANDARD` (a FULL plan must use the rich preview).
- Exactly the four blocks (Title / TL;DR / Changes / Verification), in order; the only structured anchor is the hero chip row — no metric grid, flow diagram, compare grid, or progress bar.
- The summary matches the source plan's scope and current state; missing info is omitted or marked not specified.
- No scripts, trackers, forms, auth flows, remote dependencies, or hidden metadata.
- Light and dark modes are both explicitly styled; the `body` background is `transparent` and renders correctly under host dark themes such as `[data-theme="night"]`.
- The shared design tokens from `plan-preview-guide.md` are reused — no ad-hoc palette.
- The source plan Markdown/Tiptap remains the canonical execution document.
