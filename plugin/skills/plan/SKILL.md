---
name: plan
description: Show what the plan is for the current project — what's in motion, what's next, what's blocked — by reading whichever planning sources the project uses (Jira, Confluence, a local plans/ folder, Slack-as-context, or some combination). Use this skill whenever the user asks "what's the plan", "what's next", "where are we on X", "show me the plans", "what's the plan for <area>", "catch me up on this project", "what's the status", or anything that asks for a synthesized view of the project's direction. Also use this skill in reconciliation mode when the user asks "sync plans", "reconcile with Jira", "reconcile the docs", "what's drifted", "what should graduate", "is the local plan still aligned with Jira", or anything that asks to compare local plans/ against external sources of truth. Trigger even when the user does not name a specific source — figuring out which sources apply is the skill's job. The skill reads the project's AGENTS.md to discover which sources are configured and then loads only the matching reference documents in plugin/skills/plan/.
---

# plan

The project-aware entry point for "where are we and what's next?" The plan skill doesn't assume any particular planning system. It reads the project's `AGENTS.md` to find out which sources of truth the project uses, then loads only the reference documents that match — keeping context lean and avoiding patterns that don't apply.

## What this skill does

1. **Read `AGENTS.md`** at the current project's root and locate its project-planning section.
2. **Identify which planning sources apply.** A project may use any combination of:
   - **Jira** — epics, issues, sprint boards as the shipped-work source of truth.
   - **Confluence** — long-form plans, decisions, status pages.
   - **Local `plans/` folder** — markdown-only planning, either as the sole source of truth or as a supplement.
   - **Slack** — read-only context (recent decisions, asks, blockers from a configured channel).
3. **Load only the matching reference documents** from this skill's directory:
   - `jira.md` — Jira read/write discipline.
   - `confluence.md` — Confluence read-before-write discipline.
   - `slack.md` — Slack as read-only context capture.
   - `local.md` — the `plans/` folder convention.
4. **Synthesize a short, source-cited view** of the project's plan state. Group by area when the project has multiple workstreams; flag what's awaiting the user; never paraphrase what a source says without citing the source.

If `AGENTS.md` does not exist or has no project-planning section, say so plainly and offer to run `bootstrap` (a sibling skill) — don't try to guess the planning shape from the repo contents.

## When to trigger

Fire on explicit asks for project direction or status:

- "what's the plan", "what's the plan for X", "what's next", "what should I be looking at"
- "show me the plans", "where are we on Y", "what's the status of Z"
- "catch me up on this project", "give me the rundown"
- Anything that asks for a synthesized view of work-in-flight, regardless of whether the user names a specific source.

If the user names a specific source ("check Jira", "read the Confluence page"), still go through this skill — the reference documents are the right place for the read patterns — but you can skip loading the references that don't apply.

Also fire on reconciliation asks — see the **Reconciliation mode** section at the bottom of this file:

- "sync plans", "reconcile with Jira", "reconcile the docs"
- "what's drifted", "what should graduate", "what's stale in plans/"
- "is the local plan still aligned with Jira/Confluence"

## Vocabulary

The plugin uses these terms consistently; the reference documents assume them:

- **Notes** — raw transcripts in `notes/`, captured by the `notes` skill. Inputs, not plans.
- **Plans** — supplemental local planning artifacts in `plans/`. Either *the* source of truth or supplementary to Jira/Confluence, depending on what `AGENTS.md` declares.
- **Domain documents** — files inside `plans/` organized by area of the project (UI, infra, testing, etc.), not by time.
- **Sprints** — time-boxed actionable lists in `plans/sprints/`, owned by the `sprint` skill.
- **Source of truth** — for projects that use Jira/Confluence, those are authoritative for shipped plans. `plans/` holds in-incubation ideas and personal items.

## Reading `AGENTS.md`

The project-planning section in `AGENTS.md` is free-form markdown — no enforced schema. Read it in natural language. Typical signals to look for:

- A Jira project key, epic ID, or board URL → load `jira.md`.
- A Confluence space key, page ID, or page URL → load `confluence.md`.
- A Slack channel name or ID flagged as "for context only" → load `slack.md`.
- A mention of `plans/` as a source of truth, or no external system named → load `local.md`.
- A list of GitHub repos belonging to the project — relevant for `pr-review`, but the plan skill can cite them when summarizing work-in-flight.

