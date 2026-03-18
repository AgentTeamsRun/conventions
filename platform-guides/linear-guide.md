# Linear Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

Use this guide when you need to read a Linear issue body or add a comment from the CLI.

## Available Commands

### Read an Issue

~~~bash
agentteams linear issue get --issue-id <linearIssueId>
~~~

- Returns the issue title, description, status, assignee, priority, labels, and URL.
- Use `--format json` when another tool needs structured output.

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

## Notes

- Scope is intentionally limited to issue detail lookup and comment creation.
- Issue create/update/delete and comment edit/delete are out of scope.
