# Plan Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines how to **author, execute, and complete** a high-quality tracked plan. Project conventions remain authoritative for repository-specific workflow, while this guide defines the AgentTeams plan and task lifecycle.

## What a Plan Is

A tracked unit of work (type, status, priority) with comments, assignment, and status transitions. Use a plan when the work spans multiple steps or needs review/verification. The **type** classifies the work:

- `FEATURE` — New functionality or capability
- `BUG_FIX` — Fix for a defect or unexpected behavior
- `ISSUE` — Investigation or issue resolution
- `REFACTOR` — Code restructuring without behavior change
- `CHORE` — Maintenance, config, docs, or other housekeeping

## Plan Writing Workflow

1. **Clarify requirements** — explore the codebase, interview the requester if needed
2. **Write plan body** — judge the plan's **complexity** (see Plan Complexity below) and follow the structure for that tier
3. **Gap analysis** — SHOULD run a plan-review/gap-analysis pass; use the self-check below if unavailable
4. **Register** — `agentteams plan create --title "{title}" --file {path} --type {type} --complexity {MINIMAL|STANDARD|FULL} --priority {level} --runner-type {runner-type} --model {model-id}`
5. **Link dependencies** — when creating multiple plans where one must finish before another starts, link them after creation (see Plan Dependencies below).

## Grounding: Evidence Over Memory

A plan is **specific to this repository** — and every project-specific claim in it must come from **evidence you gathered in this repo**, not from training priors, another project, or a prior session's memory. Assumed specifics are the plan-writing equivalent of the leaks platform guides forbid: they read as authoritative but silently mismatch the real project, and the executing agent acts on them.

- **Verify before you assert**: stack, build/test/lint commands, file paths, symbol/function names, and config must be confirmed from the source — read the manifest and its scripts, open the file, run the command's `--help` — before you state them as fact.
- **Do not launder memory into fact**: if a detail comes from "projects like this usually…", it is an assumption, not a finding. Keep it out of the body's factual claims.
- **Separate, don't blend**: anything you could not verify goes in `### Assumptions & Unknowns` (see below), never mixed into the body as if confirmed.
- Record what you actually consulted in `### Research Findings` and `### Conventions Referenced` — do not guess these.

> Specificity is still the goal — a vague, "neutral" plan is the failure mode the QA examples in `plan-template.md` warn against. Be concrete **about this repo**; be honest about what you have not confirmed.

## Execution Shortcuts

For standard execution flows, use lifecycle shortcuts instead of manual multi-step status updates.

```bash
agentteams plan start  --id {planId}
agentteams plan finish --id {planId}   # add report flags to attach a completion report
```

For one-shot tasks, `plan quick` collapses create + start + finish into a single command — see **Quick Log** below.

> Commit your work before finishing — both `plan finish` and report-attaching `plan quick` auto-collect commit metrics from the current git state.
> The full report-attaching parameters (report flags + quality-score / report-status semantics) are defined in `completion-report-guide.md` as the SSOT.

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

`--content` carries the plan body (format below). Adding report flags attaches a completion report in the same command.

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

## Runner Type & Model Reference

`--runner-type` and `--model` are **required** for `plan create`, `plan quick`, `report create`, `code-review create`, and report-attaching `plan finish` — the creator snapshot (`Plan.*`) at create, the executor snapshot (`CompletionReport.*`) at report; the two can differ. Not required for `plan start` or report-less `plan finish`. Always required for `plan quick`, whether or not it attaches a report.

**Inside a runner session you do not pass either flag.** The runner exports the execution snapshot to every session it spawns, and the CLI fills both in from it. Pass them explicitly only to override, or when running outside a runner session (a local terminal, a manual desktop run) — there is nothing there that knows which model you are on, so the commands above still fail without them.

| Runner Type   | Description            |
| ------------- | ---------------------- |
| `OPENCODE`    | OpenCode               |
| `CLAUDE_CODE` | Claude Code CLI        |
| `CODEX`       | OpenAI Codex CLI       |
| `ANTIGRAVITY` | Google Antigravity CLI |
| `AMP`         | Amp Code               |
| `COPILOT_CLI` | GitHub Copilot CLI     |
| `CURSOR_CLI`  | Cursor CLI             |
| `KIMI_CLI`    | Moonshot Kimi Code CLI |
| `KIRO_CLI`    | Kiro CLI               |

