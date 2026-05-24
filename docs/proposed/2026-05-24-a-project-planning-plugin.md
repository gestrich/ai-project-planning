# Project Planning Plugin

A Claude/Codex plugin that gives Bill a consistent gateway into any of his projects for capturing notes, navigating plans, running sprints, syncing with external sources of truth (Jira / Confluence / Slack), and reviewing PRs against documentation.

> Local-only context (paths to reference skill implementations on Bill's machine, validation-target project names, etc.) lives in the gitignored `private-context.md` at the repo root. The plan refers to it where needed.

## Relevant Skills

There is no project-local `AGENTS.md` yet, so skills are drawn from Bill's global skill set. Only skills directly relevant to building this plugin are listed.

| Skill | Description |
|-------|-------------|
| `skill-creator` | Create / modify / improve skills, including frontmatter conventions and description tuning for trigger accuracy |
| `update-config` | Settings.json edits (permissions, hooks) — useful if any skill needs allowlisted Bash or MCP calls |
| `claude-tools-skills:gestrich-claude-tools-plan` | The planning convention this document follows (one plan file per task in `docs/proposed/`) |

The reference skills enumerated in `private-context.md` are the canonical examples to mirror when authoring the Slack/Confluence-touching skills below.

## Background

### What this is

Bill is responsible for multiple concurrent projects and needs one consistent way to dump voice transcripts, see what's next, prepare for meetings, and keep external sources of truth aligned with the local notes he scribbles between meetings. Rather than reinvent that workflow per project, he wants a single reusable plugin he can install everywhere.

### Modeled after `swift-app-architecture`

The shape of this plugin will mirror Bill's `swift-app-architecture` repo (https://github.com/gestrich/swift-app-architecture), which is already proven to work with both Claude Code and Anthropic Codex:

```
ai-project-planning/
├── .claude-plugin/
│   └── marketplace.json            # Top-level Claude Code marketplace metadata
├── plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json             # Claude Code plugin manifest
│   ├── .codex-plugin/
│   │   └── plugin.json             # Codex plugin manifest (skills: "./skills/")
│   ├── skills/
│   │   ├── notes/SKILL.md
│   │   ├── plan/SKILL.md
│   │   ├── plan/<reference>.md     # confluence.md, jira.md, slack.md, local.md, etc.
│   │   ├── sprint/SKILL.md
│   │   ├── bootstrap/SKILL.md
│   │   ├── docs-update/SKILL.md
│   │   └── pr-review/SKILL.md
│   │   # No sync/ — reconciliation lives inside plan/
│   └── LICENSE
├── .claude/
│   └── skills/                     # Dev-time skills (skill authoring), not shipped
├── docs/
│   ├── proposed/
│   └── completed/
├── AGENTS.md                       # AI instructions (CLAUDE.md is a symlink → AGENTS.md)
└── README.md                       # Install instructions for Claude Code AND Codex
```

This is a Claude/Codex plugin, not a Claude-only one. The README must include install instructions for both runtimes.

### Reference skills as the Slack/Confluence template

When a skill in this plugin needs to talk to Slack or Confluence, follow the patterns demonstrated by the reference skills listed in `private-context.md`:

- Deferred MCP tools loaded via `ToolSearch` with `select:<tool_name>`
- Read-before-write workflow on any shared doc, with the version number used for optimistic concurrency
- Explicit-authorization-on-exact-text for any Slack send
- Capability skills that *orchestrate* and call out to single-source skills rather than reimplementing the underlying logic
- Channel IDs, page IDs, and `cloudId`s live in the skill body — not hardcoded anywhere else

Those reference skills are project-specific. The skills in this plugin are project-*agnostic*; they pick up Slack channel IDs, Jira projects, and Confluence pages from the project's `AGENTS.md` instead.

### Configuration philosophy

- A project opts in by adding a project-planning section to its `AGENTS.md` (the standard cross-runtime config file). Free-form markdown, not enforced YAML — skills read the section in natural language. Keep it portable.
- The `AGENTS.md` section is the project's declaration of: what plan storage it uses (Jira / Confluence / local-only / mixed), which Slack channel is canonical, which GitHub repos belong to it, which plugin skills the project uses (and what each is for in this project), and any project-specific notes the skills should know.
- Skills load reference documents (e.g., `plan/jira.md`, `plan/confluence.md`, `plan/slack.md`) only when `AGENTS.md` says that source applies. Reference-document loading is a native skill feature — no custom plumbing.
- Folder layout (`notes/`, `plans/`, `plans/sprints/`, etc.) lives in the *project*, not the plugin. The plugin specifies the convention; the project holds the data.

### Vocabulary (locked in for the skills)

Consistent terms across the plugin:

- **Notes** — raw voice/Apple-notes transcripts, dated, in `notes/`. Never cleaned up.
- **Plans** — supplemental local planning artifacts in `plans/`. Either *the* source of truth (when no external system is used) or *supplementary* to Jira/Confluence (when one is). The skill spells out the distinction; the project's `AGENTS.md` declares which mode.
- **Domain documents** — files inside `plans/` organized by area of the project (UI, infra, testing, etc.), not by time.
- **Sprints** — time-boxed actionable lists in `plans/sprints/`, derived from domain documents and external sources. One file per sprint.
- **Source of truth** — for projects that use Jira/Confluence, those are authoritative for shipped plans. `plans/` is for in-incubation ideas and Bill-personal items.

### Out of scope for this plugin

The voice-transcript sweeper (a daemon that watches Apple's internal voice-notes transcript directory and routes them into the right project's `notes/` folder) is **explicitly tangential**. It's listed in an appendix at the bottom of this plan as a separate side project — not a skill in the plugin.

### Simplicity bar

Keep the skills loose and the spec loose. Skills get iterated as they're used, not over-specified upfront. Each phase below adds a small, testable surface; do not bake in formats, schemas, or sub-conventions beyond what's named here.

## Phases

## - [x] Phase 1: Scaffold the plugin repo

**Skills used**: `skill-creator` (read for frontmatter/description conventions — no SKILL.md files authored in this phase, but reviewed in preparation for Phases 2+)
**Principles applied**: Mirrored `swift-app-architecture`'s dual-runtime layout exactly (top-level `.claude-plugin/marketplace.json`, `plugin/.claude-plugin/plugin.json`, `plugin/.codex-plugin/plugin.json` with `"skills": "./skills/"`). Kept manifests minimal (name, version, description, author, keywords) — nothing speculative. Used `.gitkeep` files to commit the empty `plugin/skills/`, `.claude/skills/`, and `docs/completed/` directories. Plugin shell verified loadable via `claude --plugin-dir`.

**Skills to read**: `skill-creator`

Lay down the directory shape modeled on `swift-app-architecture`. No skill content yet — just the wiring so the plugin is loadable.

- Create `.claude-plugin/marketplace.json` at repo root (one plugin entry pointing at `./plugin`).
- Create `plugin/.claude-plugin/plugin.json` with name `project-planning`, version `0.1.0`, author, description, keywords.
- Create `plugin/.codex-plugin/plugin.json` with `"skills": "./skills/"` and matching metadata (model on `swift-app-architecture/plugin/.codex-plugin/plugin.json`).
- Create empty `plugin/skills/` directory.
- Create `plugin/LICENSE` (MIT, to match the `swift-app-architecture` repo's stance).
- Create `.claude/skills/` directory for dev-time-only skills (currently empty).
- Create `docs/proposed/` and `docs/completed/` (`docs/proposed/` already exists for this plan).
- Create an initial `AGENTS.md` at the repo root with the project conventions (single-source-of-truth for skills is `plugin/skills/`; `docs/proposed/` and `docs/completed/` track planning docs). Add `CLAUDE.md` as a symlink to `AGENTS.md` so Claude Code finds it under the expected filename.
- Verify the plugin loads without skills: `claude --plugin-dir <repo>` should at least not error.

Outcome: empty but valid plugin shell.

## - [x] Phase 2: `notes` skill (foundational, no config)

**Skills used**: `skill-creator` (read for frontmatter conventions, description-tuning guidance, and progressive-disclosure structure)
**Principles applied**: Wrote a "pushy" description that names explicit trigger phrases *and* describes the implicit shape of a raw transcript, since the plan calls out both signals. Kept SKILL.md short and loose — no rigid template, no enforced slug style beyond the date prefix, since the plan's simplicity bar explicitly warns against over-specifying. Stated the negative space ("what this skill does *not* do") so downstream skills (`plan`, `sprint`) own derivation rather than this one. Used the `date` shell command for the date prefix per the global `AGENTS.md` rule against guessing the current date. Removed the placeholder `plugin/skills/.gitkeep` now that the directory has real content.

**Skills to read**: `skill-creator`

The simplest skill. No `AGENTS.md` configuration. Triggers on the user dumping a transcript into the conversation.

- `plugin/skills/notes/SKILL.md` with frontmatter `name: notes`, a description tuned to fire on phrases like "here's my brain dump", "voice transcript", "dump notes", and on visibly long unstructured first-person text resembling a transcript.
- Behavior: detect that the input is a raw transcript, write it verbatim to `notes/YYYY-MM-DD-<short-slug>.md` in the current project (no cleanup, no rewriting), and confirm the file path. If `notes/` doesn't exist, create it.
- One sentence in the skill clarifies these are *raw artifacts*, not plans — downstream skills (`plan`, `sprint`) may read them but the `notes` skill never edits them after creation.
- Don't enforce a filename convention beyond date; let the project use whatever slug style it prefers.

Outcome: a working transcript-capture entry point. Paste a transcript and get a dated file written. This is the foundation everything else extracts from.

## - [x] Phase 3: `plan` skill + reference documents

**Skills used**: `skill-creator` (read for frontmatter conventions, "pushy" description tuning, and the domain-organization pattern that maps a top-level SKILL.md to per-source reference files). Reviewed the reference skills named in `private-context.md` (`documentation` for Confluence read-before-write discipline, `vector-slack` for Slack read patterns) but only lifted the *patterns* — no project-specific IDs, cloudIds, or channel IDs were carried over, per `private-context.md`'s explicit list of things to keep out of the public repo.
**Principles applied**: Kept `SKILL.md` as the trigger surface and a router — it does *not* re-derive the read/write mechanics, just points at the matching reference doc. Used the skill-creator domain-organization pattern (one SKILL.md + per-variant reference docs in the same directory) so only relevant references load. Wrote the trigger description "pushy" with concrete user phrases. Made every reference doc project-agnostic — cloudId, space key, page IDs, project key, board ID, channel ID, and workspace URL all sourced from the project's `AGENTS.md`, with explicit "don't hardcode them here" reminders. Scoped `slack.md` to read-only per the phase spec; deferred posting to other skills. `local.md` describes the `plans/plan.md` manifest convention without prescribing internal structure beyond "the manifest exists and points at the rest." All four reference docs carry source-appropriate write discipline (Confluence read-before-write with version concurrency; Jira show-then-confirm; Slack: skill doesn't post; local: show-then-confirm). Kept all docs short and loose per the plan's simplicity bar.

**Skills to read**: `skill-creator`

The configurable, project-aware skill. Reads `AGENTS.md` to discover what planning sources apply, then loads the matching reference documents.

- `plugin/skills/plan/SKILL.md` — top-level skill, fires on "what's the plan", "what's next", "plan for X", "show me the plans", etc.
- Behavior:
  1. Read `AGENTS.md` in the project root and locate the project-planning section.
  2. From that section, identify which sources apply (Jira, Confluence, local-only, Slack-as-context).
  3. Load only the matching reference documents.
  4. Synthesize a brief view of the plan state, citing each source explicitly.
- Reference documents (one file each, in `plugin/skills/plan/`):
  - `confluence.md` — read-before-write discipline; "use the Atlassian MCP family, load schemas via `ToolSearch`, ask before any write". Model on the Confluence reference skill listed in `private-context.md`.
  - `jira.md` — same pattern for Jira. Reads epics/issues named in `AGENTS.md`. Writes only with confirmation.
  - `slack.md` — read-only context capture only. No posting from the `plan` skill. Model on the Slack reference skill listed in `private-context.md`.
  - `local.md` — for projects with no external source of truth. Describes the `plans/` folder convention: a top-level `plans/plan.md` manifest, plus per-domain documents the manifest references.
- Manifest convention (in `local.md`): `plans/plan.md` enumerates the other files in `plans/`, one line each, so the skill can navigate the project without prescribing a structure beyond "there is a manifest at the top".
- Keep all reference docs short and loose. They guide; they don't dictate format.

Outcome: land in any project, ask "what's the plan?", and the skill figures out whether to read Jira, Confluence, the local `plans/` folder, or some combination.

## - [ ] Phase 4: `sprint` skill

**Skills to read**: `skill-creator`

The "scrum master" capability. Helps pick what to do next given everything that's known about the project right now.

- `plugin/skills/sprint/SKILL.md` — fires on "what should I do next", "plan my sprint", "what's on deck this week", "next sprint".
- Behavior:
  1. Gather context: read recent `notes/`, read the relevant plan sources (delegating to the same logic as `plan`), read the previous sprint file in `plans/sprints/` if one exists, and — when `AGENTS.md` names a Slack channel — pull the last week of activity from it (read-only).
  2. Ask challenging questions about what's load-bearing for the coming sprint. This is a *coach*, not an automated planner; it surfaces decisions, doesn't make them.
  3. Draft a new sprint file at `plans/sprints/YYYY-MM-DD.md` (dated by start of week) listing on-deck items derived from domain documents and external sources. Always confirm the draft before writing.
- Loose format: bullets organized by domain (UI / infra / testing / etc.) with optional personal items. No rigid template — the skill encourages this shape but doesn't enforce it.
- Defer to `plan` reference docs for *how* to read external sources; the sprint skill should not redefine that wiring.

Outcome: an interactive sprint-planning capability that produces a single sprint file per week.

## - [ ] Phase 5: `bootstrap` skill

**Skills to read**: `skill-creator`

The migration on-ramp. Run inside an existing project to get it into a shape this plugin understands.

- `plugin/skills/bootstrap/SKILL.md` — fires on "bootstrap this project", "set up project planning here", "make this project work with the planning plugin".
- Behavior:
  1. Survey the project: look for existing notes/plans folders, look for an existing `AGENTS.md` and any project-planning section in it, check commit history / referenced URLs to *guess* whether Jira / Confluence / Slack are in use (best-effort, never assert).
  2. Produce a written plan-of-changes to approve before making any edits. The plan covers: which folders to create (`notes/`, `plans/`, `plans/sprints/`), which existing files to move into them, what to add to `AGENTS.md`'s project-planning section, and any GitHub repos to declare.
  3. Apply only after explicit confirmation. No silent edits.
- The skill must declare which plugin skills the project should use, in `AGENTS.md`, with a one-line "what it's for / when to reach for it" note per skill — so future sessions immediately know to reach for them without reading every SKILL.md.

Outcome: a repeatable way to onboard a new project to this convention without hand-editing.

## - [ ] Phase 6: `docs-update` skill

**Skills to read**: `skill-creator`

Closes the loop between Slack discussions and canonical documentation. Loose by design — this skill will evolve.

- `plugin/skills/docs-update/SKILL.md` — fires on "update the docs", "what's missing from the docs", "review last week's Slack for doc updates".
- Behavior:
  1. Pull the configured Slack channel's last week of messages (read-only; reuse the `plan/slack.md` reference document patterns).
  2. Pull the relevant documentation source — usually Confluence — using `plan/confluence.md` patterns, including the read-before-write discipline.
  3. Suggest documentation updates: clarifications, missing answers, recurring questions that deserve a permanent home.
  4. When `AGENTS.md` indicates direct write access to those docs, offer to stage a write (still requires explicit confirmation). When the docs are owned by another team, produce a draft message to send to that team instead.
- Keep this skill's behavioral spec terse. Refine once it has run against a real channel a few times.

Outcome: a weekly "what should we have written down?" sweep.

## - [ ] Phase 7: `pr-review` skill

**Skills to read**: `skill-creator`

GitHub PR review that consults the project's documentation and surfaces both PR-side and docs-side findings.

- `plugin/skills/pr-review/SKILL.md` — fires on "review this PR", "is this PR aligned with our docs", "check the open PRs for doc drift".
- Behavior:
  1. Read the GitHub repo(s) declared in `AGENTS.md` (`gh` CLI when available per the user's global `AGENTS.md` preferences; GitHub API otherwise).
  2. For a given PR, fetch the diff via the API (no local checkout in v1 — keep it simple).
  3. Cross-reference against the documentation declared in `AGENTS.md` (Confluence pages, local docs, etc.).
  4. Output two sections: (a) PR review against documented conventions, (b) documentation update suggestions when the PR introduces patterns or decisions that aren't reflected in the docs.
- Same write discipline as `docs-update`: any doc edit goes through confirmation; PR comments go through confirmation.
- Document explicitly that local checkout is out of scope for v1.

Outcome: a documentation-aware PR review that also feeds documentation improvement.

## - [ ] Phase 8: Reconciliation mode inside `plan`

**Skills to read**: `skill-creator`

No standalone `sync` skill. Reconciliation lives as a mode of the `plan` skill, with the per-source mechanics described in each reference document.

- Extend `plugin/skills/plan/SKILL.md` with a "reconciliation mode" section: fires on "sync plans", "reconcile with Jira", "what's drifted", "what should graduate".
- Behavior:
  - Compare local `plans/` (or `plans/sprints/`) against the configured external sources declared in `AGENTS.md`.
  - Report:
    - Items in `plans/` that look like they should now live in Jira/Confluence (graduation candidates).
    - Items in Jira/Confluence that contradict `plans/` (drift).
    - Stale items in `plans/` that have been overtaken by external decisions.
  - Any external mutation is gated on explicit confirmation — same discipline as the underlying reference docs.
- Extend each reference document (`plan/jira.md`, `plan/confluence.md`, `plan/local.md`) with a short "reconciliation" subsection covering what drift/graduation looks like for that source.
- Skip writing a new `plugin/skills/sync/` directory entirely.

Outcome: reconciliation is discoverable from the same entry point already used for planning, without inflating the skill count.

## - [ ] Phase 9: README, AGENTS.md, and dual-runtime install instructions

**Skills to read**: `skill-creator`

Documentation pass. The README must include install instructions for **both** Claude Code and Codex, modeled on `swift-app-architecture/README.md`.

- `README.md`:
  - Project overview (one paragraph: a Claude/Codex plugin for project planning).
  - "Installation — Claude Code" and "Installation — Codex" sections — model on `swift-app-architecture/README.md` (dual-provider framing, both subsections under one `## Installation` heading, Codex via TUI `/plugins` menu, `~/.codex/config.toml` toggle).
  - Troubleshooting block — model on the same README's troubleshooting section; same Claude Code gotchas apply here.
  - "Skills" table: skill name, trigger phrase, one-line description for each of the seven skills above.
  - "Configuration" section: how to declare project-planning in `AGENTS.md`, with one short generic example. Free-form markdown, not YAML. No company names, no internal project names — synthetic placeholders only.
- `AGENTS.md`: expand the Phase 1 starter into a fuller declaration of this repo's own conventions — `plugin/skills/` is the single source of truth, where planning docs live, where skill drafts live. Also serves as a worked example for any project bootstrapping itself onto this convention. `CLAUDE.md` stays a symlink to `AGENTS.md`.

Outcome: a project installable on either runtime by following the README.

## - [ ] Phase 10: GitHub Pages site

**Skills to read**: `skill-creator`

A public-facing HTML site hosted on GitHub Pages from this repo, walking visitors through what the plugin does, what each skill does, and how to use them. Plain static HTML (no build step, no framework).

- Create a `docs-site/` directory at the repo root (separate from `docs/proposed/` / `docs/completed/`, which are planning docs).
- Configure GitHub Pages to serve from `docs-site/` on the `main` branch via repo Settings → Pages.
- Pages to author (one file each):
  - `docs-site/index.html` — landing page. Three sections: (1) what this plugin is in two paragraphs, (2) install (links to README sections for Claude Code and Codex), (3) skills overview as a card grid linking to each skill page.
  - `docs-site/skills/notes.html`, `plan.html`, `sprint.html`, `bootstrap.html`, `docs-update.html`, `pr-review.html` — one page per skill. Each page covers: what the skill does, when to reach for it (the trigger phrases), what it reads, what it writes, configuration in `AGENTS.md` (if any), and a worked walkthrough.
  - `docs-site/configuration.html` — the `AGENTS.md` project-planning section in detail, with one generic synthetic example.
  - `docs-site/getting-started.html` — the bootstrap-a-new-project walkthrough end-to-end.
- Single shared `docs-site/style.css`. No JS frameworks. Keep it light: a serif/sans pairing, generous whitespace, code blocks with subtle background, two-column max for the card grid.
- Header on every page with a back-to-home link and a link to the GitHub repo. Footer with MIT license link.
- Use semantic HTML (`<nav>`, `<main>`, `<article>`, `<section>`) for accessibility and reasonable default rendering.
- Add a `docs-site/CNAME` only if Bill decides to use a custom domain — leave that out for v1.
- All content must be company-agnostic: no internal project names, no internal channel IDs, no internal repo paths. The site is the public face of the plugin.
- Confirm the site is live at `https://gestrich.github.io/ai-project-planning/` after the Pages config takes effect (can take a few minutes).

Outcome: a working GitHub Pages site that lets a visitor understand the plugin without reading the repo.

## - [ ] Phase 11: Validation

**Skills to read**: `skill-creator`

No traditional test suite — this is a skills plugin. Validation is hands-on against real projects.

- Install the plugin locally (`claude --plugin-dir`) and confirm all seven skills are discoverable.
- Run `bootstrap` against the smallest validation-target project listed in `private-context.md`.
- Dump a real voice transcript via the `notes` skill; confirm the file lands where the skill says it will.
- Invoke `plan` and confirm it correctly reads `AGENTS.md` and loads only the relevant reference documents.
- Invoke `sprint` and walk through producing a real sprint file.
- Invoke `docs-update` and `pr-review` against real Slack channels / real PRs (read-only — do not push any writes during validation).
- For each skill, refine the description in frontmatter if it didn't trigger when expected. The `skill-creator` skill specifically covers description tuning.
- Final pass: confirm the README install instructions actually work on a clean Codex instance (separate from the Claude Code one).
- Confirm the GitHub Pages site renders correctly on desktop and mobile, all internal links work, and no internal/company identifiers leaked into the published HTML.

Outcome: the plugin works end-to-end on at least one real project, both runtimes installable from the README, and the public Pages site is live and accurate.

---

## Appendix: Voice-transcript sweeper (side project — out of scope)

Tangential to the plugin itself. Captured here so it isn't lost; planned separately if/when it's picked up.

**Idea:** Apple stores voice-note transcripts in a (semi-hidden) system directory. A local daemon could sweep that directory hourly/daily, identify which transcript belongs to which project (by content), and route the transcript into the appropriate project's `notes/` folder via the `notes` skill convention.

**Why it's separate:**
- It's a system-level tool, not a Claude/Codex skill.
- It depends on file paths and permissions that vary per macOS version.
- It requires a registry of the user's projects with locations and identifying keywords — separate state from anything the plugin manages.

**If it ever gets built:**
- It would call into the `notes` skill convention as its output side (write a dated markdown file into the target project's `notes/` folder).
- It would read project identification hints from each project's `AGENTS.md` — possibly a `keywords:` line, possibly a free-form "this project is about X" paragraph the matcher can use.
- It would never modify anything beyond `notes/`. Routing only.

---

## Open questions

_No open questions remaining._

**Resolved:**
- *Codex install mechanism* — covered by `swift-app-architecture/README.md` and `swift-app-architecture/plugin/.codex-plugin/plugin.json`. Mirror those.
- *Sync placement* — no standalone `sync` skill. Reconciliation is a mode of the `plan` skill, with per-source details in each reference doc.
- *`AGENTS.md` schema* — free-form markdown only. No YAML frontmatter. Skills parse natural-language prose. Example shape: a `## Project Planning` section that names the planning sources (Jira epics, Confluence root, Slack channel ID), declares what `plans/` is used for, and lists GitHub repos.
- *Per-project skill enablement* — yes, the project-planning section explicitly enumerates which plugin skills the project uses, **and for each one, a short line on what it's for and when to reach for it in this project's context.** Not just bare skill names. Example: `- notes: dump raw voice transcripts into notes/. Use when I paste a brain-dump.` This gives the agent immediate orientation without needing to read each SKILL.md upfront. Skills also check the list before firing on ambiguous triggers; `bootstrap` fills it in during onboarding.
- *Head-start reference skills* — see `private-context.md` for the local-only paths and what to mirror. Draft everything else fresh.
