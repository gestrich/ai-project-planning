# Confluence reference

Load this document when the project's `AGENTS.md` names Confluence as a planning source (a space key, a root page ID, or a page URL).

This document is project-agnostic. The specific cloudId, space key, and page IDs always come from the project's `AGENTS.md` — never hardcode them here.

## The Atlassian MCP is the right tool

There are two families of Atlassian MCP tools that may be available in a session: `mcp__claude_ai_Atlassian__*` (the claude.ai integration, OAuth-scoped to the user's account) and `mcp__mcp-atlassian__*` (a self-hosted MCP server, scoped to whatever spaces it was configured for). They are not interchangeable — the same page ID can return content from one and "no permission" from the other.

The project's `AGENTS.md` should declare which family applies. If it doesn't, try the `claude_ai_Atlassian` family first (it authenticates against the user's account) and fall back to `mcp-atlassian` if the project's space isn't accessible.

Schemas for these tools are deferred. Load them via `ToolSearch` with `select:<tool_name>` before the first call in a session. Trying to call them directly without the schema returns InputValidationError.

## Tools you'll most often need

- `getConfluencePage` — fetch current content plus the version number. Use `contentFormat: "markdown"` for read-only inspection; `contentFormat: "html"` when you'll write back in the same turn (HTML round-trips preserve formatting and inline comments).
- `getConfluencePageDescendants` / `getPagesInConfluenceSpace` — walk a page tree to find the right child.
- `searchConfluenceUsingCql` — search by title/text. CQL is Confluence-specific, not JQL.
- `updateConfluencePage` — write back. Requires the version number from the most recent read.
- `createConfluenceFooterComment` / `createConfluenceInlineComment` — additive comments that don't touch body content. These do not require the read-before-write dance.

The actual tool name prefixes (`mcp__claude_ai_Atlassian__getConfluencePage`, etc.) depend on which MCP family applies — substitute accordingly.

## Reading: most invocations stop here

For the typical `plan` invocation, reading is all you need:

1. **Resolve the page.** If `AGENTS.md` names a specific root page ID, start there. If the user asked about a sub-area, walk `getConfluencePageDescendants` from the root and pick the matching child. If `AGENTS.md` only names a space, use `searchConfluenceUsingCql` to find the right page.
2. **Fetch with `contentFormat: "markdown"`** for readability.
3. **Summarize for the task.** Decisions, owners, dates, blockers, direction — surface those. Don't paste the whole page.
4. **Cite specifically.** Name the page and link to it. The page URL is the natural citation; show it inline next to the claim.

Do not paraphrase from memory or from a local note file. The live Confluence page is the source of truth; other people edit it between sessions. If the user asks "what does the doc currently say about X," re-read.

## Writing: read before write, always

Confluence pages are shared. Other people edit them directly between sessions. If you push an update built on a stale copy — a local draft, remembered content, an assumption about what the page contains — you will **silently overwrite their edits**. Confluence has no merge. The page just becomes whatever you sent. Recovering means digging through page history and reconciling by hand.

The required steps for any write:

1. **Resolve the target page.** If ambiguous across siblings, ask the user before doing anything destructive.
2. **Read the live page** with `contentFormat: "html"`. Capture both the body and the current version number — the version is what Confluence uses for optimistic concurrency.
3. **Diff against what you intended to write.** If the live page contains content you didn't expect — recent edits, inline annotations, structural changes — **stop and surface it to the user before proceeding.** Don't silently merge or silently overwrite.
4. **Compose the update as a delta** on top of the live content. Don't replace the whole page with a locally-composed version unless the user explicitly asked for a full rewrite.
5. **Write back** via `updateConfluencePage`, passing the version number from step 2. If Confluence rejects the write because the version is stale, someone edited the page between your read and your write — redo the read-diff-write cycle on the new version. Do not force-overwrite.
6. **Confirm the result.** Report what changed, on which page, and link to the page so the user can spot-check.

The read-before-write rule applies to any mutation of body content: edits, appends, status updates, formatting fixes, reorganization. The only exception is additive comments (`createConfluenceFooterComment` / `createConfluenceInlineComment`), which don't overwrite body content.

## Mapping the project's `AGENTS.md` declarations onto Confluence

The project's `AGENTS.md` should give you, at minimum:

- A **cloudId** (often the hostname, e.g., `<company>.atlassian.net`).
- A **space key** and/or **root page ID** that anchors the project's documentation.
- Optionally, a list of named sub-pages that the project considers canonical (status page, decisions log, etc.).

When the user asks about "the plan" generally, start from the root page. When they ask about a sub-area, walk the descendant tree from the root rather than guessing IDs.

If `AGENTS.md` lacks any of this and you can't infer it from one short search, ask the user — don't fabricate page IDs.
