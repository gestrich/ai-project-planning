# Local `plans/` reference

Load this document when the project's `AGENTS.md` declares the local `plans/` folder as a planning source — either as the project's sole source of truth, or as a supplement to Jira / Confluence.

## What `plans/` is for

`plans/` holds in-repo, markdown-only planning artifacts. Two flavors of project use it differently:

- **Sole source of truth.** Projects without Jira or Confluence treat `plans/` as authoritative. Domain documents inside `plans/` describe the work; sprint files derived from them drive what gets done.
- **Supplementary.** Projects that ship work through Jira/Confluence treat `plans/` as a workspace for ideas-in-incubation, the user's personal items, and synthesis that doesn't belong in the shared system yet. Once an item firms up, it graduates to the external source.

The project's `AGENTS.md` should say which flavor applies. If it doesn't, treat `plans/` as supplementary by default — graduation candidates show up in the synthesis but nothing in `plans/` is treated as a commitment.

## The manifest convention

The skill expects a top-level **manifest** at `plans/plan.md`. The manifest enumerates the other files in `plans/`, one line each, so the skill can navigate the project's planning shape without prescribing a structure beyond "there is a manifest at the top."

A typical manifest looks like:

```markdown
# Plans

This project's planning lives in this folder. The files below are the canonical documents; this manifest is the index.

## Domain documents
- [ui.md](ui.md) — UI workstream: design system, screens, navigation.
- [infra.md](infra.md) — Infra workstream: deployment, observability, cost.
- [testing.md](testing.md) — Test strategy and coverage.

## Personal / incubation
- [ideas.md](ideas.md) — half-formed ideas before they graduate to a domain doc.
- [decisions.md](decisions.md) — running log of decisions made locally.

## Sprints
Sprint files live in `sprints/`. See the `sprint` skill.
```

That shape is illustrative, not enforced. The only hard requirement is:

- `plans/plan.md` exists.
- It mentions, in some form, the other files in `plans/` so the skill can find them.

Style — section headers, bullet form, ordering — is the project's call.

## How to read

1. **Start at `plans/plan.md`.** This is the entry point. Read the whole manifest — it's small.
2. **Walk to the documents it names.** Read only those relevant to what the user asked. If they asked broadly ("what's the plan?"), skim each domain document; if they asked about a specific area, fetch only that one.
3. **Treat the documents' own structure as the source.** Don't impose headings or sections that aren't in the file. If a domain document is loose bullets, summarize it as loose bullets; if it has an explicit decisions section, surface decisions separately.
4. **Cite file paths inline.** `plans/ui.md` is the natural citation. When quoting, name the file.

If `plans/plan.md` does not exist:

- If the project's `AGENTS.md` says local is the source of truth, surface this as a setup gap — the project is missing its manifest. Offer to run `bootstrap`.
- If `plans/` exists but the manifest is missing, list what's in `plans/` and ask the user whether to treat all of it as part of the plan, or to pick a subset.

## How to write

Local files are not shared — there's no read-before-write hazard like Confluence has. Still, the user's planning files are their thinking, not yours. The rule:

**Show the proposed change before applying it.** Even small edits get a quick "I'm about to add a bullet under `## Decisions` in `plans/ui.md` — confirm?" The cost of asking is low; the cost of mangling someone's thinking files is real.

Specific patterns:

- **Appending** to a domain document is the most common write. Show the lines being added and the section they're going into.
- **Restructuring** a domain document (reordering sections, moving content) deserves a clearer diff — show before/after for the affected region.
- **Creating a new domain document** updates the manifest too. Show both changes (the new file's contents and the manifest line being added) before writing.
- **Deleting** anything in `plans/` is rare and should always confirm.

## What `plans/` is *not*

- **It is not for raw transcripts.** Those go in `notes/` via capture mode. If the user pastes a transcript and asks to "save it to plans," redirect: it belongs in `notes/`, and `plans/` synthesizes from there.
- **It is not for sprint files.** Sprints live under the top-level `sprints/` folder (sibling of `plans/`) and are owned by sprint mode. The manifest can mention sprints, but individual sprint files don't go in `plans/`.
- **It is not for everything in the repo that mentions planning.** A `ROADMAP.md` at the repo root is not `plans/`. Don't conflate them. If a project has both, surface this as something the user might want to reconcile during bootstrap.

## Mapping the project's `AGENTS.md` declarations onto local plans

The project's `AGENTS.md` should give you, at minimum:

- A statement of whether `plans/` is the project's source of truth or supplementary.
- Optionally, a list of expected domain documents (`ui`, `infra`, `testing`, …) so the skill knows what to expect even before reading the manifest.

If `AGENTS.md` is silent on `plans/` but the folder exists, treat it as supplementary and surface in the synthesis that the project's mode is undeclared.

## Reconciliation

When the `plan` skill enters reconciliation mode, this document covers the *local* half of the comparison — the half that's always present, paired with whichever external source `jira.md` and/or `confluence.md` cover.

For projects where `plans/` is the sole source of truth, reconciliation is a no-op: there's nothing external to reconcile against. Say so and stop. The user may be looking for a sprint review instead — point them at the `sprint` skill.

For projects where `plans/` is supplementary to an external source, the three reconciliation questions break down on the local side as:

- **Graduation candidates.** Bullets in domain documents that have firmed up — a "decided" section that's stable, a workstream that's now being actively executed, an idea that has crossed from incubation into commitment. These are candidates to move into Jira (as issues) or Confluence (as canonical doc content). The graduation itself is a write against the external source — defer to `jira.md` / `confluence.md` for that — but the *identification* of the candidate is a local read.
- **Drift.** Statements in `plans/` that no longer match what the external source says. The user wrote a decision down locally, then the team's Jira / Confluence record evolved without the local note catching up (or vice versa). Show both and let the user pick which is now correct.
- **Stale local items.** Content in `plans/` that's been overtaken — work that shipped, decisions that were superseded, questions that have been answered in Confluence. Candidates to archive or delete from `plans/`; the local-file write discipline below applies.

### How to read `plans/` for reconciliation

Reconciliation needs the *whole* local picture, not a slice. Read `plans/plan.md` and then read each domain document it names. Skipping documents here will under-report drift and graduation — the whole point of the mode is to be comprehensive.

The manifest's "Personal / incubation" section (`ideas.md`, `decisions.md`, etc.) matters more than usual for reconciliation: that's where graduation candidates accumulate. Read those even if the user didn't explicitly ask about them.

### Local writes from reconciliation

If reconciliation produces a local action — deleting a stale bullet, archiving a domain document, updating the manifest to drop a graduated workstream — the local write discipline from this document still applies: show the proposed change before applying it. Reconciliation tends to produce *several* small local edits at once; confirm each one individually rather than batching them under a single "go ahead." Local files are the user's thinking, and a batch confirmation tends to bury one or two items the user would have wanted to keep.
