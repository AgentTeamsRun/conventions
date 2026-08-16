# Plan Execution Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines how to **execute and complete** a plan that already exists: downloading the runbook, tracking task status, recovering from lifecycle errors, and finishing. How to _write_ a plan is a separate concern — see `plan-authoring-guide.md`.

Project conventions remain authoritative for repository-specific workflow; this guide defines the AgentTeams plan and task lifecycle.

## Plan Start → Execution Flow

When a user explicitly says to start a plan (e.g. "start plan {id}", "let's start {id}"), treat it as an explicit execution approval. Follow this flow:

```bash
# 1. Download runbook
agentteams plan download --id {planId}

# 2. Check for blocking comments
agentteams comment list --plan-id {planId}

# 3. Start lifecycle
agentteams plan start --id {planId}

# 4. (V2 plans) As you work each task, mark its start/finish — details in "During Plan Execution"
agentteams task start --plan-id {planId} --task-id {taskId}
```

**Decision after comment check:**

| Comments found                   | Action                                                       |
| -------------------------------- | ------------------------------------------------------------ |
| `RISK` or `MODIFICATION` present | Report to user and wait for confirmation before implementing |
| None (or `GENERAL` only)         | Proceed with implementation immediately                      |

The phrase "start the plan" is an explicit approval signal — do not stop after the CLI status change. Implement unless a blocking comment requires human confirmation.

## Execution Shortcuts

For standard execution flows, use lifecycle shortcuts instead of manual multi-step status updates.

```bash
agentteams plan start  --id {planId}
agentteams plan finish --id {planId}   # add report flags to attach a completion report
```

For one-shot tasks, `plan quick` collapses create + start + finish into a single command — see **Quick Log** below.

> Commit your work before finishing — both `plan finish` and report-attaching `plan quick` auto-collect commit metrics from the current git state.
> The full report-attaching parameters (report flags + quality-score / report-status semantics) are defined in `completion-report-guide.md` as the SSOT.

## Entity Reference Resolution

Plan bodies may contain `[label](type:id)` / `[label](type:id:path)` references. Resolve them per the always-on convention's **Entity References & ID Handling** rules — pass the reference token as-is to the `agentteams_resolve` tool if your tool list has it, or to `agentteams resolve` otherwise, and act on the returned `kind`. No plan-specific rules beyond those.

## During Plan Execution

Post comments to track progress:

- **Risk found**: `agentteams comment create --plan-id {planId} --type RISK --content "<risk description>" --affected-files "<paths>"`
- **Scope changed**: `agentteams comment create --plan-id {planId} --type MODIFICATION --content "<what changed and why>" --affected-files "<paths>"`
- **Status update**: `agentteams comment create --plan-id {planId} --type GENERAL --content "<current progress>"`

### Task Lifecycle (V2 plans only)

Only **V2 plans that carry tasks** get a task sidecar (`.agentteams/cli/active-plan/{filename}.tasks.json`), written alongside the runbook at download time. Each sidecar entry contains the task's `id`, number, status, and dependency IDs. Use its `id` as the `--task-id`.

For a downloaded V2 plan with tasks, lifecycle tracking is required. Do not treat the sidecar's status as permanently current: task commands update the server, not the downloaded file. Use command output as the latest state, or download the plan again when you need a refreshed local snapshot.

### Selecting the Next Task

A task is ready when it is `TODO` and every task in `dependsOnTaskIds` is `DONE` or `SKIPPED`. Tasks in the same wave may run in parallel only when there is no dependency edge between them. Do not start a dependent task while a prerequisite is `TODO`, `IN_PROGRESS`, or `BLOCKED`; the API rejects that start and returns the blocking task IDs.

### Required Execution Loop

Repeat this loop for every ready task:

```bash
# 1. Read the task's own comments before touching the task
agentteams comment list --task-id {taskId}

# 2. Immediately before implementation
agentteams task start --plan-id {planId} --task-id {taskId}

# 3. Implement the task, then run its Acceptance Criteria and QA Scenarios

# 4. Record the verified result
agentteams task finish --plan-id {planId} --task-id {taskId} --status <DONE | BLOCKED | SKIPPED>
```

The plan-level comment check in "Plan Start → Execution Flow" does not cover task comments — they hang off the task and only `--task-id` returns them. Treat a task comment as part of that task's specification: it narrows acceptance criteria, names a blocker, or carries evidence the plan body does not. Reflect it in the implementation and in the verification you run.

