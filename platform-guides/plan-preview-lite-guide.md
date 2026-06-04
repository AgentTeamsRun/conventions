# Plan Preview — Lite Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines the **lite** HTML preview used for `MINIMAL` and `STANDARD` complexity plans. It is the small-plan counterpart to the rich dashboard preview in `html-summary-guide.md`. A FULL plan uses the rich preview; a MINIMAL/STANDARD plan uses this lite one. The preview is mandatory at every tier — there is no escape hatch.

## When to Use This

- The plan's stored `complexity` is `MINIMAL` or `STANDARD`.
- The decision surface is small enough that a dashboard (metric grids, wave-flow diagrams, DoD progress bars) would be overkill and would invent structure the plan does not have.
- If the plan is `FULL`, stop and use `html-summary-guide.md` instead.

## Purpose & Safety

The lite preview follows the **same factual, HTML, and theme safety rules** as the rich preview — re-read those sections in `html-summary-guide.md`. In short:

- HTML is a human-facing **summary**, never a replacement for the canonical Markdown/Tiptap plan body.
- Do not invent facts, statuses, owners, file paths, or verification results. Omit or mark anything not derivable from the source.
- No scripts, inline handlers, trackers, forms, auth flows, or remote assets. Keep it portable inside a sandboxed iframe.
- No noisy metadata (source hashes, cache keys, plan ids, "AI-curated" footers). The host UI already supplies that context.
- Both light and dark mode must be explicitly styled; the `body` background stays `transparent`.

## Structure — Four Blocks, In Order

A lite preview is a single narrow column with exactly these blocks. Omit a block only if the source plan genuinely has nothing for it.

1. **Title** — one `<h1>` with the plan title, optionally an eyebrow line for `type · complexity`.
2. **TL;DR** — the plan's one-to-two sentence summary plus its deliverables as a short list.
3. **Changes** — what this plan will change: the files/modules or concrete steps. A short list, not the full task breakdown.
4. **Verification** — how the work is proven done: the acceptance criteria / tests / checks. A short list.

Keep the whole document scannable in a few seconds. If you find yourself adding a fifth section, a metric grid, or a flow diagram, the plan is probably FULL — reconsider the tier.

## Design Tokens — Reuse the Rich Preview's CSS

Do **not** invent a new palette. Reuse the exact `:root` design tokens, the three-layer theme resolution (light defaults → `prefers-color-scheme: dark` → host `[data-theme]` attribute), and the `body` rules defined in `html-summary-guide.md` → *Design Tokens*. The lite preview is visually consistent with the rich one; it just uses fewer components.

From the shared token set, a lite preview typically uses only:

- **Hero** — `<header><p class="eyebrow">type · COMPLEXITY</p><h1>…</h1><p class="lead">TL;DR sentence</p></header>`
- **Card / callout** — `<aside class="card">…</aside>` for the Changes and Verification blocks (`--surface` bg, 1px `--border`, radius 8).
- **File list** — `<ul class="files"><li><code>path/to/file.ts</code> <span class="tag">M</span></li>…</ul>` for the Changes block when it is file-oriented.
- **Status badge** — `<span class="badge badge-accent">…</span>` only when it carries real status meaning.

Avoid the metric grid, wave-flow, compare grid, and DoD progress bar — those are rich-preview forms for FULL plans.

## Reference Skeleton

A minimal, self-contained shape (fill the shared tokens from `html-summary-guide.md` into `:root`):

~~~html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    /* :root tokens + theme layers + body rules — copy from html-summary-guide.md */
    h1 { font-size: var(--fs-h1); line-height: var(--lh-heading); margin: 0 0 var(--s2); }
    .eyebrow { color: var(--subtle); font-size: var(--fs-caption); text-transform: uppercase; letter-spacing: .04em; margin: 0 0 var(--s1); }
    .lead { color: var(--muted); margin: 0 0 var(--s5); }
    .card { background: var(--surface); border: 1px solid var(--border); border-radius: 8px; padding: var(--s4); margin: 0 0 var(--s4); }
    .card h2 { font-size: var(--fs-h3); margin: 0 0 var(--s2); }
    .card ul { margin: 0; padding-left: var(--s4); }
    code { font-family: var(--font-mono); background: var(--code-bg); padding: 0 var(--s1); border-radius: 4px; }
  </style>
</head>
<body>
  <header>
    <p class="eyebrow">FEATURE · STANDARD</p>
    <h1>Plan title</h1>
    <p class="lead">One-to-two sentence TL;DR of what this plan achieves.</p>
  </header>

  <aside class="card">
    <h2>Changes</h2>
    <ul class="files">
      <li><code>path/to/file.ts</code> <span class="tag">M</span> — what changes</li>
    </ul>
  </aside>

  <aside class="card">
    <h2>Verification</h2>
    <ul>
      <li>Acceptance criterion / test / check that proves the work is done</li>
    </ul>
  </aside>
</body>
</html>
~~~

## Upload Workflow

Identical to the rich preview — the lite HTML is uploaded in the same `plan create` / preview-affecting `plan update` command via `--html-file` or `--html-stdin`. See `html-summary-guide.md` → *Upload Workflow* for the standalone `plan upload-html` form.

## Pre-Upload Checklist

- The plan's `complexity` is `MINIMAL` or `STANDARD` (a FULL plan must use the rich preview).
- Exactly the four blocks (Title / TL;DR / Changes / Verification), in order; no dashboard components.
- The summary matches the source plan's scope and current state; missing info is omitted or marked not specified.
- No scripts, trackers, forms, auth flows, remote dependencies, or hidden metadata.
- Light and dark modes are both explicitly styled; the `body` background is `transparent` and renders correctly under host dark themes such as `[data-theme="night"]`.
- The shared design tokens from `html-summary-guide.md` are reused — no ad-hoc palette.
- The source plan Markdown/Tiptap remains the canonical execution document.
