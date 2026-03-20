# Linear Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

Use this guide when you need to read, create, or comment on Linear issues from the CLI.

## Available Commands

### Read an Issue

~~~bash
agentteams linear issue get --issue-id <linearIssueId>
~~~

- Returns the issue title, description, status, assignee, priority, labels, and URL.
- Use `--format json` when another tool needs structured output.

### Create an Issue

~~~bash
agentteams linear issue create --team-id <linearTeamId> --title "Issue title" [--description "Details"] [--state "Done"]
~~~

- Creates a new Linear issue in the specified team.
- `--team-id` and `--title` are required.
- `--description` is optional. Supports markdown.
- `--state` is optional. Accepts a state name (e.g., "Backlog", "Todo", "In Progress", "Done", "Canceled"). The name is matched case-insensitively against the team's workflow states. If omitted, the team's default state is used.
- Returns the created issue id, identifier, title, and URL.

### Update Issue State

~~~bash
agentteams linear issue update --issue-id <linearIssueId> --state "In Progress"
~~~

- Changes the status of an existing Linear issue.
- `--issue-id` accepts both UUID and identifier (e.g., `AGE-13`).
- `--state` accepts a state name matched case-insensitively against the issue's team workflow states.

### Create a Comment

~~~bash
agentteams linear comment create --issue-id <linearIssueId> --body "Comment text"
~~~

- Creates a new Linear comment with the connected member's token.
- Returns the created comment id and timestamps.

## Authentication Rules

- These commands use the AgentTeams API key from `.agentteams/config.json` or `AGENTTEAMS_*` environment variables.
- The API key must belong to an agent config whose owner has connected a Linear account.
- If Linear is not connected, the API returns a not-connected error and the CLI exits with failure.

## Resolving Issue IDs

The `--issue-id` flag accepts both **UUID** and **identifier** (e.g., `AGE-13`).

| Source | How to extract | Example |
|---|---|---|
| Entity reference `[label](LINEAR_ISSUE:uuid)` | Use the UUID directly | `--issue-id 7c113c62-f3b6-48f5-bc76-c3a1579094fe` |
| Linear URL `https://linear.app/{workspace}/issue/{identifier}/...` | Extract the identifier segment from the URL path | `--issue-id AGE-13` |
| User mentions identifier in text (e.g., "AGE-13 확인해줘") | Use the identifier as-is | `--issue-id AGE-13` |

## Common Workflow

1. Resolve the Linear issue id from one of the sources above.
2. Read the issue first:

~~~bash
agentteams linear issue get --issue-id <issueId>
~~~

3. Add a follow-up comment if needed:

~~~bash
agentteams linear comment create --issue-id <issueId> --body "Investigation completed."
~~~

### Retroactive Issue Logging

When the user asks to log completed work as a Linear issue (e.g., "이슈 등록하고 완료로 처리해줘"), create the issue with `--state "Done"`:

~~~bash
agentteams linear issue create --team-id <teamId> --title "Fix: auth token refresh" --description "Added auto-refresh for expired Linear OAuth tokens" --state "Done"
~~~

## Notes

- Issue update/delete and comment edit/delete are out of scope.
