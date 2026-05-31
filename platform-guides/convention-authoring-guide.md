# Convention Authoring Rules (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines the baseline rules for authoring and editing convention files under `.agentteams/<category>/*.md`.
For the convention body, follow the structures and elements supported by the **Web Convention Editor toolbar** so the content stays consistent across Web/CLI/AI workflows.

## 1) File structure

- A convention file consists of **Frontmatter (optional)** + **Markdown body**.
- Frontmatter uses YAML and must be placed at the very top of the file.

Example:

~~~markdown
---
trigger: always_on
description: "..."
---

# AGENT_RULES

- Rule 1
- Rule 2

# Title

- Item
~~~

## 2) Recommended frontmatter fields

- `trigger` (optional): a hint for auto-apply/reference triggers (e.g. `always_on`, `model_decision`, `-`)
- `description` (optional): a short summary of purpose/scope

Notes:

- Frontmatter may be generated/normalized by the server, and some fields may be ignored or overwritten during updates.
- `agentInstruction` (frontmatter) is **deprecated**. Use `# AGENT_RULES` in the document body instead.

## 2-1) AGENT_RULES section

Place a `# AGENT_RULES` heading at the top of the document body (right after frontmatter) to define rules the agent must follow.

- LLMs recognize patterns like `AGENT_RULES`, `NON_NEGOTIABLE_RULES`, `GUARDRAILS` strongly because they appear frequently in training data.
- Rules in this section have higher compliance rates than rules embedded in general context.

Example:

~~~markdown
# AGENT_RULES

- Do not use .env files for development
- Use each folder's .env file instead
- Design dev tool features from the user's perspective

# PROJECT CONTEXT
...
~~~

## 3) Category / path rules

- Use one of the following values for `<category>`:
  - `rules`, `skills`, `guides`, `references`
- Use the path format `.agentteams/<category>/<fileName>.md`.
- Prefer lowercase, hyphen-based file names (e.g. `api-routes.md`).
- Avoid file name collisions within the same category; depending on policy, the server/CLI may reject conflicts.

## 4) Body authoring rules (Web editor toolbar baseline)

### Structure (Headings)

- **H1**: top-level rule section title
- **H2**: major subsections under H1
- **H3**: detailed rules under H2
- **H4**: only when further subdividing H3
- **Horizontal rule (HR)**: separate unrelated sections/topics

> Do **not** skip heading levels (e.g. H1 → H3 is not allowed).

### Blocks

- **Blockquote**: warnings/constraints that must stand out
- **Bullet list**: list rules by items
- **Ordered list**: step-by-step procedures
- **Task list**: execution/verification checklist
- **Table**: summarize options/values/exceptions
- **Code block**: runnable examples or command snippets (use language tags when possible)
- **Mermaid block**: a fenced ` ```mermaid ` code block renders as a diagram in the web viewer (e.g. `flowchart`, `sequenceDiagram`); in raw markdown and CLI output it stays as plain text, so keep nearby prose self-contained.

### Inline

- **Bold**: emphasize key conclusions/keywords
- **Italic**: optional supplemental notes
- **Inline code**: literal strings such as identifiers, file paths, commands
- **Links**: connect to related references (use descriptive link text)

### Writing style

- Prefer **imperative** phrasing for rules (e.g. "Do X", "Do not do Y").
- Wrap examples in fenced code blocks and keep them executable when possible.
- Prefer multiple smaller, focused conventions over one huge document.

### Markers (recommended prefixes)

Use these prefixes consistently across documents (include a trailing space after the emoji):

- `✅ `: correct example
- `❌ `: incorrect example
- `🚫 `: strictly forbidden (including policy/security violations)
- `⚠️ `: warning/caution
- `📌 `: key summary
- `💡 `: tip/reference
- `🎯 `: goal/intent

## 5) Useful Commands

~~~bash
# Create a new convention
agentteams convention create --file .agentteams/{category}/{convention-name}.md

# Download all conventions (required before update/delete)
agentteams convention download

# Preview update (dry-run)
agentteams convention update --file .agentteams/{category}/{convention-name}.md

# Apply update to server
agentteams convention update --file .agentteams/{category}/{convention-name}.md --apply
~~~

> Place the file under `.agentteams/<category>/` before running `convention create`. The `<category>` must be one of: `rules`, `skills`, `guides`, `references`.
