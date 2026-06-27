# AgentTeams Convention

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> ⚠️ **Priority**: When platform rules (Part 1) and project rules (Part 2) conflict, **project rules take precedence**.

---

# Part 1 — Platform Rules

Rules for interacting with the AgentTeams platform (CLI, plans, reports, conventions). These are system-level and apply to all projects.

## Convention Freshness

> ⚠️ At the start of a session, **before substantive work**, make sure your conventions are current. The always-on conventions are loaded into your context once at session start, so a stale local copy means you would work from out-of-date rules without noticing.

1. If the `agentteams` CLI is available, check for updates (read-only, no writes):

   ```bash
   agentteams convention status
   ```

2. If updates are reported (`updateAvailable: true`), sync and **reload**:

   ```bash
   agentteams convention download
   ```

   Then re-read the affected `.agentteams/` rule files (and this convention) before proceeding, so you act on the latest rules.

3. If no updates are reported, proceed.
4. If the CLI is unavailable or the project is not configured, **skip this check silently and continue** — do not block on it.

> Run this check **once per session**, not before every command. Only `agentteams convention download` writes anything; `agentteams convention status` is safe to call anytime.

## Entity References & ID Handling

User messages from the AgentTeams web UI may contain entity references in `[label](type:id)` or `[label](type:id:path)` format.

- **ID prefix stripping (IMPORTANT)**: The `id` part can include a type prefix. Always strip this prefix before passing the id to any CLI flag (`--id`, `--plan-id`, etc.). The CLI also normalizes a prefixed id automatically if one slips through, but strip it yourself to keep commands portable.
  - Example: `[Safari pull-to-refresh](plan:agentteams_pln_f62762fc-730a-4201-8586-e2541505ed1b)` → use `f62762fc-730a-4201-8586-e2541505ed1b`
  - Canonical prefix list: `agentteams_pln_` (plan) · `agentteams_rpt_` (completionReport) · `agentteams_rev_` (codeReview) · `agentteams_act_` (coAction) · `agentteams_cnv_` (convention) · `agentteams_pmt_` (postMortem) · `agentteams_doc_` (document)
- Resolution by type:
  - `convention:id:.agentteams/path` → Read the local file at the given path (e.g., `.agentteams/rules/context.md`)
  - `completionReport:id` → Download with `agentteams report download --id {id}` and read the local file
  - `postMortem:id` → Download with `agentteams postmortem download --id {id}` and read the local file
  - `coAction:id` → Download with `agentteams coaction download --id {id}` and read the local file
  - `codeReview:id` → Fetch the review record with `agentteams code-review get --id {id}` and use the response as context
  - `document:id` → Download with `agentteams document download --id {id}` and read the local file
  - `LINEAR_ISSUE:uuid` → Fetch issue details with `agentteams linear issue get --issue-id {uuid}` and use the response as context. No prefix stripping needed — the value after `LINEAR_ISSUE:` is the raw Linear issue UUID.
  - `GITHUB_ISSUE:owner/repo#number` → GitHub issue. Use `gh issue view {number} --repo {owner/repo}` or GitHub API to fetch details. No prefix stripping needed.
  - `GITHUB_PR:owner/repo#number` → GitHub pull request. Use `gh pr view {number} --repo {owner/repo}` or GitHub API to fetch details. No prefix stripping needed.
  - `GITLAB_ISSUE:projectPath#iid` → GitLab issue. Use `glab issue view {iid} --repo {projectPath}` or GitLab API to fetch details. No prefix stripping needed.
  - `GITLAB_MERGE_REQUEST:projectPath!iid` → GitLab merge request. Use `glab mr view {iid} --repo {projectPath}` or GitLab API to fetch details. No prefix stripping needed.
  - `BITBUCKET_ISSUE:workspace/repo#id` → Bitbucket issue. Resolves to `https://bitbucket.org/{workspace}/{repo}/issues/{id}`; use the URL or `agentteams search` to fetch context. No prefix stripping needed. Bitbucket issues are also extracted as plan origin issues.
  - `BITBUCKET_PR:workspace/repo#id` → Bitbucket pull request. Resolves to `https://bitbucket.org/{workspace}/{repo}/pull-requests/{id}`; use the URL or `agentteams search` to fetch context. No prefix stripping needed.

## CLI Output Rules

> ⚠️ When creating, updating, or deleting platform records (plans, conventions, reports, postmortems, co-actions, code reviews, documents), do not stop at writing local files — always register the result to the server via the CLI.

> ⚠️ **Always display `webUrl`**: When a CLI command output contains a `webUrl` field, you **must** show it to the user as a clickable markdown link (e.g., `[View in AgentTeams](https://...)`) — not as raw text or inline code. Do not omit or summarize it away — the URL is the primary way users navigate to the created or updated entity.

> If the CLI is unavailable, skip reporting and continue the task.

## Commit Attribution

