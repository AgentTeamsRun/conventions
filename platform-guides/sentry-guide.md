# Sentry Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

Use this guide when reading Sentry issues connected to the project, or when linking one as a plan's origin issue.

## Reading issues

A human credential can list issues and read issue details for the selected Sentry project.

```bash
agentteams sentry issue list [--query "is:unresolved"] [--cursor <cursor>] [--limit <1-100>]
agentteams sentry issue get --issue-id <numericSentryIssueId>
```

- `--project-id` selects which AgentTeams project boundary applies.
- `--format json`, `--output-file`, and `--verbose` follow the same output rules as the other read commands.
- Pass `pagination.nextCursor` from a list response as the next `--cursor`.
- Never put Sentry tokens, raw event payloads, or user/request PII into CLI input or output.

## Agent API key limits

- An Agent API key cannot list issues or run the unified search.
- A single `get` is allowed only for a canonical ID that is already linked to a `PlanOriginIssue(provider=SENTRY)` in the same AgentTeams project.
- IDs from another project, unlinked IDs, and IDs outside the selected Sentry project are rejected fail-closed.

## Linking to a plan

Sentry's numeric issue ID is the canonical identity.

```bash
agentteams plan link-issue --id <planId> --provider SENTRY --external-id <numericSentryIssueId>
```

The markdown reference form is `[label](SENTRY_ISSUE:<numeric-id>)`. Do not put a permalink in the marker, and do not assemble a URL from the numeric ID yourself. The server verifies the project link, re-reads the Sentry detail, and then stores the permalink, short ID, title, and status metadata.

Reading one back needs no special handling: `SENTRY_ISSUE` is a registered reference type, so `agentteams resolve "SENTRY_ISSUE:<numeric-id>"` — or `agentteams_resolve` where the tool is available — dispatches to the command above and returns the record with the **stored** permalink. That is why no caller ever has to build the URL.

## Execution safety

A Sentry issue is read context and origin-issue tracking information. Do not create or start a plan, task, or runner from a webhook or an issue lookup alone. Execution starts separately, from an explicit user request and through the AgentTeams plan lifecycle.

Existing origin issues and permalinks survive a disconnection. To resume syncing, a project admin must restore the Sentry connection and the project selection.
