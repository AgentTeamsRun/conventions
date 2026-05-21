# Code Review Guide (AgentTeams)

Code review is an independent verification workflow. It is not an automatic byproduct of plan completion, completion reports, co-actions, or post-mortems.

## When to Create a Review

Create a code review only when a user explicitly requests one, runs a dedicated code-review command, or accepts a review CTA after a risky change.

Recommend a review when a completed change includes one or more of these signals:

- Cross-workspace or shared-contract changes
- Database schema, migration, authentication, permission, billing, quota, or deployment logic changes
- Large diffs, broad refactors, or changes touching critical user workflows
- Failed or skipped verification that still needs independent inspection

Do not create a code review record merely because a plan was finished or a completion report was written.

## Reviewer Context

Prefer an independent reviewer context:

- Use a different session, agent, or fresh context when possible.
- Include the source plan, completion report, commit range, branch, diff summary, and test results as inputs.
- Avoid making the original implementation session the default reviewer because it may preserve the same assumptions and miss design or regression risks.

The review request should clearly separate:

- Original work context: plan/report/commit/diff/test evidence
- Reviewer execution context: reviewer agent/session/model and review instructions

## Creating a Review Record

Use the dedicated CLI command to register a completed review. The runner and model that performed the review are required execution-environment snapshots.

```bash
agentteams code-review create \
  --title "<review title>" \
  --target-type LOCAL_DIFF \
  --target-ref "<branch, PR, MR, or commit range>" \
  --diff-file .agentteams/cli/temp/<review-diff-summary>.md \
  --test-file .agentteams/cli/temp/<review-test-summary>.md \
  --reviewer-context "<review instructions and source context>" \
  --runner-type <runner-type> \
  --model <model-id>
```

Do not use reviewer names, agent names, or author names as substitutes for `--runner-type` or `--model`. If either value is unknown, stop and ask for the correct execution environment before creating the record.

## Severity

Use these severities consistently:

- `P0`: Must fix before merge or release. Data loss, security issue, irreversible operation, broken core flow, or severe production risk.
- `P1`: Should fix before merge. Likely regression, incorrect behavior, permission gap, migration risk, or missing required validation.
- `P2`: Fix soon. Maintainability issue, missing edge case, incomplete test coverage, confusing API or UI behavior.
- `P3`: Optional improvement. Naming, small readability issue, non-blocking cleanup, or polish.

## Finding Format

Each finding must include:

- Severity: `P0`, `P1`, `P2`, or `P3`
- File path and line range when available
- Short title
- Problem: what is wrong
- Impact: why it matters
- Suggestion: concrete fix direction

Prefer actionable findings over broad commentary. Do not include items that cannot be verified from the diff, repository context, or stated requirements.

## Creating Plans From Findings

Finding-to-plan conversion is human-in-the-loop:

- The user must choose which findings become a plan.
- Do not automatically include every finding.
- The generated plan must preserve severity, file path, problem, impact, and suggestion for each selected finding.
- The generated plan should be scoped to fixing the selected findings, not reopening the entire original implementation.

## Completion Flow

After a completion report, review whether a code review should be recommended. If it is recommended, present it as a separate explicit action. Keep co-action and post-mortem decisions separate from code review decisions.
