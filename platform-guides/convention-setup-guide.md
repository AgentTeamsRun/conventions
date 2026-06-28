# Convention Setup Guide

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> This guide helps you decide **what to turn into Conventions** and how to organize them.
> For **how to write** a Convention file (format, structure, markers), see `convention-authoring-guide.md`.

## 1) Overview

After running `agentteams init`, your project has a `.agentteams/` directory but no project-specific Conventions yet. This guide walks you through analyzing your project, converting existing rule files, and creating new Conventions — so the AI agent can follow your team's standards from day one.

**Two scenarios this guide covers:**

1. **Migration** — You already have rule files (CLAUDE.md, .cursorrules, etc.) and want to convert them into Conventions.
2. **Fresh start** — No existing rules; you need to create Conventions from scratch.

## 2) Step 1 — Analyze the Project

Before writing any Convention, understand the project:

1. **Detect the tech stack** from manifest files:
   - `package.json` (Node.js / TypeScript)
   - `pyproject.toml`, `requirements.txt` (Python)
   - `go.mod` (Go)
   - `Cargo.toml` (Rust)
   - `build.gradle`, `pom.xml` (Java / Kotlin)
   - `Gemfile` (Ruby)
2. **Check the directory structure** — monorepo vs single-app, workspace layout.
3. **Read existing documentation** — README, CONTRIBUTING, wiki links — for team workflows and coding standards.

## 3) Step 2 — Detect and Convert Existing Rule Files

Scan for AI tool rule files that may already contain valuable conventions:

| File                                 | Tool           |
| ------------------------------------ | -------------- |
| `CLAUDE.md`                          | Claude Code    |
| `.claude/settings.json`              | Claude Code    |
| `.cursorrules`, `.cursor/rules/*.md` | Cursor         |
| `.github/copilot-instructions.md`    | GitHub Copilot |
| `.windsurfrules`                     | Windsurf       |
| `.aider*`                            | Aider          |
| `codex.md`, `AGENTS.md`              | Codex          |

**How to convert:**

1. Read each file and classify rules by domain (coding style, Git workflow, testing, deployment, etc.).
2. Group related rules into separate Conventions — do **not** dump everything into one file.
3. Keep the original files intact for backward compatibility with existing tools.

## 4) Step 3 — Design Your Convention Set

### Recommended structure

| Convention            | Purpose                                                                 | Trigger                         |
| --------------------- | ----------------------------------------------------------------------- | ------------------------------- |
| `context`             | Project identity, tech stack, directory structure                       | `always_on`                     |
| `conventions`         | Common coding conventions (naming, logging, Git branching)              | `always_on`                     |
| Domain-specific rules | Detailed rules for schemas, routes, frontend, testing, deployment, etc. | `model_decision` or conditional |

### Size guidelines

- If a Convention exceeds ~200 lines, consider splitting it.
- If a Convention is under ~20 lines, consider merging it with a related one.
- Aim for **5–8 Conventions** total. More than 10 is usually too many.

## 5) Step 4 — Choose Triggers

| Trigger          | When to use                                                     | Example                          |
| ---------------- | --------------------------------------------------------------- | -------------------------------- |
| `always_on`      | Rules every task needs — project context, core coding standards | `context`, `conventions`         |
| `model_decision` | Load when relevant to the task                                  | `schema`, `frontend`, `testing`  |
| `-`              | One-time reference, guides, templates                           | `docs-style-guide`, setup guides |

> Keep `always_on` Conventions to **2–3 max** — each adds to every conversation's context cost. This is the single rule for what belongs in `always_on`.

## 6) Step 5 — Create and Register Conventions

1. Write each Convention file following `convention-authoring-guide.md` format rules.
2. Place files under `.agentteams/<category>/` (`rules`, `skills`, `guides`, or `references`).
3. Register with the CLI:

```bash
agentteams convention create --file .agentteams/{category}/{convention-name}.md
```

4. Download the updated convention index to verify:

```bash
agentteams convention download
```

## 7) Quality Checklist

- [ ] `context` Convention accurately reflects the project's tech stack and structure
- [ ] `always_on` Conventions are limited to 3 or fewer
- [ ] Each Convention has an appropriate trigger
- [ ] No content duplication across Conventions
- [ ] `AGENT_RULES` sections use clear, imperative phrasing
- [ ] File names use lowercase hyphen-case (e.g., `coding-standards.md`)

## 8) Example: Adaptable Convention Set

```
.agentteams/
  rules/
    context.md          # always_on — Project overview, tech stack, structure
    conventions.md      # always_on — Naming, logging, Git branching
    architecture.md     # model_decision — System boundaries and module patterns
    data-modeling.md    # model_decision — Persistence or schema design rules
    interfaces.md       # model_decision — API, CLI, UI, or integration contracts
    testing.md          # model_decision — Test framework, commands, and patterns
    release-process.md  # model_decision — Deploy, release, or publishing checklist
```

> This is an illustrative example. Adapt the set to your project's actual stack and workflow — not every project needs all of these, and some projects may need different domains entirely.
