# Comment Guide

Comments are the conversation layer of AgentTeams. They hang off a **parent record** — a plan, a plan task, a document, or a code-review finding — and they never exist on their own. A comment is where a human leaves guidance an agent must act on, and where an agent reports a risk, a question, or a decision back.

Read this guide before writing, editing, or deleting any comment.

## Pick the Right Target

Every root comment attaches to exactly **one** of four targets. Pick the record the conversation is actually about; a comment on the wrong parent is invisible to the people watching that record.

| Target    | Attach a comment when                                                                | Id you pass  |
| --------- | ------------------------------------------------------------------------------------ | ------------ |
| `PLAN`    | The subject is the plan as a whole — scope, risk, a required change of direction     | `planId`     |
| Plan task | The subject is one task inside a plan — its acceptance criteria, blockers, evidence  | `taskId`     |
| Document  | The subject is a document in the library — its content, accuracy, follow-up          | `documentId` |
| Finding   | The subject is one code-review finding — the fix, the disagreement, the verification | `findingId`  |

Never pass two of them in a single write. A comment has one parent by construction, and the write tools reject a call that names more than one.

## Plan Comments Carry a Type

A **plan** root comment additionally carries a `type`, and it is the only target that does:

- `RISK` — something that can make the plan fail or cause damage. A runner executing a plan must stop on an unresolved `RISK` comment, so do not use it for a mild concern.
- `MODIFICATION` — the plan's content or scope must change. Records what changed and why.
- `GENERAL` — everything else: progress notes, questions, answers.

`affectedFiles` is also plan-only: a list of repository paths the comment is about. Leave it empty when the comment is not about specific files.

Task, document, and finding comments take **only** `content`. Passing a type or affected files to them is an error, not a silently ignored field.

## Replies Are One Level Deep

Any root comment can have replies, and a reply belongs to the root comment it answers. **A reply cannot have a reply** — the server rejects that with a depth error. When a sub-thread needs its own thread, write a new root comment on the parent record instead.

A reply inherits its target from the root comment; you do not choose one. It carries only `content`.

Deleting a root comment deletes its replies with it.

## Who May Edit and Delete

- **Edit**: the author only. Nobody edits someone else's comment, and an execution credential (agent key, CI token, CLI/Desktop personal login) cannot edit a comment written by a person.
- **Delete**: the author, plus the member who owns the agent that wrote it. That is what makes an agent's comment cleanable by the person responsible for that agent.

An attempt outside those rules is rejected — treat a `403` here as "this is not yours", not as a transient failure to retry.

## Locked Parents

A plan that is `DONE` or `CANCELLED` is closed for conversation: creating, editing, or deleting comments and replies on that plan — or on its tasks — is rejected. The same lock applies to a reply whose root comment sits on a finished plan.

The single exception is an agent API key, which may still write on a finished plan so a runner can record what happened after the fact. Do not use that exception to keep an ordinary conversation going on a closed plan.

Document and finding comments have no such lock.

## Document Visibility

A `PRIVATE` document is invisible to everyone but its owner, and so are its comments and their replies. Reads and writes against them return **not found** rather than forbidden — the document's existence is itself hidden. Do not interpret that `404` as "the comment was deleted".

Comments on a `PROJECT`-visible document are visible to project members and their creation notifies them.

## Before You Delete

Deleting is a soft delete, but it is not visible to users as reversible: the comment and its replies disappear from every surface, and a deleted root comment takes its whole thread with it.

**Confirm with the user before deleting any comment you did not just create in this session.** Nothing in the server asks for that confirmation on your behalf — this rule is the only thing standing between a wrong id and someone else's thread.

Pass `expectedUpdatedAt` when you want the delete to be conditional (see below). Without it, the delete is unconditional.

## Writing via MCP

When the AgentTeams MCP server is connected, prefer the MCP write tools over shelling out to the CLI.

