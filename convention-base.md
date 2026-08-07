# AgentTeams Convention

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> ⚠️ **Priority**: When platform rules (Part 1) and project rules (Part 2) conflict, **project rules take precedence**.

---

# Part 1 — Platform Rules

Rules for interacting with the AgentTeams platform (CLI, plans, reports, conventions). These are system-level and apply to all projects.

## Convention Freshness

> ⚠️ At the start of a session, **before substantive work**, make sure your conventions are current. The always-on conventions are loaded into your context once at session start, so a stale local copy means you would work from out-of-date rules without noticing.

1. If the `agentteams` CLI is available, check for updates (read-only, no writes):

   ```bash
   agentteams convention status
   ```

2. If updates are reported (`updateAvailable: true`), sync and **reload**:

   ```bash
   agentteams convention download
   ```

   Then re-read the affected `.agentteams/` rule files (and this convention) before proceeding, so you act on the latest rules.

3. If no updates are reported, proceed.
4. If the CLI is unavailable or the project is not configured, **skip this check silently and continue** — do not block on it.

> Run this check **once per session**, not before every command. Only `agentteams convention download` writes anything; `agentteams convention status` is safe to call anytime.

## Entity References & ID Handling

User messages may carry entity references as `[label](type:id)` or `[label](type:id:path)`. Hand the reference token to the CLI verbatim — it detects the type, strips the id prefix, and dispatches:

```bash
agentteams resolve "plan:agentteams_pln_f62762fc-730a-4201-8586-e2541505ed1b"
```

Act on the returned `kind`: `file` / `localFile` → read `filePath`; `record` → use the inline payload; `external` → open `url` or run `suggestedCommand` (`gh`, `glab`). The `agentteams_resolve` tool answers with the same `kind`s minus `file` (bodies come back inline as `record`), and its `localFile` result carries `filePath` only when the session is bound to a local checkout — otherwise read its project-root-relative `path`.

- **MCP first**: when the AgentTeams MCP server is connected and the type is already known, the typed `agentteams_*_get` tool wins (see **AgentTeams Read Tools**). When the type has to be worked out from the token, use `agentteams_resolve` **if your tool list has it** — otherwise the `resolve` command above does the same job. Exception: `convention:id:path` already carries the deployed local path — read that file directly, no call.
- **Older CLI without `resolve`** (the command fails with `unknown command 'resolve'` — that is the signal, not an error to report): fall back to `plan|report|postmortem|coaction|document download --id`, `code-review get --id|--finding-id`, `task get --task-id`, `linear issue get --issue-id`; a `convention:id:path` reference is just a local file — read the path.

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

Report status to AgentTeams **if you are working under a plan**. For a downloaded V2 plan with structured
tasks, per-task lifecycle tracking is required — it is not an optional progress note.

```bash
# Prepare the runbook and check human guidance
agentteams plan download --id {planId}
agentteams comment list --plan-id {planId}
agentteams plan start --id {planId}

# Repeat for every executable task
agentteams task start --plan-id {planId} --task-id {taskId}
# ... implement and verify the task's Acceptance Criteria ...
agentteams task finish --plan-id {planId} --task-id {taskId} --status <DONE | BLOCKED | SKIPPED>

# After every task reaches a terminal status, commit (see Commit Attribution), then finish
agentteams plan finish --id {planId} ...   # registers the completion report
```

- Call `task start` immediately before beginning each task.
- Use `DONE` only after the task's Acceptance Criteria have been verified.
- `BLOCKED` and `SKIPPED` require an explanatory plan comment.
- Do not call `plan finish` while any task remains `TODO` or `IN_PROGRESS`.
- `SKIPPED` is terminal and satisfies downstream task dependencies.

> ⚠️ If your work produced code changes, **commit them before `plan finish`**. `plan finish` auto-collects commit metrics (`commitHash`, `branchName`, diff stats) from the current git state — finishing with uncommitted work records an empty or wrong snapshot. If the project requires a PR, open it before finishing too.
>
> The full `plan finish` invocation — every flag plus quality-score / report-status / review-recommendation semantics — lives in `.agentteams/platform/completion-report-guide.md`. Read it before finishing; do not guess flag values.

## Plan Workflow Rules

When executing a plan:

1. Download the runbook — `agentteams plan download --id {planId}` (saves to `.agentteams/cli/active-plan/{filename}.md`). Read it before starting.
2. Check comments, especially `RISK` — `agentteams comment list --plan-id {planId}`.
3. Start the plan lifecycle before starting any task.
4. For a V2 plan, select only a task whose dependencies are complete, read its own comments (`agentteams comment list --task-id {taskId}`), then run its required start → implement → verify → finish lifecycle.
5. Repeat until no task remains `TODO` or `IN_PROGRESS`; explain every `BLOCKED` or `SKIPPED` result in a plan comment.
6. Commit code changes before `plan finish`, then complete the report and cleanup flow.

