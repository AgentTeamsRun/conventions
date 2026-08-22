# Skill Package Guide (AgentTeams)

> ⚠️ This file is automatically deployed from the server. Do not edit it directly.

This guide is the canonical contract for **Skills** — capability packages that live alongside, but are separate from,
Conventions. For Convention authoring rules and the category decision table, see `convention-authoring-guide.md`.

## 1) Convention vs Skill

A Convention is **policy** the agent must comply with. A Skill is a **capability** the agent loads when it decides the
task calls for it.

|            | Convention                                      | Skill                                                    |
| ---------- | ----------------------------------------------- | -------------------------------------------------------- |
| Answers    | "What rules govern this work?"                  | "How do I perform this specific job?"                    |
| Shape      | Single markdown file                            | Package: `SKILL.md` + optional resources                 |
| Loading    | Trigger-driven (`always_on` / `model_decision`) | Model decides from `description`, then reads the package |
| Location   | `.agentteams/<category>/<name>.md`              | `.agentteams/skills/<slug>/SKILL.md`                     |
| Compliance | Mandatory when it applies                       | Advisory — it teaches a procedure                        |

🚫 A Skill must **not** restate project policy. If the content is a rule the agent must always obey, it belongs in a
`rules` Convention. Skills reference conventions; they do not copy them.

## 2) Package layout

```text
.agentteams/skills/<slug>/
├─ SKILL.md          # required entry file
├─ references/       # optional — supporting documents the skill points at
└─ scripts/          # optional — executable helpers the skill invokes
```

- `SKILL.md` is the only required file. A package without it is rejected on upload and on download.
- `references/` and `scripts/` are the only resource directories accepted today.
- Binary assets are not supported. A file that is not UTF-8 text is rejected — by the CLI when it collects the
  package, and by the server on upload. Keep packages text-only. The CLI skips well-known OS junk files
  (`.DS_Store`, `Thumbs.db`, `desktop.ini`, AppleDouble `._*`) during collection so a Finder or Explorer visit does
  not fail `skill push`.

### Resource file types

There is **no extension whitelist**. A resource file is judged by its location, its size, and its encoding — nothing
else. What follows is guidance, not enforcement.

- `references/` — supporting documents the entry file points at. `.md` is the default; reach for `.json` when the
  material is a structured example (a tool input payload, a fixture) that prose would distort.
- `scripts/` — helpers the skill tells the agent to run: `.sh` for shell, or a script source such as `.ts` / `.mjs`.
- The executable bit is **not preserved** through upload and download. Name the interpreter at the call site in
  `SKILL.md` — `bash scripts/<name>.sh`, not `./scripts/<name>.sh`.
- Anything that is neither a document the entry file cites nor a helper it runs usually belongs in `SKILL.md`
  itself, or nowhere.

### Frontmatter contract

`SKILL.md` starts with YAML frontmatter carrying exactly two required fields:

```markdown
---
name: <slug>
description: >-
  The situation that calls for this skill.
---

# <Human-readable title>
```

- `name` — required. Must equal the directory `<slug>`.
- `description` — required. This is the only text the model sees before deciding to load the package, and it is also
  rendered into the **Project Skill Index**, so every session pays for it. Write the **situation that calls for the
  skill**, not what the skill contains — the slug already names the subject. This mirrors the `description` rule in
  `convention-authoring-guide.md` §2-2.
  - ❌ `A skill to reference when modifying the AgentTeams MCP server; it covers tools, profiles, and the catalog.`
    The trailing "a skill to reference …" is filler, and the subject repeats the slug.
  - ✅ `When adding an agentteams_* tool, changing a tool input schema, or adjusting the tool profile/catalog.`
  - Open with the situation (`When ...`). Do not restate the slug or the title. Target **100 characters or fewer**;
    unlike convention descriptions there is no server-side cap, so the target is the only limit.
- Additional fields are ignored by the server. The published example skill in the public conventions mirror follows
  this same two-field contract; keep any change to the required set consistent with it.

### Naming and path rules

- `slug`: lowercase letters, digits, and hyphens only (`^[a-z0-9][a-z0-9-]*[a-z0-9]$`), 2–64 characters. No leading,
  trailing, or repeated hyphens.
- Resource paths are **relative to the package root**, use `/` as the separator, and must resolve inside the package.
- 🚫 Rejected by the **server**: absolute paths, `..` traversal, paths escaping the package root, duplicate paths,
  paths that collide case-insensitively (`Refs/a.md` vs `refs/a.md`), and symlinks.
- Resource files live under `references/` or `scripts/` only. Any root-level file other than `SKILL.md` is rejected.

### Size limits

