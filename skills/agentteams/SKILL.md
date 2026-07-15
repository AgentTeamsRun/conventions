---
name: agentteams
description: >-
  Use AgentTeams as the governance layer for work performed inside Orca
  worktrees. Use when the user says "AgentTeams", "start the plan", "execute
  the plan", "finish the plan", or asks an Orca agent to follow team
  conventions, report task progress, or produce an AgentTeams completion
  report.
---

# AgentTeams Governance

AgentTeams provides the shared team conventions and auditable workflow that
govern work performed by agents in Orca. Use this skill to connect a new Orca
worktree to the existing AgentTeams project, then defer to the project's
deployed convention.

## Bootstrap an Orca Worktree

Install the AgentTeams CLI if it is not already available:

```bash
npm install -g @agentteams/cli
```

In each new linked worktree, first move to the Git top-level directory returned
by `git rev-parse --show-toplevel`. Run the bootstrap once from that worktree
root:

```bash
agentteams init
```

Before this bootstrap, the worktree-local `.agentteams/convention.md` and
deployed guides have not been materialized. In a linked worktree,
`agentteams init` connects the worktree to the main checkout's existing
AgentTeams configuration without repeating project setup.

## Follow the Project Convention

After bootstrap, read `.agentteams/convention.md` completely before doing any
work and follow it as the single source of truth. It defines the current plan,
task, verification, reporting, and completion rules; do not recreate those
rules in this skill.
