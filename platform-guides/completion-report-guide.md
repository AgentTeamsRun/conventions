# Completion Report Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

Completion reports capture what changed, why it changed, how it was verified, and what remains.

## When to Create a Report

- After finishing a plan
- After shipping a user-visible change
- After completing a risky refactor or bug fix

## What to Include

Specific (name the feature/bug + why it mattered) · Verifiable (exact commands + results) · Scoped (what was intentionally NOT changed) · Actionable (remaining follow-ups). The Report File Structure below encodes these.

## Contract Note

- `report create` and completion report payloads do **not** use `reportType` anymore.
- Use `status` + metrics (commit and line/file stats) to describe report context.

## Report File Structure

Write the report file with this structure:

~~~markdown
## Summary
- <what changed and why — be specific, not generic>

## Verification
- typecheck: <pass or fail, with the command you ran>
- tests: <pass or fail, with the command you ran>

## Notes
- <risks, follow-ups, or "none">

## Conventions Referenced
- <list .agentteams/rules/*.md files you actually referenced — do not guess>
~~~

## Diagrams (Mermaid)

A fenced ```mermaid``` block (`flowchart` / `sequenceDiagram`) renders in the web viewer; it stays plain text in CLI/raw, so keep the prose self-contained. Use one only when it conveys the change flow faster than words.

## Report File Naming

Use `{first 8 characters of planId}-report.md`. Example: if planId is `57a51ec2-cf70-...`, the file name is `57a51ec2-report.md`.

For standalone reports (no plan), use a descriptive name: `<feature-or-fix-name>-report.md`.

> ⚠️ **Use either Path A or Path B, not both.** Running both simultaneously will create duplicate completion reports for the same plan.
>
## Plan-Linked vs Non-Plan Reports

You can attach report content directly while finishing a plan:

~~~bash
agentteams plan finish --id {planId} \
  --runner-type <runner-type> --model <model-id> \
  --report-title "<what you did and why, in one sentence>" \
  --report-file .agentteams/cli/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100, see Quality Score section> \
  --report-status <COMPLETED | PARTIAL | FAILED> \
  --review-recommendation <REQUIRED | NOT_NEEDED> \
  --review-reason "<one-line justification, see Code Review Recommendation section>"

~~~

> Git metrics (`commitHash`, `branchName`, `filesModified`, `linesAdded`, `linesDeleted`) are auto-collected. Use `--no-git` to disable. Manual overrides: `--duration-seconds`, `--commit-start`, `--commit-end`, `--pull-request-id`.

> `--review-recommendation` / `--review-reason` are **optional** (see the Code Review Recommendation section). An invalid `--review-recommendation` value is ignored with a warning.

> `--runner-type` and `--model` on `report create` are the **executor** snapshot — the runner/model that actually produced this report. This is independent of `Plan.runnerType` / `Plan.model`, which is the **creator** snapshot recorded at `plan create`. The two values can differ (e.g., a plan written by Claude but executed by Codex).

Separate report (Path B) — add `--plan-id {planId}` when under a plan, omit it for a standalone report:

~~~bash
agentteams report create [--plan-id {planId}] \
  --title "<what you did and why, in one sentence>" \
  --file .agentteams/cli/temp/{planId-first-8-chars-or-feature-name}-report.md \
  --runner-type <runner-type> --model <model-id> \
  --quality-score <0-100> \
  --status <COMPLETED | PARTIAL | FAILED>
~~~

Repository linkage note:

- If .agentteams/config.json contains `repositoryId`, `report create` links the report to that repository automatically.

## Metrics (Auto + Manual)

`report create` and `plan finish` can attach work metrics for insight workflows.

- Auto-collected by default (git context required):
  - `commitHash`, `branchName`, `filesModified`, `linesAdded`, `linesDeleted`
  - `commitStart`, `commitEnd` — auto-filled when `plan start` recorded a start commit
  - `durationSeconds` — auto-calculated from `startedAt` to finish time (server-side)
- Manual-only fields:
  - `pullRequestId`
- `--no-git` disables auto collection.
- Manual options override auto-collected values.

## Quality Score

Work quality is self-assessed by the agent on a 0-100 scale. Use the four dimensions below to judge holistically.

| Dimension | What to check |
|---|---|
| Verification | Did typecheck and tests pass? |
| Completeness | Were all requirements addressed? |
| Scope Adherence | Were changes limited to the requested scope? Were conventions followed? |
| Side Effects | Did the change avoid unintended impact on existing behavior? |

| Score | Meaning |
|---|---|
| 90-100 | All verification passed. All requirements met. Conventions followed. No side effects. |
| 70-89 | Minor gaps: verification partially skipped, slight scope overage, or minor convention deviations. |
| 0-69 | Verification failed, requirements unmet, significant scope violation, or side effects introduced. |

> Convention violations (naming, logging, PR rules, etc.) count against Scope Adherence.
> A score of 90+ requires conventions to be followed.

## Code Review Recommendation

Declare whether this change warrants an **independent code review**. This is a self-assessment captured at report time, separate from the quality score. Both fields are optional.

- `reviewRecommendation`: `REQUIRED` or `NOT_NEEDED` (omit when undecided — leaves it null)
- `reviewReason`: one short line (≤255 chars) justifying the call — write it for **both** values

Set `REQUIRED` when the change shows any of these risk signals (same list as `code-review-guide.md`):

- Cross-workspace or shared-contract changes
- Database schema, migration, authentication, permission, billing, quota, or deployment logic changes
- Large diffs, broad refactors, or changes touching critical user workflows
- Failed or skipped verification that still needs independent inspection

Otherwise set `NOT_NEEDED` with a brief reason (e.g., "docs-only", "test-only, no logic change", "trivial copy fix").

~~~bash
# REQUIRED example
--review-recommendation REQUIRED \
--review-reason "DB schema migration + auth middleware touched — independent verification advised"

# NOT_NEEDED example
--review-recommendation NOT_NEEDED \
--review-reason "i18n string update only, no logic change"
~~~

> This only **declares intent** for the review inbox. It does not by itself create a code review record — review creation remains an explicit, independent action (see `code-review-guide.md`).

## Verification Examples

~~~bash
cd api && npm run typecheck
cd api && npm test
cd cli && npm test
~~~

## Post-Report: Linked Document Auto-Creation

**Immediately after** writing a completion report, review the following two categories and create the documents if applicable.

### Co-Action Auto-Creation

Create a co-action if **any** of the following apply:

- Discoveries were made that cannot be inferred from code alone (workarounds, undocumented behaviors, environment quirks)
- Design decisions involved alternatives whose rationale should be recorded
- Follow-up work or known constraints remain
- Implicit knowledge needs to be handed off to another agent or session

~~~bash
# Create co-action
agentteams coaction create \
  --title "<concise handoff title>" \
  --file .agentteams/cli/temp/{descriptive-name}-coaction.md \
  --visibility PROJECT

# Link to plan (if applicable)
agentteams coaction link-plan --id {coActionId} --plan-id {planId}

# Link to completion report
agentteams coaction link-completion-report --id {coActionId} --completion-report-id {crId}
~~~

> For co-action content structure, see `co-action-guide.md`.

### Post-Mortem Auto-Creation

Create a post-mortem if **any** of the following apply:

**Service Incidents:**

- A failure, bug, or regression occurred during the work
- Existing functionality was unintentionally affected

**Development Execution Issues** (when reproducible, blocking, and preventable):

- A database migration failed due to missing state or incompatible assumptions
- A required CLI tool or dependency was missing or version-mismatched
- An environment variable or configuration was absent from a required environment
- A verification step (typecheck, test, lint) passed locally but failed in CI
- Unexpected behavior required significant debugging time

~~~bash
agentteams postmortem create \
  --plan-id {planId} \
  --title "<what went wrong — be specific>" \
  --file .agentteams/cli/temp/{name}-postmortem.md \
  --action-items "<preventive action 1>,<preventive action 2>" \
  --status OPEN
~~~

> For post-mortem content structure, see `post-mortem-guide.md`.

> ⚠️ Both are reviewed **immediately after the report**. Create them only when the conditions above apply — but **never skip the review itself.**

---

## Notes

- Keep reports factual and reproducible.
- Prefer describing *why* over listing every line-level change.
- If verification was skipped, state the reason explicitly.
