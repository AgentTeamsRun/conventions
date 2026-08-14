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

Do not use reviewer names, agent names, or author names as substitutes for `--runner-type` or `--model`. Inside a runner session both flags are filled in from the session, so omit them rather than restating what the runner already knows. Outside one, if either value is unknown, stop and ask for the correct execution environment before creating the record — never guess it.

Review status is derived from the review result and finding lifecycle:

- Omit `--findings-file`: the review is registered as `PENDING`. Use this only when the reviewer has not produced a result.
- Empty findings array (`[]`): the review is registered as `COMPLETED` with `findingCount: 0` and a `completedAt` timestamp.
- Non-empty findings array: the review is registered as `OPEN` with the findings attached.
- Some findings resolved or dismissed: the review becomes `PARTIAL`.
- Every finding resolved or dismissed: the review becomes `COMPLETED`.
- Failed review execution: submit the result with status `FAILED`.

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

## Related History Matches

Code review create and detail responses include `historyMatchStatus`; detail responses also include `relatedHistories`. Use these automatically matched plans, completion reports, and post-mortems as context for recurring risks, prior decisions, and useful verification ideas. They are references, not evidence that the current change is correct.

Interpret the status as follows:

- `MATCHED` — one or more related records were found. An empty visible list is still possible when matched records were deleted or are not visible to the current caller.
- `NO_MATCH` — matching ran successfully but found no sufficiently related record.
- `SKIPPED` — there was not enough diff or file-path context to run matching.
- `FAILED` — matching could not complete. The code review operation itself can still succeed.

A completion report's `qualityScore` describes that historical report only. Do not treat it as a score for the current review or as proof that an old approach should be repeated.

Updating `diffSummary` on a `PENDING` review recalculates and replaces its related-history matches. Updating other metadata preserves the existing matches. If matching is `FAILED`, continue the review and use `agentteams search` manually when historical context is important.

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
    "title": "Missing authorization check",
    "filePath": "src/path/to/file.ts",
    "lineStart": 42,
    "lineEnd": 45,
    "problem": "The handler accepts protected data without checking access.",
    "impact": "A user could access data they should not be able to see.",
    "suggestion": "Call the project's authorization guard before using protected data."
  }
]
```

Required fields per item: `severity`, `title`, `filePath`, `problem`, `impact`, `suggestion`. The CLI rejects the file with a clear error when any required field is missing or the top-level value is not an array.

## Finding Lifecycle

Finding status represents remediation progress:

- `OPEN` — requires action.
- `PLANNED` — selected into a generated remediation plan.
- `RESOLVED` — fixed and verified.
- `DISMISSED` — intentionally not fixed because the finding is invalid or explicitly accepted.

Mark a finding `RESOLVED` only after the described problem has been fixed, its focused verification has passed, and the fix still satisfies the source plan's acceptance criteria and conventions. Do not resolve a finding merely because implementation has started. Leave it `OPEN` when the fix or verification is incomplete.

## Fixing Findings From a Plan-Based Review

A review is plan-based when `sourcePlanId` is present, or when `sourceCompletionReportId` refers to a completion report linked to a plan.

For a plan-based review, fix findings directly in the source plan's existing branch or pull request. Do not create a separate remediation plan or call `code-review create-plan`.

### Required Flow

1. Inspect the review and identify its unresolved findings:

   ```bash
   agentteams code-review get --id {codeReviewId}
   ```

2. Read the comments on every finding you are about to fix. `code-review get` does not return them — a finding's conversation is only reachable by its own id:

   ```bash
   agentteams comment list --finding-id {findingId}
   ```

   A finding comment is where a reviewer or teammate narrows the fix: a preferred direction, a correction to the finding's premise, a note that the problem was already handled elsewhere. Reflect it in the fix and in the verification you run. It is not a stop signal — keep working. When a comment establishes that the finding is invalid or explicitly accepted, `agentteams code-review dismiss --id {codeReviewId} --finding-id {findingId}` is the correct outcome, not `resolve`.

3. Continue on the branch or pull request produced by the source plan.
4. Fix one or more related findings.
5. Run focused verification for every finding covered by the change.
6. Immediately resolve each finding whose fix has passed verification:

   ```bash
   # Resolve one verified finding
   agentteams code-review resolve \
     --id {codeReviewId} \
     --finding-id {findingId}

   # Resolve findings covered by the same fix and verification
   agentteams code-review resolve \
     --id {codeReviewId} \
     --finding-ids {findingId1},{findingId2}
   ```

7. Leave any unverified or incomplete finding `OPEN`.
8. Re-run the source plan's relevant final verification before ending the fix run.

Resolve multiple findings together only when the same verified change covers every listed finding. The code review automatically becomes `COMPLETED` when every active finding is `RESOLVED` or `DISMISSED`.

## Attaching Evidence

When a finding's basis is easier to show than to describe, attach the file to the review so the evidence travels with it. **Review this list when findings exist — attach when ANY apply:**

- A finding is backed by a failing-test or error log you captured.
- A finding has a reproduction capture (screenshot/recording).
- A perf or regression finding has a trace, benchmark, or profiling output.

Attach **after the review record exists** (the review id is required):

```bash
agentteams attachment create \
  --file .agentteams/cli/temp/repro.png \
  --code-review-id {codeReviewId}
