# AgentTeams Convention

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> ⚠️ **Priority**: When platform rules (Part 1) and project rules (Part 2) conflict, **project rules take precedence**.

---

# Part 1 — Platform Rules

Rules for interacting with the AgentTeams platform (CLI, plans, reports, conventions). These are system-level and apply to all projects.

## Entity References & ID Handling

User messages from the AgentTeams web UI may contain entity references in `[label](type:id)` or `[label](type:id:path)` format.

- **ID prefix stripping (IMPORTANT)**: The `id` part can include a type prefix. Always strip this prefix before passing the id to any CLI flag (`--id`, `--plan-id`, etc.).
  - Example: `[Safari pull-to-refresh](plan:agentteams_pln_f62762fc-730a-4201-8586-e2541505ed1b)` → use `f62762fc-730a-4201-8586-e2541505ed1b`
  - Canonical prefix list: `agentteams_pln_` (plan) · `agentteams_rpt_` (completionReport) · `agentteams_rev_` (codeReview) · `agentteams_act_` (coAction) · `agentteams_cnv_` (convention) · `agentteams_pmt_` (postMortem) · `agentteams_doc_` (document)
- Resolution by type:
  - `convention:id:.agentteams/path` → Read the local file at the given path (e.g., `.agentteams/rules/context.md`)
  - `completionReport:id` → Download with `agentteams report download --id {id}` and read the local file
  - `postMortem:id` → Download with `agentteams postmortem download --id {id}` and read the local file
  - `coAction:id` → Download with `agentteams coaction download --id {id}` and read the local file
  - `codeReview:id` → Fetch the review record with `agentteams code-review get --id {id}` and use the response as context
  - `document:id` → Download with `agentteams document download --id {id}` and read the local file
  - `LINEAR_ISSUE:uuid` → Fetch issue details with `agentteams linear issue get --issue-id {uuid}` and use the response as context. No prefix stripping needed — the value after `LINEAR_ISSUE:` is the raw Linear issue UUID.
  - `GITHUB_ISSUE:owner/repo#number` → GitHub issue. Use `gh issue view {number} --repo {owner/repo}` or GitHub API to fetch details. No prefix stripping needed.
  - `GITHUB_PR:owner/repo#number` → GitHub pull request. Use `gh pr view {number} --repo {owner/repo}` or GitHub API to fetch details. No prefix stripping needed.
  - `GITLAB_ISSUE:projectPath#iid` → GitLab issue. Use `glab issue view {iid} --repo {projectPath}` or GitLab API to fetch details. No prefix stripping needed.
  - `GITLAB_MERGE_REQUEST:projectPath!iid` → GitLab merge request. Use `glab mr view {iid} --repo {projectPath}` or GitLab API to fetch details. No prefix stripping needed.

## CLI Output Rules

> ⚠️ When creating, updating, or deleting platform records (plans, conventions, reports, postmortems, co-actions, code reviews, documents), do not stop at writing local files — always register the result to the server via the CLI.

> ⚠️ **Always display `webUrl`**: When a CLI command output contains a `webUrl` field, you **must** show it to the user as a clickable markdown link (e.g., `[View in AgentTeams](https://...)`) — not as raw text or inline code. Do not omit or summarize it away — the URL is the primary way users navigate to the created or updated entity.

> If the CLI is unavailable, skip reporting and continue the task.

## Commit Attribution

When an AgentTeams agent creates a git commit, append a co-author trailer at the
end of the commit message (separated from the body by a blank line) so the work
is attributed to AgentTeams:

```
Co-authored-by: AgentTeams <noreply@agentteams.run>
```

If another tool (e.g. Claude) adds its own `Co-authored-by` trailer, keep both —
one trailer per line, all at the very end of the message.

## Plan Lifecycle (Quick Reference)

Report status to AgentTeams **if you are working under a plan**.

```bash
# Start
agentteams plan start --id {planId}

# Finish (with completion report)
agentteams plan finish --id {planId} \
  --runner-type <runner-type> --model <model-id> \
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
- If the change has risk signals (cross-workspace, schema/auth/billing/quota/deployment, large diffs, failed verification), recommend a code review as a separate explicit action (see `.agentteams/platform/code-review-guide.md`). Keep code review decisions independent from co-action and post-mortem decisions.

### Without a Plan

Only execute the following flow when the user explicitly requests a completion report:

1. Ask the user whether to create a quick plan: "No active plan found. Would you like to create a quick plan?"
2. If approved:
   ```bash
   agentteams plan quick --title "<brief work summary>" \
     --content "<content following the format below>" \
     --type <FEATURE | BUG_FIX | ISSUE | REFACTOR | CHORE> \
     --runner-type <runner-type> \
     --model <model-id> \
     --agent <agent-name-or-id>
   ```
   If `--agent` is omitted, `AGENTTEAMS_AGENT_NAME` must be set.
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

## Guide Checks Before Writing Platform Records

Before writing or updating **platform records** (plans, reports, conventions, postmortems, code reviews, documents), read the matching guide:

| Record type | Guide to read |
|---|---|
| Plan execution | `.agentteams/platform/plan-guide.md` |
| New plan | `.agentteams/platform/plan-template.md` |
| Completion report | `.agentteams/platform/completion-report-guide.md` |
| Postmortem | `.agentteams/platform/post-mortem-guide.md` |
| Convention (create) | `.agentteams/platform/convention-authoring-guide.md` |
| Convention (update/delete) | `.agentteams/platform/convention-ud-guide.md` |
| Co-action (handoff) | `.agentteams/platform/co-action-guide.md` |
| Code review (independent verification) | `.agentteams/platform/code-review-guide.md` |
| Document (human-facing artifact) | `.agentteams/platform/document-guide.md` |
| Linear (issue/comment) | `.agentteams/platform/linear-guide.md` |

---

# Part 2 — Project Rules

Project-specific conventions defined by the team. These rules govern coding standards, workflow, and domain-specific guidelines for this repository.
