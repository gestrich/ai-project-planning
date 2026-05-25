---
name: plans
description: Help the user manage their project — capture raw transcripts and brain dumps, show what's in motion and what's next, plan the coming sprint, and reconcile local plans against external sources of truth. Fire on (capture) "here's my brain dump", "voice transcript", "dump this into notes", or any long unstructured first-person paste that reads like a spoken transcript; (status) "what's the plan", "what's next", "catch me up", "where are we on X", "what's the status"; (sprint) "what should I do next", "plan my sprint", "what's on deck this week", "what should I focus on"; (reconciliation) "sync plans", "what's drifted", "what should graduate", "is the local plan still aligned with Jira/Confluence". Trigger even when the user does not name the mode — the skill picks the right mode from the ask and reads the project's AGENTS.md to figure out which planning sources apply (Jira, Confluence, a local plans/ folder, Slack-as-context, or some combination).
---

# plans

One skill, four modes — all driven by what the user just asked for. The skill reads the project's `AGENTS.md` to discover which planning sources the project uses, then loads only the reference documents that match. No mode is assumed; the user's ask picks it.

## Modes

| The user asks… | Mode | Reference doc |
|----------------|------|---------------|
| "here's my brain dump", or pastes a transcript | **capture** | [notes.md](notes.md) |
| "what's the plan", "what's next", "catch me up" | **status** | source-specific: [jira.md](jira.md), [confluence.md](confluence.md), [slack.md](slack.md), [local.md](local.md) |
| "what should I do next", "plan my sprint" | **sprint** | [sprint.md](sprint.md) (plus the source-specific docs above) |
| "what's drifted", "what should graduate", "sync plans" | **reconciliation** | the source-specific docs above, focused on their **Reconciliation** subsections |

If the ask is ambiguous, name the modes it could fit and let the user pick — don't guess.

## Reading `AGENTS.md`

The skill finds project configuration in whatever shape the user has written it. There is no required heading and no enforced schema — read `AGENTS.md` like a teammate would, and look for signals:

- A Jira project key, epic ID, or board URL → load [jira.md](jira.md).
- A Confluence space key, page ID, or page URL → load [confluence.md](confluence.md).
- A Slack channel name or ID flagged as project context → load [slack.md](slack.md).
- A mention of `plans/` as a source, or no external system named → load [local.md](local.md).
- A list of GitHub repos — relevant for `docs-update` (which may scan recent PRs for technical-doc drift), and useful when citing work-in-flight.
- A list of which plugin skills the project uses, possibly with one-line notes on what each is for — honor that list when deciding what's in scope.

If `AGENTS.md` doesn't exist, or has no planning-relevant content, say so plainly and offer to run `bootstrap`. Don't guess the planning shape from the repo contents.

## Vocabulary

The plugin uses these terms consistently across modes and reference documents:

- **Notes** — raw transcripts in `notes/`, captured by this skill in capture mode. Inputs, not plans.
- **Plans** — supplemental local planning artifacts in `plans/`. Either *the* source of truth (when no external system is configured) or *supplementary* to Jira/Confluence (when one is). `AGENTS.md` declares which.
- **Domain documents** — files inside `plans/` organized by area of the project (UI, infra, testing, etc.), not by time.
- **Sprints** — time-boxed actionable lists in `sprints/` (top-level, sibling of `plans/`), owned by sprint mode.
- **Source of truth** — for projects that use Jira/Confluence, those are authoritative for shipped plans. `plans/` holds in-incubation ideas and personal items.

## Mode-picking heuristics

**Capture mode** fires on either of two signals:

- An explicit ask ("dump this into notes", "voice transcript", "save this").
- The shape of the input: a long, unstructured, first-person paste with conversational filler, repeated thoughts, and no headings. If it reads like the user talking to themselves, treat it as a transcript even if they didn't name it.

When in doubt, ask before writing.

**Status mode** fires on asks for *current* state — "where are we", "what's the plan", "what's the status", "catch me up". The output is a synthesized view, source-cited, grouped by area not by source.

**Sprint mode** fires on asks for *forward* state — "what should I do next", "plan the week", "what's on deck". The output is a draft sprint file at `sprints/<YYYY-MM-DD>.md`, written only after the user has approved the draft. Sprint mode is a coach, not an automated planner — surface decisions, don't make them.

**Reconciliation mode** fires on asks for *drift* — "what's drifted", "what should graduate", "is the local plan aligned with Jira". The output is a hygiene report grouped by question (graduation candidates / drift / stale) — not by source. Reconciliation surfaces drift; it never silently resolves it.

## Synthesis principles (status and sprint modes)

A good response is:

- **Source-cited.** Every statement traces to a Jira issue key, a Confluence page link, a local file path, or a Slack permalink. Inferences are labeled as such.
- **Grouped by area.** The user thinks in workstreams (UI / infra / telemetry / …), not in sources. Pull from all sources into one coherent view, then cite per item.
- **Surface what's awaiting them.** Decisions waiting on the user, blockers, asks aimed at them — top of the response.
- **Honest about gaps.** If a Jira epic hasn't moved in three weeks, a Confluence page is stale, or the Slack channel has been silent, say so. Don't fabricate motion.

Keep responses short. The user is asking for orientation, not a report.

## Reconciliation specifics

Reconciliation answers three questions, side-by-side:

1. **Graduation candidates.** Items in `plans/` firm enough to live in the external source now — a `plans/ui.md` bullet that's been "decided" for two weeks and is actively being worked on belongs as a Jira issue or on the Confluence status page.
2. **Drift.** Cases where `plans/` and the external source disagree about the same item. A pointer to a conversation, not a thing to silently resolve.
3. **Stale local items.** Bullets in `plans/` overtaken by external decisions — already shipped, already answered, already deprioritized. Candidates for archive or delete.

Mechanics:

- If `AGENTS.md` doesn't declare a local `plans/` folder, there's nothing to reconcile — say so and stop. If `plans/` is the *sole* source, there's no external source to reconcile against — say so and stop.
- Always load [local.md](local.md) (reconciliation always touches `plans/`) plus the relevant external source docs ([jira.md](jira.md), [confluence.md](confluence.md)). Each reference document carries a **Reconciliation** subsection with the source-specific mechanics — read those.
- Read both sides in full. A partial sample makes the report misleading.
- Group the report by the three questions above, not by source. Cite the source on each item.
- Show every actionable item, not the top N. Reconciliation is hygiene; under-reporting hides work.

## Writes are gated by source-specific discipline

This skill itself does not perform external mutations. When a mode surfaces an action the user wants to take — graduating a `plans/` bullet to a Jira issue, updating a Confluence page, posting a Slack reply, writing a sprint file, writing a transcript file — defer to the matching reference document's write rules:

- New Jira issue or transition → [jira.md](jira.md): show the exact change, get explicit confirmation, then execute.
- Confluence page edit → [confluence.md](confluence.md): read-before-write with version concurrency.
- Local file edit, sprint file write, or transcript capture → the relevant reference doc's writing guidance; show before writing.

Treat each acted-on item as its own confirmation. A single "go ahead" for many items hides the ones the user didn't actually want.
