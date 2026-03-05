# Co-Action Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

A co-action is a **handoff document** — it captures knowledge that cannot be inferred from code alone. Use it to transfer implicit knowledge between agents or between sessions.

## What a Co-Action Is

- A co-action records **discoveries, decisions, and context** accumulated during a work session.
- It is NOT a session context dump. Only include information that has lasting value for the next reader.
- Co-actions link to related plans, completion reports, and post-mortems for full traceability.
- Takeaways track the key insights and lessons learned from the work.

## When to Create a Co-Action

- After completing a multi-session or multi-agent project
- When handing off work to another agent or team member
- When a body of implicit knowledge would be lost without documentation
- Before a long pause in active development on a feature

## Content Structure

Write the co-action content with these sections. Include only sections that have meaningful content — skip empty ones.

~~~markdown
## Key Discoveries
- <findings that are not obvious from the code>
- <workarounds, undocumented behaviors, environment quirks>

## Design Decisions
- <decision>: <rationale — why this approach, what alternatives were rejected>

## Usage Scenarios
- <how the feature/system is intended to be used>
- <edge cases or non-obvious interaction patterns>

## Non-Goals
- <what was intentionally excluded and why>

## Risks / Trade-offs
- <known compromises and their potential impact>
- <technical debt introduced and conditions for revisiting>

## Follow-up / Known Constraints
- <what the next agent should continue or address>
- <temporary workarounds that need permanent solutions>
- <incomplete items with context on where they left off>
~~~

## Section Writing Tips

### Key Discoveries
Focus on things the next reader cannot find by reading code. Examples: "Prisma 7 delegate requires wrapper pattern due to type inference bug", "Docker build caches COPY layer even when file content changes if mtime is unchanged".

### Design Decisions
State the decision and the reason. Bad: "Used Redis". Good: "Chose Redis over in-memory cache because the service runs multi-instance and cache must be shared; accepted the ops overhead."

### Non-Goals
Prevent scope creep by the next agent. Be explicit about what was considered and deliberately excluded.

### Follow-up / Known Constraints
This is the most critical section for handoff. Be specific enough that the next agent can resume without re-investigating.

## Visibility

- `PRIVATE` (default): only the creator can view/access
- `PROJECT`: all project members can view; only the creator can change status, visibility, or delete

## Takeaways

Takeaways capture **key insights** that emerged during the work — things the next agent should know but might not discover on their own.

Use takeaways when:
- You discovered a non-obvious constraint or behavior (e.g., "Fastify response schema silently strips undefined fields")
- A design decision was made that isn't documented elsewhere
- You found a workaround that future agents should reuse (or avoid)
- The task revealed risks or follow-up items that don't belong in the main content

Takeaways are separate from content. Content is the structured handoff document; takeaways are concise, standalone insights attached to it.

~~~bash
agentteams coaction takeaway-create --id {coActionId} --content "<insight>"
~~~

## Useful Commands

~~~bash
# Create
agentteams coaction create \
  --title "<concise handoff title>" \
  --file .agentteams/cli/temp/{descriptive-name}-coaction.md \
  --visibility PRIVATE

# Download for reference
agentteams coaction download --id {coActionId}

# Link related entities
agentteams coaction link-plan --id {coActionId} --plan-id {planId}
agentteams coaction link-completion-report --id {coActionId} --completion-report-id {crId}
agentteams coaction link-post-mortem --id {coActionId} --post-mortem-id {pmId}

# Takeaways
agentteams coaction takeaway-create --id {coActionId} --content "<insight>"
agentteams coaction takeaway-list --id {coActionId}
agentteams coaction takeaway-update --id {coActionId} --takeaway-id {takeawayId} --content "<updated insight>"
agentteams coaction takeaway-delete --id {coActionId} --takeaway-id {takeawayId}
# Update content
agentteams coaction update --id {coActionId} --file .agentteams/cli/temp/{name}-coaction.md

# Change status
agentteams coaction update --id {coActionId} --status CLOSED
~~~

## Output Behavior

- `coaction create` and `coaction update` print post-action tips to **stderr**.
- Structured command results (for `--format json`) remain on **stdout**.
- This separation prevents JSON pipeline parsing failures when tips are shown.

~~~bash
# Keep stdout clean for JSON consumers
agentteams coaction create \
  --title "<title>" \
  --file .agentteams/cli/temp/{name}-coaction.md \
  --format json \
  2>/tmp/coaction-tips.log
~~~

## Common Pitfalls

- Dumping raw session logs instead of curated knowledge
- Writing implementation details that belong in a completion report
- Skipping the "Follow-up / Known Constraints" section — this is the most valuable part for handoff
- Forgetting to link related plans and reports
- Using PRIVATE visibility when the knowledge should be shared with the team
- Assuming info tips are printed to stdout; they are intentionally sent to stderr

## References

- Related guides: `plan-guide.md`, `completion-report-guide.md`, `post-mortem-guide.md`
