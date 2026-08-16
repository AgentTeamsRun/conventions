# Plan Template (STANDARD Tier)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> This template is for **STANDARD complexity** plans (2–3 tasks · known, bounded scope).
> Set `--complexity STANDARD` at creation. V2 plans render this structured content directly in the web UI.
> Smaller scope? Use `plan-template-minimal.md`. Multi-wave work or any risk signal (schema / auth / billing / quota / deployment, cross-workspace edits, large diff, unfamiliar domain) means FULL — use `plan-template-full.md`. The tier rule is in `plan-authoring-guide.md` (**Plan Complexity**).

## TL;DR

> **In Plain Terms**: <!-- 1-2 sentences a non-expert (the requester/stakeholder) can read to confirm the plan matches their intent. State the problem and what visibly changes for the user. Write in the plan's language. NO file paths, code identifiers, or internal jargon (e.g. SSOT, projectId). See plan-authoring-guide.md → TL;DR Audience. -->
>
> **Quick Summary**: <!-- 1-2 sentence technical summary of what this plan achieves -->
>
> **Deliverables**:
>
> - <!-- file/module 1 — what changes -->
> - <!-- file/module 2 — what changes -->
>
> **Estimated Effort**: <!-- Short / Medium / Long -->
> **Dependencies**: <!-- blocking plan IDs if creating multiple plans, or "none" -->

---

## Context

### Original Request

<!-- What the user/requester asked for and why -->

### Research Findings

<!-- What you discovered from exploring the codebase, docs, or external sources -->

### Assumptions & Unknowns

<!--
Project-specific claims you could NOT verify against this repo — kept separate
from the confirmed facts above (see plan-authoring-guide.md → Grounding: Evidence
Over Memory). Do not blend these into the body as if confirmed. Write "none" if
every stack / command / path / symbol used in this plan was verified from the source.

Example:
- Assumed the test command is the package's default script — NOT verified against the manifest.
- Unsure whether <module> already exists; plan creates it, revisit if it does.
-->

### Conventions Referenced

<!--
Required at every tier. List the project convention files you actually consulted
while drafting this plan — do not guess. Same format as completion reports so the
platform can auto-link plan-stage convention usage to execution-stage usage.

Example:
- .agentteams/rules/<convention-name>.md
- .agentteams/rules/<convention-name>.md - <why it was relevant>
-->

---

## Work Objectives

### Core Objective

<!-- One sentence: what success looks like -->

### Concrete Deliverables

- <!-- file/module — description -->

### Definition of Done

- <!-- Criterion 1 -->
- <!-- Criterion 2 -->
- Completion Report is created, then Co-Action and Post-Mortem needs are reviewed immediately. `completion-report-guide.md` owns the criteria for each.

### Must Have

- <!-- Non-negotiable requirement -->

### Must NOT Have (Guardrails)

- <!-- Explicitly forbidden action -->

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed.

### QA Policy

- <!-- Tool/command mapping per this project's testing convention -->
- <!-- Test-first: list the tests that prove each Definition of Done item. For BUG_FIX, include a failing reproduction test that a later task makes pass. -->

---

## TODOs

> **Structured task contract**: Keep the exact `## TODOs` heading and numbered `### N. Task title` headings. The server parses dependencies from `Blocked By: Task N` or `Depends On: Task N`, and waves from `Parallel Group: Wave N`. Reference only task numbers that exist in this plan.

---

### 1. Task title

- **Parallel Group**: <!-- Wave 1; keep this label so the server can parse the wave -->
- **Blocked By**: <!-- none; keep this label so the server can parse dependencies -->

**What to do**:

  <!-- Detailed steps -->

**Must NOT do**:

- <!-- Forbidden action -->

**Required Conventions**:

  <!-- Project conventions the executing agent MUST read before starting this task. -->
  <!-- Use project-level rule files relevant to this task (for example, -->
  <!-- .agentteams/rules/<relevant-convention>.md). If none are required, write "none". -->

- <!-- convention file — why it's required for this task -->

**References**:

- <!-- file:lines — description -->

**Acceptance Criteria**:

- <!-- Verifiable criterion — the SSOT for this task being complete: the evidence required before marking the task `DONE`; prefer a test assertion -->

---

### 2. Task title

- **Parallel Group**: <!-- Wave 2; use Wave 1 instead when this task is independent of Task 1 -->
- **Blocked By**: <!-- Task 1; write none when this task is independent -->

**What to do**:

  <!-- Detailed steps -->

**Must NOT do**:

- <!-- Forbidden action -->

**Required Conventions**:

  <!-- Project conventions the executing agent MUST read before starting this task. -->
  <!-- Use project-level rule files relevant to this task (for example, -->
  <!-- .agentteams/rules/<relevant-convention>.md). If none are required, write "none". -->

- <!-- convention file — why it's required for this task -->

**References**:

- <!-- file:lines — description -->

**Acceptance Criteria**:

- <!-- Verifiable criterion — the SSOT for this task being complete: the evidence required before marking the task `DONE`; prefer a test assertion -->
