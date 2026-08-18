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

## Writing via MCP

When the AgentTeams MCP server is connected, prefer the MCP write tools over shelling out to the CLI. The tools below are the document ones; every other record kind is covered by its own guide. A record kind whose guide has no `Writing via MCP` section has no MCP write tool yet — write it through the CLI.

| Tool                               | Purpose                                                                                                                                                                      |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agentteams_guide_get("document")` | Fetch this guide's current text plus its `guideHash` (from the local sync, or from the server when this session has no local copy). **Call this before any document write.** |
| `agentteams_document_create`       | Create a document.                                                                                                                                                           |
| `agentteams_document_update`       | Update a document. Only the fields you pass change.                                                                                                                          |
| `agentteams_document_delete`       | Delete a document (destructive).                                                                                                                                             |

The tools operate on the single project the MCP server is bound to. There is no `projectId` argument — a different project cannot be reached from an MCP session.

### The three optional contract fields

All three are optional; omitting them all is valid and behaves exactly like a plain write.

- **`guideHash`** — the hash returned by `agentteams_guide_get`. Pass it so the server can confirm you followed the current rules. All three write tools accept it, delete included. If your local copy is stale, the write is rejected with `GUIDE_OUTDATED` and the response names the hash the server expects. Recover with `agentteams convention download`, re-read the guide, and retry.
- **`idempotencyKey`** — a key of your choosing that makes a retry safe. Repeating a call with the same key and the same request replays the first result instead of creating a second document; a repeated delete stays successful instead of turning into "not found". Reusing a key with a _different_ request is rejected as a conflict — pick a new key for a new write. Two details worth knowing: a retry that arrives while the first call is **still running** is rejected with a conflict that says so (wait a moment and repeat it to get the replay), and a key is remembered for **24 hours**, after which the same key may be used again for a different write.
- **`expectedUpdatedAt`** — the `updatedAt` you last read (from `agentteams_document_get`). Pass it on update and delete so a concurrent edit is rejected instead of silently overwritten. **Without it, a delete is unconditional**: it removes the document even if someone edited it after you read it.

### Fallback to the CLI

When MCP is unavailable or the tool is missing, use `agentteams document create/update/delete` — the CLI reaches the same endpoints with the same server-side validation and the same error codes, and accepts the same three fields as `--guide-hash`, `--idempotency-key`, and `--expected-updated-at`.

If `agentteams_guide_get` reports that its hash is unknown, run `agentteams convention download` in the project. A missing hash is not fatal — writes still work, they just skip the freshness check. When there is no local copy at all (an MCP session started outside the project), the tool reads the guide from the server instead and reports `source: "server"`.

## Visibility (Private vs Project)

Documents are `PRIVATE` by default. When you omit `--visibility`, the document is saved as private and is visible only to the creator.

Use `--visibility PROJECT` only when the user explicitly asks to share the document with the project or team. Project-visible documents are visible to project members, and creating one sends a notification to project members, so avoid using project visibility for routine agent-generated drafts, notes, and handoffs unless sharing was requested.

## Tagging (two-tier: suggested vs confirmed)

Tags have two layers. Only **confirmed** tags (`tags`) drive the document folder tree, list filters, and search. **Suggested** tags (`suggestedTags`) are a staging area that a person reviews.

- **Agents cannot set confirmed tags directly.** Whatever you pass via `--tags` or `--suggested-tags` on create/update — or via `suggestedTags` on the MCP write tools — is stored as a **suggestion**; the server absorbs it. This holds for every execution credential (agent API key, CI service token, and CLI/Desktop personal login), so an MCP session cannot bypass it. Only a human confirms a tag: promoting a suggestion and applying a tag relabel are both rejected for execution credentials, so those routes cannot be used as a side door either. This keeps the tree clean even when many documents are auto-generated.
- **Reuse is auto-confirmed; new tags wait for review.** A suggestion that exactly matches a tag already used in the project is promoted to confirmed automatically (so reused tags show up in the tree immediately). A brand-new tag stays a suggestion until the user accepts it.
- **Reuse-first — always check existing tags before suggesting.** Run `agentteams document tags` to see the project's existing tags (with usage counts) and prefer reusing them. Inventing near-duplicates (`planning` vs `plans`) fragments the tree, so match an existing tag whenever one fits.
- **Your tags are added, never subtracted.** On update, whatever you send is appended to the document's existing suggestions instead of replacing them — suggesting one tag cannot wipe out other suggestions a person has not reviewed yet. Removing a suggestion is a human action (promote or dismiss), so do not expect an update to clear one.
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