```

- Pass exactly one target id (`--code-review-id` here; `--completion-report-id` for reports).
- Allowed types: images (jpg/png/gif/webp), text/markdown, pdf, html. Max 10 MB and 10 attachments per record.
- The CLI uploads the bytes straight to object storage and registers only metadata with the server.
- Keep the written finding self-contained — the attachment supplements, not replaces, the `problem` / `impact` / `suggestion` text. Skip only when no finding has capturable evidence.

## Diagrams (Mermaid)

A fenced `mermaid` block (`flowchart` / `sequenceDiagram`) renders in the web viewer; it stays plain text in CLI/raw. Use one when it explains a regression path or control flow faster than words.

## Creating Plans From Standalone Review Findings

Create a remediation plan only when the review has no source plan or plan-linked completion report. Finding-to-plan conversion is **human-in-the-loop** — the user chooses which `OPEN` findings become a plan; do not auto-include every finding. The generated plan preserves each selected finding's severity / file path / problem / impact / suggestion, and is scoped to fixing those findings, not reopening the original implementation.

Do not call `code-review create-plan` for a plan-based review. Plan-based findings are fixed directly in the source branch or pull request.

### Generated Remediation Plans

When `code-review create-plan` creates a remediation plan:

- Selected findings move from `OPEN` to `PLANNED`.
- Completing the generated plan as `DONE` automatically moves its remaining `PLANNED` findings to `RESOLVED`.
- Cancelling the generated plan moves its remaining `PLANNED` findings back to `OPEN`.

Do not manually resolve planned findings before the generated plan is verified unless the remediation was explicitly completed outside that plan.

## Completion Flow

## Cross-repo Change Set

When code reviews across multiple repositories belong to one delivery unit, create a Change Set and link each review with its merge order.

```bash
agentteams change-set create \
  --project-id {projectId} \
  --title "{changeSetTitle}"

agentteams change-set add-item \
  --change-set-id {changeSetId} \
  --repository-remote-url {repositoryRemoteUrl} \
  --branch-name {branchName} \
  --target-url {pullRequestUrl} \
  --merge-order 1 \
  --code-review-id {codeReviewId}
```

Add every paired review and repository separately, and use `--merge-order` to record the actual merge sequence. The free-text workaround of placing sibling review UUIDs in `reviewerContext` is deprecated. A v1 Change Set is informational only; it neither blocks nor automatically performs merges.

After a completion report, review whether a code review should be recommended. If it is recommended, present it as a separate explicit action. Keep co-action and post-mortem decisions separate from code review decisions.
