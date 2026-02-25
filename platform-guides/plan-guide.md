# Plan Guide (AgentTeams)

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
4. **Register** — `agentteams plan create --file {path} --type {type} --priority {level}`

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
  --report-title "<what you did and why, in one sentence>" \
  --report-file .agentteams/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100, see Quality Score dimensions> \
  --report-status <COMPLETED | PARTIAL | FAILED>

# Finish and include a minimal completion report template
agentteams plan finish --id {planId} --report-template minimal
~~~

## Plan Tiers — Pick the Right Level

Not every plan needs the same structure. Pick the tier that matches your task size.

### Minimal (1 task, 1–2 files, <30 min)

- `## TL;DR` — 1–2 sentence summary, deliverables
- `## TODOs` — What to do + Acceptance Criteria per task

### Standard (2–3 tasks, known scope)

Everything in Minimal, plus:

- `## Context` — Original Request / Research Findings
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
