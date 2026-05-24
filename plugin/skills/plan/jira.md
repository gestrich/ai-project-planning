# Jira reference

Load this document when the project's `AGENTS.md` names Jira as a planning source (a project key, an epic ID, a board URL, or a list of issue keys).

This document is project-agnostic. Project keys, epic IDs, board IDs, and the cloudId always come from the project's `AGENTS.md` — never hardcode them here.

## The Atlassian MCP is the right tool

The same two MCP families that apply to Confluence apply to Jira: `mcp__claude_ai_Atlassian__*` (claude.ai integration) and `mcp__mcp-atlassian__*` (self-hosted). They are not interchangeable. The project's `AGENTS.md` should declare which family applies; if not, try the `claude_ai_Atlassian` family first.

Schemas are deferred. Load them via `ToolSearch` with `select:<tool_name>` before the first call in a session.

## Tools you'll most often need

- `getJiraIssue` — fetch a single issue by key.
- `searchJiraIssuesUsingJql` — query across the project, including filtering by epic link, sprint, assignee, status, label, or update time. JQL is Jira-specific; CQL won't work here.
- `getTransitionsForJiraIssue` — list the transitions available on an issue before trying to move it.
- `transitionJiraIssue` — move an issue through its workflow. A write.
- `addCommentToJiraIssue` — add a comment. Additive but still visible to others; treat as a write.
- `createJiraIssue` / `editJiraIssue` — create or modify issues. Writes.
- `getVisibleJiraProjects` — discover what's accessible if `AGENTS.md` doesn't name a project key.

The exact tool name prefixes depend on which MCP family applies — substitute accordingly.

## Reading: the common case

For a typical `plan` invocation, you're answering "what's in motion right now?" The shape:

1. **Pick the right query.** If `AGENTS.md` names epics, query their child issues (`"Epic Link" = <EPIC-KEY>` or the project's equivalent). If it names a board or sprint, query `sprint in openSprints()` scoped to the project. If it names neither, query open issues in the project, ordered by recent updates.
2. **Pull the fields you need, not the whole issue.** Status, assignee, summary, last update, and parent/epic are usually enough. Avoid pulling description/comments unless the synthesis demands it.
3. **Summarize by area.** Group issues by epic, component, or label depending on what the project uses. Don't list 40 issues flat — that's a dump, not a synthesis.
4. **Cite issue keys inline.** The key (`PROJ-123`) is the natural citation; the user can paste it into Jira directly.

Stale issues matter. If an issue hasn't been touched in weeks but is still "In Progress," flag it as drift rather than reporting it as active.

## Writing: confirm before any mutation

Jira writes have a smaller blast radius than Confluence overwrites (you can't accidentally erase someone's edits — Jira tracks comments, transitions, and field changes as discrete events). But they are visible to the team, sometimes ping people (status transitions, assignee changes), and are awkward to undo. So the rule is:

**Show the user the exact change you're about to make, get explicit confirmation, then execute.**

This applies to:

- Status transitions (Done, In Progress, Blocked, …)
- Assignee changes
- Adding or editing comments
- Creating new issues
- Editing summary, description, or any field
- Linking issues to epics or to each other

What "explicit confirmation" looks like:

- For a transition: "I'm about to move `PROJ-123` from `In Progress` → `Code Review`. Confirm?"
- For a comment: show the exact text, including any @mentions, and wait for go-ahead.
- For a new issue: show the summary, description, type, parent epic, and any labels, and wait for go-ahead.

A general request like "update the Jira ticket with what we just discussed" is **authorization to draft**, not to send. Draft, show, wait, execute.

## Mentions and notifications

Jira @mentions notify the mentioned user. Resolve usernames via `lookupJiraAccountId` (or the project's equivalent tool) before composing a comment, and surface to the user that the mention will notify someone. Never add an @mention they didn't ask for.

## Mapping the project's `AGENTS.md` declarations onto Jira

The project's `AGENTS.md` should give you, at minimum:

- A **cloudId** (often the hostname, e.g., `<company>.atlassian.net`).
- One or more **project keys**, **epic IDs**, or **board IDs** that scope the project.
- Optionally, named saved filters or JQL snippets that the project uses for its standard views.

When the user asks about "the plan" generally, default to: open issues in the named project(s), ordered by recent updates, grouped by epic. When they ask about a sub-area, narrow the JQL by component, label, or epic.

If `AGENTS.md` doesn't name the project key and you can't find it from one short search, ask — don't fabricate keys.
