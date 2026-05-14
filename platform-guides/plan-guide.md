# Plan Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines how to **write** a high-quality plan. For execution details (status transitions, CLI commands, comments), follow your project's workflow conventions and CLI help.

## What a Plan Is

- A plan is a tracked unit of work with a type, status, and priority.
- Use plans when the work spans multiple steps or requires review and verification.
- Plans support comments, assignment, and status transitions.
- Plans have a **type** that classifies the nature of the work:
  - `FEATURE` — New functionality or capability
  - `BUG_FIX` — Fix for a defect or unexpected behavior
  - `ISSUE` — Investigation or issue resolution
  - `REFACTOR` — Code restructuring without behavior change
  - `CHORE` — Maintenance, config, docs, or other housekeeping

## Plan Writing Workflow

1. **Clarify requirements** — explore the codebase, interview the requester if needed
2. **Write plan body** — follow Plan Tiers below to pick the right structure
3. **Gap analysis** — SHOULD run Metis review; use the self-check below if unavailable
4. **Register** — `agentteams plan create --file {path} --type {type} --priority {level} --runner-type {runner-type} --model {model-id}`
5. **Link dependencies** — when creating multiple plans at once and one plan must finish before another can start, link them after creation:
   ~~~bash
   agentteams dependency create --plan-id {blockedPlanId} --blocking-plan-id {blockingPlanId}
   ~~~

Repository linkage note:

- If .agentteams/config.json contains `repositoryId`, `plan create` links the new plan to that repository automatically.

## Execution Shortcuts

For standard execution flows, use lifecycle shortcuts instead of manual multi-step status updates.

~~~bash
# Start plan lifecycle
agentteams plan start --id {planId}

# Finish plan lifecycle
agentteams plan finish --id {planId}

# Finish and include completion report with metrics
agentteams plan finish --id {planId} \
  --runner-type <runner-type> --model <model-id> \
  --report-title "<what you did and why, in one sentence>" \
  --report-file .agentteams/cli/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100, see Quality Score dimensions> \
  --report-status <COMPLETED | PARTIAL | FAILED>

~~~

## Runner Type & Model Reference

Two snapshots, two sources:

- `Plan.runnerType` / `Plan.model` — **creator** snapshot. Recorded by `plan create` (and `plan quick`). Required at creation time.
- `CompletionReport.runnerType` / `CompletionReport.model` — **executor** snapshot. Recorded by `report create` (and by `plan finish` when generating a report). Required when creating a report.

`--runner-type` and `--model` are therefore **required** for `plan create`, `plan quick`, and `report create`. `plan start` and `plan finish` (without a report) do not accept them.

| Runner Type | Description |
|---|---|
| `CLAUDE_CODE` | Claude Code CLI |
| `CODEX` | OpenAI Codex CLI |
| `GEMINI` | Google Gemini CLI |
| `AMP` | Amp Code |
| `OPENCODE` | OpenCode |

`--model` accepts any model ID string used by the runner engine (e.g., `claude-opus-4-6`, `o3`, `gemini-2.5-pro`).

## Plan Start → Execution Flow

When a user explicitly says to start a plan (e.g. "start plan {id}", "let's start {id}"), treat it as an explicit execution approval. Follow this flow:

~~~bash
# 1. Download runbook
agentteams plan download --id {planId}

# 2. Check for blocking comments
agentteams comment list --plan-id {planId}

# 3. Start lifecycle
agentteams plan start --id {planId}
~~~

**Decision after comment check:**

| Comments found | Action |
|---|---|
| `RISK` or `MODIFICATION` present | Report to user and wait for confirmation before implementing |
| None (or `GENERAL` only) | Proceed with implementation immediately |

The phrase "start the plan" is an explicit approval signal — do not stop after the CLI status change. Implement unless a blocking comment requires human confirmation.

## Entity Reference Resolution

Plans may contain entity references in `[label](type:id)` or `[label](type:id:path)` format. Resolve them as follows:

1. **ID prefix stripping (IMPORTANT)**: The `id` part may include a type prefix such as `plan_`, `cr_`, `ca_`, `conv_`, or `pm_`. Always strip this prefix before passing the id to any CLI flag (`--id`, `--plan-id`, etc.).
   - Example: `[My Plan](plan:plan_f62762fc-730a-4201-8586-e2541505ed1b)` → use `f62762fc-730a-4201-8586-e2541505ed1b`
   - Full prefix list: `plan_` · `cr_` · `ca_` · `conv_` · `pm_`
2. Resolution by type:
   - `convention:id:.agentteams/path` → Read the local file at the given path (e.g., `.agentteams/rules/context.md`)
   - `completionReport:id` → Download with `agentteams report download --id {id}` and read the local file
   - `postMortem:id` → Download with `agentteams postmortem download --id {id}` and read the local file
   - `coAction:id` → Download with `agentteams coaction download --id {id}` and read the local file

## Origin Issue Linking

When creating a plan based on an external issue (GitHub, GitLab, Linear), link the origin issue so the platform can track the relationship and sync status.

**Supported providers:** `LINEAR`, `GITHUB`, `GITLAB`

**Preferred method — CLI flag on creation:**

~~~bash
# Linear
agentteams plan create --file plan.md --type FEATURE --priority HIGH \
  --origin-issue "LINEAR:<issueUuid>:<issueUrl>:<issueTitle>"

# GitHub
agentteams plan create --file plan.md --type BUG_FIX --priority HIGH \
  --origin-issue "GITHUB:<owner/repo#number>:<issueUrl>:<issueTitle>"

