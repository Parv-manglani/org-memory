# org-memory

A Claude skill that turns sessions into shared organisational memory in a Notion database.

- **Pull:** at the start of a task, Claude checks Org Learnings for relevant past entries.
- **Push:** at the end of a task, say **"push learnings"**. Claude shows candidate learnings, and you pick which go to Notion.
- **Migrate:** say **"migrate memory"**. Claude moves what it already remembers locally into Notion, in batches you approve.

The learnings live in Notion. This repo holds only the skill: [`org-memory/SKILL.md`](org-memory/SKILL.md).

---

## Quick start: give this repo to your Claude

Paste this to Claude Code (or the Claude app):

> Set up the org-memory skill from https://github.com/Parv-manglani/org-memory

Claude follows the section below.

---

## For Claude: setup steps

If a user gave you this repo and asked you to set it up, do these in order. Tell the user what you are doing at each step.

### 1. Install the skill

- **Claude Code:** install it personally, for all projects:
  ```bash
  tmp=$(mktemp -d) && git clone --depth 1 https://github.com/Parv-manglani/org-memory "$tmp/org-memory"
  mkdir -p ~/.claude/skills
  ```
  If `~/.claude/skills/org-memory` already exists, show the user the diff and ask before replacing it. Then:
  ```bash
  cp -R "$tmp/org-memory/org-memory" ~/.claude/skills/
  ```
  Result: `~/.claude/skills/org-memory/SKILL.md`. The skill loads automatically in new sessions. For this session, read that file and follow it directly.
- **Claude app / claude.ai:** get the skill folder (`org-memory/` from this repo, or the zip at https://github.com/Parv-manglani/org-memory/releases/latest/download/org-memory.zip), package it as a skill file, and present it so the user can save it to their skills in one click. If no save option appears, tell the user to download that zip and upload it under Settings → Capabilities → Skills → Upload skill.

### 2. Check the Notion connector

Try a Notion search for the **Org Learnings** database. If Notion tools are missing or the database is not found, give the user the steps below and wait until they confirm it is connected.

The user needs access to the org workspace first: accept the Notion invite sent to their email, and sign in with that same email. When Notion asks which workspace to connect, they must pick the org workspace, not a personal one. Writing needs "Can edit" access or higher on the Org Learnings database.

- **Claude app:** Settings → Connectors → Notion → Connect, signed in to the org workspace.
- **Claude Code:**
  ```bash
  claude mcp add --transport http notion https://mcp.notion.com/mcp
  ```
  Then run `/mcp` inside Claude Code and authenticate Notion with the org workspace account.

### 3. Set the author name

Look for a line `Org-memory author: Name` in `~/.claude/CLAUDE.md` (Claude Code) or the user's preferences (Claude app). If it is missing, ask the user for their name.

- **Claude Code:** append `Org-memory author: <Name>` to `~/.claude/CLAUDE.md`.
- **Claude app:** ask the user to add that line under Settings → Profile → personal preferences.

### 4. Migrate existing memory

Ask the user whether to move their existing Claude memory into Notion now. If yes, run the **Migrate** mode in `SKILL.md`. Nothing goes to Notion without their confirmation, and local memory is never changed.

---

## Manual install

- **Claude app:** download `org-memory.zip` from [Releases](https://github.com/Parv-manglani/org-memory/releases/latest) → Settings → Capabilities → Skills → Upload skill.
- **Claude Code (personal, all projects):** copy the `org-memory/` folder to `~/.claude/skills/`, so you get `~/.claude/skills/org-memory/SKILL.md`.
- **Claude Code (shared via a project repo):** commit it as `.claude/skills/org-memory/SKILL.md` in that repo. Everyone who clones it gets the skill.

Then connect Notion and set your author name, as in steps 2 and 3 above.
