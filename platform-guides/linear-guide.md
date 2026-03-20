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

## Common Workflow

1. Resolve the Linear issue id from a message reference such as `[label](LINEAR_ISSUE:uuid)`.
2. Read the issue first:

~~~bash
agentteams linear issue get --issue-id <uuid>
~~~

3. Add a follow-up comment if needed:

~~~bash
agentteams linear comment create --issue-id <uuid> --body "Investigation completed."
~~~

### Retroactive Issue Logging

When the user asks to log completed work as a Linear issue (e.g., "이슈 등록하고 완료로 처리해줘"), create the issue with `--state "Done"`:

~~~bash
agentteams linear issue create --team-id <teamId> --title "Fix: auth token refresh" --description "Added auto-refresh for expired Linear OAuth tokens" --state "Done"
~~~

## Notes

- Issue update/delete and comment edit/delete are out of scope.
