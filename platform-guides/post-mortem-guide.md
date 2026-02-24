# Post Mortem Guide (AgentTeams)

A post mortem is a written analysis of an incident, focusing on root cause and prevention.

## Goals

- Capture what happened and when
- Identify the root cause (not symptoms)
- Document the impact (users, revenue, SLA, operations)
- Define concrete action items to prevent recurrence

## Before You Start

- Collect logs, alerts, and key timestamps.
- Identify the affected systems and user segments.
- Confirm whether this was a one-off or part of a recurring pattern.

## Structure

- Title: short incident identifier
- Content: narrative of what happened and what was learned
- Action items: specific tasks with owners and deadlines
- Status: OPEN, IN_PROGRESS, or RESOLVED

## Root Cause Notes

- Prefer a clear causal chain over vague labels.
- Distinguish: trigger vs contributing factors vs detection failure.
- Include "why it was not caught earlier".

## Tips

- Use a clear timeline (timestamps if available)
- Avoid blame; focus on systems and process
- Include what detection/monitoring failed and how to improve it

## Action Items

- Make action items concrete and testable.
- Prefer small items with clear ownership.
- Track them like normal plans if they require engineering work.

## Post Mortem Content Template

Use this structure for the `--content` field:

~~~markdown
## Summary
[What happened - 1-2 sentences]

## Timeline
- HH:MM - ...
- HH:MM - ...

## Root Cause
[Specific cause, not symptoms]

## Impact
[Who/what was affected and for how long]

## What Went Wrong
- ...

## What Went Well
- ...
~~~

## Useful Commands

~~~bash
agentteams postmortem create \
  --title "Incident title" \
  --content "## Summary\n- ...\n\n## Timeline\n- ...\n\n## Root Cause\n- ...\n\n## Impact\n- ..." \
  --action-items "item1,item2" \
  --status RESOLVED
~~~

Repository linkage note:

- If .agentteams/config.json contains `repositoryId`, `postmortem create` links the postmortem to that repository automatically.
