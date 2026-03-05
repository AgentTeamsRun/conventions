# AgentTeams Reporting Rule

Report status to AgentTeams **if you are working under a plan**.
If you are not working under a plan, see the **Completion report without a plan** section below.

> If the CLI is unavailable, skip reporting and continue the task.

## On work start

```bash
agentteams plan start --id {planId}
```

## On work completion

```bash
agentteams plan finish --id {planId} \
  --report-title "<what you did and why, in one sentence>" \
  --report-file .agentteams/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100, see scoring rules below> \
  --report-status <see status rules below>
```

### Report file naming

Use `{first 8 characters of planId}-report.md`. Example: if planId is `57a51ec2-cf70-...`, the file name is `57a51ec2-report.md`.

### Report file structure

Write the report file with this structure before running `plan finish`:

```markdown
## Summary
- <what changed and why — be specific, not generic>

## Verification
- typecheck: <pass or fail, with the command you ran>
- tests: <pass or fail, with the command you ran>

## Notes
- <risks, follow-ups, or "none">

## Conventions Referenced
- <list .agentteams/rules/*.md files you actually referenced — do not guess>
```

### Quality score rules

Assign `--quality-score` by evaluating these four dimensions:

| Dimension | What to check |
|---|---|
| Verification | Did typecheck and tests pass? |
| Completeness | Were all requirements addressed? |
| Scope Adherence | Were changes limited to the requested scope? Were conventions followed? |
| Side Effects | Did the change avoid unintended impact on existing behavior? |

**Score**: 90-100 = all four pass · 70-89 = minor gaps · 0-69 = failures or violations.

### Report status rules

| Status | When to use |
|---|---|
| `COMPLETED` | All verification passed, all requirements met |
| `PARTIAL` | Some requirements done, but work remains (e.g., blocked by external dependency) |
| `FAILED` | Verification failed, critical requirements unmet, or changes had to be reverted |

### Git metrics

Git metrics (`commitHash`, `branchName`, `filesModified`, `linesAdded`, `linesDeleted`) are auto-collected. Use `--no-git` to disable. Manual-only fields: `--duration-seconds`, `--commit-start`, `--commit-end`, `--pull-request-id`.

---

## Completion report without a plan

When you complete work **without a plan**, follow this decision flow:

1. Ask the user: "No active plan found. Should I create a quick plan to record this work?"
2. If approved:
   ```bash
   agentteams plan quick --title "<brief work summary>" \
     --content "<TL;DR and actual tasks performed>" \
     --type <FEATURE | BUG_FIX | ISSUE | REFACTOR | CHORE>
   ```
   Then create the report separately:
   ```bash
   agentteams report create \
     --plan-id <planId from quick plan> \
     --title "<what you did and why, in one sentence>" \
     --file .agentteams/temp/<planId-first-8-chars>-report.md \
     --quality-score <0-100, see scoring rules above> \
     --status <COMPLETED | PARTIAL | FAILED>
   ```
3. If declined, create a standalone report (no plan link):
   ```bash
   agentteams report create \
     --title "<what you did and why, in one sentence>" \
     --file .agentteams/temp/<descriptive-name>-report.md \
     --quality-score <0-100, see scoring rules above> \
     --status <COMPLETED | PARTIAL | FAILED>
   ```

The same report file structure, quality score rules, and status rules apply.

---

## Plan Workflow Rules

### Before starting work on a plan

1. Download the plan as a local runbook:
   ```bash
   agentteams plan download --id {planId}
   ```
   This saves to `.agentteams/active-plan/{filename}.md`. Read this file at the start of your work.

3. If the plan contains entity references in `[label](type:id)` or `[label](type:id:path)` format, resolve them:
   - **ID prefix stripping (IMPORTANT)**: The `id` part may include a type prefix such as `plan_`, `cr_`, `ca_`, `conv_`, or `pm_`. Always strip this prefix before passing the id to any CLI flag (`--id`, `--plan-id`, etc.).
     - Example: `[My Plan](plan:plan_f62762fc-730a-4201-8586-e2541505ed1b)` → use `f62762fc-730a-4201-8586-e2541505ed1b`
     - Full prefix list: `plan_` · `cr_` · `ca_` · `conv_` · `pm_`
   - `convention:id:.agentteams/path` → Read the local file at the given path (e.g., `.agentteams/rules/context.md`)
   - `completionReport:id` → Download with `agentteams report download --id {id}` and read the local file
   - `postMortem:id` → Download with `agentteams postmortem download --id {id}` and read the local file
   - `coAction:id` → Download with `agentteams coaction download --id {id}` and read the local file

2. Check for comments (especially `RISK` comments):
   ```bash
   agentteams comment list --plan-id {planId}
   ```

### During work

Post comments to track progress:

- **Risk found**: `agentteams comment create --plan-id {planId} --type RISK --content "<describe the risk and its potential impact>" --affected-files "<comma-separated file paths>"`
- **Scope changed**: `agentteams comment create --plan-id {planId} --type MODIFICATION --content "<what changed from the original plan and why>" --affected-files "<comma-separated file paths>"`
- **Status update**: `agentteams comment create --plan-id {planId} --type GENERAL --content "<current progress with specific verification results>"`

### After completing or cancelling a plan

Clean up the local runbook:

```bash
agentteams plan cleanup --id {planId}
```

### Guide checks before writing documents

Before writing or updating **platform documents** (plans, reports, conventions, postmortems), read the matching guide:

| Document type | Guide to read |
|---|---|
| Plan execution | `.agentteams/platform/plan-guide.md` |
| New plan | `.agentteams/platform/plan-template.md` |
| Completion report | `.agentteams/platform/completion-report-guide.md` |
| Postmortem | `.agentteams/platform/post-mortem-guide.md` |
| Convention (create) | `.agentteams/platform/convention-authoring-guide.md` |
| Convention (update/delete) | `.agentteams/platform/convention-ud-guide.md` |
| Co-action (handoff) | `.agentteams/platform/co-action-guide.md` |