When an AgentTeams agent creates a git commit, append a co-author trailer at the
end of the commit message (separated from the body by a blank line) so the work
is attributed to AgentTeams:

```
Co-authored-by: AgentTeams <noreply@agentteams.run>
```

If another tool (e.g. Claude) adds its own `Co-authored-by` trailer, keep both —
one trailer per line, all at the very end of the message.

## Plan Lifecycle (Quick Reference)

Report status to AgentTeams **if you are working under a plan**.

```bash
agentteams plan start --id {planId}
# ... do the work, then commit it (see Commit Attribution) ...
agentteams plan finish --id {planId} ...   # registers the completion report
```

> ⚠️ If your work produced code changes, **commit them before `plan finish`**. `plan finish` auto-collects commit metrics (`commitHash`, `branchName`, diff stats) from the current git state — finishing with uncommitted work records an empty or wrong snapshot. If the project requires a PR, open it before finishing too.
>
> The full `plan finish` invocation — every flag plus quality-score / report-status / review-recommendation semantics — lives in `.agentteams/platform/completion-report-guide.md`. Read it before finishing; do not guess flag values.

## Plan Workflow Rules

Before starting work on a plan:

1. Download the runbook — `agentteams plan download --id {planId}` (saves to `.agentteams/cli/active-plan/{filename}.md`). Read it before starting.
2. Check comments, especially `RISK` — `agentteams comment list --plan-id {planId}`.
3. Full execution workflow (entity refs, comments, cleanup): `.agentteams/platform/plan-guide.md`.

## Work Completion Rules

### With a Plan

- `plan finish` auto-creates the completion report.
- Evidence produced (verification output saved to a file, before/after screenshots, repro/perf logs) → attach it to the report so the claim travels with proof (`agentteams attachment create --completion-report-id {id}`; see Attaching Evidence in `.agentteams/platform/completion-report-guide.md`).
- Handoff needed → also create a co-action (`.agentteams/platform/co-action-guide.md`).
- Risk signals (cross-workspace; schema/auth/billing/quota/deployment; large diffs; failed verification) → recommend a code review as a separate explicit action (`.agentteams/platform/code-review-guide.md`). Keep this decision independent of co-action / post-mortem decisions. Evidence behind a finding → attach it to the review (see Attaching Evidence in the code-review guide).

### Without a Plan (only when the user explicitly requests a report)

A completion report is always tied to a plan — there is no standalone (plan-less) report. To record work you already finished without a pre-existing plan, use a **quick log** (`plan quick`), which registers the plan and the report in a single step.

1. No active plan → ask: "No active plan found. Record it as a quick log?"
2. Approved → run `agentteams plan quick` with completion report flags to register the plan and report in a single step (see `.agentteams/platform/completion-report-guide.md` for report flags).
3. Declined → no report is recorded (a report cannot be created without a plan).
4. Handoff needed → also create a co-action (`.agentteams/platform/co-action-guide.md`).

```bash
# Record already-done work as a quick log (plan + completion report in one workflow)
# (see completion-report-guide.md for all available report flags)
agentteams plan quick --title "<brief work summary>" \
  --content "<see format below>" \
  --type <FEATURE | BUG_FIX | ISSUE | REFACTOR | CHORE> \
  --runner-type <runner-type> --model <model-id> \
  --report-file "<path to report markdown>" \
  [report-flags...]
```

> The `--content` body format (and when to choose a quick log over the full lifecycle) is the SSOT in `.agentteams/platform/plan-guide.md` (**Quick Log**).

## Unified Search

Use `agentteams search` to find entities (plans, co-actions, reports, post-mortems, conventions) across the project in a single call.

```bash
agentteams search --query "<keyword>" --format json
```

## Guide Checks Before Writing Platform Records

Before writing or updating **platform records** (plans, reports, conventions, postmortems, code reviews, documents), read the matching guide:

| Record type                            | Guide to read                                        |
| -------------------------------------- | ---------------------------------------------------- |
| Plan execution                         | `.agentteams/platform/plan-guide.md`                 |
| New plan                               | `.agentteams/platform/plan-template.md`              |
| Completion report                      | `.agentteams/platform/completion-report-guide.md`    |
| Postmortem                             | `.agentteams/platform/post-mortem-guide.md`          |
| Convention (create)                    | `.agentteams/platform/convention-authoring-guide.md` |
| Convention (update/delete)             | `.agentteams/platform/convention-ud-guide.md`        |
| Co-action (handoff)                    | `.agentteams/platform/co-action-guide.md`            |
| Code review (independent verification) | `.agentteams/platform/code-review-guide.md`          |
| Document (human-facing artifact)       | `.agentteams/platform/document-guide.md`             |
| Linear (issue/comment)                 | `.agentteams/platform/linear-guide.md`               |

---

# Part 2 — Project Rules

Project-specific conventions defined by the team. These rules govern coding standards, workflow, and domain-specific guidelines for this repository.
