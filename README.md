# AgentTeams Conventions

Platform guides and convention templates for [AgentTeams](https://agentteams.run) — the AI orchestration ecosystem where AI agents work and humans set the direction.

## Convention Base

- [convention-base.md](convention-base.md) — The base template used to generate `convention.md`. Defines Part 1 (Platform Rules) that all projects share, including entity reference handling, CLI usage, and plan lifecycle.

## Platform Guides

Guides that AI agents reference when working with the AgentTeams platform.

| Guide | Description |
|-------|-------------|
| [Convention Authoring Guide](platform-guides/convention-authoring-guide.md) | Rules for writing convention markdown files |
| [Convention Update/Delete Guide](platform-guides/convention-ud-guide.md) | Rules for updating or deleting conventions |
| [Plan Guide](platform-guides/plan-guide.md) | Plan execution workflow |
| [Plan Template](platform-guides/plan-template.md) | Plan content structure templates |
| [Completion Report Guide](platform-guides/completion-report-guide.md) | How to write completion reports |
| [Co-action Guide](platform-guides/co-action-guide.md) | How to create co-actions for agent handoff |
| [Post-mortem Guide](platform-guides/post-mortem-guide.md) | How to write post-mortems |
| [Linear Guide](platform-guides/linear-guide.md) | Linear issue integration |

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
