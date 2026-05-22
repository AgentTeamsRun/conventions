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
  --findings-file .agentteams/cli/temp/<review-findings>.json \
  --runner-type <runner-type> \
  --model <model-id>
```

Do not use reviewer names, agent names, or author names as substitutes for `--runner-type` or `--model`. If either value is unknown, stop and ask for the correct execution environment before creating the record.

Review status is decided by whether the reviewer has produced a result:

- Omit `--findings-file`: review is registered as `PENDING`. Use only when the reviewer agent has not yet produced findings.
- `--findings-file` pointing at an empty array (`[]`): review is registered as `COMPLETED` with `findingCount: 0` and a `completedAt` timestamp. Use this when the reviewer ran and explicitly found no issues.
- `--findings-file` with a non-empty array: review is registered as `COMPLETED` with the findings attached.

Do not omit `--findings-file` to represent a "no issues found" result — that incorrectly leaves the review in `PENDING`.

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

### Findings JSON File Format

`--findings-file` expects a JSON array. Each item must include the six required fields below; `lineStart` / `lineEnd` are optional.

```json
[
  {
    "severity": "P1",
    "title": "Missing permission check",
    "filePath": "api/src/routes/example.ts",
    "lineStart": 42,
    "lineEnd": 45,
    "problem": "The route accepts project data without checking membership.",
    "impact": "A member could access another project's data.",
    "suggestion": "Call requireProjectMemberAccess before the handler."
  }
]
```

Required fields per item: `severity`, `title`, `filePath`, `problem`, `impact`, `suggestion`. The CLI rejects the file with a clear error when any required field is missing or the top-level value is not an array.

## Creating Plans From Findings

Finding-to-plan conversion is human-in-the-loop:

- The user must choose which findings become a plan.
- Do not automatically include every finding.
- The generated plan must preserve severity, file path, problem, impact, and suggestion for each selected finding.
- The generated plan should be scoped to fixing the selected findings, not reopening the entire original implementation.

## Completion Flow

After a completion report, review whether a code review should be recommended. If it is recommended, present it as a separate explicit action. Keep co-action and post-mortem decisions separate from code review decisions.
