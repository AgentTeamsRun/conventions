# Document Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

Documents are human-facing artifacts. Use Documents for long-lived written content that a person will read, edit, or share in the Web Tiptap editor.

## When to Create a Document

Create or edit a Document when the user explicitly asks for a human-readable artifact:

- Korean triggers: 문서, 메모, 글, 초안, 원고, 노트, 요약, 회의록, 공지, 설명서
- English triggers: document, doc, memo, draft, write-up, note, summary, minutes, announcement
- Any request that asks to write or revise long-lived prose for humans

Do **not** create a Document for:

- Coding conventions, naming rules, or architecture guidance → use **Convention**
- Trackable work units or TODOs → use **Plan**
- Execution results tied to a plan → use **Completion Report**
- Failure analysis → use **Post-Mortem**
- Agent-to-agent handoffs → use **Co-Action**

> Document vs Convention: a Convention is auto-loaded as agent **rules**; a Document is human-readable **prose**, editable and shareable in the Tiptap editor.

## Creating a Document

Use the dedicated CLI to register the document on the server. The body comes from a local markdown file.

```bash
# Inspect existing project tags first (reuse-first — see Tagging below)
agentteams document tags

# Create a new document (tags you pass are SUGGESTIONS, not confirmed — see Tagging)
agentteams document create \
  --title "<document title>" \
  --file .agentteams/cli/temp/<doc-slug>.md \
  --suggested-tags "<comma,separated,tags>"

# Update an existing document
agentteams document update --id <documentId> \
  --file .agentteams/cli/temp/<doc-slug>.md

# Download (fetch) a document
agentteams document download --id <documentId>

# List documents
agentteams document list --query "<keyword>"
```

If `--title` is omitted on create, the file name is used.

## Tagging (two-tier: suggested vs confirmed)

Tags have two layers. Only **confirmed** tags (`tags`) drive the document folder tree, list filters, and search. **Suggested** tags (`suggestedTags`) are a staging area that a person reviews.

- **Agents cannot set confirmed tags directly.** Whatever you pass via `--tags` or `--suggested-tags` on create/update is stored as a **suggestion** — the server absorbs it. Only a human (web UI) or the promote action confirms a tag. This keeps the tree clean even when many documents are auto-generated.
- **Reuse is auto-confirmed; new tags wait for review.** A suggestion that exactly matches a tag already used in the project is promoted to confirmed automatically (so reused tags show up in the tree immediately). A brand-new tag stays a suggestion until the user accepts it.
- **Reuse-first — always check existing tags before suggesting.** Run `agentteams document tags` to see the project's existing tags (with usage counts) and prefer reusing them. Inventing near-duplicates (`planning` vs `plans`) fragments the tree, so match an existing tag whenever one fits.
- **Use `/` for hierarchy.** A tag like `work/planning/ai` renders as a nested folder (`work > planning > ai`) in the document tree. Keep paths shallow and consistent.
- Suggestions are capped at 20 per document; confirmed tags at 10.

## Markdown Structure Tips

Write Documents in Tiptap-compatible markdown so they remain editable in the web editor:

- Use `#` / `##` for headings.
- Use `-` for unordered lists.
- Use fenced code blocks with triple backticks.
- Use GFM table syntax for tables.
- Use a fenced `mermaid` code block for diagrams — it renders in the web viewer (see below).
- Avoid raw HTML — the Tiptap editor strips most inline HTML.

## Diagrams (Mermaid)

A fenced `mermaid` block renders in the web viewer; it stays plain text in CLI/raw, so keep the prose self-contained. Types: `flowchart`, `sequenceDiagram`, `erDiagram`, `gantt`, `stateDiagram-v2`.
