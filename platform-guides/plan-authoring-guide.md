# Plan Authoring Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide defines how to **write** a high-quality tracked plan: what a plan is, how to judge its complexity, how to structure the body, and how to register it. Everything that happens **after** the plan exists — running it, tracking tasks, finishing it — lives in `plan-execution-guide.md`. You do not need that guide to write a plan.

Project conventions remain authoritative for repository-specific workflow; this guide defines the AgentTeams plan authoring contract.

## What a Plan Is

A tracked unit of work (type, status, priority) with comments, assignment, and status transitions. Use a plan when the work spans multiple steps or needs review/verification. The **type** classifies the work:

- `FEATURE` — New functionality or capability
- `BUG_FIX` — Fix for a defect or unexpected behavior
- `ISSUE` — Investigation or issue resolution
- `REFACTOR` — Code restructuring without behavior change
- `CHORE` — Maintenance, config, docs, or other housekeeping

## Plan Writing Workflow

1. **Clarify requirements** — explore the codebase, interview the requester if needed
2. **Write plan body** — judge the plan's **complexity** (see Plan Complexity below) and copy the template for that tier
3. **Gap analysis** — SHOULD run a plan-review/gap-analysis pass; use the self-check below if unavailable
4. **Register** — `agentteams plan create --title "{title}" --file {path} --type {type} --complexity {MINIMAL|STANDARD|FULL} --priority {level} --runner-type {runner-type} --model {model-id}`
5. **Link dependencies** — when you create several plans and one must complete before another, link them after creation. The `dependency` commands are in `plan-execution-guide.md` (**Plan Dependencies**).

> Recording work you already finished, with no plan for it yet? That is a **quick log** (`plan quick`), not a plan draft — see `plan-execution-guide.md` (**Quick Log**).

## Grounding: Evidence Over Memory

A plan is **specific to this repository** — and every project-specific claim in it must come from **evidence you gathered in this repo**, not from training priors, another project, or a prior session's memory. Assumed specifics are the plan-writing equivalent of the leaks platform guides forbid: they read as authoritative but silently mismatch the real project, and the executing agent acts on them.

- **Verify before you assert**: stack, build/test/lint commands, file paths, symbol/function names, and config must be confirmed from the source — read the manifest and its scripts, open the file, run the command's `--help` — before you state them as fact.
- **Do not launder memory into fact**: if a detail comes from "projects like this usually…", it is an assumption, not a finding. Keep it out of the body's factual claims.
- **Separate, don't blend**: anything you could not verify goes in `### Assumptions & Unknowns` (see below), never mixed into the body as if confirmed.
- Record what you actually consulted in `### Research Findings` and `### Conventions Referenced` — do not guess these.

> Specificity is still the goal — a vague, "neutral" plan is the failure mode the QA examples in `plan-template-full.md` warn against. Be concrete **about this repo**; be honest about what you have not confirmed.

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

The tier determines how much structure the plan body needs. Each tier has its own copyable template — take that one and fill it in, rather than copying a larger template and deleting sections.

| Tier       | Template                    | Body carries                                                                                                                                                                                                                                      |
| ---------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MINIMAL`  | `plan-template-minimal.md`  | `## TL;DR` (In Plain Terms line + summary + deliverables) · `## Conventions Referenced` · `## TODOs` (What to do + Acceptance Criteria per task)                                                                                                  |
| `STANDARD` | `plan-template-standard.md` | Everything in MINIMAL, plus `## Context` (Original Request / Research Findings / Assumptions & Unknowns / Conventions Referenced) · `## Work Objectives` · `## Verification Strategy` · TODOs add Must NOT do / References / Required Conventions |
| `FULL`     | `plan-template-full.md`     | Everything in STANDARD, plus Context adds Interview Summary / Gap-Analysis Review · `## Execution Strategy` (Parallel Waves / Diagrams) · `## Final Verification Wave` · TODOs add Agent Profile / Parallelization / QA Scenarios / Commit        |

`### Conventions Referenced` — the project convention files you consulted while planning — is **required at every tier**. Do not guess. Place it under Context, or top-level when Context is omitted.

`### Assumptions & Unknowns` — project-specific claims you could **not** verify against this repo (see Grounding above), kept separate from the body's confirmed facts. Required at STANDARD/FULL whenever any such claim exists; write `none` if everything was verified. At MINIMAL (no Context section), note an unverified assumption inline in the TODO it affects rather than stating it as fact.

### TL;DR Audience (every tier)

`## TL;DR` opens with an **In Plain Terms** line — the one part of the plan a non-expert (requester, stakeholder) reads to confirm the work matches their intent. Write it in everyday language in the plan's own language: state the problem and what visibly changes for the user. Do **not** put file paths, code identifiers, or internal abbreviations (e.g. `SSOT`, `projectId`) in it. Everything below the TL;DR — Context, TODOs, verification — is the work order for the executing agent and may be as technical and detailed as the task requires. Keep the easy summary and the technical detail separate, not blended.

