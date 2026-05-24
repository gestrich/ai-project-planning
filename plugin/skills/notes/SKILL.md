---
name: notes
description: Capture raw voice transcripts and brain dumps into a project's notes/ folder verbatim. Use this skill whenever the user says things like "here's my brain dump", "voice transcript", "dump notes", "capture this", "drop this into notes", or pastes a visibly long, unstructured, first-person block of text that reads like a spoken transcript (rambling sentences, repeated thoughts, no headings, conversational filler). Trigger even when the user does not explicitly say "notes" — if the input looks like a raw transcript, this skill is the right home for it.
---

# notes

The simplest entry point in the project-planning plugin: take a raw transcript or brain dump from the user and write it, verbatim, into the current project's `notes/` folder.

Notes captured here are **raw artifacts**, not plans. Downstream skills (`plan`, `sprint`) may read them later, but this skill never edits a note after creating it and never rewrites the content on the way in.

## When to trigger

Fire on either of these signals:

- An explicit ask: "here's my brain dump", "voice transcript", "dump this into notes", "capture this", "save this transcript", "drop this into notes", and similar.
- An implicit signal: the user pastes a long, unstructured, first-person block that looks like a spoken transcript — rambling sentences, conversational filler ("um", "so", "yeah"), repeated ideas, no headings or bullet structure. If it reads like Bill talking to himself, treat it as a transcript even if he didn't name it.

If it's ambiguous (e.g., a short paragraph that could be a question or a note), ask before writing.

## Behavior

1. **Determine the current date** with `date +%Y-%m-%d` rather than guessing.
2. **Pick a short slug** (2–4 words, kebab-case) that summarizes the transcript's topic. Slug style is loose — whatever fits the content. Only the date prefix is required.
3. **Construct the path**: `notes/<YYYY-MM-DD>-<slug>.md` at the project root. If `notes/` does not exist, create it. If a file with that exact name already exists, append a short suffix (e.g., `-2`) rather than overwriting.
4. **Write the transcript verbatim.** No cleanup, no rewriting, no reformatting, no summarization, no headers added. Preserve the user's words exactly as given.
5. **Confirm the file path** back to the user in one sentence, so they can find it later.

## What this skill does *not* do

- It does not edit, summarize, restructure, or "clean up" the transcript. The whole point is that the raw form is the artifact.
- It does not extract action items, decisions, or todos. Those belong to downstream skills.
- It does not modify or delete previously written notes.
- It does not enforce a filename convention beyond the leading `YYYY-MM-DD-` date prefix; the slug is the project's call.
