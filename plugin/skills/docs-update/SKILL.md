---
name: docs-update
description: Analyze recent team discussion (Slack, or whatever the project uses for chat) against the project's documentation and propose updates — clarifications, recurring questions that deserve a permanent home, decisions that were made in discussion but never written down. Use this skill whenever the user says "update the docs", "what's missing from the docs", "review last week's discussion for doc updates", "what should we have written down", "what doc updates do we need", or otherwise asks for a documentation hygiene pass driven by what's been happening recently. Trigger even when the user does not name the skill — if they're asking what should be captured in the docs based on what's happened in chat, this is the right home. The skill reads the project's AGENTS.md to discover where discussion happens, where docs live, and how docs get updated; it then proposes updates the user can act on. Read-only by default; any write or hand-off only happens after explicit confirmation.
---

# docs-update

Closes the loop between recent team discussion and the project's documentation. Reads the discussion source and the documentation source declared in `AGENTS.md`, looks for gaps, and proposes updates the user can act on.

Documentation lives in wildly different shapes across projects — a Confluence space, an in-repo `docs/` folder, a wiki, a Notion database, an S3 bucket of PDFs, an owned-by-another-team page tree. The skill makes no assumptions about that shape. It reads what `AGENTS.md` points it at, and follows whatever update workflow `AGENTS.md` describes. Refine over time as the skill is used against real projects.

## When to trigger

Fire on any phrasing that asks for documentation hygiene driven by recent discussion:

- "update the docs", "what should be in the docs", "what's missing from the docs"
- "review last week's discussion for doc updates", "sweep chat for doc updates"
- "what should we have written down", "what doc updates do we need"
- A recurring weekly check on whether the documented state still matches reality.

If the user is asking a *one-shot* question that just happens to need docs context (e.g., "what does the spec say about X"), that's the `plans` skill in status mode, not this one. `docs-update` is specifically the sweep — discussion in, doc proposals out.

## What `AGENTS.md` should tell you

Before doing anything, read the project's `AGENTS.md` and pick up:

- **Where team discussion happens.** Usually a Slack channel; could be Discord, GitHub Discussions, a mailing list, or anything else. Without a named source, the skill has nothing to read — say so and stop.
- **Where the docs live.** A Confluence space, an in-repo folder, a wiki, a Notion workspace, etc. Without a named source, the skill can still report what came out of discussion, but framed as "things the team is talking about that don't appear to be written down anywhere".
- **How docs get updated.** Free-form prose in `AGENTS.md`. Project-specific. It might say "I have direct edit access to Confluence", "the docs live in this repo and are PR-reviewed", "another team owns these pages — file an issue against their repo", "ask on `#docs-team` and they'll edit". Honor whatever workflow is described. Don't invent a workflow if `AGENTS.md` doesn't describe one — ask.

If a required piece is missing, ask the user once and offer to run `bootstrap` (or its refresh mode) to declare it.

## Behavior

The skill runs in three phases: **gather → propose → act**. Gather and propose are read-only; nothing leaves the project until the user confirms.

### 1. Gather

- **Resolve the time window.** Default to the last seven days. Use `date +%Y-%m-%d` to anchor the window; never guess. Honor a different window if the user names one.
- **Read the discussion source** named in `AGENTS.md`, read-only. When the source is Slack and the project's planning content references it, the reference patterns in `plans/slack.md` apply. For other sources, use the project's available tools to read recent activity (capture permalinks, resolve user references to names when possible).
- **Read the current docs.** Whatever `AGENTS.md` points to. If it's Confluence and `plans/confluence.md` applies, follow that. If it's in-repo files, read them directly. If it's something else, read it however that source supports. Don't skip this step — proposing updates without seeing what the docs currently say is how you propose duplicates and contradictions.
- Keep this phase quiet — the user sees the proposals, not the raw read output, unless they ask.

### 2. Propose

Produce a structured list of proposed documentation updates. Show it inline. Each item should carry:

- **What the update is** — a clarification, a missing answer, a recurring question, a decision that needs a permanent home.
- **Where it should land** — the specific doc/page/section if known. If the right home is ambiguous, surface that as a question rather than picking one silently.
- **Why it's worth landing** — the discussion signal that prompted it (a link if available), and what's at stake if it stays only in chat.

Good proposals usually fall into a few shapes:

- A question got asked more than once in different threads → the answer deserves a permanent home.
- A decision was made but the doc still describes the prior approach → the doc needs an update.
- A concept the team is now using freely doesn't appear in the docs at all → it needs to be introduced.
- The doc is correct but ambiguous enough that people are asking the same clarifying question repeatedly → tighten the wording.

Drop noise. A one-off question that got a one-off answer and nobody asked again isn't a doc update. Use judgment about what's recurring vs. transient.

### 3. Act

Only after the user picks one or more proposals to act on. *How* the action happens is driven by what `AGENTS.md` says about the update workflow:

- If the user can edit the docs directly, offer to stage the edit using whatever protocol the source requires (e.g., the read-before-write discipline in `plans/confluence.md` for Confluence; a normal file edit for in-repo docs; whatever the source needs).
- If the docs are owned by another team or otherwise out of the user's reach, hand off appropriately — draft a message, draft an issue, draft a PR comment, depending on what the workflow described in `AGENTS.md` calls for. Do not perform the hand-off itself; produce the artifact and let the user (or another skill) send it.
- If the workflow isn't clear, ask. Never invent a write path.

Always wait for explicit confirmation before any write or hand-off. The proposal list is not the confirmation; each acted-on item is its own ask.

## What this skill does *not* do

- It does not assume a particular doc storage or update workflow. `AGENTS.md` describes the workflow; the skill follows it.
- It does not summarize the discussion source for catch-up purposes. That's the `plans` skill in status mode. `docs-update` is specifically about extracting *documentation-shaped* gaps from recent discussion.
- It does not modify local `notes/` or `plans/`. Those are owned by the `plans` skill.

## Relationship to other skills

- `bootstrap` is the right answer when `AGENTS.md` is missing the discussion source, the docs source, or the update workflow.
- `pr-review` shares the "propose doc updates" output shape. The two skills compose into a weekly cadence: PRs in, docs swept, doc gaps proposed once from both sides.
- When `AGENTS.md` points the discussion source at Slack or the docs source at Confluence, the matching reference docs (`plans/slack.md`, `plans/confluence.md`) carry the read/write mechanics. The skill calls into them rather than redefining them.
