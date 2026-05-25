---
name: bootstrap
description: Onboard an existing project to the project-planning plugin's conventions — survey what's already there (notes, plans, AGENTS.md, commit history hints of Jira/Confluence/Slack usage), then propose and apply a structured plan of changes (folders to create, files to move, AGENTS.md content to add, planning sources to declare, plugin skills to enable). Use this skill whenever the user says "bootstrap this project", "set up project planning here", "make this project work with the planning plugin", "onboard this repo to project planning", "get this project ready for the planning skills", "wire this up to project planning", or anything along the lines of "land me in a new repo and make it usable by these skills". Trigger even when the user does not name the skill explicitly — if they're asking how to get a fresh project into shape so the other skills will work, this is the right home. Always shows the plan-of-changes for confirmation before writing anything; never silently edits.
---

# bootstrap

The on-ramp for the project-planning plugin. Bootstrap looks at an existing project, figures out what's already there, and proposes the minimum set of changes needed to get it into a shape the rest of the plugin's skills (`planning`, `docs-update`, `pr-review`) can work with. It never edits silently — every change goes through an explicit plan-of-changes the user signs off on first.

## When to trigger

Fire on any phrasing that asks for project onboarding to the planning convention:

- "bootstrap this project", "bootstrap the project for planning"
- "set up project planning here", "set this project up for the planning plugin"
- "make this project work with the planning plugin", "wire this repo up to the planning skills"
- "onboard this repo", "get this project ready for planning"
- A user landing in a fresh repo and asking what they need to add so the other skills will work.

If the user has already bootstrapped the project (planning content already exists in `AGENTS.md`), still trigger — but treat the run as a *refresh*: surface what's there, propose deltas (e.g., a Slack channel they've started using but haven't declared), and don't duplicate content.

## Behavior

The skill runs in three phases: **survey**, **propose**, **apply**. Don't collapse them — the user needs to see the proposed plan before any edits happen.

### 1. Survey

Read the project's current state. This is read-only.

- **Determine the project root.** Usually the current working directory; if ambiguous, ask.
- **Check for `AGENTS.md`** at the root. If it exists, read it and look for any existing planning-relevant content (declared planning sources, skill enablement list, etc.) under whatever heading or shape the user has it in. (`CLAUDE.md` may be a symlink to `AGENTS.md` — follow it.)
- **Check for existing planning folders**: `notes/`, `plans/`, `plans/sprints/`. Note what's there and what's missing.
- **Check for stray planning artifacts** that should probably move into the convention — e.g., a top-level `NOTES.md`, a `TODO.md`, a `planning/` directory, dated transcript files at the root.
- **Best-effort source-of-truth detection** (never assert — these are hints to ask the user about):
  - **Jira**: grep recent commit messages and PR descriptions for issue keys (e.g., `ABC-123`); check `README.md` and any `CONTRIBUTING.md` for Jira links; look for Jira URLs in `.github/` templates.
  - **Confluence**: scan `README.md` and docs for `*.atlassian.net/wiki/` URLs.
  - **Slack**: look for `slack.com/archives/` links in docs or commit messages.
  - **GitHub**: read `git remote -v` to enumerate the repo(s) the project owns. If the project spans multiple repos, ask the user which others belong to it.
- Use `date +%Y-%m-%d` if a date is needed; never guess it.

Keep the survey output short — a bulleted summary the user can scan, not a wall of text.

### 2. Propose

Produce a written plan-of-changes. Show it inline. Do not write anything yet.

The plan must cover, in order:

