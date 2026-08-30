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

## Writing via MCP

When the AgentTeams MCP server is connected, prefer the MCP write tools over shelling out to the CLI.

| Tool                                       | Purpose                                                                                           |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| `agentteams_guide_get("code-review")`      | Fetch this guide's current text plus its `guideHash`. **Call this before any code review write.** |
| `agentteams_codereview_create`             | Create a review. Pass `findings` upfront when the result is already known.                        |
| `agentteams_codereview_update`             | Update metadata on a `PENDING` review, or cancel it by passing `status: "CANCELLED"`.             |
| `agentteams_codereview_finding_status_set` | Move one finding to `DISMISSED` (dismiss), `OPEN` (undismiss), or `RESOLVED`.                     |

The tools operate on the single project the MCP server is bound to. There is no `projectId` argument — a different project cannot be reached from an MCP session.

**A finding has no standalone create or delete tool.** Supply findings when you create the review; afterwards only their status changes. Submitting the result of a `PENDING` review is likewise a CLI-only path (`agentteams code-review submit-result`), as is creating a remediation plan from selected findings.

### The three optional contract fields

All three are optional; omitting them all is valid and behaves exactly like a plain write.

- **`guideHash`** — the hash returned by `agentteams_guide_get("code-review")`. Pass it so the server can confirm you followed the current rules. Every write tool accepts it. If your local copy is stale, the write is rejected with `GUIDE_OUTDATED` and the response names the hash the server expects. Recover with `agentteams convention download`, re-read this guide, and retry.
- **`idempotencyKey`** — a key of your choosing that makes a retry safe. Repeating a call with the same key and the same request replays the first result instead of creating a second review. Reusing a key with a _different_ request is rejected as a conflict — pick a new key for a new write. A retry that arrives while the first call is **still running** is rejected with a conflict that says so (wait a moment and repeat it to get the replay), and a key is remembered for **24 hours**.
- **`expectedUpdatedAt`** — the `updatedAt` you last read (from `agentteams_codereview_get` for a review, `agentteams_codereview_finding_get` for a finding). Pass it on update and on a finding transition so a concurrent change is rejected instead of silently overwritten. **Without it, the write is unconditional**: it applies even if someone changed the record after you read it.

**Retrying a create keeps the `findings` order you first sent.** The request fingerprint behind `idempotencyKey` preserves array order, so a retry that lists the same findings in a different order is not a retry — it is rejected as key reuse. Either resend the array exactly as before, or use a new key. If you regenerate findings between attempts and cannot guarantee the order, use a new key.

### Fallback to the CLI

When MCP is unavailable or a tool is missing, use `agentteams code-review create/update/cancel/dismiss/undismiss/resolve` — the CLI reaches the same endpoints with the same server-side validation and the same error codes, and accepts the same three fields as `--guide-hash`, `--idempotency-key`, and `--expected-updated-at`. One difference is worth knowing: `code-review resolve` accepts several findings at once, and each one is a separate request, so a single `--idempotency-key` cannot cover them. Resolve one finding per call when you need that key.

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

## Review Summary

`resultSummary` is the review's conclusion. It is rendered above the findings on the review detail screen, and it is the only part a reader sees before deciding whether to open the findings at all. Write it so that decision can be made without reading them and without opening the diff.

Write it as three blocks, in this order:

1. **Verdict** — one line. Whether the change can merge, plus the finding counts by severity. If a required input could not be recovered or a command failed, say so here; it changes how far the verdict can be trusted.
2. **What changes for people** — at most three lines. Who experiences what: the user, the screen, the data, the operator, or the team. When a finding changes UI, name the affected screen and the condition that triggers it. When a finding changes a business rule, state the rule before and after.
3. **Remaining actions** — a short list, most severe first. One line per item that still needs a fix or a decision.

Constraints:

- Do not open with code narration. Symbol names, call paths, and file layout belong in block 3 or in the findings themselves — never in blocks 1 and 2.
- Do not write the summary as a single paragraph. A reader who has to parse one long block to reach the consequence gains nothing over reading the findings.
- Say so plainly when a finding has no user-visible effect. Contract regressions, refactors, and internal cleanups are legitimate findings; report what they mean for the team or the operator instead of inventing a user-facing consequence.
- Keep it shorter than the findings it summarizes. A summary approaching the length of the findings is a second copy of them.

Example:

> **Not mergeable as is** — P0 0 / P1 2 / P2 3 / P3 3.
>
> Two P1s let a chained runner request run on a machine whose owner never shared that agent, and run in write mode on a runner restricted to planning only. Both appear only once this change enables chained dispatch for the first time, so neither is reachable in production today.
>
> Remaining:
>
> - P1 — route chain publishing through the shared-agent and plan-mode checks the direct API already enforces.
> - P1 — same root cause as above; one shared predicate fixes both.
> - P2 ×3 — a misclassified error message, a disabled-state UI conflict, and a missing server-side guard for agent-key callers.

## Finding Format

Each finding must include:

- Severity: `P0`, `P1`, `P2`, or `P3`
- File path and line range when available
- Short title
- Problem: what is wrong
- Impact: who experiences what — see below
- Suggestion: concrete fix direction

Prefer actionable findings over broad commentary. Do not include items that cannot be verified from the diff, repository context, or stated requirements.

### Writing `impact`

Open with the consequence, not the mechanism. The first sentence must name who experiences what: the user, the screen, the data, the operator, or the team. Symbol names, call paths, and ordering belong in the sentences after it, or in `problem`.

When the finding changes UI, name the affected screen and the condition that triggers it. When it changes a business rule, state the rule before and after.

Not every finding reaches a user, and that is fine. A contract regression, an internal refactor, or a missing test still has a real consequence for the team or the operator — report that one. Do not invent a user-facing effect to satisfy the format, and do not stretch a maintainability issue into a user harm it does not cause.

- ❌ `printMcpRegistration` filters on `outcome !== 'SKIPPED_NOT_DETECTED'`, so the entry is removed from the list and the remaining count is printed instead.
- ✅ A user whose client was not detected sees no row for it in the install output and cannot tell whether it was skipped or failed. `printMcpRegistration` drops the entry before printing the count.

Both sentences describe the same defect. Only the second lets a reader judge severity without opening the file.

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
   agentteams code-review finding-list --id {codeReviewId}
   ```

   Repeat with `--page` until `meta.page` equals `meta.totalPages`. The review header from `code-review get --id` no longer inlines findings.

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

`--idempotency-key` and `--expected-updated-at` each apply to one finding, so do not combine either option with `--finding-ids`. To keep retry or concurrency protection, resolve findings one at a time with `--finding-id`; pass that finding's own `updatedAt` as `--expected-updated-at`.

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
