# Project: ai-project-planning

This repository is the source for the **project-planning** plugin — a Claude/Codex plugin that helps you manage a project: capture brain dumps as notes, see what's in motion across Jira/Confluence/Slack/local `plans/`, plan the week, reconcile drift, and review PRs against documented conventions.

`CLAUDE.md` at the repo root is a symlink to this file. Update `AGENTS.md`; Claude Code reads either name.

## Repository conventions

- **`plugin/skills/`** — single source of truth for all skill content the plugin ships. Every published skill lives under `plugin/skills/<skill-name>/SKILL.md`, with reference documents in the same directory (e.g., `plugin/skills/planning/confluence.md`). Do not write skill content anywhere else; do not duplicate skill content between `plugin/skills/` and `.claude/skills/`.
- **`plugin/.claude-plugin/plugin.json`** — Claude Code manifest.
- **`plugin/.codex-plugin/plugin.json`** — Codex manifest (`"skills": "./skills/"`).
- **`.claude-plugin/marketplace.json`** — top-level marketplace metadata used by Claude Code's `plugin marketplace add` flow.
- **`docs/proposed/`** — in-progress planning documents. One markdown file per plan; phases tracked as `## - [ ]` / `## - [x]` checkboxes inside the file.
- **`docs/completed/`** — planning documents whose phases are all checked off. Move plans here once finished.
- **`docs-site/`** — public GitHub Pages site. Deployed via `.github/workflows/pages.yml`.
- **`.claude/skills/`** — dev-time-only skills used while authoring this plugin. **Nothing in here ships.** Anything that belongs in the published plugin goes under `plugin/skills/` instead.
- **`private-context.md`** — gitignored, local-only references. Plans may reference it; nothing in the public repo depends on it.

## Shipped skills

- **`planning`** — one skill, four modes (capture / status / sprint / reconciliation). Reference documents in `plugin/skills/planning/`: `notes.md`, `sprint.md`, `jira.md`, `confluence.md`, `slack.md`, `local.md`.
- **`bootstrap`** — onboards an existing project (folders, file moves, `AGENTS.md` content). Single SKILL.md.
- **`docs-update`** — sweeps a configured Slack channel and proposes documentation updates.
- **`pr-review`** — reviews a GitHub PR against the project's documented conventions; outputs PR-side and docs-side findings.

## Writing skills in this repo

- Use the `skill-creator` skill for frontmatter conventions, description tuning, and the progressive-disclosure structure (SKILL.md + per-variant reference docs).
- Descriptions should be "pushy": name explicit user trigger phrases *and* the implicit shape that should fire the skill. The skill description is the primary triggering mechanism.
- Channel IDs, page IDs, project keys, `cloudId`s, repo URLs, and Slack workspace URLs come from the consuming project's `AGENTS.md` — never hardcode them in a skill body.
- Defer external-source read/write mechanics to the matching reference document in `plugin/skills/planning/<source>.md` rather than redefining them in each skill that needs them.
- Read-by-default. Any external write — Confluence edit, Jira transition, Slack send, PR comment, local file write — requires explicit user confirmation. (This is the contract; don't restate it as a feature in user-facing docs.)

## Worked example: planning content in a consuming project's `AGENTS.md`

A project that uses this plugin declares its setup in `AGENTS.md`. There's no required heading — put it under whatever fits the file you already have. The shape below is illustrative:

```markdown
## Project Planning

**Planning sources:** Jira (project key `EXAMPLE`, epic `EXAMPLE-100`), Confluence (space `EX`, root page id `123456`), local `plans/` for in-incubation ideas. Source of truth for shipped work is Jira/Confluence; `plans/` is supplementary.

**Slack channel:** `C0123456789` (canonical discussion channel).

**GitHub repos:**
- `acme-org/example-service` — primary backend
- `acme-org/example-web` — web client

**Plugin skills enabled:**
- `planning` — capture brain dumps, show what's in motion, plan the week, reconcile drift.
- `pr-review` — review PRs against the documented conventions.
```

Free-form markdown. The skills read it in natural language; they don't validate it.

## Useful commands

- Load the plugin locally without installing: `claude --plugin-dir ./plugin` (point at the directory holding `.claude-plugin/plugin.json`, not the repo root)
- List the shipped skills: `ls plugin/skills`
- Check the marketplace metadata: `cat .claude-plugin/marketplace.json`
