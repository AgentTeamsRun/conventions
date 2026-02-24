# AgentTeams Reporting Rule

When work starts or ends, report status to AgentTeams **only if you are working under a plan**.

## On work start

If you are working under a plan, start the plan:

```bash
agentteams plan start --id {planId}
```

If you are not working under a plan, skip reporting and continue the task.

## On work completion

If you are working under a plan:

```bash
agentteams plan finish --id {planId} --report-title "Work completion summary" --report-file .agentteams/temp/<report-file-name>.md
```

If you are not working under a plan, skip reporting and continue the task.

> If the CLI is unavailable, skip reporting and continue the task.

### Completion Report Writing Guide

Before writing a completion report, read: `.agentteams/platform/completion-report-guide.md`

Assign `--quality-score` based on four dimensions: Verification, Completeness, Scope Adherence (includes conventions), and Side Effects.
90+ = all passing · 70-89 = minor gaps · 0-69 = failures or violations.

Include reproducible verification evidence (commands + outcomes), but keep outcomes short:
write pass/fail plus 1-3 lines of summary; do not paste long raw logs into the report body.

If you are not working under a plan and need a standalone report:

```bash
agentteams report create \
  --title "Work completion summary" \
  --file .agentteams/temp/<report-file-name>.md \
  --quality-score 95
```

### Postmortem Submission

If you had an incident or a high-severity quality issue, create a postmortem.

Before writing a postmortem, read: `.agentteams/platform/post-mortem-guide.md`

```bash
agentteams postmortem create \
  --plan-id {planId} \
  --title "Incident postmortem" \
  --file .agentteams/temp/<postmortem-file-name>.md \
  --action-items "Follow-up 1,Follow-up 2" \
  --status OPEN
```

## Quick Plan

When a completion report is requested but no plan exists, suggest creating a quick plan first:

1. Ask the user whether to create a quick plan.
2. If approved:
   ```bash
   agentteams plan quick --title "[work summary]" --report-title "Work completion summary" --report-file .agentteams/temp/<report-file-name>.md
   ```
   This single command creates a plan (template: quick-minimal, priority: LOW), starts it, and finishes it with the report attached.
3. If declined, create a standalone report:
   ```bash
   agentteams report create --title "Work completion summary" --file .agentteams/temp/<report-file-name>.md
   ```

## Plan Workflow Rules

### Download plan snapshot before starting

Download the plan as a local runbook before starting work.

```bash
agentteams plan download --id {planId}
```

This saves the plan to `.agentteams/active-plan/{filename}.md`.
Always reference this file during work. After completing or cancelling the plan, clean up:

```bash
agentteams plan cleanup --id {planId}
```

### Check comments before starting

Before working on a plan, read its comments first.

```bash
agentteams comment list --plan-id {planId}
```

Pay special attention to `RISK` comments.

### Post comments during work

- **RISK**: `agentteams comment create --plan-id {planId} --type RISK --content "[risk details]"`
- **MODIFICATION**: `agentteams comment create --plan-id {planId} --type MODIFICATION --content "[what changed and why]"`
- **GENERAL**: `agentteams comment create --plan-id {planId} --type GENERAL --content "Done. Verified (typecheck/tests)."`

### Required guide checks before writing documents

You MUST read the relevant platform guide before writing or updating related documents.

- Plan execution: `.agentteams/platform/plan-guide.md`
- New plan: `.agentteams/platform/plan-template.md`
- Completion reports: `.agentteams/platform/completion-report-guide.md`
- Post mortems: `.agentteams/platform/post-mortem-guide.md`
- Conventions: `.agentteams/platform/convention-authoring-guide.md`
- Convention update/delete: `.agentteams/platform/convention-ud-guide.md`

If the required guide is not checked, do not proceed with document creation.
