# Plan Template (Full Tier)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

> This template is for **Full tier** plans (4+ tasks, multi-wave, unfamiliar domain).
> For Minimal or Standard plans, refer to the Plan Tiers section in `plan-guide.md` and extract only the sections you need.

## TL;DR

> **Quick Summary**: <!-- 1-2 sentence summary of what this plan achieves -->
>
> **Deliverables**:
>
> - <!-- file/module 1 — what changes -->
> - <!-- file/module 2 — what changes -->
>
> **Estimated Effort**: <!-- Short / Medium / Long -->
> **Type**: <!-- FEATURE / BUG_FIX / ISSUE / REFACTOR / CHORE -->
> **Parallel Execution**: <!-- YES / NO -->
> **Critical Path**: <!-- Task X → Task Y → ... -->
> **Dependencies**: <!-- blocking plan IDs if creating multiple plans, or "none" -->

---

## Context

### Original Request

<!-- What the user/requester asked for and why -->

### Interview Summary

<!-- Key decisions from Q&A with the requester (if applicable) -->

### Research Findings

<!-- What you discovered from exploring the codebase, docs, or external sources -->

### Metis Review

<!-- Gaps identified by Metis review (or self-check results) -->

---

## Work Objectives

### Core Objective

<!-- One sentence: what success looks like -->

### Concrete Deliverables

- <!-- file/module — description -->

### Definition of Done

- <!-- Criterion 1 -->
- <!-- Criterion 2 -->

### Must Have

- <!-- Non-negotiable requirement -->

### Must NOT Have (Guardrails)

- <!-- Explicitly forbidden action -->

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed.

### QA Policy

- <!-- Tool/command mapping: API→typecheck+test, CLI→test, Web→build+playwright -->

---

## Execution Strategy

### Parallel Execution Waves

~~~
Wave 1:
├── Task 1: description [category]
└── Task 2: description [category]

Wave 2 (Wave 1 complete):
└── Task 3: description [category]
~~~

---

## TODOs

---

- 1. Task title

  **What to do**:

  <!-- Detailed steps -->

  **Must NOT do**:

  - <!-- Forbidden action -->

  **Recommended Agent Profile**:

  - **Category**: `<!-- category -->`
    - Reason: <!-- why this category -->
  - **Skills**: <!-- skill list or none -->

  **Parallelization**:

  - **Can Run In Parallel**: <!-- YES / NO -->
  - **Parallel Group**: <!-- Wave N -->
  - **Blocks**: <!-- Task N or none -->
  - **Blocked By**: <!-- Task N or none -->

  **References**:

  - <!-- file:lines — description -->

  **Acceptance Criteria**:

  - <!-- Verifiable criterion -->

  **QA Scenarios**:

  ~~~
  Scenario: description
    Tool: Bash
    Steps:
      1. command
    Expected Result: what success looks like
    Failure Indicators: what failure looks like
    Evidence: .sisyphus/evidence/task-N-name.txt
  ~~~

  **Commit**: <!-- YES / NO -->

  - Message: `<!-- type(scope): description -->`
  - Files: <!-- file list -->
  - Pre-commit: `<!-- verification command -->`

---

## Final Verification Wave

- F1. **Final check** — `quick`

  1. <!-- verification command 1 -->
  2. <!-- verification command 2 -->

  Output: `<!-- result format -->`

---

## Commit Strategy

- **Task 1**: `<!-- commit message -->`
- **Task 2**: `<!-- commit message -->`

## Success Criteria

~~~bash
# <!-- description -->
<!-- command -->
# Expected: <!-- result -->
~~~

---

## QA Scenario Examples

### Good (specific, verifiable)

~~~
Scenario: API returns 404 for non-existent plan
  Tool: Bash
  Steps:
    1. curl -s -o /dev/null -w "%{http_code}" http://localhost:3001/api/plans/nonexistent-id
  Expected Result: 404
  Failure Indicators: 200 or 500
  Evidence: .sisyphus/evidence/plan-404.txt
~~~

### Bad (vague, unverifiable)

~~~
Scenario: API works correctly
  Tool: Manual
  Steps:
    1. Test the API
  Expected Result: It works
~~~