Task comments carry no `type` and are not a stop signal — unlike a plan `RISK` or `MODIFICATION` comment, they do not pause execution. Keep working. When a task comment contradicts the plan body or demands a decision outside your scope, reflect what you can, then raise it on the plan with `agentteams comment create --plan-id {planId} --type RISK` and follow the existing plan-comment rules.

Do not mark a task `DONE` merely because code was written. Verification is part of the task. If verification fails and cannot be resolved within scope, use `BLOCKED` and record the failure evidence in a plan comment.

### Finish Status Semantics

| Status        | Meaning                                                                    | Counts as complete | Required follow-up                    |
| ------------- | -------------------------------------------------------------------------- | ------------------ | ------------------------------------- |
| `TODO`        | Not started                                                                | No                 | Start it, or intentionally skip it    |
| `IN_PROGRESS` | Work or verification is active                                             | No                 | Move it to a terminal status          |
| `DONE`        | Acceptance Criteria and required verification passed                       | Yes                | Preserve useful verification evidence |
| `BLOCKED`     | Work cannot proceed because of a decision, dependency, or external blocker | No                 | Add a `RISK` or `GENERAL` comment     |
| `SKIPPED`     | Intentionally not performed because it is out of scope or obsolete         | Yes                | Add a comment explaining why          |

`DONE` and `SKIPPED` satisfy downstream task dependencies. Task status changes do not finish the parent plan; `plan finish` performs that separate lifecycle transition.

### Lifecycle Error Recovery

| Reason                | Action                                                                 |
| --------------------- | ---------------------------------------------------------------------- |
| `DEPENDENCY_BLOCKED`  | Inspect `blockedByTaskIds`; finish those prerequisites first           |
| `NOT_V2`              | This plan has no V2 task lifecycle; track execution with plan comments |
| `NO_STRUCTURED_TASKS` | Re-check the authored `## TODOs` structure or use comment tracking     |
| `PLAN_TERMINAL`       | Do not mutate tasks on a completed or cancelled plan                   |

### Finish Preflight

Before `plan finish`, verify every item:

- [ ] No task remains `TODO`.
- [ ] No task remains `IN_PROGRESS`.
- [ ] Every `DONE` task passed its Acceptance Criteria and required QA Scenarios.
- [ ] Every `BLOCKED` or `SKIPPED` task has an explanatory plan comment.
- [ ] Useful verification evidence has been preserved for the completion report.
- [ ] Code changes have been committed, and a required PR has been opened.

V1 plans (or V2 plans without tasks) have no sidecar and no per-task lifecycle — track progress with comments only.

## After Completing or Cancelling a Plan

After the completion report, do not stop there:

1. Review whether a **Co-Action** and/or **Post-Mortem** is needed, then create and link them — criteria and commands are the SSOT in `completion-report-guide.md` (Post-Report: Linked Document Auto-Creation).
2. Clean up the local runbook once linked documents are created or explicitly ruled out:

```bash
agentteams plan cleanup --id {planId}
```

## Quick Log

A **quick log** is the way to record work you already finished when there is no plan for it yet. The `plan quick` command performs create + start + finish in a single command and optionally attaches a completion report in the same call, so the work is captured as a plan-linked report without a separate up-front planning step. Use it for work that does not need a downloaded runbook or multi-step status tracking.

> Because every completion report is plan-linked, a quick log is the standard path for logging already-done work — it provides the plan the report attaches to, in one shot.

### When to Use a Quick Log

| Use a quick log when…                                            | Use the full `plan create` lifecycle when…                        |
| ---------------------------------------------------------------- | ----------------------------------------------------------------- |
| Single, small, well-understood task finished in one session      | Work spans multiple steps, waves, or sessions                     |
| No separate runbook / step-by-step execution is needed           | Reviewers need to inspect the plan before execution               |
| You are recording work you just did (often with `--report-file`) | Risk signals apply (schema / auth / billing / quota / deployment) |

When unsure, prefer the full lifecycle — a quick log cannot be reviewed before the work happens.

### Command

```bash
agentteams plan quick --title "<title>" --content "<plan content>" \
  --runner-type <runner> --model <model> \
  [--assigned-to <agent config id or name>] \
  [--report-file <path> --report-title <report title> ...]
```

`--content` carries the plan body (format below). Adding report flags attaches a completion report in the same command. The `--runner-type` / `--model` contract is defined in `plan-authoring-guide.md` (**Runner Type & Model Reference**).

### Agent Assignment

A quick log is assigned to an agent, and where that agent comes from depends on how you authenticated:

