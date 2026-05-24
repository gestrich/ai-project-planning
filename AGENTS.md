# Project: ai-project-planning

This repository is the source for the **project-planning** plugin — a Claude/Codex plugin that ships skills for capturing notes, navigating plans, planning sprints, syncing with Jira/Confluence/Slack, and reviewing PRs against documentation.

`CLAUDE.md` at the repo root is a symlink to this file. Update `AGENTS.md`; Claude Code reads either name.

## Repository conventions

- **`plugin/skills/`** — single source of truth for all skill content the plugin ships. Every published skill lives under `plugin/skills/<skill-name>/SKILL.md`, with reference documents in the same directory (e.g., `plugin/skills/plan/confluence.md`). Do not write skill content anywhere else; do not duplicate skill content between `plugin/skills/` and `.claude/skills/`.
- **`plugin/.claude-plugin/plugin.json`** — Claude Code manifest.
- **`plugin/.codex-plugin/plugin.json`** — Codex manifest (`"skills": "./skills/"`).
- **`.claude-plugin/marketplace.json`** — top-level marketplace metadata used by Claude Code's `plugin marketplace add` flow.
- **`docs/proposed/`** — in-progress planning documents. One markdown file per plan; phases tracked as `## - [ ]` / `## - [x]` checkboxes inside the file.
- **`docs/completed/`** — planning documents whose phases are all checked off. Move plans here once finished.
- **`.claude/skills/`** — dev-time-only skills used while authoring this plugin. **Nothing in here ships.** Anything that belongs in the published plugin goes under `plugin/skills/` instead.
- **`private-context.md`** — gitignored, local-only references (paths to reference skills on Bill's machine, validation-target project names). Plans may reference it; nothing in the public repo depends on it.

## Writing skills in this repo

- Use the `skill-creator` skill for frontmatter conventions, description tuning, and the progressive-disclosure structure (SKILL.md + per-variant reference docs).
- Descriptions should be "pushy": name explicit user trigger phrases *and* the implicit shape that should fire the skill. The skill description is the primary triggering mechanism.
- Skills in this plugin are project-*agnostic*. Channel IDs, page IDs, project keys, `cloudId`s, repo URLs, and Slack workspace URLs come from the consuming project's `AGENTS.md` — never hardcode them in a skill body.
- Defer external-source read/write mechanics to the matching reference document in `plugin/skills/plan/<source>.md` rather than redefining them in each skill that needs them.
- Read-by-default: any external write (Confluence edit, Jira transition, Slack send, PR comment, local file write) requires explicit user confirmation.

## Worked example: the project-planning section in `AGENTS.md`

This repo is itself a project, and a project that uses the project-planning plugin declares its setup with a `## Project Planning` section. This repo doesn't need that section (the plugin doesn't plan itself with the plugin), but the example below — copied from the README — is the shape consuming projects should mirror:

```markdown
## Project Planning

**Planning sources:** Jira (project key `EXAMPLE`, epic `EXAMPLE-100`), Confluence (space `EX`, root page id `123456`), and the local `plans/` folder for in-incubation ideas. Source of truth for shipped work is Jira/Confluence; `plans/` is supplementary.

**Slack channel:** `C0123456789` (canonical discussion channel).

**GitHub repos:**
- `acme-org/example-service` — primary backend
- `acme-org/example-web` — web client

**Plugin skills enabled:**
- `notes` — dump raw voice transcripts into `notes/`. Use when I paste a brain-dump.
- `plan` — show the current direction. Use when I ask "what's the plan" or "what's next".
- `sprint` — plan the week. Use when I ask "what should I do this week".
- `pr-review` — review PRs against the documented conventions. Use when I paste a PR URL.
```

Free-form markdown, not enforced YAML. The skills read it in natural language.

## Useful commands

- Load the plugin locally without installing: `claude --plugin-dir .`
- List the shipped skills: `ls plugin/skills`
- Check the marketplace metadata: `cat .claude-plugin/marketplace.json`
