# AgentTeams Convention

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> ⚠️ **Priority**: When platform rules (Part 1) and project rules (Part 2) conflict, **project rules take precedence**.

---

# Part 1 — Platform Rules

Rules for interacting with the AgentTeams platform (CLI, plans, reports, conventions). These are system-level and apply to all projects.

## Session Start — Freshness

> ⚠️ Run this **once per session, before substantive work** — not before every command. Always-on conventions are loaded into your context once at session start, so a stale local copy means you work from out-of-date rules without noticing. Skill packages have a separate owner and are **not** covered by the convention check: a fresh clone or a new git worktree starts with no packages at all, and the Skill Index below then points at files that do not exist.

1. Check both surfaces (read-only, no writes — only the `download` commands write):

   ```bash
   agentteams convention status
   agentteams skill status
   ```

2. For each surface reporting `updateAvailable: true`, sync:

   ```bash
   agentteams convention download
   agentteams skill download
   ```

3. After `convention download`, re-read the affected `.agentteams/` rule files (and this convention) before proceeding, so you act on the latest rules.
4. `agentteams skill download` also copies each package into the engine-native paths. If you are `CLAUDE_CODE` and the repository has no `.claude/` directory, that mirror is skipped by default — pass `--skill-targets agents,claude` so your own engine gets one. See `.agentteams/platform/skill-package-guide.md` for the per-engine paths.
5. If the CLI is unavailable, the project is not configured, or a command fails with `unknown command 'skill'` (the installed CLI predates skill packages), **skip silently and continue** — that is the signal, not an error to report, and never a reason to block.

> **A skill you need right now**: read `.agentteams/skills/<slug>/SKILL.md` directly from the Skill Index below. Downloading is enough to make that possible. Engine-native loading happens when a session starts, so a package fetched mid-session is picked up natively from the next session on.

## Reading Entities (MCP First)

User messages may carry entity references as `[label](type:id)` or `[label](type:id:path)`.

Always prefer MCP for read-only entity access when the AgentTeams MCP server is connected.

- **Type already known**: the typed `agentteams_*_get` MCP tool wins. **Each tool's own description is the SSOT** for paging, filter defaults, and truncation — read it and decide, rather than following a copy kept here.
- **Type must be worked out from the token**: use `agentteams_resolve` if your tool list has it; otherwise hand the token to the CLI verbatim — it detects the type, strips the id prefix, and dispatches:

  ```bash
  agentteams resolve "plan:agentteams_pln_f62762fc-730a-4201-8586-e2541505ed1b"
  ```

  Act on the returned `kind`: `file` / `localFile` → read `filePath`; `record` → use the inline payload; `external` → open `url` or run `suggestedCommand` (`gh`, `glab`). The `agentteams_resolve` tool answers with the same `kind`s minus `file` (bodies come back inline as `record`), and its `localFile` result carries `filePath` only when the session is bound to a local checkout — otherwise read its project-root-relative `path`.

- Exception: `convention:id:path` already carries the deployed local path — read that file directly, no call.
- **`SENTRY_ISSUE:<numeric-id>`**: carries only Sentry's canonical issue ID. Never build a permalink from that ID. Read it through `agentteams sentry issue get --issue-id <numeric-id>` so the server verifies the project binding and returns the stored Sentry permalink (`.agentteams/platform/sentry-guide.md`).
- **Older CLI without `resolve`** (fails with `unknown command 'resolve'` — the signal, not an error to report): fall back to `plan|report|postmortem|coaction|document download --id`, `code-review get --id|--finding-id`, `task get --task-id`, `linear issue get --issue-id`, `sentry issue get --issue-id`.

Use the `agentteams` CLI for everything else: mutations without an MCP write tool, downloads, workflows that create local files, and any environment where MCP or the matching tool is unavailable. CLI commands call AgentTeams HTTP APIs directly; they do not use MCP as an internal transport. Both surfaces share the same authenticated APIs and project scope.

```bash
# CLI fallback example when MCP is unavailable
agentteams search --query "<keyword>"
```

## Platform Records — Register, Then Link

> ⚠️ When creating, updating, or deleting platform records (plans, conventions, skills, reports, postmortems, co-actions, code reviews, documents), do not stop at writing local files — always register the result to the server via the CLI. If the CLI is unavailable, skip reporting and continue the task.

- **Skills are the easiest one to leave unregistered**, because a skill package is a local directory and writing the files feels like finishing. It is not: until `agentteams skill create --dir .agentteams/skills/<slug> --apply` runs, the package exists only on this machine — the web list, the Skill Index in this file, and every other runner stay empty.
- **Always display `webUrl`**: when a CLI command output contains a `webUrl` field, you **must** show it to the user as a clickable markdown link (e.g. `[View in AgentTeams](https://...)`) — not as raw text or inline code. The URL is the primary way users navigate to the created or updated entity.
- **Read the matching guide before writing or updating a record.** Do not guess flag values or document structure.

