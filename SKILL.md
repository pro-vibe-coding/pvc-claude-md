---
name: pvc-claude-md
description: "Audits a project's CLAUDE.md on four pillars and proposes small diffs that it writes only after approval. Also captures session learnings into the file as dated lab notes and rotates lab notes past the latest 10 into LABS.md. Use when asked to 'audit my CLAUDE.md', 'review claude.md', 'check CLAUDE.md', 'is my CLAUDE.md good', 'improve CLAUDE.md', 'update CLAUDE.md with what we learned', 'my project instructions file is a mess', 'rotate lab notes', 'compact lab notes' or 'archive old lab notes'. Not for closing a whole work set; that is pvc-end-set. Not for a first CLAUDE.md, which /init writes, or for auto memory."
metadata:
  author: pvc
  version: "0.0.20"
license: Apache-2.0
---

<!-- SPDX-License-Identifier: Apache-2.0 -->

# PVC CLAUDE.md

## The 4 Pillars

Score each pillar 0-3 (0 = missing, 1 = weak, 2 = adequate, 3 = strong). A line earns its place only when Claude could not find it by reading the repo, or would not go and look (Claude Code docs, Best practices: Tell Claude what it can't see).

1. **Knowledge Compression.** What Claude cannot work out from the repo: how to confirm work (the exact test, typecheck and lint commands, and how to run one test file), gotchas and places where the code misleads (an old pattern new code must not copy, a folder that looks unused but is live), decisions and constraints that are not in the repo, and corrections the user has had to give twice. Facts Claude finds by exploring (file listings, obvious folder structure, dependency lists, architecture overviews) count against the score: when auditing, list each such line as a removal candidate.
2. **User Preferences.** Style rules, conventions, voice, banned vocabulary.
3. **Capability Declarations.** What Claude cannot see about its tools: which skill, MCP server or script to prefer for a job and when, how to run the project's own scripts, and the outside services the work depends on. Claude already sees the name and description of each skill it can invoke and the tool names of connected MCP servers, so a plain list of them counts against the score: flag it as a removal candidate.
4. **Lab Notes.** Running log of what worked, what didn't, gotchas. Format: `### YYYY-MM-DD: short title` then 1-2 sentences. CLAUDE.md keeps the latest 10 entries; older ones rotate to `LABS.md`.

## Lab Notes Rotation

CLAUDE.md keeps the **latest 10 lab notes**. Older entries archive to a `LABS.md` in the same folder as that CLAUDE.md (the project root for the root CLAUDE.md). `LABS.md` is append-only: never reorder an archived entry or edit it, not even to fix style such as an em dash, and delete one only when the user asks.

### The two files

**`CLAUDE.md`** (Lab notes section structure):
```markdown
## Lab notes

Showing the latest 10. **N older entries archived in [`LABS.md`](LABS.md).** New lab notes get added here; once the count exceeds 10, the oldest rotates to `LABS.md` (handled by `/pvc-claude-md`). Grep both files when looking up historical findings.

When a mistake is made, a dead end is hit, or a non-obvious approach pays off, add an entry at the bottom of this section: `### YYYY-MM-DD: short title`, then 1-2 sentences on what happened and what to do (or not do) next time. Leave out secrets, machine paths and client names.

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
2. Count entries in the section (`grep -Ec '^### [0-9]{4}-'` or the Grep tool, in the Lab notes section)
3. If count > 10, MOVE the oldest (count - 10) entries, oldest first, from CLAUDE.md to the BOTTOM of `LABS.md`
4. When step 3 moved entries, raise the count in the section header by the number moved: ``**N older entries archived in [`LABS.md`](LABS.md).**``

If the count is exactly 10 after adding the new entry, no rotation needed.

### Manual rotation command

If user says "rotate lab notes", "compact lab notes", or "archive old lab notes" without proposing new content, run the rotation WITHOUT adding a new entry and without an audit. Move any entries beyond the latest 10 to `LABS.md` and update the header count. Show the move as a diff first, in the form of Step 4, and write after approval. Useful after manual editing or to fix drift.

### First-time setup

If `LABS.md` doesn't exist:
1. Count lab notes in CLAUDE.md
2. If count > 10: create `LABS.md` with the archive header from the "The two files" section, move entries 1 through (count - 10) to `LABS.md` (oldest first), update CLAUDE.md's Lab notes section header
3. If count <= 10: do nothing; the rotation kicks in only after the 11th entry is added

## Workflow

Pick the route from the request before any step:

1. **Audit** (audit, review, check or improve a CLAUDE.md): run Steps 1 to 5.
2. **Capture** ("update CLAUDE.md with what we learned", or a session end): find the CLAUDE.md the lesson belongs to, the one with the Lab notes section or else the one at the project root, then run Steps 3 to 5. Print no scores and no audit report.
3. **Rotation** ("rotate lab notes", "compact lab notes", "archive old lab notes"): run only the manual rotation above.

The shell commands below are for Bash. Where Claude has the Glob and Grep tools (the default on Windows), use those instead, with the same file names and the same skipped folders. Glob returns at most 100 files, so with Glob, search one file name at a time, drop paths under `node_modules`, `.git` and `.claude/worktrees`, and narrow the pattern when the result says it was cut.

### Step 1: Discovery

Find all CLAUDE.md, LABS.md, CLAUDE.local.md and AGENTS.md files, and the rule files under `.claude/rules/`:

```bash
find . \( -name node_modules -o -name .git -o -path "*/.claude/worktrees" \) -prune -o -type f \( -name "CLAUDE.md" -o -name "LABS.md" -o -name "CLAUDE.local.md" -o -name "AGENTS.md" -o -path "*/.claude/rules/*.md" \) -print 2>/dev/null
```

`CLAUDE.md` is shared with the team. `LABS.md` is the lab notes archive (also shared, but rarely loaded into context). `CLAUDE.local.md` is personal, but git ignores it only when an ignore rule lists it, such as a line in the project's `.gitignore`. `AGENTS.md` is the instruction file other coding agents read; Step 3 says when it matters.

A rule file under `.claude/rules/` with no `paths` field in its frontmatter loads at launch with the same priority as `.claude/CLAUDE.md`. Read the rule files beside each CLAUDE.md before scoring: a style rule or a gotcha kept there counts for its pillar, and the audit proposes no addition that a rule already holds. A checkout under `.claude/worktrees/` is a copy of the project for another session: skip its files, say in the report which worktrees were skipped, and propose no edit inside them.

With `AGENTS.md` and no `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` in the folder or above it, Claude Code (v2.1.277 or later) already reads `AGENTS.md` as the project instructions. Say so, and offer to audit `AGENTS.md` on the same four pillars. Offer a new `CLAUDE.md` only with the import of `AGENTS.md` as its first line, since any other first `CLAUDE.md` stops `AGENTS.md` from loading; Step 3 gives the import line. With both, and no such import in the `CLAUDE.md`, note that `AGENTS.md` does not load by default, and offer the import line as the first line of the `CLAUDE.md` next to `AGENTS.md`.

### Step 2: Audit

Read each CLAUDE.md from disk with the Read tool, since the copy loaded into the session has its HTML comments stripped. For each CLAUDE.md, score the 4 pillars and output:

```markdown
## CLAUDE.md Audit Report
**File:** [path]
**Token estimate:** ~[N] tokens, imports included; [L] lines (target under 200)
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

### Removal candidates
[Each line Claude can find in the repo or already sees, quoted, with its reason]
```

### Step 2b: Sync Project Structure (if markers present)

If a CLAUDE.md holds a `STRUCTURE:BEGIN` marker, read [references/structure-sync.md](references/structure-sync.md) and follow it as part of that file's audit; without the marker, skip this step.

### Step 3: Capture Session Learnings (if applicable)

If the user is at session end or asks to "update CLAUDE.md with what we learned", reflect on what was missing earlier in the session: bash commands discovered, patterns followed, gotchas hit, workflows that worked or failed. Draft additions that would help future sessions.

Before drafting, strip private detail from every proposed line. Secrets (API keys, tokens, passwords, connection strings) never go into any instruction file, `CLAUDE.local.md` included, since Claude Code loads these files into every session and into subagents: name the environment variable or the secret store that holds the value instead. Absolute machine paths (home folders, drive letters) and client or customer names stay out of CLAUDE.md, which is committed and shared. When a lesson needs one of those to make sense, propose it for `CLAUDE.local.md` instead and say so in the diff. Before the first write to `CLAUDE.local.md`, run `git check-ignore -q CLAUDE.local.md`. Exit 0 means it is ignored and untracked. Any other result: add the line `CLAUDE.local.md` to the project's `.gitignore` in the same diff, and when `git ls-files CLAUDE.local.md` prints the path, the file is tracked already, so keep the private detail out of it and tell the user.

By default, Claude Code reads `AGENTS.md` as the project instructions only while no `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` sits in the working folder or above it, so in a project that relies on `AGENTS.md`, the first such file stops that. Tell the user before that first write, and keep `AGENTS.md` loaded. Put a new `CLAUDE.md` next to `AGENTS.md` and make its first line the import: the at sign with `AGENTS.md` right after it, no space. For a `CLAUDE.local.md` alone, point to the `/config` setting **Project instructions** with the value `claude-md-and-agents-md`.

For each new lab note: follow the rotation steps above so CLAUDE.md stays at <=10 entries.

### Step 4: Propose Targeted Updates

Show diffs before editing. For each addition or removal:

````markdown
### Update: ./CLAUDE.md
**Why:** [one-line reason]

```diff
+ [an addition, kept brief]
- [a removal candidate, quoted as it stands]
```
````

If the rotation will run (entries about to exceed 10), also show this, with one **Entry being archived** line for each entry that moves:

```markdown
### Rotation: oldest entry moves to LABS.md
**Entry being archived:** YYYY-MM-DD: short title
```

Keep additions minimal. Avoid verbose explanations, things obvious from the code, one-off fixes unlikely to recur, or generic best practices. The private-detail rule of Step 3 holds for every addition, also those that come from the audit.

### Step 5: Apply with Approval

Ask before writing. Only edit files the user approves. After applying, suggest the user keep a brief Lab Notes entry going forward.

## Lab Notes Meta-Prompt

If Pillar 4 is missing, propose adding to the bottom of CLAUDE.md:

```markdown
## Lab notes

When a mistake is made, a dead end is hit, or a non-obvious approach pays off, add an entry at the bottom of this section: `### YYYY-MM-DD: short title`, then 1-2 sentences on what happened and what to do (or not do) next time. Leave out secrets, machine paths and client names.

<!-- When this section holds more than 10 entries, /pvc-claude-md moves the oldest to LABS.md in this folder. -->
```

Claude Code strips block-level HTML comments from a CLAUDE.md when it loads the file into Claude's context, so an instruction inside one does not reach Claude through the loaded file. Keep a comment for notes to people only, and where an audit finds an instruction inside a comment (such as an older form of this prompt), propose it as visible text. Leave the STRUCTURE markers as they are.

## Style

Follow the existing style rules in the target CLAUDE.md. Read them first if any exist. For a file with no style rules, default to plain, concrete instructions a reader can check, simple words, short sentences, and no em dashes in new content.
