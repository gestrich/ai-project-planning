# Project Planning

A plugin for Claude Code and OpenAI Codex that gives you a consistent gateway into any project for capturing notes, navigating plans, running sprints, syncing with external sources of truth (Jira / Confluence / Slack), and reviewing PRs against documentation. Drop the plugin into a new project, declare a small project-planning section in the project's `AGENTS.md`, and the skills figure out which sources apply and read only what's relevant.

## Installation

### Claude Code

1. Add the marketplace:
   ```bash
   claude plugin marketplace add https://github.com/gestrich/ai-project-planning
   ```

2. Install the plugin:
   ```bash
   claude plugin install project-planning@gestrich-project-planning --scope user
   ```
   > Use `--scope user` to install system-wide for all projects.

3. Restart Claude Code if necessary.

**Uninstall:**
```bash
claude plugin uninstall project-planning@gestrich-project-planning --scope user
```

### OpenAI Codex

1. Add the marketplace:
   ```bash
   codex plugin marketplace add gestrich/ai-project-planning
   ```

2. Open a Codex session and install via the `/plugins` menu — select **project-planning** → **Install plugin**.
   > There is no `codex plugin install` CLI command yet; installation must be done through the interactive TUI.

3. After installing, the plugin can be toggled without the TUI by editing `~/.codex/config.toml`:
   ```toml
   [plugins."project-planning"]
   enabled = true   # set to false to disable without uninstalling
   ```

**Uninstall:** Remove the `[plugins."project-planning"]` entry from `~/.codex/config.toml` and run:
```bash
codex plugin marketplace remove gestrich-project-planning
```

### Local Testing (Claude Code)

```bash
claude --plugin-dir ~/path/to/ai-project-planning/plugin
```

`--plugin-dir` points at the directory that holds `.claude-plugin/plugin.json` — that's `./plugin` inside this repo, not the repo root.

## Skills

Skills trigger from natural-language prompts. You don't need to name the skill — the descriptions are tuned to fire on the kinds of phrases you'd actually say.

| Skill | Example trigger | What it does |
|-------|-----------------|--------------|
| `notes` | "here's my brain dump" / pasting a long transcript | Writes the transcript verbatim to `notes/YYYY-MM-DD-<slug>.md`. No cleanup, no rewriting. |
| `plan` | "what's the plan?", "what's next?", "catch me up" | Reads `AGENTS.md` to find which sources apply (Jira, Confluence, local `plans/`, Slack-as-context), loads only matching reference docs, synthesizes a brief view. |
| `plan` (reconciliation mode) | "sync plans", "what's drifted?", "what should graduate?" | Same skill, different mode — compares local `plans/` against external sources and reports graduation candidates, drift, and stale items. |
| `sprint` | "what should I do next?", "plan my sprint", "what's on deck?" | Acts as a scrum-master coach: gathers context, asks challenging questions, drafts a sprint file at `plans/sprints/YYYY-MM-DD.md` after confirmation. |
| `bootstrap` | "bootstrap this project", "set up project planning here" | Surveys an existing project, proposes a plan-of-changes (folders, file moves, `AGENTS.md` section), applies it after confirmation. |
| `docs-update` | "update the docs", "what's missing from the docs?" | Sweeps the configured Slack channel for the last week, cross-references the docs, proposes updates. Stages a write or drafts a message to the owning team. |
| `pr-review` | "review this PR", "is this PR aligned with our docs?" | Fetches the PR diff, cross-references the project's documented conventions, produces both PR-side findings and doc-update suggestions. |

All skills are read-by-default. Any external write (Jira transition, Confluence edit, Slack send, PR comment, local file change) requires explicit confirmation.

## Configuration

A project opts into this plugin by adding a `## Project Planning` section to its `AGENTS.md`. Free-form markdown — there's no enforced schema. The skills read it in natural language.

A minimal example:

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

**Project-specific notes:** The web client mirrors the backend's REST contract; PR reviews should check both sides when an endpoint changes.
```

The skills don't validate this section — they read it like a teammate would. Add what's useful; leave out what isn't.

Folder layout lives in the project, not the plugin:

```
your-project/
├── AGENTS.md            # contains the section above
├── notes/               # raw transcripts (created by `notes`)
└── plans/               # local planning artifacts
    ├── plan.md          # manifest pointing at the other plan files
    └── sprints/         # one file per sprint, dated by start-of-week
```

Run the `bootstrap` skill in an existing project to get this scaffolding set up.

## Troubleshooting

### Slash commands not showing in terminal

Skill commands may not appear in terminal autocomplete but will show up in the Claude Code VSCode extension. Use the VSCode extension for the best discovery experience.

### Updating the plugin

```
claude plugin marketplace update gestrich-project-planning
claude plugin uninstall project-planning@gestrich-project-planning
claude plugin install project-planning@gestrich-project-planning --scope user
```

Asking Claude to update the plugin via the CLI does not always work — it may report success but won't actually pull the latest version. Uninstall and reinstall, or use the Claude Code UI.

### Skills not accessible after installation

If skills aren't triggering after installation, try uninstalling and reinstalling with explicit scope:

```
/plugin uninstall project-planning@gestrich-project-planning
/plugin install project-planning@gestrich-project-planning --scope user
```

### Plugin not available after adding marketplace

Sometimes the plugin won't appear as an installation option even after adding the marketplace. Work around it by manually enabling the plugin in your Claude Code configuration.

Create or edit `~/.claude/config.json` and add:

```json
{
  "enabledPlugins": {
    "project-planning@gestrich-project-planning": true
  }
}
```

Then restart Claude Code.

## License

MIT License — see [LICENSE](plugin/LICENSE) for details.
