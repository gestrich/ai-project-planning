---
name: docs-update
description: Analyze recent project activity against the project's documentation and propose updates — clarifications, recurring questions that deserve a permanent home, decisions made in chat that the docs don't yet reflect, and gaps between code and the technical docs that describe it. Use this skill whenever the user says "update the docs", "what's missing from the docs", "review last week's discussion for doc updates", "what should we have written down", "are our docs in sync with the code", "any doc gaps from recent PRs", or otherwise asks for a documentation hygiene pass driven by what's been happening recently. Trigger even when the user does not name the skill — if they're asking what should be captured in the docs based on what's happened recently in chat or in the code, this is the right home. The skill reads the project's AGENTS.md to discover what activity sources to scan, where docs live, and how docs get updated; it then proposes updates the user can act on. Read-only by default; any write or hand-off only happens after explicit confirmation.
---

# docs-update

Closes the loop between recent project activity and the project's documentation. Reads whatever activity sources and documentation sources `AGENTS.md` declares, looks for gaps, and proposes updates the user can act on.

A project usually has documentation in more than one shape. There are **project docs** — process, status, decisions, plans — and there are **technical docs associated with code repositories** — API references, architecture notes, conventions, runbooks. Both go stale, and the work to keep them in sync with reality is a chore. This skill is the chore: scan recent activity, scan the current docs, propose what to update.

Documentation lives in wildly different shapes across projects — a Confluence space, an in-repo `docs/` folder, a wiki, a Notion database, an owned-by-another-team page tree, a folder of release-notes markdown. Update workflows vary just as much. The skill makes no assumptions about either. It reads what `AGENTS.md` points it at, and follows whatever update workflow `AGENTS.md` describes.

## When to trigger

Fire on any phrasing that asks for documentation hygiene driven by recent activity:

- "update the docs", "what should be in the docs", "what's missing from the docs"
- "review last week's discussion for doc updates", "sweep chat for doc updates"
- "are our docs in sync with the code", "any doc gaps from recent PRs"
- "what should we have written down", "what doc updates do we need"
- A recurring weekly check on whether the documented state still matches reality.

If the user is asking a *one-shot* question that just happens to need docs context (e.g., "what does the spec say about X"), that's the `plans` skill in status mode, not this one. `docs-update` is specifically the sweep — activity in, doc proposals out.

## What `AGENTS.md` should tell you

Before doing anything, read the project's `AGENTS.md` and pick up:

- **Activity sources to scan.** A project may have any combination:
  - Team discussion (a Slack channel, a Discord, GitHub Discussions, etc.) — surfaces decisions, recurring questions, concepts the team is using in conversation but haven't written down.
  - A code repository (or several) — recent PRs and commits surface changes that may have invalidated the technical docs that describe the code.
  - Anything else the project considers signal for doc staleness.
- **Documentation sources to compare against.** Could be one place; often two — high-level project docs (Confluence, a wiki, a Notion workspace) plus repo-associated technical docs (an in-repo `docs/` folder, a README, generated API references). `AGENTS.md` should say which docs cover what.
- **How docs get updated.** Free-form prose in `AGENTS.md`. Project-specific. It might say "I have direct edit access to Confluence", "the in-repo docs are PR-reviewed", "another team owns these pages — file an issue against their repo", "ask on `#docs-team` and they'll edit". Honor whatever workflow is described. Don't invent a workflow if `AGENTS.md` doesn't describe one — ask.

If a required piece is missing, ask the user once and offer to run `bootstrap` (or its refresh mode) to declare it.

## Behavior

The skill runs in three phases: **gather → propose → act**. Gather and propose are read-only; nothing leaves the project until the user confirms.

### 1. Gather

- **Resolve the time window.** Default to the last seven days. Use `date +%Y-%m-%d` to anchor the window; never guess. Honor a different window if the user names one.
- **Read each activity source** named in `AGENTS.md`, read-only. For team discussion, pull the recent window of messages and load-bearing threads (capture permalinks, resolve user references). For code repositories, pull recent merged PRs and their diffs (and recent commits if PRs aren't the unit of work). The mechanics depend on the source; when a reference doc in `plans/` applies (e.g., `plans/slack.md`), use it.
- **Read the current docs.** Whatever `AGENTS.md` points to. Read enough to be able to compare against what came out of activity — don't skip this step. Proposing updates without seeing what the docs currently say is how you propose duplicates and contradictions.
- Keep this phase quiet — the user sees the proposals, not the raw read output, unless they ask.

### 2. Propose

Produce a structured list of proposed documentation updates. Show it inline. Each item should carry:

- **What the update is** — a clarification, a missing answer, a recurring question, a decision that needs a permanent home, a technical detail that no longer matches the code.
- **Where it should land** — the specific doc/page/section if known. If the right home is ambiguous, surface that as a question rather than picking one silently.
- **Why it's worth landing** — the signal that prompted it (a discussion link, a PR link, a commit), and what's at stake if the doc stays as-is.

Common patterns:

- A question got asked more than once → the answer deserves a permanent home.
- A decision was made but the doc still describes the prior approach → the doc needs an update.
- A concept the team is using freely doesn't appear in the docs at all → it needs to be introduced.
- A merged PR changed a public API, an architectural boundary, or a convention, and the technical docs still describe the old shape → the docs need to be brought current.
- The doc is correct but ambiguous enough that people keep asking the same clarifying question → tighten the wording.

Drop noise. A one-off question, a tiny PR that didn't change documented behavior, a commit that just renamed a local variable — not doc updates. Use judgment about what's recurring or load-bearing.

### 3. Act

Only after the user picks one or more proposals to act on. *How* the action happens is driven by what `AGENTS.md` says about the update workflow. If the user can edit the docs directly, stage the edit using whatever protocol the source requires. If updates flow through another team, an issue tracker, or a PR-reviewed repo, produce the artifact that workflow calls for and let the user (or another skill) deliver it. If the workflow isn't clear, ask.

Always wait for explicit confirmation before any write or hand-off. The proposal list is not the confirmation; each acted-on item is its own ask.

## What this skill does *not* do

- It does not assume a particular doc storage or update workflow. `AGENTS.md` describes the workflow; the skill follows it.
- It does not summarize activity for catch-up purposes. That's the `plans` skill in status mode. `docs-update` is specifically about extracting *documentation-shaped* gaps from recent activity.
- It does not modify local `notes/` or `plans/`. Those are owned by the `plans` skill.

## Relationship to other skills

- `bootstrap` is the right answer when `AGENTS.md` is missing the activity sources, the docs sources, or the update workflow.
- When `AGENTS.md` points an activity source at Slack or a docs source at Confluence, the matching reference docs (`plans/slack.md`, `plans/confluence.md`) carry the read/write mechanics. The skill calls into them rather than redefining them.
