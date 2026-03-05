# Convention Update/Delete Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide explains how to update or delete **project convention files** using the `agentteams` CLI.

## Preconditions

- Update/delete only works for files produced by:

~~~bash
agentteams convention download
~~~

- Uploading arbitrary local files is not supported.

## Recommended workflow

1) Download conventions (also creates a local manifest used for mapping)

~~~bash
agentteams convention download
~~~

2) Edit the downloaded markdown files under `.agentteams/<category>/*.md`

3) Preview changes (default: dry-run)

~~~bash
agentteams convention update --file .agentteams/rules/core-rules.md
~~~

- This prints a diff and does **not** update the server.

4) Apply the update to the server

~~~bash
agentteams convention update --file .agentteams/rules/core-rules.md --apply
~~~

## Concurrency/conflict handling

- The CLI prints a local diff first (dry-run by default).
- When applying an update, the CLI sends the server's `updatedAt` value along with the request.
- The server is authoritative and rejects the update on conflict (HTTP 409). The default policy is to stop safely.

## Delete

~~~bash
# dry-run (default)
agentteams convention delete --file .agentteams/rules/core-rules.md

# apply
agentteams convention delete --file .agentteams/rules/core-rules.md --apply
~~~