### Immutability & Change History

- `estimatedComplexity` is set once at creation and is **never changed** by updates — it preserves the original AI judgment for later comparison.
- `complexity` can be changed via `plan update --complexity <tier> [--complexity-reason "<why>"]`. When it changes, the server **automatically** records a `MODIFICATION` comment (`complexity: A→B · reason: …`) on every path (CLI or web) — you do not create this comment yourself.

### Raising Complexity → Re-investigation Loop

When a user judges the scope is larger than the plan assumes, they raise `complexity` (typically MINIMAL/STANDARD → FULL). This is the signal to **investigate more deeply and rewrite the plan body at the higher tier**, using that tier's template. The plan's status is unchanged by a complexity change — it is not a new plan, just a re-scoped one.

## Structured Task Authoring Contract

V2 task rows are parsed from the plan body. The following markup is a machine-readable contract, not merely a visual recommendation:

- Put tasks under the exact `## TODOs` heading.
- Start each task with a numbered third-level heading in the form `### N. Task title`. Keep task numbers unique and sequential.
- Express a dependency as `Blocked By: Task N` or `Depends On: Task N`. Use `Blocks: Task N` only on the blocking task.
- Express an execution wave as `Parallel Group: Wave N`.
- Reference only task numbers that exist in the same plan. Self-dependencies and unknown task numbers do not create usable dependency links.
- Treat each task's Acceptance Criteria as the evidence required before that task can be marked `DONE`.

The parser derives task rows, dependency links, and waves from these labels. Changing the headings or inventing equivalent labels can leave the plan without the intended structured task metadata.

## Task Required Elements

FULL tier requires all items. STANDARD tier requires ★ items only. MINIMAL needs ★ What to do + ★ Acceptance Criteria.

- ★ What to do / Must NOT do
- Recommended Agent Profile (category + skills + reason)
- Parallelization (Wave / Blocks / Blocked By)
- ★ References (Pattern / API / External)
- ★ Acceptance Criteria
- QA Scenarios (Tool / Steps / Expected Result)
- Commit (message + files + pre-commit)

## Test-First Planning

Bake verification into the plan at writing time, not as a trailing afterthought:

- Write each task's **Acceptance Criteria as verifiable tests** — what you would assert to prove the task is done. A criterion that cannot be expressed as a check is underspecified.
- For `BUG_FIX` plans, make the first task a **failing test that reproduces the bug**; a later task makes it pass. This proves both that the bug existed and that it is gone.
- For `FEATURE` work, plan the test in the **same task** as the code it covers — not as a separate trailing "write tests" task that gets dropped under time pressure.
- The connected project's testing convention is the source of truth for framework, file location, and run command. This guide governs _that_ you plan tests, not _how_ they are written.

## Verification Expectations

Plan the verification the work will actually need:

- If the plan touches API code: API typecheck + tests.
- If the plan touches CLI code: CLI tests.
- If the plan introduces a new endpoint: at least one request-level test.
- If the plan changes a template: update its tests to match.

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

## Runner Type & Model Reference

`--runner-type` and `--model` are **required** for `plan create`, `plan quick`, `report create`, `code-review create`, and report-attaching `plan finish` — the creator snapshot (`Plan.*`) at create, the executor snapshot (`CompletionReport.*`) at report; the two can differ. Lifecycle commands that only move status, and a report-less `plan finish`, do not take them. Always required for `plan quick`, whether or not it attaches a report.

**Inside a runner session you do not pass either flag.** The runner exports the execution snapshot to every session it spawns, and the CLI fills both in from it. Pass them explicitly only to override, or when running outside a runner session (a local terminal, a manual desktop run) — there is nothing there that knows which model you are on, so the commands above still fail without them.

`--runner-type` takes one of the engine identifiers the platform currently supports. **This guide does not list them.** A copy of that list goes stale the moment an engine is added, renamed, or retired, and a stale value is rejected at create time. Read the accepted values from the command itself — `agentteams plan create --help` prints them for `--runner-type` — and let the platform's runner-type constant stay the single source of truth.

`--model` accepts any model ID string used by the runner engine (e.g., `claude-opus-4-6`, `o3`).

## Common Pitfalls

- Stating stack, commands, or file paths from memory instead of verifying them against this repo
- Blending unverified assumptions into the body as confirmed facts instead of isolating them in `### Assumptions & Unknowns`
- Copying the FULL template for a one-task plan, then leaving empty scaffolding behind
- Skipping tests because changes "look small"
- Changing API contracts without updating schemas/tests
- Writing files to project-specific directories when they should be platform-wide
- Mixing platform content with project conventions (keep them separate)
- Scope creep beyond task spec

## References

- `plan-template-minimal.md` · `plan-template-standard.md` · `plan-template-full.md` — copyable bodies, one per complexity tier
- `plan-execution-guide.md` — running a registered plan, quick logs, origin issue links, plan dependencies
- `completion-report-guide.md` — report flags and the quality-score / review-recommendation semantics
