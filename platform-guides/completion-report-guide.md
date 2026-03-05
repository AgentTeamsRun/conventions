# Completion Report Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

Completion reports capture what changed, why it changed, how it was verified, and what remains.

## When to Create a Report

- After finishing a plan
- After shipping a user-visible change
- After completing a risky refactor or bug fix

## What Makes a Good Report

- Specific: names the feature/bug and the reason it mattered
- Verifiable: includes the commands you ran (and results)
- Scoped: states what was intentionally NOT changed
- Actionable: calls out follow-ups if any remain

## What to Include

- Purpose: why this work was needed
- Scope: what areas/modules were touched
- Verification: exact commands you ran and whether they passed
- Risks: potential side effects or follow-up work

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

## Report File Naming

Use `{first 8 characters of planId}-report.md`. Example: if planId is `57a51ec2-cf70-...`, the file name is `57a51ec2-report.md`.

For standalone reports (no plan), use a descriptive name: `<feature-or-fix-name>-report.md`.

> ⚠️ **Use either Path A or Path B, not both.** Running both simultaneously will create duplicate completion reports for the same plan.
>
## Plan-Linked vs Non-Plan Reports

You can attach report content directly while finishing a plan:

~~~bash
agentteams plan finish --id {planId} \
  --report-title "<what you did and why, in one sentence>" \
  --report-file .agentteams/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100, see Quality Score section> \
  --report-status <COMPLETED | PARTIAL | FAILED>

~~~

> Git metrics (`commitHash`, `branchName`, `filesModified`, `linesAdded`, `linesDeleted`) are auto-collected. Use `--no-git` to disable. Manual overrides: `--duration-seconds`, `--commit-start`, `--commit-end`, `--pull-request-id`.

If you were working under a plan, link the report to the plan:

~~~bash
agentteams report create \
  --plan-id {planId} \
  --title "<what you did and why, in one sentence>" \
  --file .agentteams/temp/{planId-first-8-chars}-report.md \
  --quality-score <0-100> \
  --status <COMPLETED | PARTIAL | FAILED>
~~~

If you were not working under a plan, omit the plan id:

~~~bash
agentteams report create \
  --title "<what you did and why, in one sentence>" \
  --file .agentteams/temp/<feature-or-fix-name>-report.md \
  --quality-score <0-100> \
  --status <COMPLETED | PARTIAL | FAILED>
~~~

Repository linkage note:

- If .agentteams/config.json contains `repositoryId`, `report create` links the report to that repository automatically.

## Metrics (Auto + Manual)

`report create` and `plan finish` can attach work metrics for insight workflows.

- Auto-collected by default (git context required):
  - `commitHash`, `branchName`, `filesModified`, `linesAdded`, `linesDeleted`
- Manual-only fields:
  - `durationSeconds`, `commitStart`, `commitEnd`, `pullRequestId`
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

## Verification Examples

~~~bash
cd api && npm run typecheck
cd api && npm test
cd cli && npm test
~~~

## Notes

- Keep reports factual and reproducible.
- Prefer describing *why* over listing every line-level change.
- If verification was skipped, state the reason explicitly.