| Record type                            | Guide to read                                                                 |
| -------------------------------------- | ----------------------------------------------------------------------------- |
| Plan authoring                         | `.agentteams/platform/plan-authoring-guide.md` and the tier template it names |
| Plan execution                         | `.agentteams/platform/plan-execution-guide.md` (**Task Lifecycle**)           |
| Completion report                      | `.agentteams/platform/completion-report-guide.md`                             |
| Postmortem                             | `.agentteams/platform/post-mortem-guide.md`                                   |
| Convention (create)                    | `.agentteams/platform/convention-authoring-guide.md`                          |
| Convention (update/delete)             | `.agentteams/platform/convention-ud-guide.md`                                 |
| Skill (capability package)             | `.agentteams/platform/skill-package-guide.md`                                 |
| Co-action (handoff)                    | `.agentteams/platform/co-action-guide.md`                                     |
| Code review (independent verification) | `.agentteams/platform/code-review-guide.md`                                   |
| Document (human-facing artifact)       | `.agentteams/platform/document-guide.md`                                      |
| Comment or reply                       | `.agentteams/platform/comment-guide.md`                                       |
| Linear (issue/comment)                 | `.agentteams/platform/linear-guide.md`                                        |
| Sentry (project issue context)         | `.agentteams/platform/sentry-guide.md`                                        |

## Plan Execution

Report status to AgentTeams **if you are working under a plan**. For a downloaded V2 plan with structured tasks, per-task lifecycle tracking is required — it is not an optional progress note.

```bash
# Prepare the runbook (saves to .agentteams/cli/active-plan/{filename}.md) and check human guidance
agentteams plan download --id {planId}
agentteams comment list --plan-id {planId}        # read RISK comments before starting
agentteams plan start --id {planId}

# Repeat for every executable task whose dependencies are complete
agentteams comment list --task-id {taskId}
agentteams task start --plan-id {planId} --task-id {taskId}
# ... implement and verify the task's Acceptance Criteria ...
agentteams task finish --plan-id {planId} --task-id {taskId} --status <DONE | BLOCKED | SKIPPED>

# After every task reaches a terminal status, commit (see Commit Attribution), then finish
agentteams plan finish --id {planId} ...   # registers the completion report
```

- Call `task start` immediately before beginning each task; select only a task whose dependencies are complete.
- Use `DONE` only after the task's Acceptance Criteria have been verified.
- `BLOCKED` and `SKIPPED` require an explanatory plan comment.
- `SKIPPED` is terminal and satisfies downstream task dependencies.
- Do not call `plan finish` while any task remains `TODO` or `IN_PROGRESS`.

> ⚠️ If your work produced code changes, **commit them before `plan finish`**. `plan finish` auto-collects commit metrics (`commitHash`, `branchName`, diff stats) from the current git state — finishing with uncommitted work records an empty or wrong snapshot. If the project requires a PR, open it before finishing too.

The detailed execution workflow (task selection, statuses, errors, comments, and cleanup) is in `.agentteams/platform/plan-execution-guide.md` (**During Plan Execution**). The full `plan finish` invocation — every flag plus quality-score / report-status / review-recommendation semantics — is in `.agentteams/platform/completion-report-guide.md`.

## Commit Attribution

When an AgentTeams agent creates a git commit, append a co-author trailer at the end of the commit message (separated from the body by a blank line) so the work is attributed to AgentTeams:

```
Co-authored-by: AgentTeams <noreply@agentteams.run>
```

If another tool (e.g. Claude) adds its own `Co-authored-by` trailer, keep both — one trailer per line, all at the very end of the message.

## Work Completion

`plan finish` auto-creates the completion report. Then decide each of the following **independently** — one does not imply or replace another:

- **Evidence produced** (verification output saved to a file, before/after screenshots, repro/perf logs) → attach it so the claim travels with proof: `agentteams attachment create --completion-report-id {id}` (see Attaching Evidence in `.agentteams/platform/completion-report-guide.md`). Evidence behind a review finding → attach it to the review instead (see Attaching Evidence in the code-review guide).
- **Handoff needed** → create a co-action (`.agentteams/platform/co-action-guide.md`).
- **Risk signals** — cross-workspace; schema/auth/billing/quota/deployment; large diffs; failed verification → recommend a code review as a separate explicit action (`.agentteams/platform/code-review-guide.md`).

### Without a Plan (only when the user explicitly requests a report)

A completion report is always tied to a plan — there is no standalone (plan-less) report. To record work you already finished without a pre-existing plan, use a **quick log** (`plan quick`), which registers the plan and the report in a single step.

1. No active plan → ask: "No active plan found. Record it as a quick log?"
2. Approved → run the command below.
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

> The `--content` body format (and when to choose a quick log over the full lifecycle) is the SSOT in `.agentteams/platform/plan-execution-guide.md` (**Quick Log**).

---

# Part 2 — Project Rules

Project-specific conventions defined by the team. These rules govern coding standards, workflow, and domain-specific guidelines for this repository.