# GitLab
agentteams plan create --file plan.md --type ISSUE --priority MEDIUM \
  --origin-issue "GITLAB:<projectPath#iid>:<issueUrl>:<issueTitle>"
~~~

The `--origin-issue` flag is repeatable for multiple issues.

**Fallback 1 — Explicit link after creation:**

~~~bash
agentteams plan issue --id {planId} --provider <LINEAR|GITHUB|GITLAB> \
  --external-id {id} --external-url {url} --title "{title}"
~~~

**Fallback 2 — Entity reference in plan content:**

If the plan body includes an issue entity reference (e.g., `[Issue title](LINEAR_ISSUE:uuid)`, `[Issue title](GITHUB_ISSUE:owner/repo#number)`), the server automatically extracts and links it on creation.

**Duplicate handling:** If the same issue is linked multiple times (via flag, manual command, or content extraction), only one record is kept — duplicates are silently ignored.

## Plan Dependencies

When creating multiple plans at once, use dependencies to define execution order. A blocked plan should not start until its blocking plans are completed.

~~~bash
# Add dependency (blockedPlan waits for blockingPlan to finish)
agentteams dependency create --plan-id {blockedPlanId} --blocking-plan-id {blockingPlanId}

# List dependencies for a plan
agentteams dependency list --plan-id {planId}

# Remove a dependency
agentteams dependency delete --plan-id {planId} --dep-id {depId}
~~~

Typical flow: create all plans first, then link dependencies using the returned IDs.

## During Plan Execution

Post comments to track progress:

- **Risk found**: `agentteams comment create --plan-id {planId} --type RISK --content "<risk description>" --affected-files "<paths>"`
- **Scope changed**: `agentteams comment create --plan-id {planId} --type MODIFICATION --content "<what changed and why>" --affected-files "<paths>"`
- **Status update**: `agentteams comment create --plan-id {planId} --type GENERAL --content "<current progress>"`

## After Completing or Cancelling a Plan

When completing a plan, do not treat the completion report as the final step by itself.

1. Write or generate the completion report.
2. Immediately review whether a Co-Action is needed. Create and link one when the work produced implicit knowledge, durable design decisions, follow-up work, known constraints, or handoff context that cannot be inferred from the code alone.
3. Immediately review whether a Post-Mortem is needed. Create one only when a failure, regression, or unexpected execution issue occurred and the issue is reproducible or systemic, significantly delayed or blocked the work, and can be prevented by a process, tooling, or environment change.
4. Clean up the local runbook after required linked documents have been created or explicitly ruled out.

Clean up the local runbook:

~~~bash
agentteams plan cleanup --id {planId}
~~~

## Plan Tiers — Pick the Right Level

Not every plan needs the same structure. Pick the tier that matches your task size.

### Minimal (1 task, 1–2 files, <30 min)

- `## TL;DR` — 1–2 sentence summary, deliverables
- `## TODOs` — What to do + Acceptance Criteria per task
- `### Conventions Referenced` (under Context, or top-level if Context omitted) — `.agentteams/rules/*.md` files you consulted while planning. Same format as completion reports. Required at every tier — do not guess.

### Standard (2–3 tasks, known scope)

Everything in Minimal, plus:

- `## Context` — Original Request / Research Findings / Conventions Referenced
- `## Work Objectives` — Deliverables / Definition of Done / Must Have / Must NOT Have
- `## Verification Strategy` — QA tool mapping (API→typecheck+test, CLI→test, Web→build)
- TODOs add: Must NOT do / References

### Full (4+ tasks, multi-wave, unfamiliar domain)

Everything in Standard, plus:

- Context adds: Interview Summary / Metis Review
- `## Execution Strategy` — Parallel Waves / Dependency Matrix / Agent Dispatch
- TODOs add: Agent Profile / Parallelization / QA Scenarios / Commit plan

> When unsure, start with Standard. Upgrade to Full when tasks reach 4+.
> `plan-template.md` provides a copyable Full-tier template. For Minimal/Standard, extract only the sections you need.

## Task Required Elements

Full tier requires all items. Standard tier requires ★ items only.

- ★ What to do / Must NOT do
- Recommended Agent Profile (category + skills + reason)
- Parallelization (Wave / Blocks / Blocked By)
- ★ References (Pattern / API / External)
- ★ Acceptance Criteria
- QA Scenarios (Tool / Steps / Expected Result / Evidence)
- Commit (message + files + pre-commit)

## Gap Analysis

SHOULD: Ask the Metis agent to review the plan draft before registering.

If Metis is unavailable, self-check:

- [ ] All required sections for the chosen tier present?
- [ ] Must NOT Have guardrails defined?
- [ ] Each TODO has acceptance criteria?
- [ ] Dependency graph correct (no circular blocks)?
- [ ] File references verified to exist?

## Verification Expectations

- If you touched API code: run API typecheck + tests.
- If you touched CLI code: run CLI tests.
- If you introduced a new endpoint: add at least one request-level test.
- If you changed a template: update its tests to match.

## Common Pitfalls

- Skipping tests because changes "look small"
- Changing API contracts without updating schemas/tests
- Writing files to project-specific directories when they should be platform-wide
- Mixing platform content with project conventions (keep them separate)
- Excessive comments that restate the code
- Scope creep beyond task spec
- Over-abstraction or premature generalization
- Generic names (data, result, item) that obscure intent

## References

- `plan-template.md` — copyable Full-tier plan template
