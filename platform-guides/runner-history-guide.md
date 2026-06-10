# Runner History Guide

How to write the runner history file. The runner prompt gives you the history path; this guide defines the format, length, and content rules.

## Purpose

- The history file is a **handoff document for the next session** — include only what the next session needs to know.
- It is also the **only channel for user-facing questions**. stdout is not shown to the user.

## Format Rules

- Save as a Markdown (`.md`) file at the history path given in the runner prompt.
- Overwrite the file with the latest full summary for this run.
- Do not add a top-level title (e.g., `# Runner History`).
- Use `###` (h3) headings. Include at least the two required sections below; add extra sections only when they materially improve the handoff.
- Target length: keep under 8,000 characters. Be concise, but include enough context for a reliable handoff.

## Required Sections

- `### Summary` — 3-10 bullet points focused on what the next session needs to know (final state, key decisions, remaining work). If any CLI command output includes a `webUrl` field during this run, include it as a clickable markdown link in the relevant summary bullet.
- `### Questions for User` — include only blocking or decision-required questions (up to 3). Write `None` if there are no questions.

## Do NOT Include

- Code diffs, full file contents, CLI/terminal output, step-by-step execution logs, or verification command results.

## Continuation Runs

When the runner prompt provides a previous history path:

- The previous history file has two sections: `## Requests` (all prior user requests in chronological order) and `## History (latest)` (the most recent cumulative summary, may be absent on the first follow-up).
- Read both sections to understand the conversation context, then continue without repeating completed work.
- If the previous history has a `Suggestions for User` section, consider those suggestions in the context of the user's current prompt and proceed accordingly.
- When writing the new history file, do NOT copy or append previous session content. Write a single up-to-date summary that supersedes prior history.
