---
name: pr-review
description: Review a GitHub pull request against the project's own documentation — both directions. Fetch the diff from the GitHub repo(s) named in the project's AGENTS.md, cross-reference the change against the documented conventions (Confluence pages, local docs, plan files declared in AGENTS.md), and produce two outputs in one pass: (a) a PR review surfacing where the diff diverges from documented conventions, and (b) doc-update suggestions for patterns or decisions in the diff that aren't reflected in the docs yet. Use this skill whenever the user asks "review this PR", "review PR #123", "is this PR aligned with our docs", "check this PR against the spec", "what's wrong with this PR", "does this PR match our conventions", "check the open PRs for doc drift", "sweep open PRs", or pastes a GitHub PR URL and asks for feedback. Trigger even when the user doesn't name the skill — if they're handing you a PR and asking what it means against the project's documented direction, this is the right home. Read-only by default; any PR comment or doc edit requires explicit confirmation. Local checkout is out of scope in v1 — diffs come from the GitHub API.
---

# pr-review

Documentation-aware GitHub PR review. Fetches a PR from the repo(s) declared in the project's `AGENTS.md`, cross-references the diff against the project's documented conventions, and surfaces findings in two directions:

1. **PR-side findings** — where the diff diverges from what the docs say should happen.
2. **Docs-side findings** — where the diff introduces patterns or decisions that the docs don't reflect yet.

The two are deliberately produced together. A PR that adopts a new pattern isn't necessarily wrong; sometimes the docs are stale. Reporting both sides lets the user decide which way to close the gap.

## When to trigger

Fire on any phrasing that hands you a PR and asks for an opinion on it:

- "review this PR", "review PR #123", "what do you think of this PR"
- "is this PR aligned with our docs", "does this match our conventions"
- "check the open PRs for doc drift", "sweep open PRs"
- A bare GitHub PR URL pasted into the conversation when the project's `AGENTS.md` declares that repo.

If the user is asking for a *generic* code review without doc cross-referencing — "look for bugs in this PR" — that's a different job; this skill is specifically the documented-conventions read. You can still run it and just note that the docs-cross-reference step found nothing relevant.

## Behavior

The skill runs in three phases: **fetch → cross-reference → propose**. Everything before phase 3 is read-only. PR comments and doc edits both require explicit confirmation in phase 3.

### 1. Fetch

- **Resolve the repo.** Read the project's `AGENTS.md` Project Planning section and pick up the GitHub repo(s) it declares. If the user gave a full PR URL, take the repo from the URL and confirm it appears in `AGENTS.md` — if it doesn't, ask once before continuing (the PR may belong to a different project).
- **Use the `gh` CLI when available** per Bill's global `AGENTS.md` preferences (`gh pr view`, `gh pr diff`, `gh pr view --json`). Fall back to the GitHub REST API only when `gh` isn't installed. Both work; `gh` is the preferred surface.
- **Fetch the diff and metadata.** Pull the PR description, the list of changed files, and the diff itself. Capture the head SHA — if the user later asks to comment, you want to make sure the PR hasn't moved underneath you.
- **No local checkout in v1.** This is intentional: it keeps the skill portable and avoids dragging in build/test machinery the skill has no business running. If the diff is so large that the API view isn't enough, say so and surface the file list rather than guessing.

### 2. Cross-reference

For each meaningful change in the diff, look up what the project's documentation says about that area. The documentation sources are whichever ones `AGENTS.md` declares for this project — usually some mix of:

- **Confluence pages** — follow `plan/confluence.md` to read them (load Atlassian MCP schemas via `ToolSearch`, fetch with `contentFormat: "markdown"`). Don't redefine those patterns here.
- **Local docs** — files inside the project, typically under `docs/` or `plans/`. Read directly.
- **Plan files** — `plans/plan.md` and the domain documents it references, per `plan/local.md`.

The goal of the cross-reference isn't to mechanically grep for filenames mentioned in the docs. It's to ask, for each load-bearing change in the diff: *is there documented guidance that speaks to this, and does the change match it?* Use judgment about what's load-bearing. A renamed local variable doesn't need a docs lookup; a new public API, a changed architectural boundary, a new dependency, a different error-handling pattern, a deviation from a naming convention — those do.

