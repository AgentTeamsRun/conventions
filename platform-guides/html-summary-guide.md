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