`--model` accepts any model ID string used by the runner engine (e.g., `claude-opus-4-6`, `o3`).

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

## Entity Reference Resolution

Plan bodies may contain `[label](type:id)` / `[label](type:id:path)` references. Resolve them per the always-on convention's **Entity References & ID Handling** rules — pass the reference token as-is to the `agentteams_resolve` tool if your tool list has it, or to `agentteams resolve` otherwise, and act on the returned `kind`. No plan-specific rules beyond those.

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

## Structured Task Authoring Contract

V2 task rows are parsed from the plan body. The following markup is a machine-readable contract, not merely a visual recommendation:

- Put tasks under the exact `## TODOs` heading.
- Start each task with a numbered third-level heading in the form `### N. Task title`. Keep task numbers unique and sequential.
- Express a dependency as `Blocked By: Task N` or `Depends On: Task N`. Use `Blocks: Task N` only on the blocking task.
- Express an execution wave as `Parallel Group: Wave N`.
- Reference only task numbers that exist in the same plan. Self-dependencies and unknown task numbers do not create usable dependency links.
- Treat each task's Acceptance Criteria as the evidence required before that task can be marked `DONE`.

The parser derives task rows, dependency links, and waves from these labels. Changing the headings or inventing equivalent labels can leave the plan without the intended structured task metadata.

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

## Plan Complexity — A Stored Field, Not Just a Document Concept

Every plan carries a **complexity** tier (`MINIMAL` / `STANDARD` / `FULL`) that is **stored on the plan in the database**, not merely implied by how the body is written. You set it at creation time with `--complexity`; the server records it as both `estimatedComplexity` (the immutable snapshot of your first judgment) and `complexity` (the effective, adjustable value). The stored value drives plan simplification and the user-triggered re-investigation loop, so judging it honestly matters.

### Judging the Tier

