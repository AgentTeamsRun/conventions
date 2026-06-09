# Code Review Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

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

Prefer an **independent** reviewer — a different session, agent, or fresh context. The original implementation session preserves the same assumptions and misses design/regression risks.

Separate the two contexts clearly in the request:

- Original work: plan / report / commit range / branch / diff summary / test results
- Reviewer execution: reviewer agent / session / model + review instructions

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

## Updating a PENDING Review

Use `code-review update` only to correct metadata on a review that is still `PENDING`. Once a review result has been submitted or findings exist, update the review through the result/finding lifecycle instead.

When executing an existing `PENDING` review, inspect the review record before reviewing the diff. If the review input context is incomplete, update it before calling `submit-result`. A completed review should preserve both sides of the workflow:

- Original work: target ref, source/base branch or commit range, diff summary, and test summary
- Reviewer execution: reviewer context/instructions, runner type, and model

Before `submit-result`, confirm these fields are recorded when the information is available:

- `targetRef` or source commit range
- `sourceBranchName` and `baseBranchName` for branch diffs
- `diffSummary`
- `testSummary`
- `reviewerContext`
- `runnerType`
- `model`

If any required context cannot be recovered or the CLI cannot update it, mention that limitation in `resultSummary` before submitting the result.

```bash
agentteams code-review update \
  --id <code-review-id> \
  --title "<corrected review title>" \
  --target-type BRANCH_DIFF \
  --target-ref "<branch, PR, MR, or commit range>" \
  --diff-file .agentteams/cli/temp/<updated-review-diff-summary>.md \
  --test-file .agentteams/cli/temp/<updated-review-test-summary>.md \
  --reviewer-context "<updated review instructions and source context>" \
  --runner-type <runner-type> \
  --model <model-id>
```

Editable fields are `title`, `targetType`, `targetRef`, `sourceCommitStart`, `sourceCommitEnd`, `sourceBranchName`, `baseBranchName`, `diffSummary`, `testSummary`, `reviewerContext`, `recommendationReason`, `runnerType`, and `model`. Pass only the fields that need to change.

Do not use `code-review update` for `findings`, `status`, `resultSummary`, `errorMessage`, source links, repository links, or creator metadata. Use `code-review submit-result` for results and the finding commands for finding state changes.

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

## Diagrams (Mermaid)

A fenced ```mermaid``` block (`flowchart` / `sequenceDiagram`) renders in the web viewer; it stays plain text in CLI/raw. Use one when it explains a regression path or control flow faster than words.

## Creating Plans From Findings

Finding-to-plan conversion is **human-in-the-loop** — the user chooses which findings become a plan; do not auto-include every finding. The generated plan preserves each selected finding's severity / file path / problem / impact / suggestion, and is scoped to fixing those findings, not reopening the original implementation.

## Completion Flow

After a completion report, review whether a code review should be recommended. If it is recommended, present it as a separate explicit action. Keep co-action and post-mortem decisions separate from code review decisions.
