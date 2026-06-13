# Convention Authoring Rules (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines the baseline rules for authoring and editing convention files under `.agentteams/<category>/*.md`.
For the convention body, follow the structures and elements supported by the **Web Convention Editor toolbar** so the content stays consistent across Web/CLI/AI workflows.

## 1) File structure

- A convention file consists of **Frontmatter (optional)** + **Markdown body**.
- Frontmatter uses YAML and must be placed at the very top of the file.

Example:

```markdown
---
trigger: always_on
description: '...'
---

# AGENT_RULES

- Rule 1
- Rule 2

# Title

- Item
```

## 2) Recommended frontmatter fields

- `trigger` (optional): a hint for auto-apply/reference triggers (e.g. `always_on`, `model_decision`, `-`)
- `description` (optional): a short summary of purpose/scope

Notes:

- Frontmatter may be generated/normalized by the server, and some fields may be ignored or overwritten during updates.
- `agentInstruction` (frontmatter) is **deprecated**. Use `# AGENT_RULES` in the document body instead.

## 2-1) AGENT_RULES section

Place a `# AGENT_RULES` heading at the top of the body (right after frontmatter) for rules the agent must follow. LLMs comply with `AGENT_RULES` / `NON_NEGOTIABLE_RULES` / `GUARDRAILS` headings more reliably than with rules buried in general context.

```markdown
# AGENT_RULES

- Do not use .env files for development
- Use each folder's .env file instead

# PROJECT CONTEXT

...
```

## 3) Category / path rules

- Use one of the following values for `<category>`:
  - `rules`, `skills`, `guides`, `references`
- Use the path format `.agentteams/<category>/<fileName>.md`.
- Prefer lowercase, hyphen-based file names (e.g. `api-routes.md`).
- Avoid file name collisions within the same category; depending on policy, the server/CLI may reject conflicts.

## 4) Body authoring rules (Web editor toolbar baseline)

Use standard Tiptap-toolbar markdown — headings, blockquotes (warnings/constraints), bullet/ordered/task lists, tables (options/values), code blocks (with language tags). Non-obvious rules:

- **Headings**: H1 = section, H2/H3/H4 = nesting. Do **not** skip levels (H1 → H3 is not allowed).
- **Mermaid**: a fenced ` ```mermaid ` block renders in the web viewer; it stays plain text in CLI/raw, so keep nearby prose self-contained.
- **Writing style**: prefer **imperative** phrasing ("Do X", "Do not do Y"); keep examples in fenced, executable code blocks; prefer multiple small focused conventions over one huge document.

### Markers (recommended prefixes)

Use consistently (trailing space after the emoji): `✅ ` correct · `❌ ` incorrect · `🚫 ` strictly forbidden · `⚠️ ` warning · `📌 ` key summary · `💡 ` tip · `🎯 ` goal/intent.

## 5) Single source of truth (no duplication)

A convention must not duplicate facts whose authoritative source lives elsewhere. Duplicated values and policy inevitably drift out of sync with their source and erode trust in the convention.

- Define only what the convention **owns**: implementation patterns, naming, and process — rules that have no other authoritative source.
- For values or policy whose source of truth is **code or a Document, link to it instead of restating it** (e.g. reference the constant in source code; reference the Document for policy and rationale).
- Heuristic: _"If this changes, must I edit it in two places?"_ If yes, it is duplication — remove it from the convention and replace it with a pointer.
- Ownership split: **values/logic → code**, **human-facing policy/rationale → Document**, **patterns/naming/process → convention**.

## 6) Useful Commands

```bash
# Create a new convention
agentteams convention create --file .agentteams/{category}/{convention-name}.md

# Download all conventions
agentteams convention download
```

> Place the file under `.agentteams/<category>/` before running `convention create`. The `<category>` must be one of: `rules`, `skills`, `guides`, `references`.
>
> Updating or deleting conventions: see `convention-ud-guide.md`.