1. **Folders to create** — typically some subset of `notes/`, `plans/`, `plans/sprints/`. Skip ones that already exist.
2. **Existing files to move** — any stray planning artifacts found during survey, with proposed destinations. Always propose `git mv` (preserve history) rather than copy-delete.
3. **`AGENTS.md` changes** — either:
   - Add planning content to an existing `AGENTS.md` (pick a heading and location that fits how the user already structured it; don't force a particular section name), or
   - Create `AGENTS.md` (and the `CLAUDE.md` symlink to it) if neither exists, or
   - Propose targeted edits if planning content is already present but missing pieces (e.g., a newly-confirmed Slack channel).
4. **Planning sources to declare** — based on the survey hints, confirmed by the user. List Jira project keys / epic IDs, Confluence space + page IDs, Slack channel IDs, and whether `plans/` is the source of truth or supplementary.
5. **GitHub repos to declare** — the current repo plus any others the user names.
6. **Plugin skills to enable** — see the skill-enablement section below; this is the most important piece.

Ask any clarifying questions inline before finalizing the plan. Common ones: "is this Jira project the source of truth or just a place issues get filed?", "do you want `plans/` for personal in-incubation work, or as the canonical plan?", "this commit references `ABC-123` — is that the project key you want declared?". When the survey turned up nothing for a category, ask outright rather than omitting it silently.

### 3. Apply

Only after explicit confirmation. "Looks good", "go ahead", "do it", "apply" — wait for one of those. Then:

- Create folders.
- `git mv` any files being relocated.
- Edit or create `AGENTS.md`. If creating it fresh, also create the `CLAUDE.md` → `AGENTS.md` symlink.
- Report what was done with file paths, in one short summary.

If the user pushes back on a piece of the plan, revise just that piece and re-confirm — don't re-survey the whole project.

## The planning content in `AGENTS.md`

Free-form markdown. Skills read `AGENTS.md` in natural language — no required heading, no enforced schema. Put the content under whatever heading fits the file the user already has, or pick a reasonable one (`## Project Planning` works, but isn't required). The shape below is a starting point; adapt to what the project actually uses.

```markdown
## Project Planning

This project uses the [project-planning plugin](https://github.com/gestrich/ai-project-planning).

### Sources of truth

- **Jira**: project key `XYZ`, primary epic `XYZ-100`. Issues here are authoritative for shipped work.
- **Confluence**: space `TEAM`, root page id `123456789`. Long-form plans and decisions live here.
- **Local `plans/`**: in-incubation ideas and personal items only. Not authoritative for shipped work.
- **Slack**: `#team-channel` (id `C0123ABC456`) is the canonical channel — used as read-only context.

### GitHub repos

- `org/primary-repo` (this repo)
- `org/secondary-repo`

### Plugin skills used

- `planning` — capture transcripts into `notes/`, show what's in motion across Jira/Confluence/`plans/`, plan the week, and reconcile local plans against external sources. Use when I paste a brain-dump, ask "what's the plan", ask "what should I do this week", or ask what's drifted.
- `bootstrap` — re-run if the planning shape changes (new Slack channel, new repo, etc.).
- `docs-update` — sweep last week of `#team-channel` for things that should land in Confluence. Use weekly.
- `pr-review` — review PRs against the docs declared above. Use when reviewing open PRs.
```

Two things matter most about this content:

- **It enumerates which planning sources apply**, so `planning` knows which reference documents to load.
- **It lists the plugin skills the project uses, each with a one-line "what it's for / when to reach for it"** in this project's context. This gives any future session immediate orientation without needing to read each SKILL.md upfront, and lets skills check the list before firing on ambiguous triggers.

Omit subsections that don't apply (e.g., no Confluence → drop that bullet). Don't pad with skills the project isn't actually using.

## What this skill does *not* do

- It does not write to external systems. No Jira project creation, no Confluence page creation, no Slack channel setup. It only writes to the local repo.
- It does not run other skills as part of bootstrap. After applying the plan, the user invokes `planning`, `docs-update`, etc. themselves.
- It does not enforce a folder layout beyond the conventions named in the plugin (`notes/`, `plans/`, `plans/sprints/`). If a project already uses a different layout and wants to keep it, propose declaring that in `AGENTS.md` instead of moving files.
- It does not silently edit. Every change goes through the proposed plan and explicit confirmation.

## Relationship to other skills

- `bootstrap` is run once per project (and re-run as a refresh when the planning shape changes). The other skills (`planning`, `docs-update`, `pr-review`) are used continuously and rely on the planning content in `AGENTS.md` that `bootstrap` lands.
- If `planning` is invoked in a project with no planning content in `AGENTS.md`, it should suggest running `bootstrap` rather than guessing the planning shape from the repo contents.