If a section is ambiguous (e.g., a Jira URL is mentioned but the project clearly uses local `plans/` as its primary source), use both — but mark in the synthesis which is authoritative.

## How to synthesize

A good `plan` response is:

- **Source-cited.** Every statement traces to either Jira (with issue keys), Confluence (with page link), the local `plans/` file path, or the Slack message permalink. If a fact has no source, say it's your inference.
- **Grouped by area, not by source.** The user thinks in terms of workstreams (UI, infra, telemetry, …), not in terms of "things Jira told me." Pull from all sources into one coherent view, then cite per item.
- **Surface what's awaiting them.** Decisions waiting on them, blockers, asks aimed at them — these go at the top.
- **Honest about gaps.** If a source is stale, or a Jira epic hasn't been touched in three weeks, or the Slack channel has gone quiet, say so. Don't fabricate motion.

Keep the synthesis short. The user is asking for orientation, not a full report.

## Writes are gated by source-specific discipline

When the user asks for a write that touches a planning source (updating a Confluence page, transitioning a Jira issue, posting to Slack), defer to the reference document for that source. Each reference document carries the read-before-write or explicit-authorization rules appropriate to its system. The `plan` skill itself does not do writes — it routes.

## Reconciliation mode

Reconciliation between local `plans/` and external sources of truth (Jira, Confluence) lives as a mode of this skill rather than a separate `sync` skill. The trigger phrases are listed under **When to trigger** above. Use this mode when the user wants to know what's drifted, what should graduate, or what's gone stale — not when they want a fresh status read (that's the default mode).

### What reconciliation answers

Three questions, side-by-side:

1. **Graduation candidates.** Items in `plans/` that are firm enough they should live in the external source of truth now. A `plans/ui.md` bullet that has been sitting under "decided" for two weeks and is being actively worked on belongs in a Jira issue or on the Confluence status page, not buried in a markdown file.
2. **Drift.** Cases where `plans/` and the external source disagree about the same item — a decision recorded one way locally and a different way in Confluence, a Jira issue marked `Done` while the local plan still has it as "in flight," and so on. Drift is a pointer to a conversation the user needs to have, not a thing to silently resolve.
3. **Stale local items.** Bullets in `plans/` that the external source has overtaken — an idea that's now shipped, a question that's been answered in a Confluence page, a workstream that's been deprioritized in Jira. These are candidates for archiving or deletion, not for graduation.

### How to run reconciliation

1. **Confirm `plans/` is in scope.** If `AGENTS.md` doesn't declare a local `plans/` folder, there is nothing to reconcile — say so and stop. If `plans/` is the project's *sole* source of truth, there are no external sources to reconcile against either — say so and stop.
2. **Load the matching reference documents.** Same logic as the default mode: read `AGENTS.md`, identify which external sources apply, load `jira.md` and/or `confluence.md`. Always load `local.md` since reconciliation always touches `plans/`. Each reference document carries a **Reconciliation** subsection with the source-specific mechanics — read those, don't re-derive them here.
3. **Read both sides in full.** Reconciliation is not a synthesis from a partial sample. Read the manifest and the named domain documents from `plans/`; read the configured Jira queries / Confluence pages in full. If you skip a source the report is misleading.
4. **Group the report by the three questions above**, not by source. The user wants "what should graduate / what's drifted / what's stale", not "here's what Jira says, then here's what Confluence says, then here's what local says". Cite the source on each item.
5. **Show every item the user could act on, not the top N.** Reconciliation is a hygiene pass — under-reporting hides the work.

### Writes are gated, same as the rest of `plan`

This skill still does not perform mutations on its own. When reconciliation surfaces an item the user wants to act on — graduating a `plans/` bullet to a new Jira issue, updating a Confluence page to reflect a decision recorded locally, deleting stale local content — defer to the matching reference document's write discipline:

- New Jira issue or transition → `jira.md` "show the user the exact change, get explicit confirmation, then execute."
- Confluence page edit → `confluence.md` read-before-write cycle with version concurrency.
- Local file edit or delete → `local.md` "show the proposed change before applying it."

Treat each acted-on item as its own confirmation. A single "go ahead and apply everything" is too coarse — reconciliation reports often contain 10+ items spanning multiple sources, and "everything" tends to obscure a few items the user didn't actually want.