| Limit                     | Value          | Applies to                                  |
| ------------------------- | -------------- | ------------------------------------------- |
| `SKILL.md` size           | 64 KB          | Entry file                                  |
| Single resource file size | 256 KB         | Each file under `references/` or `scripts/` |
| Files per package         | 50             | Including `SKILL.md`                        |
| Total package size        | 2 MB           | Sum of all file contents                    |
| Path length               | 200 characters | Relative path per file                      |

- Encoding is **UTF-8 text only**. The CLI rejects a file whose bytes are not valid UTF-8 or that contains a null
  byte while collecting the local package. The server rejects an upload whose content contains a null byte, a lone
  surrogate, or a Unicode replacement character (U+FFFD). Fastify's JSON parser replacement-decodes invalid UTF-8
  into U+FFFD rather than rejecting the body, so the server treats U+FFFD as evidence of that path — a legitimate
  U+FFFD in otherwise valid text is also refused.
- Every file records a `sha256` hash of its content. The package version is the hash of the sorted
  `(relativePath, sha256)` pairs, so an unchanged package produces an unchanged version.

## 3) Local sync ownership

- `.agentteams/skills/` belongs to `skill download` alone. Convention sync never sweeps or recreates that directory,
  so a convention download can never delete a skill package.
- Downloads are validated in a temporary directory and swapped in atomically. An interrupted sync leaves the previous
  package in place rather than a half-written one.
- Every file the CLI writes — including mirrors — is recorded in `.agentteams/skills.manifest.json`, so a later sync
  removes only files it wrote itself.

## 4) Runner discovery

Some agent CLIs discover skill packages from their own repository paths; others have no skill mechanism and reach
skills through the Skill Index in `.agentteams/convention.md`. Each engine carries one of four verdicts:

- `native` — the engine reads a project path itself.
- `index-reference` — no native discovery, so the engine reaches the package through the Skill Index in
  `.agentteams/convention.md`. **Skills still work here** — the agent opens `.agentteams/skills/<slug>/SKILL.md`
  itself instead of the engine loading it.
- `unsupported` — no native discovery **and** no index path either. No measured engine is in this state; every agent
  reads `convention.md`, so a verdict of `index-reference` is what "the engine has no skill feature" looks like here.
- `unverified` — not measured yet. ⚠️ Never treat `unverified` as `unsupported`; state that it was not measured.

⚠️ **A missing `skill` subcommand is not evidence of `unsupported`.** Discovery is usually implicit — the engine reads
a directory and never exposes a command for it. Every verdict below comes from a probe package installed in a
throwaway project, one candidate path per project, read back through the engine's own inspection surface or through a
non-interactive prompt. The prompt method is calibrated against `CLAUDE_CODE`: it returns the probe token where the
package is discoverable and `NO-SKILL` in a project with no package, so a negative result is a real negative.

