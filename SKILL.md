---
name: pvc-claude-md
description: Audit, optimize, and update CLAUDE.md files with session learnings. Pro Vibe Coding's all-in-one CLAUDE.md tool. Use when the user says "audit my CLAUDE.md", "improve CLAUDE.md", "update CLAUDE.md", "review claude.md", "is my CLAUDE.md good", "check CLAUDE.md", "PVC CLAUDE.md", "rotate lab notes", "compact lab notes", "archive old lab notes", or wants to capture session learnings into project memory. Combines a 4-pillar quality framework, audit-and-improve workflow, session-learning capture, and lab notes rotation (latest 10 in CLAUDE.md, older archived to LABS.md) into one tool. Trigger this skill whenever a user mentions CLAUDE.md maintenance, project memory, or wants to keep their CLAUDE.md current. Even if they don't explicitly say "CLAUDE.md", trigger this when they reference project context, project memory, system prompts for their codebase, or "what Claude knows about my project".
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
license: Apache-2.0
---

<!-- SPDX-License-Identifier: Apache-2.0 -->

# PVC CLAUDE.md

**Version:** v0.0.13

Audit, optimize, and update CLAUDE.md files using the 4-pillar quality framework. Combines audit, improvement, session-learning capture, and lab notes rotation into one workflow.

See `README.md` for what this skill does, why it exists, install instructions, and the changelog.

## When to Use

Trigger when the user mentions:

- Auditing, reviewing, or improving CLAUDE.md
- Updating CLAUDE.md with session learnings
- Project memory or context optimization
- "Is my CLAUDE.md good?" or similar
- Lab notes rotation ("rotate lab notes", "compact lab notes", "archive old lab notes")

## The 4 Pillars

Score each pillar 0-3 (0 = missing, 1 = weak, 2 = adequate, 3 = strong).

1. **Knowledge Compression.** Project name, purpose, tech stack, key files, commands, audience. **Claude 5 era rule (v0.0.6):** a strong score means compressed NON-OBVIOUS knowledge: gotchas, constraints, decisions, and patterns Claude cannot infer from the repo itself. Facts Claude discovers by exploring (file listings, obvious folder structure, standard commands a manifest already declares) count against the score, not toward it. When auditing, list any discoverable-fact lines as removal candidates. (Anthropic context-engineering guidance for Claude 5 generation models, 2026-07.)
2. **User Preferences.** Style rules, conventions, voice, banned vocabulary.
3. **Capability Declarations.** MCP servers, skills, scripts, integrations.
4. **Lab Notes.** Running log of what worked, what didn't, gotchas. Format: `### YYYY-MM-DD: short title` then 1-2 sentences. **In v0.0.2+, CLAUDE.md keeps the latest 10 entries; older ones rotate to `LABS.md`.**

## Lab Notes Rotation (v0.0.2+)

CLAUDE.md keeps the **latest 10 lab notes**. Older entries archive to a sibling `LABS.md` in the project root.

**Why this matters:** CLAUDE.md is loaded into every Claude session's context. Too many lab notes = too many tokens spent on stale lessons. Rolling 10 keeps per-session cost low while preserving all history in `LABS.md`.

### The two files

**`CLAUDE.md`** (Lab notes section structure):
```markdown
## Lab notes

Showing the latest 10. **N older entries archived in [`LABS.md`](LABS.md).** New lab notes get added here; once the count exceeds 10, the oldest rotates to `LABS.md` (handled by `/pvc-claude-md` skill v0.0.2+). Grep both files when looking up historical findings.

### YYYY-MM-DD: oldest entry title
body...

[9 more entries, newest at bottom]
```

**`LABS.md`** (full structure):
```markdown
# Archived Lab Notes

Older lab notes rotated out of `CLAUDE.md`. `CLAUDE.md` keeps the latest 10 entries; everything older is archived here, oldest at top, newest at bottom. This file is append-only: entries are never edited or reordered once archived. Grep both files when looking up past findings.

### YYYY-MM-DD: oldest archived entry title
body...

[all older entries, oldest at top, newest at bottom]
```

