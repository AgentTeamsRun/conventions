# AgentTeams Convention

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> ⚠️ **Priority**: When platform rules (Part 1) and project rules (Part 2) conflict, **project rules take precedence**.

---

# Part 1 — Platform Rules

Rules for interacting with the AgentTeams platform (CLI, plans, reports, conventions). These are system-level and apply to all projects.

## Session Start

Run once, before substantive work. Re-read every file the output lists under `reread`.

```bash
agentteams session sync
```

## Reading Entities (MCP First)

User messages may carry entity references as `[label](type:id)` or `[label](type:id:path)`.

Always prefer MCP for read-only entity access when the AgentTeams MCP server is connected. **Each tool's own description is the SSOT** for paging, filter defaults, and truncation.

- **Type already known**: the typed `agentteams_*_get` tool wins.
- **Type must be worked out from the token**: hand the token over verbatim and do what the result's `message` says — it names the file to read, the payload to use, or the command to run.

  ```bash
  agentteams resolve "plan:agentteams_pln_f62762fc-730a-4201-8586-e2541505ed1b"
  ```

  `agentteams_resolve`, if your tool list has it, answers the same, except record bodies come back inline instead of as a downloaded file.

Use the `agentteams` CLI for everything else: mutations without an MCP write tool, downloads, workflows that create local files, and any environment where MCP or the matching tool is unavailable. CLI commands call AgentTeams HTTP APIs directly; they do not use MCP as an internal transport. Both surfaces share the same authenticated APIs and project scope.

## Platform Records — Register, Then Link

> ⚠️ Creating, updating, or deleting a platform record (plan, convention, skill, report, postmortem, co-action, code review, document) is **not finished when the local files are written** — register the result to the server via the CLI. If the CLI is unavailable, skip reporting and continue the task.

- **Read the matching guide before you write.** `agentteams_guide_get`, or `agentteams guide get --record-kind <kind>` (`agentteams guide list` names them). Do not guess flag values or document structure.
- **Show every `webUrl` a command returns as a clickable markdown link** (e.g. `[View in AgentTeams](https://...)`) — it is how the user reaches what you just created.

## Plan Execution and Completion

**If you are working under a plan**, per-task lifecycle tracking is required — it is not an optional progress note: `plan start`, then `task start` / `task finish` for every task, then `plan finish`. Task selection, statuses, comments, commit handling, and the full `plan finish` invocation are in `.agentteams/platform/plan-execution-guide.md` and `.agentteams/platform/completion-report-guide.md`.

After `plan finish`, decide evidence, handoff, and code review **independently** — one does not imply or replace another.

Without a plan there is no standalone report: recording already-done work means a quick log (`plan quick`), and only when the user asks for one. No active plan → ask "No active plan found. Record it as a quick log?" — declined means nothing is recorded.

## Commit Attribution

When an AgentTeams agent creates a git commit, append a co-author trailer at the end of the commit message (separated from the body by a blank line) so the work is attributed to AgentTeams:

```
Co-authored-by: AgentTeams <noreply@agentteams.run>
```

If another tool (e.g. Claude) adds its own `Co-authored-by` trailer, keep both — one trailer per line, all at the very end of the message.

---

# Part 2 — Project Rules

Project-specific conventions defined by the team. These rules govern coding standards, workflow, and domain-specific guidelines for this repository.