Keep this phase quiet. The user sees the proposals, not the raw doc dumps, unless they ask.

### 3. Propose

Produce two clearly-separated sections. Show them inline.

**A. PR review against documented conventions**

For each PR-side finding:

- **What the divergence is** — the specific change, the file/line range, what the docs say should happen instead.
- **Where the doc lives** — the page, file, or section being relied on (link or path so the user can verify).
- **Severity** — a quick read on how load-bearing the divergence is. A naming convention slip is different from contradicting an architectural decision. Don't grade everything as critical; that defeats the point.
- **Suggested resolution** — change the PR, or update the doc, or accept the divergence as a one-off with a comment in the PR explaining why.

If the diff doesn't diverge from anything documented, say so plainly. An empty PR-side section is a valid outcome.

**B. Documentation update suggestions**

For each docs-side finding:

- **What the docs are missing** — a pattern, decision, dependency, or convention the diff introduces or relies on that isn't in the docs.
- **Where it should land** — the specific page, file, or section. If the right home is ambiguous, surface that as a question rather than picking one silently.
- **Why it's worth landing** — what's at stake if the doc stays as-is while this PR ships.

The output shape here intentionally mirrors `docs-update`'s proposal format so the two skills compose cleanly in a weekly cadence.

### 3a. Act (only after explicit confirmation)

- **PR comments**: when the user picks one or more PR-side findings to post as comments, draft each comment and show it inline first. Wait for explicit go-ahead before posting (`gh pr review --comment` or `gh pr comment`). Re-check the PR head SHA before posting — if it's moved, surface that and re-fetch rather than commenting on a stale diff.
- **Doc edits**: when the user picks one or more docs-side findings to act on, follow the same write discipline as `docs-update`. For Confluence, follow `plan/confluence.md`'s read-before-write protocol strictly (read live HTML, capture version, write back with the version for optimistic concurrency). For in-repo docs, show the proposed diff and wait for confirmation before writing.
- The proposal list is not the confirmation. Treat each action as a separate explicit ask.

## Mapping the project's `AGENTS.md` declarations onto this skill

The Project Planning section of `AGENTS.md` should give you:

- One or more **GitHub repos** in `owner/repo` form. Without at least one, this skill has nothing to read; tell the user and stop.
- A **documentation source** — Confluence space + root page id, local docs folder path, or the local `plans/` convention. Without one, the skill can still produce the diff summary but should be explicit that there's no documented baseline to cross-reference against.
- Optionally, an **ownership note** for the docs — whether the project has direct write access or another team owns them. Default to "project owns it" unless `AGENTS.md` says otherwise. The ownership routing applies to docs-side findings exactly the way it does in `docs-update`.

If a required piece is missing, ask the user once and offer to run `bootstrap` (or its refresh mode) to declare it. Don't fabricate repo names or page IDs.

## What this skill does *not* do

- **It does not check out the PR locally** in v1. Diffs come from the GitHub API. If a future version adds local checkout, it'll be opt-in, not the default.
- **It does not run tests, linters, or builds.** The PR's own CI handles that. This skill is specifically about documented-conventions alignment.
- **It does not post PR comments or edit docs without explicit confirmation.** Read-before-write applies to both directions.
- **It does not generate generic code review feedback** (style nits, micro-optimizations, subjective preferences). Findings must trace back to something the project's docs actually say, or to a docs-shaped gap the PR exposes.
- **It does not modify local `notes/` or `plans/`.** Those are owned by `notes`, `plan`, and `sprint`.

## Relationship to other skills

- The Confluence and local-docs read patterns come from `plan/confluence.md` and `plan/local.md`. If those patterns need to change, change them once in the reference docs — don't duplicate here.
- The docs-side output shape intentionally mirrors `docs-update`. The two skills can run side-by-side in a weekly cadence: open PRs reviewed, recent Slack swept, doc gaps proposed once from both sides.
- `bootstrap` is the right answer when `AGENTS.md` is missing the repo or docs declaration this skill needs.