### When adding a new lab note (extends Step 3 of the workflow)

1. Insert the new entry at the BOTTOM of the `## Lab notes` section in CLAUDE.md (chronological: oldest first / newest last)
2. Count entries in the section (grep `^### [0-9]{4}-` in the Lab notes section)
3. If count > 10, MOVE the topmost (oldest) entry from CLAUDE.md to the BOTTOM of `LABS.md`
4. Update the count in the section header: ``**N older entries archived in [`LABS.md`](LABS.md).**`` (increment N by 1)

If the count is exactly 10 after adding the new entry, no rotation needed.

### Manual rotation command

If user says "rotate lab notes", "compact lab notes", or "archive old lab notes" without proposing new content, run the rotation WITHOUT adding a new entry. Move any entries beyond the latest 10 to `LABS.md` and update the header count. Useful after manual editing or to fix drift.

### Searching for a past lab note

Grep BOTH `CLAUDE.md` AND `LABS.md`, because the entry might be archived:

```bash
grep -n "pattern" CLAUDE.md LABS.md
```

### First-time setup

If `LABS.md` doesn't exist:
1. Count lab notes in CLAUDE.md
2. If count > 10: create `LABS.md` with the archive header from the "The two files" section, move entries 1 through (count - 10) to `LABS.md` (oldest first), update CLAUDE.md's Lab notes section header
3. If count <= 10: do nothing; the rotation kicks in only after the 11th entry is added

## Project Structure Sync (v0.0.4+)

Some projects keep an auto-managed structure block in CLAUDE.md between these markers:

```markdown
<!-- STRUCTURE:BEGIN ... -->
[folder overview, usually a mermaid diagram plus a link to the project's diagrams file]
<!-- STRUCTURE:END -->
```

When the markers exist, every run of this skill refreshes the block. No markers means no structure sync; skip silently.

### How to sync

1. Find where the mapped folders live before listing anything. Read the block: if it names a home folder for the steps (for example `.pvc_content_pipeline/`), list inside that folder (`ls -a`). Otherwise list the project root (`ls -d */`). Skip infrastructure dotfolders like `.claude` and `.git`, but never skip a dotfolder the block itself names as the home
2. Compare that listing against the folder names inside the block (mermaid node labels and any folder lists must match the real folder names exactly)
3. In sync: leave the block untouched, report `Structure: in sync` in the audit
4. Drifted (folder added, renamed, or removed): regenerate the mermaid and folder notes inside the markers from the real listing, report `Structure: drifted (N changes)` with a diff. Node labels stay identical to the exact folder names
5. If the block links a diagrams file (for example `_diagrams/content_pipeline/content_pipeline.md`), check its node labels against the same listing and propose updating it in the same pass so CLAUDE.md and the diagram never disagree

Structure sync edits follow the same approval rule as everything else in this skill: show the diff, ask before writing.

This feeds Pillar 1 (Knowledge Compression) in one direction only. A structure block is a folder listing, so an in-sync block does not raise the score. A drifted block misleads Claude, so it caps the pillar at weak until regenerated.

## Workflow

### Step 1: Discovery

Find all CLAUDE.md and LABS.md files:

```bash
find . -name "CLAUDE.md" -o -name "LABS.md" -o -name "CLAUDE.local.md" 2>/dev/null | head -20
```

`CLAUDE.md` is shared with the team. `LABS.md` is the lab notes archive (also shared, but rarely loaded into context). `CLAUDE.local.md` is personal and gitignored.

### Step 2: Audit

For each CLAUDE.md, score the 4 pillars and output:

