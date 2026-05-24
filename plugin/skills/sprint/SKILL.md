---
name: sprint
description: Plan the next sprint for the current project by acting as a scrum-master coach — gather context from notes, plans, the previous sprint file, and the project's configured Slack channel, then ask challenging questions about what's load-bearing for the week before drafting a new sprint file at plans/sprints/YYYY-MM-DD.md. Use this skill whenever the user says "what should I do next", "plan my sprint", "what's on deck this week", "next sprint", "let's plan the week", "what should I focus on this sprint", "help me figure out the sprint", or otherwise asks for help deciding what to take on over the coming week. Trigger even if the user does not say the word "sprint" — if they're asking for a focused, time-boxed picture of what to do next given everything currently in motion, this is the right skill. Surfaces decisions, doesn't make them; always confirms the draft before writing.
---

# sprint

The "scrum master" capability of the project-planning plugin. The sprint skill helps the user decide what to do next given everything that's known about the project right now — recent notes, plans, the previous sprint, and configured external context (Slack, Jira, Confluence). It is a coach, not an automated planner: it surfaces what's load-bearing, asks the questions that need answering, and only writes a sprint file once the user has signed off on the draft.

## When to trigger

Fire on explicit asks for sprint planning or weekly focus:

- "what should I do next", "what should I focus on this week", "what's on deck"
- "plan my sprint", "let's plan the sprint", "next sprint", "kick off the new sprint"
- "what's load-bearing right now", "what should I prioritize"
- Anything that asks for a focused, time-boxed view of the coming week's work, even if the word "sprint" isn't used.

If the user is asking for a *current-state* synthesis rather than a forward-looking decision ("what's the plan", "where are we"), that's the `plan` skill — defer to it. The sprint skill is specifically about *picking what to do next*.

## Behavior

1. **Determine the sprint start date** with `date +%Y-%m-%d` (use the Monday of the current or upcoming week — ask the user if it's ambiguous which one they mean). Never guess the date.
2. **Gather context** in this order:
   - **Recent notes**: list and read recent files in `notes/`. Recency matters more than completeness — the last week or two is usually enough.
   - **Plan sources**: defer to the `plan` skill's reference documents to read whichever sources `AGENTS.md` declares (Jira, Confluence, local `plans/`, Slack-as-context). Do not redefine the read mechanics here — the `plan` skill owns them.
   - **Previous sprint**: if `plans/sprints/` contains a previous file, read the most recent one. Note what carried over, what got dropped, and what was completed.
   - **Slack activity (when configured)**: if `AGENTS.md` names a Slack channel for the project, pull the last week of activity from it read-only, using the same patterns as `plan/slack.md`. This is for surfacing decisions, asks, and blockers — not for posting.
3. **Coach, don't dictate.** Before drafting anything, surface the decisions the user is implicitly making by picking one sprint shape over another. Ask challenging questions when the load-bearing choice is unclear:
   - Which workstream is most at risk of slipping if it doesn't get attention this week?
   - What's been carried over from the previous sprint, and is it still the right call?
   - What's surfaced in notes that hasn't been triaged into a plan yet?
   - What's awaiting a decision the user can make right now (and is therefore unblocking)?
   - What's *not* going in the sprint, and is the user okay with that?
   - The skill surfaces these; the user answers them.
4. **Draft the sprint file.** Show the draft inline before writing. Path: `plans/sprints/<YYYY-MM-DD>.md`, dated to the start of the sprint week. If `plans/sprints/` does not exist, create it.
5. **Confirm explicitly, then write.** Do not write the file until the user has approved the draft. Edits are cheap; surprise files are not.

## Sprint file shape (loose)

Bullets organized by domain, with optional personal items. The skill encourages this shape; it does not enforce it. A reasonable default:

```markdown
# Sprint: <YYYY-MM-DD>

## On deck

### <Domain — e.g. UI>
- [ ] item, with a one-line *why* and a source link (Jira key, Confluence page, plans/ file, or note path)

### <Domain — e.g. infra>
- [ ] ...

### Personal / cross-cutting
- [ ] ...

## Carried over
- [ ] items from last sprint that are still in flight, with a note on what's blocking them

## Decisions awaiting Bill
- the choice that's gating something on this sprint

## Out of scope this week
- things considered and explicitly deferred (so the next sprint isn't confused about why they're missing)
```

Domains come from the project itself — read the project's `plans/` domain documents or `AGENTS.md` to discover them. Do not invent new domains the project hasn't already named.

## What this skill does *not* do

- It does not write to external systems. No Jira transitions, no Confluence edits, no Slack posts. Those are gated by their reference documents in `plan/` and require explicit user direction.
- It does not modify notes or domain documents in `plans/`. It reads them.
- It does not enforce a sprint cadence or template. If the user wants two-week sprints, or a sprint shape that breaks the default above, follow their lead.
- It does not auto-roll items from the previous sprint. Carrying something over is a decision, not a default.

## Relationship to `plan`

The `plan` skill answers "where are we?". The `sprint` skill answers "given where we are, what should this week be?". They share the same reference documents for reading external sources — the sprint skill should call into them, not duplicate them. If a planning source's read patterns need to change, change them in `plan/<source>.md` once.
