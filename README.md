# Project Planning

A plugin for Claude Code and OpenAI Codex that helps you manage a project — capture brain dumps as notes, see what's in motion across Jira / Confluence / Slack / a local `plans/` folder, plan the week, reconcile local plans against external sources, and review PRs against the project's documented conventions.

The skills read your `AGENTS.md` to discover which planning sources your project uses (Jira, Confluence, a local `plans/` folder, Slack-as-context, GitHub repos, or some combination). You write it in whatever shape makes sense for the project — there's no required heading and no enforced schema — and the skills parse it like a teammate would.

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
| `planning` | "here's my brain dump" / "what's the plan?" / "what should I do this week?" / "what's drifted?" | One skill, four modes. **Capture** writes raw transcripts to `notes/YYYY-MM-DD-<slug>.md`. **Status** synthesizes what's in motion across the configured sources. **Sprint** acts as a scrum-master coach and drafts a sprint file at `plans/sprints/YYYY-MM-DD.md`. **Reconciliation** compares local `plans/` against Jira/Confluence and reports drift, graduation candidates, and stale items. |
| `bootstrap` | "bootstrap this project", "set up project planning here" | Surveys an existing project, proposes a plan-of-changes (folders, file moves, `AGENTS.md` content), applies it after confirmation. |
| `docs-update` | "update the docs", "what's missing from the docs?" | Sweeps the configured Slack channel for the last week, cross-references the docs, proposes updates. Stages a write or drafts a message to the owning team. |
| `pr-review` | "review this PR", "is this PR aligned with our docs?" | Fetches the PR diff, cross-references the project's documented conventions, produces both PR-side findings and doc-update suggestions. |

## Configuration

The skills read your project's `AGENTS.md` and look for planning-relevant information in whatever shape you've written it. No required heading, no enforced schema — put the content under whatever heading fits your file. Skills look for these signals:

- A Jira project key, epic ID, or board URL.
- A Confluence space, page ID, or page URL.
- A Slack channel name or ID flagged as project context.
- A local `plans/` folder declared as a source.
- A list of GitHub repos belonging to the project.
- A list of which plugin skills the project uses, ideally with a one-line "what it's for / when to use it" per skill.

An illustrative shape (use it as a starting point, not a template to follow exactly):

```markdown
## Project Planning

**Planning sources:** Jira (project key `EXAMPLE`, epic `EXAMPLE-100`), Confluence (space `EX`, root page id `123456`), and the local `plans/` folder for in-incubation ideas. Source of truth for shipped work is Jira/Confluence; `plans/` is supplementary.

**Slack channel:** `C0123456789` (canonical discussion channel).

**GitHub repos:**
- `acme-org/example-service` — primary backend
- `acme-org/example-web` — web client

**Plugin skills enabled:**
- `planning` — capture brain dumps, show what's in motion, plan the week, reconcile drift. Use when I paste a transcript, ask "what's the plan", "what should I do this week", or "what's drifted".
- `pr-review` — review PRs against the documented conventions. Use when I paste a PR URL.

**Project-specific notes:** The web client mirrors the backend's REST contract; PR reviews should check both sides when an endpoint changes.
```

Folder layout lives in the project, not the plugin:

```
your-project/
├── AGENTS.md            # planning content lives somewhere in here
├── notes/               # raw transcripts (created by capture mode)
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