```markdown
## CLAUDE.md Audit Report
**File:** [path]
**Token estimate:** ~[N] tokens
**Lab notes:** [latest count] in CLAUDE.md, [archived count] in LABS.md
**Structure:** [in sync | drifted (N changes) | no STRUCTURE markers]

| Pillar | Score | Status |
|--------|-------|--------|
| Knowledge Compression | [0-3] | [Missing/Weak/Adequate/Strong] |
| User Preferences | [0-3] | [Missing/Weak/Adequate/Strong] |
| Capability Declarations | [0-3] | [Missing/Weak/Adequate/Strong] |
| Lab Notes | [0-3] | [Missing/Weak/Adequate/Strong] |

**Overall:** [N]/12

### Recommendations
[Specific additions for each missing or weak pillar]
```

### Step 2b: Sync Project Structure (if markers present)

If the CLAUDE.md contains a `STRUCTURE:BEGIN` marker, run the Project Structure Sync (section above) and fill the `**Structure:**` line of the audit report. If the block drifted, the regenerated block goes into Step 4's proposed updates like any other diff.

### Step 3: Capture Session Learnings (if applicable)

If the user is at session end or asks to "update CLAUDE.md with what we learned", reflect on what was missing earlier in the session: bash commands discovered, patterns followed, gotchas hit, workflows that worked or failed. Draft additions that would help future sessions.

Before drafting, strip private detail from every proposed line: API keys, tokens, passwords, connection strings, absolute machine paths (home folders, drive letters), and client or customer names. CLAUDE.md is committed and shared. When a lesson needs a private detail to make sense, propose it for `CLAUDE.local.md` instead and say so in the diff.

For each new lab note: follow the rotation steps above so CLAUDE.md stays at <=10 entries.

### Step 4: Propose Targeted Updates

Show diffs before editing. For each addition:

````markdown
### Update: ./CLAUDE.md
**Why:** [one-line reason]

```diff
+ [the addition, kept brief]
```
````

If the rotation will run (entries about to exceed 10), also show:

```markdown
### Rotation: oldest entry moves to LABS.md
**Entry being archived:** YYYY-MM-DD: short title
```

Keep additions minimal. Avoid verbose explanations, things obvious from the code, one-off fixes unlikely to recur, or generic best practices.

### Step 5: Apply with Approval

Ask before writing. Only edit files the user approves. After applying, suggest the user keep a brief Lab Notes entry going forward.

## Lab Notes Meta-Prompt

If Pillar 4 is missing, propose adding to the bottom of CLAUDE.md:

```markdown
## Lab notes

<!-- When a mistake is made, a dead end is hit, or a non-obvious approach pays off,
     add an entry here with what happened and what to do (or not do) next time.
     Format: `### YYYY-MM-DD: short title` then 1-2 sentences.
     Once this section exceeds 10 entries, /pvc-claude-md will rotate the oldest
     entry to LABS.md. Both files live at the project root. -->
```

## Style

Follow the existing style rules in the target CLAUDE.md. Read them first if any exist. Default to hedged honest voice, simple words, short sentences. Avoid absolutes. No em dashes in any new content.

## Anti-Patterns

- Do NOT exceed 10 entries in CLAUDE.md's Lab notes section. Always rotate.
- Do NOT edit `LABS.md` ordering retroactively. New archives always append at the bottom (newest archived at the end).
- Do NOT delete entries from `LABS.md` unless explicitly asked. The archive is append-only.
- Do NOT skip the rotation header update when archiving. The `**N older entries...**` count must stay accurate.
- Do NOT modify entries that are already in `LABS.md` for stylistic reasons (e.g. em-dash fixes). Archive entries are frozen historical snapshots; preserve fidelity.
- Do NOT add or reward facts Claude discovers by exploring the repo (file listings, obvious structure, commands a manifest already declares). CLAUDE.md tokens belong to gotchas and non-obvious patterns; flag discoverable-fact lines as removal candidates in every audit. The one exception is a STRUCTURE block between markers: keep it accurate and report drift, but never let it raise the score; a drifted block caps the pillar at weak.
- Do NOT copy secrets, absolute machine paths, or client names into CLAUDE.md. Strip them, or route the lesson to `CLAUDE.local.md`.
