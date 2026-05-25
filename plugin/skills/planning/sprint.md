# Sprint mode

Reference document for the `planning` skill when the user wants to plan the coming sprint. Sprint mode is a scrum-master coach — surface what's load-bearing, ask the questions that need answering, and only write the sprint file after the user has signed off on the draft.

## When to invoke sprint mode

Fire on explicit asks for sprint planning or weekly focus:

- "what should I do next", "what should I focus on this week", "what's on deck"
- "plan my sprint", "let's plan the sprint", "next sprint", "kick off the new sprint"
- "what's load-bearing right now", "what should I prioritize"
- Anything that asks for a focused, time-boxed view of the coming week's work, even if the word "sprint" isn't used.

If the user is asking for a *current-state* synthesis ("what's the plan", "where are we"), that's status mode — defer. Sprint mode is specifically about *picking what to do next*.

## Behavior

1. **Determine the sprint start date** with `date +%Y-%m-%d` (use the Monday of the current or upcoming week — ask the user if it's ambiguous which one they mean). Never guess the date.
2. **Gather context** in this order:
   - **Recent notes**: list and read recent files in `notes/`. Recency matters more than completeness — the last week or two is usually enough.
   - **Plan sources**: read whichever sources `AGENTS.md` declares (Jira, Confluence, local `plans/`, Slack-as-context). Use the corresponding reference documents ([jira.md](jira.md), [confluence.md](confluence.md), [slack.md](slack.md), [local.md](local.md)) — don't redefine the read mechanics here.
   - **Previous sprint**: if `plans/sprints/` contains a previous file, read the most recent one. Note what carried over, what got dropped, and what was completed.
   - **Slack activity (when configured)**: if `AGENTS.md` names a Slack channel for the project, pull the last week of activity from it read-only, using [slack.md](slack.md). This is for surfacing decisions, asks, and blockers — not for posting.
3. **Coach, don't dictate.** Before drafting anything, surface the decisions the user is implicitly making by picking one sprint shape over another. Ask challenging questions when the load-bearing choice is unclear:
   - Which workstream is most at risk of slipping if it doesn't get attention this week?
   - What's been carried over from the previous sprint, and is it still the right call?
   - What's surfaced in notes that hasn't been triaged into a plan yet?
   - What's awaiting a decision the user can make right now (and is therefore unblocking)?
   - What's *not* going in the sprint, and is the user okay with that?
   - Sprint mode surfaces these; the user answers them.
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

## Decisions awaiting the user
- the choice that's gating something on this sprint

## Out of scope this week
- things considered and explicitly deferred (so the next sprint isn't confused about why they're missing)
```

Domains come from the project itself — read the project's `plans/` domain documents or `AGENTS.md` to discover them. Do not invent new domains the project hasn't already named.

## What sprint mode does *not* do

- It does not write to external systems. No Jira transitions, no Confluence edits, no Slack posts. Those are gated by their reference documents and require explicit user direction.
- It does not modify notes or domain documents in `plans/`. It reads them.
- It does not enforce a sprint cadence or template. If the user wants two-week sprints, or a sprint shape that breaks the default above, follow their lead.
- It does not auto-roll items from the previous sprint. Carrying something over is a decision, not a default.

## Relationship to status mode

Status mode answers "where are we?". Sprint mode answers "given where we are, what should this week be?". They share the same reference documents for reading external sources — sprint mode should call into them, not duplicate them. If a planning source's read patterns need to change, change them in the source's reference doc once.
