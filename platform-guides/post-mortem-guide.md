# Post Mortem Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

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

Use this structure for the `--content` field or the `--file` content:

~~~markdown
## Summary
<what happened — 1-2 sentences, be specific>

## Timeline
- HH:MM - <event 1>
- HH:MM - <event 2>

## Root Cause
<specific cause, not symptoms — describe the causal chain>

## Impact
<who/what was affected and for how long>

## What Went Wrong
- <contributing factor 1>
- <contributing factor 2>

## What Went Well
- <what helped limit the damage or speed up recovery>
~~~

## Post Mortem File Naming

Use `{first 8 characters of planId}-postmortem.md`. Example: if planId is `57a51ec2-cf70-...`, the file name is `57a51ec2-postmortem.md`.

## Useful Commands

~~~bash
agentteams postmortem create \
  --plan-id {planId} \
  --title "<what broke — e.g., 'API 500 on plan finish after schema migration'>" \
  --file .agentteams/temp/{planId-first-8-chars}-postmortem.md \
  --action-items "<specific preventive action 1>,<specific preventive action 2>" \
  --status OPEN
~~~

Repository linkage note:

- If .agentteams/config.json contains `repositoryId`, `postmortem create` links the postmortem to that repository automatically.