| Tier       | When                                                                                                                                                        |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MINIMAL`  | 1 task · 1–2 files · single domain · no risk signals                                                                                                        |
| `STANDARD` | 2–3 tasks · known, bounded scope                                                                                                                            |
| `FULL`     | 4+ tasks · multi-wave, **or** any risk signal: schema / auth / billing / quota / deployment change · cross-workspace edits · large diff · unfamiliar domain |

> When unsure between two tiers, pick the higher one. Under-scoping a FULL plan as MINIMAL is the failure mode this field exists to prevent.

### Body Structure per Tier

The tier determines how much structure the plan body needs.

- **MINIMAL** — `## TL;DR` (In Plain Terms line + 1–2 sentence summary + deliverables) · `## TODOs` (What to do + Acceptance Criteria per task)
- **STANDARD** — everything in MINIMAL, plus `## Context` (Original Request / Research Findings / Assumptions & Unknowns) · `## Work Objectives` (Deliverables / Definition of Done / Must Have / Must NOT Have) · `## Verification Strategy` (QA tool mapping per the project's stack) · TODOs add Must NOT do / References
- **FULL** — everything in STANDARD, plus Context adds Interview Summary / Gap-Analysis Review · `## Execution Strategy` (Parallel Waves / Dependency Matrix / Agent Dispatch) · TODOs add Agent Profile / Parallelization / QA Scenarios / Commit plan

`### Conventions Referenced` — `.agentteams/rules/*.md` files you consulted while planning — is **required at every tier**. Do not guess. Place it under Context, or top-level when Context is omitted.

`### Assumptions & Unknowns` — project-specific claims you could **not** verify against this repo (see Grounding above), kept separate from the body's confirmed facts. Required at STANDARD/FULL whenever any such claim exists; write `none` if everything was verified. At MINIMAL (no Context section), note an unverified assumption inline in the TODO it affects rather than stating it as fact.

> `plan-template.md` provides a copyable FULL-tier template. For MINIMAL/STANDARD, extract only the sections you need.

### TL;DR Audience (every tier)

`## TL;DR` opens with an **In Plain Terms** line — the one part of the plan a non-expert (requester, stakeholder) reads to confirm the work matches their intent. Write it in everyday language in the plan's own language: state the problem and what visibly changes for the user. Do **not** put file paths, code identifiers, or internal abbreviations (e.g. `SSOT`, `projectId`) in it. Everything below the TL;DR — Context, TODOs, verification — is the work order for the executing agent and may be as technical and detailed as the task requires. Keep the easy summary and the technical detail separate, not blended.

### Preview

V2 plans render their structured sections and tasks directly in the web UI; they do **not** render an uploaded HTML preview. Do not generate or upload a preview for a V2 plan.

The HTML preview workflow in `plan-preview-guide.md` remains only for legacy V1 plans that still expose the visualization tab.

### Immutability & Change History

- `estimatedComplexity` is set once at creation and is **never changed** by updates — it preserves the original AI judgment for later comparison.
- `complexity` can be changed via `plan update --complexity <tier> [--complexity-reason "<why>"]`. When it changes, the server **automatically** records a `MODIFICATION` comment (`complexity: A→B · reason: …`) on every path (CLI or web) — you do not create this comment yourself.

### Raising Complexity → Re-investigation Loop

When a user judges the scope is larger than the plan assumes, they raise `complexity` (typically MINIMAL/STANDARD → FULL). This is the signal to **investigate more deeply and rewrite the plan body at the higher tier**. The plan's status is unchanged by a complexity change — it is not a new plan, just a re-scoped one.

## Task Required Elements

FULL tier requires all items. STANDARD tier requires ★ items only. MINIMAL needs ★ What to do + ★ Acceptance Criteria.

- ★ What to do / Must NOT do
- Recommended Agent Profile (category + skills + reason)
- Parallelization (Wave / Blocks / Blocked By)
- ★ References (Pattern / API / External)
- ★ Acceptance Criteria
- QA Scenarios (Tool / Steps / Expected Result)
- Commit (message + files + pre-commit)

## Gap Analysis

SHOULD: Ask a plan-review/gap-analysis agent to review the plan draft before registering.

If no such reviewer is available, self-check:

- [ ] All required sections for the chosen tier present?
- [ ] Must NOT Have guardrails defined?
- [ ] Each TODO has acceptance criteria?
- [ ] Dependency graph correct (no circular blocks)?
- [ ] File references verified to exist?
- [ ] Stack / commands / paths stated as fact were confirmed against this repo, not recalled from memory?
- [ ] Every unverified claim lives in `### Assumptions & Unknowns`, not blended into the body?

## Verification Expectations

- If you touched API code: run API typecheck + tests.
- If you touched CLI code: run CLI tests.
- If you introduced a new endpoint: add at least one request-level test.
- If you changed a template: update its tests to match.

## Test-First Planning

Bake verification into the plan at writing time, not as a trailing afterthought:

- Write each task's **Acceptance Criteria as verifiable tests** — what you would assert to prove the task is done. A criterion that cannot be expressed as a check is underspecified.
- For `BUG_FIX` plans, make the first task a **failing test that reproduces the bug**; a later task makes it pass. This proves both that the bug existed and that it is gone.
- For `FEATURE` work, plan the test in the **same task** as the code it covers — not as a separate trailing "write tests" task that gets dropped under time pressure.
- The connected project's testing convention is the source of truth for framework, file location, and run command. This guide governs _that_ you plan tests, not _how_ they are written.

## Common Pitfalls

- Stating stack, commands, or file paths from memory instead of verifying them against this repo
- Blending unverified assumptions into the body as confirmed facts instead of isolating them in `### Assumptions & Unknowns`
- Skipping tests because changes "look small"
- Changing API contracts without updating schemas/tests
- Writing files to project-specific directories when they should be platform-wide
- Mixing platform content with project conventions (keep them separate)
- Scope creep beyond task spec

## References

- `plan-template.md` — copyable FULL-tier plan template
- `plan-preview-guide.md` — HTML preview for legacy V1 plans only
