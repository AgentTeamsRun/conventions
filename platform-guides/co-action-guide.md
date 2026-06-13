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

- `coaction create` / `update` print tips to **stderr**; structured result (`--format json`) stays on **stdout**. Redirect `2>` when piping JSON.
- Do not dump raw session logs — curate.
- Do not put implementation detail here — that belongs in the completion report.
- Never skip `## Follow-up / Known Constraints` — it is the highest-value section for handoff.
- Link related plans/reports; do not leave a co-action orphaned.
- Use `PRIVATE` only when truly private; use `PROJECT` for shared knowledge.

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

## Commands

```bash
# Create
agentteams coaction create \
  --title "<handoff title>" \
  --file .agentteams/cli/temp/{name}-coaction.md \
  --visibility PRIVATE

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
agentteams coaction create --title "<t>" --file <f> --format json 2>/tmp/tips.log
```

> Full flags for any subcommand: `agentteams coaction <subcommand> --help`

## References

`plan-guide.md` · `completion-report-guide.md` · `post-mortem-guide.md`