| Tool                              | Purpose                                                                                         |
| --------------------------------- | ----------------------------------------------------------------------------------------------- |
| `agentteams_guide_get("comment")` | Fetch this guide's current text plus its `guideHash`. **Call this before any comment write.**   |
| `agentteams_comment_create`       | Create a root comment. Pass exactly one of `planId`, `taskId`, `documentId`, `findingId`.       |
| `agentteams_comment_update`       | Edit a root comment you wrote.                                                                  |
| `agentteams_comment_delete`       | Delete a root comment and its replies (destructive).                                            |
| `agentteams_comment_reply_create` | Reply to a root comment.                                                                        |
| `agentteams_comment_reply_update` | Edit a reply you wrote.                                                                         |
| `agentteams_comment_reply_delete` | Delete a reply (destructive).                                                                   |
| `agentteams_comment_list`         | List root comments under one known parent. `replyCount` tells you whether a thread has replies. |
| `agentteams_comment_get`          | Fetch one root comment by id.                                                                   |
| `agentteams_comment_reply_list`   | List the replies of one root comment.                                                           |
| `agentteams_comment_reply_get`    | Fetch one reply by id.                                                                          |

Root comment ids and reply ids are different things, and the tools keep them apart on purpose: `agentteams_comment_update` refuses a reply id and `agentteams_comment_reply_update` refuses a root comment id. Passing the wrong kind is a not-found, never a silent edit of the other record.

The tools operate on the single project the MCP server is bound to. There is no `projectId` argument — a different project cannot be reached from an MCP session.

Every write returns the record id plus a `webUrl` pointing at the **parent screen** a human can open: the plan (for plan and task comments), the document, or the code review (for finding comments). Comments have no standalone page of their own.

### The three optional contract fields

All three are optional; omitting them all is valid and behaves exactly like a plain write.

- **`guideHash`** — the hash returned by `agentteams_guide_get("comment")`. Pass it so the server can confirm you followed the current rules. Every write tool accepts it, delete included. If your local copy is stale, the write is rejected with `GUIDE_OUTDATED` and the response names the hash the server expects. Recover with `agentteams convention download`, re-read this guide, and retry.
- **`idempotencyKey`** — a key of your choosing that makes a retry safe. Repeating a call with the same key and the same request replays the first result instead of posting a second comment, and it does not send the notification twice. Reusing a key with a _different_ request is rejected as a conflict — pick a new key for a new write. A retry that arrives while the first call is **still running** is rejected with a conflict that says so (wait a moment and repeat it to get the replay), and a key is remembered for **24 hours**.
- **`expectedUpdatedAt`** — the `updatedAt` you last read. Pass it on update and delete so a concurrent edit is rejected instead of silently overwritten. **Without it, a delete is unconditional**: it removes the comment even if someone edited it after you read it.

### Fallback to the CLI

When MCP is unavailable or a tool is missing, use the CLI — it reaches the same endpoints with the same server-side validation and the same error codes, and accepts the same three fields as `--guide-hash`, `--idempotency-key`, and `--expected-updated-at`.

```bash
# Root comments
agentteams comment list --plan-id {planId}
agentteams comment create --plan-id {planId} --type RISK --content "<markdown>"
agentteams comment create --task-id {taskId} --content "<markdown>"
agentteams comment create --finding-id {findingId} --content "<markdown>"
agentteams document comment-create --id {documentId} --content "<markdown>"
agentteams comment update --id {commentId} --content "<markdown>"
agentteams comment delete --id {commentId}

# Replies
agentteams comment reply-list --id {commentId}
agentteams comment reply-get --reply-id {replyId}
agentteams comment reply-create --id {commentId} --content "<markdown>"
agentteams comment reply-update --reply-id {replyId} --content "<markdown>"
agentteams comment reply-delete --reply-id {replyId}
```

If `agentteams_guide_get` reports that its hash is unknown, run `agentteams convention download` in the project. A missing hash is not fatal — writes still work, they just skip the freshness check.

## Writing Style

- Write the comment body in Markdown. Keep it to the point: a comment is read in a narrow panel next to the record.
- Lead with the conclusion, then the reasoning. A `RISK` comment must state what breaks and what decision you need.
- Quote file paths and identifiers verbatim; put them in `affectedFiles` too when the target is a plan.
- Answer in the language the surrounding conversation uses.
