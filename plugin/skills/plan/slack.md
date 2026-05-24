# Slack reference

Load this document when the project's `AGENTS.md` names a Slack channel as a context source.

**Scope: this skill uses Slack read-only.** The `plan` skill does not post to Slack. Posting lives in other skills (`docs-update`, the standalone `slack-message` skill, project-specific Slack skills) — each carries its own write discipline. This document covers reading for project context only.

This document is project-agnostic. Channel IDs, workspace URLs, and user IDs always come from the project's `AGENTS.md` (or from runtime lookups) — never hardcode them here.

## The Slack MCP is the right tool

When available, the `mcp__claude_ai_Slack__*` family is the right way to read Slack. Schemas are deferred. Load them via `ToolSearch` with `select:<tool_name>` before the first call in a session.

## Tools you'll most often need

- `slack_read_channel` — paginated message history, newest first. Default limit is 100; for "what's new" reads, 20–30 is usually plenty.
- `slack_read_thread` — replies under a parent message. Channel reads return parent messages with a `thread_ts` and reply count; follow up with this when threads look load-bearing.
- `slack_search_public` — keyword + semantic search. Use `in:<channel-name>`, `from:<@USERID>`, `after:YYYY-MM-DD`, `before:YYYY-MM-DD`, `has:link`, `is:thread` modifiers.
- `slack_search_channels` / `slack_search_users` — resolve names to IDs.
- `slack_read_user_profile` — turn a user ID back into a name when summarizing.
- `slack_read_canvas` — read a channel's Canvas (the pinned doc), when the project has one and the user asks about it.

## Reading patterns

What the user is asking for shapes the read. A few common shapes:

### "Catch me up" / "What's happening in the channel?"

1. `slack_read_channel(channel_id=<from AGENTS.md>, limit=20)`.
2. Scan parents for non-trivial reply counts or reactions — those are the load-bearing threads. Pull each interesting one with `slack_read_thread`.
3. Summarize **by topic**, not by timestamp. Surface:
   - Asks aimed at the user (questions, decisions waiting on them) — these go at the top.
   - Decisions made or direction shifts.
   - Blockers, especially anything blocking on the user.
   - New people joining the channel, only if it signals a new team coming aboard.
4. Drop bot noise (Slackbot join messages, automated alerts that aren't load-bearing) unless they're signal.

### "What did <person> say about <X>?"

1. If you don't have their user ID, `slack_search_users(query=<name>)`.
2. `slack_search_public(query="in:<channel> from:<@Uxxx> <X>")`. Keyword search first; if zero hits, retry with a natural-language query — semantic search often catches what keyword search misses.
3. Pull surrounding context only when it matters — `slack_read_thread` for replies, `slack_read_channel` with `oldest`/`latest` for the conversation around a single message.

### "What did the team decide about <X>?"

1. `slack_search_public(query="in:<channel> <X>")`, default relevance sort.
2. For each promising hit, fetch the full thread. Decisions live in thread replies more often than in parent messages.
3. Report the decision *and* who made it (or "unclear, last word from <person>"), so the user can follow up.

### "What's in the Canvas?"

If the project's channel has a Canvas pinned in its topic and `AGENTS.md` mentions it, call `slack_read_canvas`. Canvas IDs aren't always handy; if needed, look at recent Slackbot messages about Canvas updates — they include the file ID.

## Output shape

A good `plan`-via-Slack synthesis is:

- **Short prose**, 3–8 bullets, organized by topic, prioritizing things that need the user's attention.
- **Permalinks** to source messages so the user can jump in. The tool usually returns a permalink — use it verbatim. Otherwise, Slack message URLs are `https://<workspace>.slack.com/archives/<channel-id>/p<ts-without-the-dot>`.
- **Names, not user IDs.** Resolve `<@U…>` before showing them.
- **Don't paste full message text** unless wording matters (the user asked "what exact phrasing did <person> use?"). Summarize.

If a question can't be answered from what's in the channel, say so plainly. Don't paper over gaps with confident-sounding paraphrase.

## Timestamps and time windows

- Slack timestamps come back as Unix floats (`1779204458.747489`). Pass the same format when bracketing reads with `oldest` / `latest`.
- "Recent" is fuzzy. When the user asks about "last week" or "Tuesday's discussion," resolve the actual date first (via a `date` shell call if needed) — projects move fast, off-by-one days matter.
- Don't trust your own memory for what was discussed recently. The channel is the source of truth — always re-read for "recent" questions.

## When the auth token is dead

If a call returns `invalid_auth_token`:

1. Tell the user — don't keep retrying. Tokens don't always refresh automatically mid-session.
2. Suggest: run `/mcp` to reconnect the Slack integration, then **disconnect and reconnect** (a single reconnect sometimes isn't enough — the cached token may need a full disconnect cycle). Then retry the simplest possible call (e.g., `slack_read_user_profile` with no args) to confirm before resuming the actual work.
3. If a session restart is needed, say so — better than silently looping on a dead token.

## Mapping the project's `AGENTS.md` declarations onto Slack

The project's `AGENTS.md` should give you, at minimum:

- A **channel name** and **channel ID** for the project's main channel.
- Optionally, the workspace URL and any sibling channels worth knowing about (e.g., a leadership channel that talks about the project).
- Optionally, a Canvas ID if the channel has a pinned doc that the project treats as a planning artifact.

If `AGENTS.md` only names the channel by name, resolve to an ID once via `slack_search_channels` and use the ID for subsequent calls.
