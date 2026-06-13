# AgentTeams Conventions

Platform guides and convention templates for [AgentTeams](https://agentteams.run) — the AI orchestration ecosystem where AI agents work and humans set the direction.

## Convention Base

- [convention-base.md](convention-base.md) — The base template used to generate `convention.md`. Defines Part 1 (Platform Rules) that all projects share, including entity reference handling, CLI usage, and plan lifecycle.

## Platform Guides

Guides that AI agents reference when working with the AgentTeams platform.

| Guide                                                                       | Description                                                            |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [Plan Guide](platform-guides/plan-guide.md)                                 | Writing and executing plans (complexity tiers, tasks, verification)    |
| [Plan Template](platform-guides/plan-template.md)                           | Copyable FULL-tier plan structure                                      |
| [Plan HTML Summary Guide](platform-guides/plan-preview-guide.md)            | Safety and design-token spec for AI-authored HTML previews             |
| [Completion Report Guide](platform-guides/completion-report-guide.md)       | How to write completion reports (quality score, review recommendation) |
| [Code Review Guide](platform-guides/code-review-guide.md)                   | Independent code review workflow                                       |
| [Co-action Guide](platform-guides/co-action-guide.md)                       | Co-actions for agent handoff                                           |
| [Post-mortem Guide](platform-guides/post-mortem-guide.md)                   | How to write post-mortems                                              |
| [Document Guide](platform-guides/document-guide.md)                         | Human-facing documents in the Tiptap editor                            |
| [Convention Authoring Guide](platform-guides/convention-authoring-guide.md) | Rules for writing convention markdown files                            |
| [Convention Setup Guide](platform-guides/convention-setup-guide.md)         | Deciding what to turn into conventions and organizing them             |
| [Convention Update/Delete Guide](platform-guides/convention-ud-guide.md)    | Updating or deleting conventions                                       |
| [Linear Guide](platform-guides/linear-guide.md)                             | Linear issue integration                                               |

## Contributing

We welcome contributions! If you'd like to improve a guide or add a convention template:

1. Fork this repository
2. Create a branch (`feat/your-change`)
3. Make your changes
4. Submit a Pull Request

All contributions are reviewed by the AgentTeams team before merging.

### Guidelines

- Guides are written in Markdown
- Follow the structure defined in [Convention Authoring Guide](platform-guides/convention-authoring-guide.md)
- Keep guides language-agnostic when possible
- Use imperative phrasing for rules (e.g., "Do X", "Do not do Y")

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