| Credential                               | Where the agent comes from                                                                                              |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Agent API key (`key_…`)                  | Inferred from the key itself. `--assigned-to` is ignored — a proven identity is not overridable.                        |
| Personal login (`agentteams auth login`) | `$AGENTTEAMS_AGENT_NAME` when a runner spawned the session; otherwise the server identifies the agent from the session. |

The server's step is a fallback, not a guess: it matches the calling machine against the agents **you** registered in this project, and narrows by project root when one machine holds several. If that does not land on exactly one agent, it assigns nothing rather than picking — the call then fails with `Agent assignment is required`, and you pass `--assigned-to` with an id or name from `agentteams agent-config list`.

`--assigned-to` always beats the server's judgment, so naming a different agent still works.

> This applies to `plan start` and `plan quick`. Both accept `--assigned-to`; neither requires it when the session identifies one agent.

### `--content` Format

A quick log's body **scales to whether you attach a completion report**, so the same work is never described twice. Decide ownership first:

| You attach a report (`--report-file`)? | `--content` carries                                 | Why                                                                                                                                                   |
| -------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Yes** (the common path)              | `## TL;DR` only — the intent/scope, one anchor line | The report owns _what changed_ (`## Summary`) and _how it was verified_ (`## Verification`). Restating them in the plan duplicates the report's SSOT. |
| **No**                                 | The full three sections below                       | With no report attached, the plan body is the **only** record of the work, so it must carry the work and verification itself.                         |

**With a report (minimal anchor):**

```markdown
## TL;DR

<!-- 1-2 sentence intent: what this work was and why. The report carries the detail. -->
```

**Without a report (sole record):**

```markdown
## TL;DR

<!-- 1-2 sentence summary -->

## Work Performed

- <!-- changed files / description -->

## Verification Results

- <!-- build/test pass status -->
```

> Ownership split: the **quick-log plan** = _why/what_ (intent) + the anchor the report links to; the **completion report** = _what changed + how verified + risks/follow-ups + conventions_ (the outcome SSOT). When a report is attached, do not repeat Work Performed / Verification in `--content`.

### Completion Report

`completion-report-guide.md` is the SSOT for the report flags (report status, quality score, review recommendation) and for the quick-log-with-report path. Commit your work first — report-attaching `plan quick` auto-collects commit metrics from the current git state.

## Origin Issue Linking

When creating a plan based on an external issue (GitHub, GitLab, Bitbucket, Linear), link the origin issue so the platform can track the relationship and sync status.

**Supported providers:** `LINEAR`, `GITHUB`, `GITLAB`, `BITBUCKET`

**Preferred method — Explicit link after creation:**

```bash
# Linear
agentteams plan link-issue --id {planId} --provider LINEAR \
  --external-id <issueUuid> --external-url <issueUrl> --title "{issueTitle}"

# GitHub
agentteams plan link-issue --id {planId} --provider GITHUB \
  --external-id <owner/repo#number> --external-url <issueUrl> --title "{issueTitle}"

# GitLab
agentteams plan link-issue --id {planId} --provider GITLAB \
  --external-id <projectPath#iid> --external-url <issueUrl> --title "{issueTitle}"

# Bitbucket
agentteams plan link-issue --id {planId} --provider BITBUCKET \
  --external-id <workspace/repo#id> --external-url <issueUrl> --title "{issueTitle}"
```

Repeat `agentteams plan link-issue` for multiple issues.

**Fallback — Entity reference in plan content:**

If the plan body includes an issue entity reference (e.g., `[Issue title](LINEAR_ISSUE:uuid)`, `[Issue title](GITHUB_ISSUE:owner/repo#number)`, `[Issue title](BITBUCKET_ISSUE:workspace/repo#id)`), the server automatically extracts and links it on creation. (Pull-request references such as `GITHUB_PR`/`BITBUCKET_PR` are resolved to URLs but are not extracted as origin issues.)

**Duplicate handling:** If the same issue is linked multiple times (via manual command or content extraction), only one record is kept — duplicates are silently ignored.

## Plan Dependencies

When creating multiple plans at once, use dependencies to define execution order. A blocked plan should not start until its blocking plans are completed.

```bash
# Add dependency (blockedPlan waits for blockingPlan to finish)
agentteams dependency create --plan-id {blockedPlanId} --blocking-plan-id {blockingPlanId}

# List dependencies for a plan
agentteams dependency list --plan-id {planId}

# Remove a dependency
agentteams dependency delete --plan-id {planId} --dep-id {depId}
```

Typical flow: create all plans first, then link dependencies using the returned IDs.

## References

- `plan-authoring-guide.md` — writing or re-scoping a plan body
- `completion-report-guide.md` — report flags, quality score, review recommendation, and the linked-document decision
