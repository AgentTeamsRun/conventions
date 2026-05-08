# Set Up Project Conventions

## TL;DR

Seed plan auto-registered on `agentteams init`. Follow the steps in `.agentteams/platform/convention-setup-guide.md` to write and register Conventions so the AI agent can work against consistent project standards from day one.

- **Deliverables**:
  - `.agentteams/rules/context.md` (`always_on`) — project identity, tech stack, structure
  - `.agentteams/rules/conventions.md` (`always_on`) — common coding rules
  - Domain-specific rule files as needed (`schema.md`, `routes.md`, `frontend.md`, etc.)
  - All convention files registered via `agentteams convention create --file <path>`
- **Estimated Effort**: Medium
- **Type**: CHORE
- **Parallel Execution**: NO

## Context

A freshly initialized project has the `.agentteams/` directory but no project-specific Conventions yet. Initial Conventions are not the final answer — they are a starting point you will refine as the project evolves. The agent's job is to investigate the project, draft a first set, and clearly mark items that need a human decision.

- Authoring procedure: `.agentteams/platform/convention-setup-guide.md`
- Authoring format: `.agentteams/platform/convention-authoring-guide.md`

## Work Objectives

### Definition of Done

- Every item in the Quality Checklist of `convention-setup-guide.md` is satisfied or marked for human review.
- `always_on` Conventions are limited to 2–3.
- All Conventions are registered on the server via the CLI.
- Items requiring human decision are listed under Human Review Points or in the completion report.

### Must Have

- Read `convention-setup-guide.md` (Steps 1–5) and `convention-authoring-guide.md` before writing any Convention.
- Leave guesses unconfirmed; record them as items for human review.

### Must NOT Have

- Do not copy secrets, tokens, or personal env file contents into Conventions.
- Do not mix unrelated domains into one file or grow a single file beyond ~200 lines.
- Do not state unverified commands or deployment steps as confirmed facts.

## TODOs

### 1. Study guides and analyze the project

- Follow Steps 1–2 of `convention-setup-guide.md` to detect tech stack, directory structure, and existing rule files (CLAUDE.md, AGENTS.md, .cursorrules, README, etc.).
- Identify build/test/lint/typecheck/dev-server commands within the verifiable scope.
- Do not run destructive or environment-dependent commands; record cautions instead.

**Acceptance Criteria**

- Tech stack and primary work units are identified.
- Verification commands and their limits are recorded.

### 2. Write and register Conventions

- Following Steps 3–6 of `convention-setup-guide.md`, write `context`, `conventions`, and any domain-specific rule files.
- Set appropriate `trigger` and `description` in each frontmatter.
- Register each file via `agentteams convention create --file <path>`.
- Verify the index with `agentteams convention download`.

**Acceptance Criteria**

- Reference documents the agent should read before any task are in place.
- `always_on` Conventions are limited to 2–3.
- All Convention files are registered on the server.

### 3. Compile Human Review Points

- Run through the Quality Checklist of `convention-setup-guide.md`.
- List decisions a human must confirm (tech stack accuracy, forbidden actions, test/deploy commands, decisions the agent should not make alone).

**Acceptance Criteria**

- Every Quality Checklist item is either checked off or marked as awaiting human review.
- Items requiring human attention are listed clearly.

## Success Criteria

- `.agentteams/rules/context.md` and `conventions.md` exist and are registered.
- The Quality Checklist of `convention-setup-guide.md` is satisfied.
- Open items remain explicit, not silently confirmed.
