# Completion Report Guide (AgentTeams)

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

## Minimal Template

Use a short structure so future readers can scan it quickly:

~~~
## Summary
- What changed and why

## Verification
- typecheck: ...
- tests: ...

## Notes
- risks / follow-ups

## Conventions Referenced
- .agentteams/rules/...  # list conventions you referenced during this work
~~~

## Plan-Linked vs Non-Plan Reports

You can attach report content directly while finishing a plan:

~~~bash
agentteams plan finish --id {planId} --report-title "Work completion summary" --report-file .agentteams/temp/<report-file-name>.md

# Alternative: attach a minimal template without writing a file
agentteams plan finish --id {planId} --report-template minimal
~~~

If you were working under a plan, link the report to the plan.

~~~bash
agentteams report create \
  --plan-id {planId} \
  --title "Feature X implemented" \
  --content "## Summary\n- What changed and why\n\n## Verification\n- typecheck: pass\n- tests: pass\n\n## Notes\n- risks / follow-ups"
~~~

If you were not working under a plan, omit the plan id.

~~~bash
agentteams report create \
  --title "Feature X implemented" \
  --content "## Summary\n- What changed and why\n\n## Verification\n- typecheck: pass\n- tests: pass\n\n## Notes\n- risks / follow-ups"
~~~

Repository linkage note:

- If .agentteams/config.json contains `repositoryId`, `report create` links the report to that repository automatically.

## Metrics (Auto + Manual)

`report create` can attach work metrics for insight workflows.

- Auto-collected by default (git context required):
  - `commitHash`, `branchName`, `filesModified`, `linesAdded`, `linesDeleted`
- Manual-only fields:
  - `durationSeconds`, `commitStart`, `commitEnd`, `pullRequestId`
- `--no-git` disables auto collection.
- Manual options override auto-collected values.

Example:

~~~bash
agentteams report create \
  --plan-id {planId} \
  --title "Feature X implemented" \
  --content "## Summary\n- What changed and why" \
  --files-modified 5 \
  --lines-added 120 \
  --lines-deleted 30 \
  --quality-score 95
~~~

## Quality Score

Work quality is self-assessed by the agent on a 0-100 scale. Use the four dimensions below to judge holistically.

**Dimensions:**
- **Verification**: Did typecheck and tests pass?
- **Completeness**: Were all requirements addressed?
- **Scope Adherence**: Were changes limited to the requested scope? Were conventions followed?
- **Side Effects**: Did the change avoid unintended impact on existing behavior?

**Score Tiers:**

| Score | Color | Criteria |
|-------|-------|----------|
| 90-100 | Green | All verification passed. All requirements met. Changes within scope and conventions followed. No side effects. |
| 70-89 | Yellow | Minor gaps: verification partially skipped, slight scope overage, or minor convention deviations. |
| 0-69 | Red | Verification failed, requirements unmet, significant scope violation, or side effects introduced. |

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
