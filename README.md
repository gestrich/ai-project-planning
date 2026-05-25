# Project Planning

A plugin for Claude Code and OpenAI Codex that helps you manage a project — capture brain dumps, see what's in motion, plan the week, and review PRs against documented conventions.

## How it works

Each project gets its own **local repo**. The repo represents your project: it's where your notes and local plans live, and it's the gateway the skills operate through. If you don't have one yet, the `bootstrap` skill creates and organizes it for you.

From there, you chat about your project the way you would with a teammate — paste a brain dump, ask what's next, plan the week, hand over a PR for review. The skills know about your project's external sources of truth (Confluence pages, Jira issues, Slack channels, GitHub repos) and read from them on demand, so the local repo doesn't duplicate what already lives there. It holds what's *not* in those systems: in-incubation ideas, personal items, raw transcripts.

Discovery of those sources is driven by your project's `AGENTS.md` — written in whatever shape makes sense. See [Configuration](#configuration) for the details.

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
| `bootstrap` | "bootstrap this project", "set up project planning here" | Surveys an existing project, proposes a plan-of-changes (folders, file moves, `AGENTS.md` content), applies it after confirmation. |
| `plans` | "here's my brain dump" / "what's the plan?" / "what should I do this week?" / "what's drifted?" | One skill, four modes. **Capture** writes raw transcripts to `notes/YYYY-MM-DD-<slug>.md`. **Status** synthesizes what's in motion across the configured sources. **Sprint** acts as a scrum-master coach and drafts a sprint file at `sprints/YYYY-MM-DD.md`. **Reconciliation** compares local `plans/` against Jira/Confluence and reports drift, graduation candidates, and stale items. |
| `docs-update` | "update the docs", "what's missing from the docs?" | Sweeps the configured Slack channel for the last week, cross-references the docs, proposes updates. Stages a write or drafts a message to the owning team. |
| `pr-review` | "review this PR", "is this PR aligned with our docs?" | Fetches the PR diff, cross-references the project's documented conventions, produces both PR-side findings and doc-update suggestions. |

## Configuration

The `bootstrap` skill sets up everything below the first time you run it; you usually don't write it by hand.

Each project's `AGENTS.md` carries the planning content — which Jira project, Confluence space, Slack channel, and GitHub repos belong to it. Free-form markdown, no enforced schema; the skills read it like a teammate would. See the [Configuration page on the site](https://gestrich.github.io/ai-project-planning/configuration.html) for the full reference and an example shape.

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
