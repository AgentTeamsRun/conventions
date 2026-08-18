# Co-Action Guide (AgentTeams)

> ⚠️ Auto-deployed from the server. Do not edit directly.

A co-action is a **handoff document**: knowledge that cannot be inferred from code.
It is NOT a session log dump — include only what has lasting value for the next reader.
Co-actions link to plans / completion reports / post-mortems for traceability.

## When to Create

- Handing off to another agent or session
- After a multi-session / multi-agent project
- Before a long pause, when implicit knowledge would otherwise be lost

If the work just needs "what I did", use a completion report instead — not a co-action.

## Guardrails (read first)

- `coaction create` / `update` print tips to **stderr**; the structured result stays on **stdout**. Redirect `2>` when piping JSON.
- Do not dump raw session logs — curate.
- Do not put implementation detail here — that belongs in the completion report.
- Never skip `## Follow-up / Known Constraints` — it is the highest-value section for handoff.
- Link related plans/reports; do not leave a co-action orphaned.

## Content Structure

Fill these sections; omit any that have no real content.

```markdown
## Key Discoveries

- <non-obvious findings, workarounds, undocumented behavior, env quirks>

## Design Decisions

- <decision>: <why this, what was rejected>

## Usage Scenarios

- <intended use, non-obvious interaction patterns>

## Non-Goals

- <deliberately excluded, and why — prevents next-agent scope creep>

## Risks / Trade-offs

- <known compromises, tech debt + when to revisit>

## Follow-up / Known Constraints

- <what the next agent must continue; be specific enough to resume without re-investigating>
```

Tips: state decisions _with_ their reason ("Redis over in-memory because multi-instance, accepted ops overhead"). Key Discoveries = things unreadable from code. Non-Goals = explicit exclusions.

A fenced `mermaid flowchart` renders in the web viewer; stays plain text in CLI/raw — keep prose self-contained.

## Visibility

- `PRIVATE` (default): creator only
- `PROJECT`: all project members view; only creator changes status/visibility/deletes

## Takeaways

Standalone insights attached to (but separate from) the content. Create one when you found a non-obvious constraint, an undocumented decision, a reusable workaround, or a risk that doesn't fit the main sections.

## Writing via MCP

When the AgentTeams MCP server is connected, prefer the MCP write tools over shelling out to the CLI.

| Tool                                | Purpose                                                                                           |
| ----------------------------------- | ------------------------------------------------------------------------------------------------- |
| `agentteams_guide_get("co-action")` | Fetch this guide's current text plus its `guideHash`. **Call this before any co-action write.**   |
| `agentteams_coaction_create`        | Create a co-action. Requires at least one of `planId`, `completionReportId`, `postMortemId`.      |
| `agentteams_coaction_update`        | Update a co-action, including the `OPEN` to `CLOSED` transition. Only the fields you pass change. |
| `agentteams_coaction_delete`        | Delete a co-action (destructive).                                                                 |

The tools operate on the single project the MCP server is bound to. There is no `projectId` argument — a different project cannot be reached from an MCP session.

Takeaways have no MCP tool of their own: create and list them through the CLI (see Commands below).

### The three optional contract fields

All three are optional; omitting them all is valid and behaves exactly like a plain write.

- **`guideHash`** — the hash returned by `agentteams_guide_get("co-action")`. Pass it so the server can confirm you followed the current rules. Every write tool accepts it, delete included. If your local copy is stale, the write is rejected with `GUIDE_OUTDATED` and the response names the hash the server expects. Recover with `agentteams convention download`, re-read this guide, and retry.
- **`idempotencyKey`** — a key of your choosing that makes a retry safe. Repeating a call with the same key and the same request replays the first result instead of creating a second co-action. Reusing a key with a _different_ request is rejected as a conflict — pick a new key for a new write. A retry that arrives while the first call is **still running** is rejected with a conflict that says so (wait a moment and repeat it to get the replay), and a key is remembered for **24 hours**.
- **`expectedUpdatedAt`** — the `updatedAt` you last read (from `agentteams_coaction_get`). Pass it on update and delete so a concurrent edit is rejected instead of silently overwritten. **Without it, an update or delete is unconditional**: it overwrites or removes the co-action even if someone edited it after you read it.

### Fallback to the CLI

When MCP is unavailable or a tool is missing, use `agentteams coaction create/update/delete` — the CLI reaches the same endpoints with the same server-side validation and the same error codes, and accepts the same three fields as `--guide-hash`, `--idempotency-key`, and `--expected-updated-at`.

## Commands

```bash
# Create. Omit `--visibility` for `PRIVATE`; pass `--visibility PROJECT` explicitly to share with project members.
agentteams coaction create \
  --title "<handoff title>" \
  --file .agentteams/cli/temp/{name}-coaction.md

# Update content / status
agentteams coaction update --id {id} --file .agentteams/cli/temp/{name}-coaction.md
agentteams coaction update --id {id} --status CLOSED

# Download / link
agentteams coaction download --id {id}
agentteams coaction link-plan              --id {id} --plan-id {planId}
agentteams coaction link-completion-report --id {id} --completion-report-id {crId}
agentteams coaction link-post-mortem       --id {id} --post-mortem-id {pmId}

# Takeaways
agentteams coaction takeaway-create --id {id} --content "<insight>"
agentteams coaction takeaway-list   --id {id}

# Keep stdout clean for JSON consumers
agentteams coaction create --title "<t>" --file <f> 2>/tmp/tips.log
```

> Full flags for any subcommand: `agentteams coaction <subcommand> --help`

## References

`plan-execution-guide.md` · `completion-report-guide.md` · `post-mortem-guide.md`