| RunnerType    | Verdict         | Project-local path                                                                                   | Evidence                                                                                                                                                                                                            |
| ------------- | --------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CLAUDE_CODE` | native          | `.claude/skills/<slug>/SKILL.md`                                                                     | Prompt probe returned the probe token; control project returned `NO-SKILL` (2026-08-15)                                                                                                                             |
| `COPILOT_CLI` | native          | `.github/skills/`, `.agents/skills/`, `.claude/skills/`                                              | `copilot skill --help` names all three project sources; `copilot skill list` reported the probe (1.0.78, 2026-08-15)                                                                                                |
| `AMP`         | native          | `.agents/skills/<slug>/`                                                                             | `amp skill list` enumerated a project-local `.agents/skills/*` package (build 0.0.1786715939, 2026-08-15)                                                                                                           |
| `CODEX`       | native          | `.agents/skills/<slug>/`, `.codex/skills/<slug>/`                                                    | `codex debug prompt-input` carried the probe in the model-visible prompt from those two paths only (0.147.0, 2026-08-15)                                                                                            |
| `OPENCODE`    | native          | `.agents/skills/<slug>/`, `.claude/skills/<slug>/`, `.opencode/skills/<slug>/`                       | `opencode debug skill` resolved the probe to each of those three paths (1.18.18, 2026-08-15)                                                                                                                        |
| `GROK_BUILD`  | native          | `.agents/skills/<slug>/`, `.claude/skills/<slug>/`, `.cursor/skills/<slug>/`, `.grok/skills/<slug>/` | `grok inspect` listed the probe as `project` from each of those four paths (1.0.3, 2026-08-15)                                                                                                                      |
| `CURSOR_CLI`  | index-reference | —                                                                                                    | Prompt probe returned `NO-SKILL` for `.cursor/`, `.claude/`, and `.agents/` packages. The binary carries those path strings, so support may be gated or unreleased — re-measure on upgrade (2026.07.16, 2026-08-15) |
| `KIRO_CLI`    | index-reference | —                                                                                                    | Prompt probe returned `NO-SKILL` (`kiro-cli` 2.18.1, 2026-08-15). ⚠️ The runner executes `kiro-cli`, not the `kiro` IDE launcher — probe the former                                                                 |
| `KIMI_CLI`    | unverified      | —                                                                                                    | Installed (0.29.0) and carries `--skills-dir`, documented as overriding "auto-discovered user and project directories". Runtime probe blocked: the CLI was not signed in on the measuring machine (2026-08-15)      |
| `ANTIGRAVITY` | unverified      | —                                                                                                    | Binary absent on the measuring machine (2026-08-15)                                                                                                                                                                 |

`.agents/skills/` is read by five of the six measured engines. It is the single highest-value mirror path.

## 5) Mirror policy

`.agentteams/skills/<slug>/` is the single source of truth. The CLI **copies** the package into the native paths above
(copies, not symlinks — symlinks break on Windows, in CI, and in git).

| Priority | Mirror path                      | Read by                                        | Rule                                                                   |
| -------- | -------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------- |
| 1        | `.agents/skills/<slug>/SKILL.md` | COPILOT_CLI, AMP, CODEX, OPENCODE, GROK_BUILD  | Written by default — vendor-neutral path with the widest support       |
| 2        | `.claude/skills/<slug>/SKILL.md` | CLAUDE_CODE, COPILOT_CLI, OPENCODE, GROK_BUILD | Written only when a `.claude/` marker directory exists                 |
| 3        | `.github/skills/<slug>/SKILL.md` | Copilot on github.com                          | Written only when `.github/` exists                                    |
| —        | `index-reference` engines (§4)   | CURSOR_CLI, KIRO_CLI                           | No mirror — they follow the Skill Index in `.agentteams/convention.md` |

- **Detection gating** — a mirror is written only for clients whose marker directory already exists. `.agents/` is the
  one exception and is always written. `--skill-targets=agents,claude,github,none` overrides detection explicitly.
- **gitignore by default** — mirrors are derived artifacts. Use `skill download --commit-mirrors` to opt a project into
  committing them.
- 🚫 Never create, modify, or delete skills under the user's home directory (for example `~/.claude`, `~/.copilot`,
  `~/.agents`, `~/.config/agents`). Mirrors are repository-local only.
- ⚠️ Writing both `.agents/` and `.claude/` can make a single engine load the same skill twice. Check the engine's
  loaded-skill event (it reports each skill's source and path) and skip the `.claude/` mirror when the duplicate
  appears.

## 6) Commands and when to run them

Nothing fetches skill packages for you. No pre-run hook, no runner step, no side effect of the convention sync —
`.agentteams/skills/` has a separate owner and only these commands write to it. A fresh clone or a new git worktree
therefore starts with **no packages**, which is exactly when the Skill Index in `.agentteams/convention.md` starts
pointing at files that do not exist. You are the one who closes that gap.

### At the start of a session

```bash
agentteams session sync
```

One command covers both surfaces: it checks conventions and skills, downloads only what changed, and reports which
always-on files you must re-read. It fetches the packages **and** writes the engine mirrors from §5 in the same step.

- ⚠️ **The skill fetch is gated on `agentteams skill status`, and that gate is not optional.**
  `agentteams skill download` overwrites every local package without consulting `updateAvailable`, so calling it
  ungated erases a package you are still authoring. Running the pair by hand is fine, but keep that order — `status`
  first, `download` only when `updateAvailable` is true.
- `unknown command 'skill'` means the installed CLI predates skill packages. Skip silently and continue — that is the
  signal, not an error to report.
- The mirror for the engine you are running is written even when its marker directory is absent, as long as the
  session carries a runner type. In a session started outside the runner there is no runner type to read, so a
  marker-gated mirror is skipped: working as `CLAUDE_CODE` in a repository with no `.claude/`, run
  `agentteams skill download --skill-targets agents,claude`. Match the target to your own engine's row in §4; do not
  guess targets for engines you are not running.

### When you need a skill during this session

Open `.agentteams/skills/<slug>/SKILL.md` yourself, using the Skill Index in `.agentteams/convention.md` to find it.
A download is enough to make that possible. Engines that discover skills natively build that inventory when the
session starts, so a package fetched mid-session is loaded natively **from the next session on** — do not wait for it
to appear in this one.

### When you author a skill

```bash
# Create a skill package from a local directory (dry-run without --apply)
agentteams skill create --dir .agentteams/skills/{slug} --apply
```

Writing the files is not the end of the job. Until `skill create --apply` runs, the package exists only on this
machine — the web list, the Skill Index, and every other checkout stay empty.

```bash
# Inspect what is registered and whether the local copy is current
agentteams skill list
agentteams skill status
```
