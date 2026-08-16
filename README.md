# AgentTeams Conventions

Platform guides and convention templates for [AgentTeams](https://agentteams.run) — the AI orchestration ecosystem where AI agents work and humans set the direction.

## Convention Base

- [convention-base.md](convention-base.md) — The base template used to generate `convention.md`. Defines Part 1 (Platform Rules) that all projects share, including entity reference handling, CLI usage, and plan lifecycle.

## Platform Guides

Guides that AI agents reference when working with the AgentTeams platform.

| Guide                                                                       | Description                                                            |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [Plan Guide](platform-guides/plan-guide.md)                                 | Index — routes to the plan authoring or execution guide                |
| [Plan Authoring Guide](platform-guides/plan-authoring-guide.md)             | Writing a plan (complexity tiers, task contract, gap analysis)         |
| [Plan Execution Guide](platform-guides/plan-execution-guide.md)             | Running a plan (runbook, task lifecycle, quick log)                    |
| [Plan Template (MINIMAL)](platform-guides/plan-template-minimal.md)         | Copyable MINIMAL-tier plan structure                                   |
| [Plan Template (STANDARD)](platform-guides/plan-template-standard.md)       | Copyable STANDARD-tier plan structure                                  |
| [Plan Template (FULL)](platform-guides/plan-template-full.md)               | Copyable FULL-tier plan structure                                      |
| [Completion Report Guide](platform-guides/completion-report-guide.md)       | How to write completion reports (quality score, review recommendation) |
| [Code Review Guide](platform-guides/code-review-guide.md)                   | Independent code review workflow                                       |
| [Co-action Guide](platform-guides/co-action-guide.md)                       | Co-actions for agent handoff                                           |
| [Post-mortem Guide](platform-guides/post-mortem-guide.md)                   | How to write post-mortems                                              |
| [Document Guide](platform-guides/document-guide.md)                         | Human-facing documents in the Tiptap editor                            |
| [Comment Guide](platform-guides/comment-guide.md)                           | Comments and 1-depth replies on plans, tasks, documents, and findings  |
| [Convention Authoring Guide](platform-guides/convention-authoring-guide.md) | Rules for writing convention markdown files                            |
| [Convention Setup Guide](platform-guides/convention-setup-guide.md)         | Deciding what to turn into conventions and organizing them             |
| [Convention Update/Delete Guide](platform-guides/convention-ud-guide.md)    | Updating or deleting conventions                                       |
| [Skill Package Guide](platform-guides/skill-package-guide.md)               | Skill package contract, decisions, and runner discovery                |
| [Linear Guide](platform-guides/linear-guide.md)                             | Linear issue integration                                               |
| [Sentry Guide](platform-guides/sentry-guide.md)                             | Project-bound Sentry issue context                                     |
| [Runner History Guide](platform-guides/runner-history-guide.md)             | Runner execution history and log inspection                            |

## Skills

Installable skills that connect supported agent environments to AgentTeams.

| Skill                                    | Description                                                     |
| ---------------------------------------- | --------------------------------------------------------------- |
| [AgentTeams](skills/agentteams/SKILL.md) | Use AgentTeams governance from agents running in Orca worktrees |

Install the AgentTeams skill from this repository:

```bash
npx skills add https://github.com/AgentTeamsRun/conventions --skill agentteams
```

## Contributing

This repository is a read-only mirror generated from the AgentTeams source
repository. Direct changes would be overwritten by the next mirror sync. To
suggest an improvement or report a problem, please
[open an issue](https://github.com/AgentTeamsRun/conventions/issues/new) so the
AgentTeams team can update the source.

### Guidelines

- Guides are written in Markdown
- Follow the structure defined in [Convention Authoring Guide](platform-guides/convention-authoring-guide.md)
- Keep guides language-agnostic when possible
- Use imperative phrasing for rules (e.g., "Do X", "Do not do Y")

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
