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
2. **Write plan body** — judge the plan's **complexity** (see Plan Complexity below) and follow the structure for that tier
3. **Gap analysis** — SHOULD run Metis review; use the self-check below if unavailable
4. **Register** — `agentteams plan create --file {path} --type {type} --complexity {MINIMAL|STANDARD|FULL} --priority {level} --runner-type {runner-type} --model {model-id}` — an HTML preview is also required (see Plan Complexity → Preview)
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
| `ANTIGRAVITY` | Google Antigravity CLI |
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

1. **ID prefix stripping (IMPORTANT)**: The `id` part may include a type prefix. Always strip this prefix before passing the id to any CLI flag (`--id`, `--plan-id`, etc.).
   - Example: `[My Plan](plan:agentteams_pln_f62762fc-730a-4201-8586-e2541505ed1b)` → use `f62762fc-730a-4201-8586-e2541505ed1b`
   - Canonical prefix list: `agentteams_pln_` (plan) · `agentteams_rpt_` (completionReport) · `agentteams_rev_` (codeReview) · `agentteams_act_` (coAction) · `agentteams_cnv_` (convention) · `agentteams_pmt_` (postMortem) · `agentteams_doc_` (document)
2. Resolution by type:
   - `convention:id:.agentteams/path` → Read the local file at the given path (e.g., `.agentteams/rules/context.md`)
   - `completionReport:id` → Download with `agentteams report download --id {id}` and read the local file
   - `postMortem:id` → Download with `agentteams postmortem download --id {id}` and read the local file
   - `coAction:id` → Download with `agentteams coaction download --id {id}` and read the local file
   - `codeReview:id` → Fetch the review record with `agentteams code-review get --id {id}` and use the response as context
   - `document:id` → Download with `agentteams document download --id {id}` and read the local file

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

## Plan Complexity — A Stored Field, Not Just a Document Concept

Every plan carries a **complexity** tier (`MINIMAL` / `STANDARD` / `FULL`) that is **stored on the plan in the database**, not merely implied by how the body is written. You set it at creation time with `--complexity`; the server records it as both `estimatedComplexity` (the immutable snapshot of your first judgment) and `complexity` (the effective, adjustable value). The stored value drives plan simplification (which preview template is used) and the user-triggered re-investigation loop, so judging it honestly matters.

### Judging the Tier

| Tier | When |
|---|---|
| `MINIMAL` | 1 task · 1–2 files · single domain · no risk signals |
| `STANDARD` | 2–3 tasks · known, bounded scope |
| `FULL` | 4+ tasks · multi-wave, **or** any risk signal: schema / auth / billing / quota / deployment change · cross-workspace edits · large diff · unfamiliar domain |

> When unsure between two tiers, pick the higher one. Under-scoping a FULL plan as MINIMAL is the failure mode this field exists to prevent.

### Body Structure per Tier

The tier determines how much structure the plan body needs.

- **MINIMAL** — `## TL;DR` (1–2 sentence summary + deliverables) · `## TODOs` (What to do + Acceptance Criteria per task)
- **STANDARD** — everything in MINIMAL, plus `## Context` (Original Request / Research Findings) · `## Work Objectives` (Deliverables / Definition of Done / Must Have / Must NOT Have) · `## Verification Strategy` (QA tool mapping: API→typecheck+test, CLI→test, Web→build) · TODOs add Must NOT do / References
- **FULL** — everything in STANDARD, plus Context adds Interview Summary / Metis Review · `## Execution Strategy` (Parallel Waves / Dependency Matrix / Agent Dispatch) · TODOs add Agent Profile / Parallelization / QA Scenarios / Commit plan

`### Conventions Referenced` — `.agentteams/rules/*.md` files you consulted while planning — is **required at every tier**. Do not guess. Place it under Context, or top-level when Context is omitted.

> `plan-template.md` provides a copyable FULL-tier template. For MINIMAL/STANDARD, extract only the sections you need.

### Preview — Complexity Selects the Template

The HTML preview is **mandatory at every tier** (there is no escape hatch), but the complexity selects which template to author:

| Complexity | Preview template |
|---|---|
| `MINIMAL` / `STANDARD` | **lite** — see `plan-preview-lite-guide.md` (title + TL;DR + changes + verification) |
| `FULL` | **rich** — see `plan-preview-guide.md` (dashboard: metric grid, wave-flow, DoD progress) |

The preview is always authored by an AI agent and uploaded in the same `plan create` / preview-affecting `plan update` command.

### Immutability & Change History

- `estimatedComplexity` is set once at creation and is **never changed** by updates — it preserves the original AI judgment for later comparison.
- `complexity` can be changed via `plan update --complexity <tier> [--complexity-reason "<why>"]`. When it changes, the server **automatically** records a `MODIFICATION` comment (`complexity: A→B · reason: …`) on every path (CLI or web) — you do not create this comment yourself.

### Raising Complexity → Re-investigation Loop

When a user judges the scope is larger than the plan assumes, they raise `complexity` (typically MINIMAL/STANDARD → FULL). This is the signal to **investigate more deeply and rewrite the plan body at the higher tier** (and regenerate the rich preview). The plan's status is unchanged by a complexity change — it is not a new plan, just a re-scoped one.

## Task Required Elements

FULL tier requires all items. STANDARD tier requires ★ items only. MINIMAL needs ★ What to do + ★ Acceptance Criteria.

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

## Test-First Planning

Bake verification into the plan at writing time, not as a trailing afterthought:

- Write each task's **Acceptance Criteria as verifiable tests** — what you would assert to prove the task is done. A criterion that cannot be expressed as a check is underspecified.
- For `BUG_FIX` plans, make the first task a **failing test that reproduces the bug**; a later task makes it pass. This proves both that the bug existed and that it is gone.
- For `FEATURE` work, plan the test in the **same task** as the code it covers — not as a separate trailing "write tests" task that gets dropped under time pressure.
- The connected project's testing convention is the source of truth for framework, file location, and run command. This guide governs *that* you plan tests, not *how* they are written.

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

- `plan-template.md` — copyable FULL-tier plan template
- `plan-preview-guide.md` — rich HTML preview (FULL plans)
- `plan-preview-lite-guide.md` — lite HTML preview (MINIMAL/STANDARD plans)
