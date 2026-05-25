# Capture mode: notes

Reference document for the `planning` skill when the user is dumping a raw transcript or brain dump. Take the input and write it, verbatim, into the current project's `notes/` folder.

Notes captured here are **raw artifacts**, not plans. Other modes (status, sprint, reconciliation) may read them later, but capture mode never edits a note after creating it and never rewrites the content on the way in.

## When to invoke capture mode

Fire on either of these signals:

- An explicit ask: "here's my brain dump", "voice transcript", "dump this into notes", "capture this", "save this transcript", "drop this into notes", and similar.
- An implicit signal: the user pastes a long, unstructured, first-person block that looks like a spoken transcript — rambling sentences, conversational filler ("um", "so", "yeah"), repeated ideas, no headings or bullet structure. If it reads like the user talking to themselves, treat it as a transcript even if they didn't name it.

If it's ambiguous (e.g., a short paragraph that could be a question or a note), ask before writing.

## Behavior

1. **Determine the current date** with `date +%Y-%m-%d` rather than guessing.
2. **Pick a short slug** (2–4 words, kebab-case) that summarizes the transcript's topic. Slug style is loose — whatever fits the content. Only the date prefix is required.
3. **Construct the path**: `notes/<YYYY-MM-DD>-<slug>.md` at the project root. If `notes/` does not exist, create it. If a file with that exact name already exists, append a short suffix (e.g., `-2`) rather than overwriting.
4. **Write the transcript verbatim.** No cleanup, no rewriting, no reformatting, no summarization, no headers added. Preserve the user's words exactly as given.
5. **Confirm the file path** back to the user in one sentence, so they can find it later.

## What capture mode does *not* do

- It does not edit, summarize, restructure, or "clean up" the transcript. The whole point is that the raw form is the artifact.
- It does not extract action items, decisions, or todos. Those belong to status or sprint mode.
- It does not modify or delete previously written notes.
- It does not enforce a filename convention beyond the leading `YYYY-MM-DD-` date prefix; the slug is the project's call.
