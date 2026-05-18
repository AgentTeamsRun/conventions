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
- The source plan Markdown/Tiptap remains the canonical execution document.
- Light and dark modes are both explicitly styled; the document does not depend on the host app's theme.
