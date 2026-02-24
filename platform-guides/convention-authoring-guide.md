# Convention Authoring Rules (AgentTeams)

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
agentInstruction: |
  ...
---

# Title

- Item
~~~

## 2) Recommended frontmatter fields

- `trigger` (optional): a hint for auto-apply/reference triggers (e.g. `always_on`, `model_decision`, `-`)
- `description` (optional): a short summary of purpose/scope
- `agentInstruction` (optional): explicit instructions for agents (multi-line supported)

Notes:

- Frontmatter may be generated/normalized by the server, and some fields may be ignored or overwritten during updates.

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
