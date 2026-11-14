---
name: org-memory
description: Push session learnings to the org's Notion learnings database when a task wraps up or the user asks, pull relevant past learnings at session start, and migrate the user's existing local Claude memory into Notion when asked ("migrate memory", "transfer memory to Notion").
---

# Org Memory

Turns sessions into shared organisational memory in Notion. Three modes: **Pull** (start), **Push** (end), **Migrate** (one-time move of existing local memory).
Requires the Notion connector / MCP, signed into the org workspace.

## Config

- Database: **Org Learnings**
- Data source: `collection://4189506d-5fe0-8119-bd49-000bcef96db6`

Always write to this data source. Never create a second database.

## Author identity

The Notion account is shared, so record the person explicitly. Look for a line `Org-memory author: Name` in the user's CLAUDE.md or preferences. If absent, ask once before pushing. Set Author name on every new page and in every Log entry.

## Schema

| Property | Type |
|---|---|
| Title | title — one-line claim, e.g. "Razorpay webhooks retry 3x, make handlers idempotent" |
| Type | select: Decision, Gotcha, How-to, Domain/Customer, Failure |
| Project | select (add new options as needed) |
| Status | select: Draft, Verified, Superseded |
| Author name | text |
| Date | date |
| Superseded by / Supersedes | relation (self) |

Page body: **Context** (1–2 lines) → **Learning** → **Why / evidence** → **Log** (dated entries appended on updates).

## Pull — at session start

When the task names a project or topic, search the database (Project + keywords). Fetch at most 3–5 relevant non-Superseded entries and use them silently as context. Mention one only if it changes the approach.

## Push — when a task wraps up, or when the user asks ("push learnings")

1. **Extract** 3–5 candidate pointers from the session.
   - Keep: decisions + rationale, non-obvious gotchas, what failed, domain/customer facts, reusable procedures.
   - Skip: chit-chat, transient task state, anything already in code/docs.
2. **Redact**: drop credentials, personal data, client-confidential details, raw numbers tied to named clients.
3. **Review**: show the pointers as a numbered list. Ask: "Push to Notion? Reply with numbers to keep, edits, or 'skip'." Push nothing without confirmation.
4. **Match** each confirmed pointer: search the database by Project + key terms, then fetch top hits.
   - Same learning exists → append a dated Log entry; don't duplicate.
   - Contradicts an existing entry → show both, ask the user. If the new one wins: set old Status = Superseded and link via Superseded by.
   - No match → create a new page, Status = Draft, Author name = current user.
5. **Report**: one line per pointer — created / updated / superseded, with page link.

## Migrate — when the user asks ("migrate memory", "transfer my memory to Notion"), or right after first install

Moves what this Claude already remembers into Org Learnings. Same pipeline as Push, applied to stored memory instead of the current session.

1. **Inventory** the memory you can read. Read only; never edit or delete local memory.
   - **Claude Code:** `~/.claude/CLAUDE.md`; every `~/.claude/projects/*/memory/*.md` (each file is one memory; `MEMORY.md` is only an index); the `CLAUDE.md` / `CLAUDE.local.md` of the current project. Use the folder name under `projects/` to infer the Project.
   - **Claude app:** the memory and preferences visible in your context. List what you remember; don't invent.
   - Tell the user how many sources and items you found before going further.
2. **Filter** to org-worthy learnings: decisions + rationale, gotchas, failures, domain/customer facts, reusable procedures.
   - Skip personal preferences about how Claude should talk or behave (tone, language, formatting), the user's private details, transient task state, and anything that only restates code or docs.
   - Personal working rules that would help teammates (e.g. "never do X in this repo, because Y") are candidates; mark them so the user can decide.
3. **Redact** exactly as in Push.
4. **Review in batches** of up to 10, numbered, each with its proposed Title, Type and Project. Ask: "Push to Notion? Reply with numbers to keep, edits, 'skip', or 'stop'." Push nothing without confirmation. On 'stop', end the migration and report what was done.
5. **Match** each confirmed item exactly as in Push (dedupe, append Log, supersede only with the user's say-so). Several local memories saying the same thing become one page.
6. **Report** per batch: created / updated / superseded, with page links. At the end: totals, plus the items skipped.

## Rules

- Never write outside the Org Learnings database.
- Never delete; supersede.
- One claim per page. Title must stand alone without the body.
- If the user ignores or declines the push prompt, drop it; don't re-ask in the same session.
- Migrate never modifies or deletes the local memory files it reads.
