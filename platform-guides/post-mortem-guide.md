# Post Mortem Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

A post mortem is a written analysis of an incident or a recurring development-process issue, focusing on root cause and prevention.

## Scope

Post mortems cover two categories:

### 1. Service Incidents

User-facing outages, regressions, or degraded behavior — the traditional post mortem scope.

### 2. Development Execution Issues

Issues that surfaced during development execution and are likely to recur if left unaddressed. These are **not** user-visible incidents but affect reliability, velocity, or correctness of the development process itself.

**Include when all three conditions are met:**

1. The issue is **reproducible or systemic** — not a one-off typo or local misconfiguration.
2. The issue **blocked or significantly delayed** the planned work.
3. The issue can be **prevented by a process, tooling, or environment change**.

**Examples:**

- Database migration failure due to missing prerequisite state or incompatible schema assumptions
- Missing or version-mismatched CLI tool / dependency that the workflow assumes is present
- Environment variable or configuration absent from a required environment (staging, CI, etc.)
- Verification step (typecheck, test, lint) that passed locally but failed in CI due to environment differences
- Unexpected behavioral regression discovered during execution, not caught by existing tests

**Do NOT create a post mortem for:**

- One-off typos, syntax errors, or trivial local mistakes that were fixed immediately
- Known limitations already documented elsewhere
- Issues that only affect a single developer's local setup with no recurrence risk

## Writing Notes

- **Root cause, not symptoms** — describe the causal chain; distinguish trigger vs contributing factors vs detection failure, and include "why it wasn't caught earlier".
- **Timeline** — collect logs, alerts, and key timestamps first; identify the affected systems and user segments. Use a clear timeline with timestamps when available.
- **No blame** — focus on systems and process, including what detection/monitoring failed and how to improve it.
- **Action items** — concrete, testable, with clear ownership; track like normal plans when they need engineering work.
- **Status**: `OPEN`, `IN_PROGRESS`, or `RESOLVED`.

## Diagrams (Mermaid)

A fenced `mermaid` block (`gantt` / `sequenceDiagram`) renders in the web viewer; it stays plain text in CLI/raw. Use one to visualize the incident timeline or chain of events.

## Post Mortem Content Template

Use this structure for the `--content` field or the `--file` content:

```markdown
## Summary

<what happened — 1-2 sentences, be specific>

## Category

<Service Incident | Development Execution Issue>

## Timeline

- HH:MM - <event 1>
- HH:MM - <event 2>

## Root Cause

<specific cause, not symptoms — describe the causal chain>

## Impact

<who/what was affected and for how long — for development execution issues, describe the delay or blocked work>

## What Went Wrong

- <contributing factor 1>
- <contributing factor 2>

## What Went Well

- <what helped limit the damage or speed up recovery>

## Prevention

- <specific process, tooling, or environment change to prevent recurrence>
```

## Post Mortem File Naming

- **Plan-linked post mortem:** use `{first 8 characters of planId}-postmortem.md`. Example: if planId is `57a51ec2-cf70-...`, use `57a51ec2-postmortem.md`.
- **Standalone service incident:** use a concise descriptive name such as `checkout-outage-postmortem.md`.

## Useful Commands

```bash
# Plan-linked incident or development execution issue
agentteams postmortem create \
  --plan-id {planId} \
  --title "<what broke — e.g., 'API 500 on plan finish after schema migration'>" \
  --file .agentteams/cli/temp/{planId-first-8-chars}-postmortem.md \
  --action-items "<specific preventive action 1>,<specific preventive action 2>" \
  --status OPEN

# Standalone service incident. Omit `--plan-id` when no plan is associated.
agentteams postmortem create \
  --title "<user-facing incident summary>" \
  --file .agentteams/cli/temp/{descriptive-name}-postmortem.md \
  --action-items "<specific preventive action 1>,<specific preventive action 2>" \
  --status OPEN
```