The detailed execution workflow (task selection, statuses, errors, comments, and cleanup) is in
`.agentteams/platform/plan-guide.md` (**During Plan Execution**).

## Work Completion Rules

### With a Plan

- `plan finish` auto-creates the completion report.
- Evidence produced (verification output saved to a file, before/after screenshots, repro/perf logs) → attach it to the report so the claim travels with proof (`agentteams attachment create --completion-report-id {id}`; see Attaching Evidence in `.agentteams/platform/completion-report-guide.md`).
- Handoff needed → also create a co-action (`.agentteams/platform/co-action-guide.md`).
- Risk signals (cross-workspace; schema/auth/billing/quota/deployment; large diffs; failed verification) → recommend a code review as a separate explicit action (`.agentteams/platform/code-review-guide.md`). Keep this decision independent of co-action / post-mortem decisions. Evidence behind a finding → attach it to the review (see Attaching Evidence in the code-review guide).

### Without a Plan (only when the user explicitly requests a report)

A completion report is always tied to a plan — there is no standalone (plan-less) report. To record work you already finished without a pre-existing plan, use a **quick log** (`plan quick`), which registers the plan and the report in a single step.

1. No active plan → ask: "No active plan found. Record it as a quick log?"
2. Approved → run `agentteams plan quick` with completion report flags to register the plan and report in a single step (see `.agentteams/platform/completion-report-guide.md` for report flags).
3. Declined → no report is recorded (a report cannot be created without a plan).
4. Handoff needed → also create a co-action (`.agentteams/platform/co-action-guide.md`).

```bash
# Record already-done work as a quick log (plan + completion report in one workflow)
# (see completion-report-guide.md for all available report flags)
agentteams plan quick --title "<brief work summary>" \
  --content "<see format below>" \
  --type <FEATURE | BUG_FIX | ISSUE | REFACTOR | CHORE> \
  --runner-type <runner-type> --model <model-id> \
  --report-file "<path to report markdown>" \
  [report-flags...]
```

> The `--content` body format (and when to choose a quick log over the full lifecycle) is the SSOT in `.agentteams/platform/plan-guide.md` (**Quick Log**).

## AgentTeams Read Tools (MCP First)

When the AgentTeams MCP server is connected, prefer MCP for read-only entity access. **Each tool's own description is the SSOT** for paging, filter defaults, and truncation, and it always matches the tools actually exposed — read it and decide, rather than following a copy kept here.

Use the `agentteams` CLI for everything else: mutations without an MCP write tool, downloads, workflows that create local files, and any environment where MCP or the matching tool is unavailable.

CLI commands call AgentTeams HTTP APIs directly; they do not use MCP as an internal transport. Both surfaces share the same authenticated APIs and project scope.

```bash
# CLI fallback example when MCP is unavailable
agentteams search --query "<keyword>" --format json
```

## Guide Checks Before Writing Platform Records

Before writing or updating **platform records** (plans, reports, conventions, postmortems, code reviews, documents), read the matching guide:

| Record type                            | Guide to read                                                                    |
| -------------------------------------- | -------------------------------------------------------------------------------- |
| Plan authoring                         | `.agentteams/platform/plan-guide.md` and `.agentteams/platform/plan-template.md` |
| Plan execution                         | `.agentteams/platform/plan-guide.md` (**Task Lifecycle**)                        |
| Completion report                      | `.agentteams/platform/completion-report-guide.md`                                |
| Postmortem                             | `.agentteams/platform/post-mortem-guide.md`                                      |
| Convention (create)                    | `.agentteams/platform/convention-authoring-guide.md`                             |
| Convention (update/delete)             | `.agentteams/platform/convention-ud-guide.md`                                    |
| Co-action (handoff)                    | `.agentteams/platform/co-action-guide.md`                                        |
| Code review (independent verification) | `.agentteams/platform/code-review-guide.md`                                      |
| Document (human-facing artifact)       | `.agentteams/platform/document-guide.md`                                         |
| Comment or reply                       | `.agentteams/platform/comment-guide.md`                                          |
| Linear (issue/comment)                 | `.agentteams/platform/linear-guide.md`                                           |

---

# Part 2 — Project Rules

Project-specific conventions defined by the team. These rules govern coding standards, workflow, and domain-specific guidelines for this repository.
