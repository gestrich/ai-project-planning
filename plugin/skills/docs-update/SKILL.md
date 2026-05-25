---
name: docs-update
description: Sweep the project's configured Slack channel for the last week, cross-reference against the project's documentation source (usually Confluence), and propose documentation updates — clarifications, recurring questions that deserve a permanent home, decisions that were made in Slack but never written down. Use this skill whenever the user says "update the docs", "what's missing from the docs", "review last week's Slack for doc updates", "what should we have written down", "sweep Slack for doc updates", "what doc updates do we need", "anything to land in Confluence from Slack", or otherwise asks for a recurring documentation hygiene pass against recent team discussion. Trigger even when the user does not name the skill — if they're asking what should be captured in the docs based on what's happened in chat, this is the right home. Read-only by default; offers to stage a doc edit (or a draft message to the owning team) only after explicit confirmation.
---

# docs-update

Closes the loop between team discussion (Slack) and canonical documentation (usually Confluence). Reads the configured channel's last week of activity, reads the relevant docs, and proposes updates — never edits silently. When the project owns the docs, offers to stage a write. When another team owns them, drafts a message instead.

This skill is loose by design and expected to evolve. Refine it once it has run against a real channel a few times.

## When to trigger

Fire on any phrasing that asks for documentation hygiene driven by recent discussion:

- "update the docs", "what should be in the docs", "what's missing from the docs"
- "review last week's Slack for doc updates", "sweep Slack for doc updates"
- "what should we have written down", "anything to land in Confluence from Slack"
- A recurring weekly check on whether the documented state still matches reality.

If the user is asking a *one-shot* question that just happens to need docs context (e.g., "what does the spec say about X"), that's the `planning` skill in status mode, not this one. `docs-update` is specifically the sweep — discussion in, doc proposals out.

## Behavior

The skill runs in three phases: **gather → propose → stage**. The gather and propose phases are read-only; nothing leaves the project until the user confirms.

### 1. Gather

- **Resolve the time window.** Default to the last seven days. Use `date +%Y-%m-%d` to anchor the window; never guess. If the user names a different window ("last two weeks", "since the offsite"), honor it.
- **Read the Slack channel** named in the project's `AGENTS.md`, read-only. Follow the patterns in `planning/slack.md` — load Slack MCP schemas via `ToolSearch`, read the channel, pull load-bearing threads, resolve user IDs to names, capture permalinks. Don't redefine those patterns here.
- **Read the documentation source.** When `AGENTS.md` declares Confluence, follow `planning/confluence.md` — resolve the root page from `AGENTS.md`, walk descendants when the discussion implicates a specific sub-area, fetch with `contentFormat: "markdown"` for inspection. When `AGENTS.md` declares a local docs source (e.g., a `docs/` folder), read those files directly. Do not skip this step — proposing doc updates without re-reading what the docs currently say is how you propose duplicates and contradictions.
- Keep this phase quiet — the user sees the proposals, not the raw read output, unless they ask.

### 2. Propose

Produce a structured list of proposed documentation updates. Show it inline. Each item should carry:

- **What the update is** — a clarification, a missing answer, a recurring question, a decision that needs a permanent home.
- **Where it should land** — the specific doc/page/section. If the right home is ambiguous, surface that as a question rather than picking one silently.
- **Why it's worth landing** — the Slack signal that prompted it (permalink to the message or thread), and what's at stake if it stays only in Slack.
- **Who owns the doc** — the project (we can stage the write) or another team (we draft a message instead). This determines what happens in phase 3.

Good proposals usually fall into a few shapes:

- A question got asked more than once in different threads → the answer deserves a permanent home.
- A decision was made in a thread but the doc still describes the prior approach → the doc needs an update.
- A concept the team is now using freely in chat doesn't appear in the docs at all → it needs to be introduced.
- The doc is correct but ambiguous enough that people are asking the same clarifying question repeatedly → tighten the wording.

Drop noise. A one-off question that got a one-off answer and nobody asked again isn't a doc update. Use judgment about what's recurring vs. transient.

### 3. Stage

Only after the user picks one or more proposals to act on.

- **When the project owns the doc** (`AGENTS.md` declares Confluence direct-write access, or the docs live in this repo): offer to stage the edit. For Confluence, follow `planning/confluence.md`'s read-before-write discipline strictly — read the live page with `contentFormat: "html"`, capture the version number, compose the update as a delta, surface unexpected content before proceeding, write back with the version for optimistic concurrency. For in-repo docs, show the proposed diff inline and wait for confirmation before writing.
- **When another team owns the doc**: draft a short message to send to that team — what's changed, the Slack permalink for context, a concrete suggested edit. Do not post the message from this skill. Hand it to the user (or to a Slack-posting skill they invoke separately) for the actual send.
- Always wait for explicit confirmation before any write or any message send. The proposal list is not the confirmation.

## Mapping the project's `AGENTS.md` declarations onto this skill

`AGENTS.md` should give you:

- A **Slack channel** to sweep (name + id). Without one, this skill has nothing to read; tell the user and stop.
- A **documentation source** — Confluence space + root page id, or a local docs folder path. Without one, the skill can still report what came out of Slack, but framed as "things the team is talking about that don't appear to be written down anywhere", with no specific target proposed.
- Optionally, an **ownership note** for the docs — whether the project has direct write access or another team owns them. Default to "project owns it" unless `AGENTS.md` says otherwise.

If a required piece is missing, ask the user once and offer to run `bootstrap` (or its refresh mode) to declare it. Don't fabricate channel IDs or page IDs.

## What this skill does *not* do

- It does not post to Slack. Drafting a message to another team is fine; sending it is not this skill's job.
- It does not write to documentation without explicit confirmation. Read-before-write applies to every body-content mutation.
- It does not summarize the channel for catch-up purposes. That's the `planning` skill in status mode (with Slack as a context source). `docs-update` is specifically about extracting *documentation-shaped* gaps from the channel.
- It does not modify local `notes/` or `plans/`. Those are owned by the `planning` skill.

## Relationship to other skills

- Reads use the same patterns as `planning/slack.md` and `planning/confluence.md`. If those patterns need to change, change them once in the reference docs — don't duplicate here.
- `pr-review` (next skill in the plugin) shares the "propose doc updates" output shape. The two skills can run alongside each other in a weekly cadence: PRs in, docs swept, doc gaps proposed once from both sides.
- `bootstrap` is the right answer when `AGENTS.md` is missing the channel or docs declaration this skill needs.
