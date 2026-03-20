# AgentTeams Convention

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> ⚠️ **Priority**: When platform rules (Part 1) and project rules (Part 2) conflict, **project rules take precedence**.

---

# Part 1 — Platform Rules

Rules for interacting with the AgentTeams platform (CLI, plans, reports, conventions). These are system-level and apply to all projects.

## Entity References & ID Handling

User messages from the AgentTeams web UI may contain entity references in `[label](type:id)` or `[label](type:id:path)` format.

- **ID prefix stripping (IMPORTANT)**: The `id` part includes a type prefix (`plan_`, `cr_`, `ca_`, `conv_`, `pm_`). Always strip this prefix before passing the id to any CLI flag (`--id`, `--plan-id`, etc.).
  - Example: `[Safari pull-to-refresh](plan:plan_f62762fc-730a-4201-8586-e2541505ed1b)` → use `f62762fc-730a-4201-8586-e2541505ed1b`
  - Full prefix list: `plan_` · `cr_` · `ca_` · `conv_` · `pm_`
- Resolution by type:
  - `convention:id:.agentteams/path` → Read the local file at the given path (e.g., `.agentteams/rules/context.md`)
  - `completionReport:id` → Download with `agentteams report download --id {id}` and read the local file
  - `postMortem:id` → Download with `agentteams postmortem download --id {id}` and read the local file
  - `coAction:id` → Download with `agentteams coaction download --id {id}` and read the local file
  - `LINEAR_ISSUE:uuid` → Fetch issue details with `agentteams linear issue get --issue-id {uuid}` and use the response as context. No prefix stripping needed — the value after `LINEAR_ISSUE:` is the raw Linear issue UUID.
  - `GITHUB_ISSUE:owner/repo#number` → GitHub issue. Use `gh issue view {number} --repo {owner/repo}` or GitHub API to fetch details. No prefix stripping needed.
  - `GITHUB_PR:owner/repo#number` → GitHub pull request. Use `gh pr view {number} --repo {owner/repo}` or GitHub API to fetch details. No prefix stripping needed.
  - `GITLAB_ISSUE:projectPath#iid` → GitLab issue. Use `glab issue view {iid} --repo {projectPath}` or GitLab API to fetch details. No prefix stripping needed.
  - `GITLAB_MERGE_REQUEST:projectPath!iid` → GitLab merge request. Use `glab mr view {iid} --repo {projectPath}` or GitLab API to fetch details. No prefix stripping needed.

## CLI Output Rules

> ⚠️ When creating, updating, or deleting platform documents (plans, conventions, reports, postmortems, co-actions), do not stop at writing local files — always register the result to the server via the CLI.

> ⚠️ **Always display `webUrl`**: When a CLI command output contains a `webUrl` field, you **must** show it to the user as a clickable markdown link (e.g., `[View in AgentTeams](https://...)`) — not as raw text or inline code. Do not omit or summarize it away — the URL is the primary way users navigate to the created or updated entity.

> If the CLI is unavailable, skip reporting and continue the task.

## Plan Lifecycle (Quick Reference)

Report status to AgentTeams **if you are working under a plan**.

```bash
# Start
agentteams plan start --id {planId}

# Finish (with completion report)
agentteams plan finish --id {planId} \
  --report-title "<what you did and why, in one sentence>" \
  --report-file .agentteams/cli/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100> \
  --report-status <COMPLETED | PARTIAL | FAILED>
```

For report file structure, quality score dimensions, and status rules, see `.agentteams/platform/completion-report-guide.md`.

## Plan Workflow Rules

### Before starting work on a plan

1. Download the plan as a local runbook:
   ```bash
   agentteams plan download --id {planId}
   ```
   This saves to `.agentteams/cli/active-plan/{filename}.md`. Read this file at the start of your work.

2. Check for comments (especially `RISK` comments):
   ```bash
   agentteams comment list --plan-id {planId}
   ```

3. For detailed execution workflow (entity references, comments, cleanup), see `.agentteams/platform/plan-guide.md`.

## Work Completion Rules

### With a Plan

- Completion report is automatically created when `plan finish` is executed
- If handoff to another agent is needed, also create a co-action (see `.agentteams/platform/co-action-guide.md`)

### Without a Plan

Only execute the following flow when the user explicitly requests a completion report:

1. Ask the user whether to create a quick plan: "No active plan found. Would you like to create a quick plan?"
2. If approved:
   ```bash
   agentteams plan quick --title "<brief work summary>" \
     --content "<content following the format below>" \
     --type <FEATURE | BUG_FIX | ISSUE | REFACTOR | CHORE>
   ```
   Then follow the report creation steps in `.agentteams/platform/completion-report-guide.md`.
3. If declined: create a standalone completion report (no plan link). See `.agentteams/platform/completion-report-guide.md`.
4. If handoff to another agent is needed, also create a co-action (see `.agentteams/platform/co-action-guide.md`)

#### Quick Plan `--content` Format

```markdown
## TL;DR
<!-- 1-2 sentence summary -->

## Work Performed
- <!-- changed files / description -->

## Verification Results
- <!-- build/test pass status -->
```

## Unified Search

Use `agentteams search` to find entities (plans, co-actions, reports, post-mortems, conventions) across the project in a single call.

```bash
agentteams search --query "<keyword>" --format json
```

## Guide Checks Before Writing Documents

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
| Linear (issue/comment) | `.agentteams/platform/linear-guide.md` |

---

# Part 2 — Project Rules

Project-specific conventions defined by the team. These rules govern coding standards, workflow, and domain-specific guidelines for this repository.
