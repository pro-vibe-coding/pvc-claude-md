# pvc-claude-md

<p align="center"><img src="assets/pvc-claude-md-banner.jpg" alt="pvc-claude-md"></p>

![version](https://img.shields.io/badge/version-v0.0.20-blue) ![type](https://img.shields.io/badge/Claude%20Code-skill-7C5CFF) ![license](https://img.shields.io/badge/license-Apache%202.0-green)

> Audit, optimize, update, and rotate CLAUDE.md files. Pro Vibe Coding's all-in-one CLAUDE.md tool.

`pvc-claude-md` (display name: **PVC CLAUDE.md**) is a Claude Code skill that keeps your `CLAUDE.md` honest. It scores the file against a 4-pillar quality framework, proposes concrete diffs for whatever is weak, captures what a session learned, and rotates stale lab notes out to a sibling `LABS.md` so per-session context stays cheap. Made by Pro Vibe Coding for anyone who runs long Claude Code sessions and watches their CLAUDE.md drift over time.

## Table of contents

- [Why this exists](#why-this-exists)
- [What it does](#what-it-does)
- [The 4 pillars](#the-4-pillars)
- [Install](#install)
- [Usage](#usage)
- [Lab notes rotation](#lab-notes-rotation)
- [How it compares](#how-it-compares)
- [Output rubric](#output-rubric)
- [Known limitations](#known-limitations)
- [Changelog](#changelog)
- [License](#license)

## Why this exists

Most CLAUDE.md files drift. Three problems compound over a project's life:

1. **Stale context.** Claude follows guidance that no longer matches the code.
2. **Missing capability notes.** Claude sees which skills and MCP tools you have, but not which one to prefer for a job or how your project's scripts run, so it reinvents work you already automated.
3. **No memory of past failures.** The same dead ends get hit again because nothing recorded them.

This skill addresses all three in one pass instead of three separate habits you have to remember.

## What it does

- **Audits** an existing `CLAUDE.md` against 4 pillars and returns a scored report (0 to 12).
- **Suggests targeted diffs** for whatever pillar is missing or weak, kept minimal.
- **Captures session learnings** as dated lab notes so the next session inherits them.
- **Rotates lab notes** so `CLAUDE.md` holds the latest 10 while older history archives to `LABS.md` (v0.0.2+).
- **Syncs the project structure block** when `CLAUDE.md` carries `STRUCTURE:BEGIN`/`STRUCTURE:END` markers: every audit compares the block (mermaid folder diagram, folder notes) against the real folder listing and regenerates it on drift (v0.0.4+).

## The 4 pillars

Each pillar scores 0 to 3: 0 = missing, 1 = weak, 2 = adequate, 3 = strong.

| Pillar | Covers |
|---|---|
| Knowledge Compression | What Claude cannot work out from the repo: the exact test, typecheck and lint commands, gotchas and misleading code, decisions that are not in the repo. Folder listings and other facts Claude finds by exploring count against it |
| User Preferences | Style rules, conventions, voice, banned vocabulary |
| Capability Declarations | Which skill, MCP server or script to prefer for a job and when, how to run your scripts, the outside services the work depends on. A plain list of skills or MCP servers counts against it, since Claude already sees them |
| Lab Notes | Running log of what worked, what did not, gotchas |

The scoring follows the Claude Code docs. Best practices (Tell Claude what it can't see, and Keep CLAUDE.md short) says to write down only what Claude cannot find by reading the repo, or would otherwise spend time finding again every session. The docs page on CLAUDE.md and memory says `/doctor` cuts content Claude can derive from the codebase and keeps pitfalls, rationale and conventions.

## Install

This is a Claude Code skill. Pick the path that matches where you want it available.

**From the GitHub release zip:** it unpacks to a folder named `pvc-claude-md-<version>` (for example `pvc-claude-md-0.0.11`). Rename that folder to `pvc-claude-md` before you place it. The `name` field in SKILL.md sets the command `/pvc-claude-md`, the folder name also invokes the skill, and the Agent Skills standard requires a folder name that matches `name`. A `git clone` gives you the right folder name already.

### Project-local (one repo)

Drop the folder into the repo's `.claude/skills/`:

```
your-project/
├── CLAUDE.md         (latest 10 lab notes)
├── LABS.md           (archived lab notes, auto-created on first rotation)
└── .claude/
    └── skills/
        └── pvc-claude-md/
            ├── SKILL.md
            ├── README.md
            ├── references/      (structure sync steps)
            ├── assets/          (README artwork)
            ├── REQUIREMENTS.md
            ├── LICENSE
            ├── NOTICE
            └── TRADEMARK.md
```

### Personal (all your projects)

Copy the `pvc-claude-md/` folder to `~/.claude/skills/` for user-global access across every project.

### Polyrepo / monorepo

Place it once at `.claude/skills/pvc-claude-md/` in the repository root. A session started in any folder of that repository loads it, because Claude Code looks for project skills in every parent folder up to the repository root. A copy in a subfolder's `.claude/skills/` loads once Claude reads or edits a file in that subfolder.

### As part of a plugin

If you bundle PVC skills as a Claude Code plugin, include `pvc-claude-md/` in the plugin's `skills/` directory. Claude Code auto-discovers skills in that folder when the plugin loads.

### Verify

Claude Code picks up a project or personal skill folder in the running session. Ask "What skills are available?" and confirm `pvc-claude-md` appears, or type `/pvc-claude-md`. If it does not show, run `/reload-skills`. For a plugin install, run `/reload-plugins` after the change.

## Usage

Trigger it with plain language. Any of these work:

- "audit my CLAUDE.md"
- "review my CLAUDE.md" or "check my CLAUDE.md"
- "improve my CLAUDE.md"
- "update CLAUDE.md with what we learned"
- "is my CLAUDE.md good?"
- "my project instructions file is a mess"
- "rotate lab notes", "compact lab notes", or "archive old lab notes"

It is not the skill for closing a whole work set (pvc-end-set does that), for writing a first CLAUDE.md (`/init` does that), or for auto memory.

### Example

```
You: audit my CLAUDE.md
```

Agent response:

```markdown
## CLAUDE.md Audit Report
**File:** ./CLAUDE.md
**Token estimate:** ~380 tokens, imports included; 36 lines (target under 200)
**Lab notes:** 0 in CLAUDE.md, no LABS.md exists
**Structure:** no STRUCTURE markers

| Pillar | Score | Status |
|--------|-------|--------|
| Knowledge Compression  | 0 | Missing |
| User Preferences        | 3 | Strong  |
| Capability Declarations | 0 | Missing |
| Lab Notes               | 0 | Missing |

**Overall:** 3/12

### Recommendations
- Add the exact test and lint commands, and how to run one test file (Knowledge Compression)
- Say which tool to use for database questions and when to run the seed script (Capability Declarations)
- Start a Lab notes section so future sessions inherit lessons
```

It then shows the proposed diffs and asks before writing. It only edits files you approve:

```diff
### Update: ./CLAUDE.md
+ ## Tools
+ - Database questions: use the postgres MCP server, which points at staging
+ - Run scripts/seed.sh before the integration tests
```

## Lab notes rotation

`CLAUDE.md` is loaded into every session's context, so a long lab-notes log quietly taxes every prompt. From v0.0.2, the skill keeps the latest 10 entries in `CLAUDE.md` and archives older ones to a sibling `LABS.md`.

- Adding the 11th entry moves the oldest to the bottom of `LABS.md`.
- The section header tracks the count: ``**N older entries archived in [`LABS.md`](LABS.md).**``
- When the 11th entry triggers a move, the skill reports it: `### Rotation: oldest entry moves to LABS.md`.
- `LABS.md` is append-only and frozen. The skill does not rewrite entries already there.
- Searching history? Grep both files: `grep -n "pattern" CLAUDE.md LABS.md`

## How it compares

Claude Code comes with tools of its own for project instructions and memory. Here is how this skill compares with them, so you can tell before installing whether you need it.

| | What it does with your CLAUDE.md | Scores it | Lab notes |
|---|---|---|---|
| pvc-claude-md | Audits an existing file, proposes small diffs, writes only what you approve, captures session learnings | Yes, 4 pillars, 0 to 12 | Dated entries, latest 10 in CLAUDE.md, older ones in LABS.md |
| `/init` | Writes a first CLAUDE.md, or suggests improvements to the one you have | No | No |
| `/doctor` | Proposes trims: cuts content Claude can derive from the code, keeps pitfalls, rationale and conventions | No | No |
| `/doctor prompt-audit` | Reports outdated or conflicting instructions with proposed edits, and changes nothing until you ask (Claude Code v2.1.283 or later) | No | No |
| Auto memory | Leaves it alone: Claude saves its own notes from your corrections in a separate memory folder | No | No |
| Editing by hand | Whatever you write | No | If you remember |

The 4-pillar score and the lab-notes rotation are what this skill adds. Those tools work beside it: `/doctor` and `/doctor prompt-audit` for trims and conflicting rules, auto memory for notes Claude keeps for itself.

## Output rubric

Every audit report the skill produces aims to pass on:

- **Accurate scoring.** Each pillar judged against what the file contains.
- **Minimal diffs.** Additions are brief, no padding, nothing already obvious from the code.
- **Voice match.** Follows the target project's own style rules. A file with none gets plain, concrete instructions a reader can check, simple words, short sentences, and no em dashes in new content.
- **Rotation integrity.** `CLAUDE.md` never exceeds 10 lab notes, and the archive count stays accurate.

## Known limitations

- Token estimates are approximate, not a tokenizer count.
- Rotation assumes the `### YYYY-MM-DD: title` lab-note format. Free-form notes may not rotate cleanly.
- It edits only files you approve, so a fully unattended run still pauses for confirmation.

## Changelog

- **v0.0.20 (2026-11-27).** The same skill text as v0.0.18. Version 0.0.19 added a look at the folders above the working folder, and a last check showed that it could take the user file `~/.claude/CLAUDE.md` for a project file. Those changes were taken back before the public release.
- **v0.0.18 (2026-11-27).** Fixed what a second check against the Claude Code docs found. A secret such as an API key, a token, a password or a connection string never goes into any instruction file, `CLAUDE.local.md` included: the skill names the environment variable or the secret store instead. Only machine paths and client names may move to `CLAUDE.local.md`. The audit report has a list of removal candidates, and Step 4 shows a removal as a minus line. Discovery also reads the rule files under `.claude/rules/`, so a rule kept there counts for its pillar and gets no duplicate in CLAUDE.md. A checkout under `.claude/worktrees/` is skipped and named in the report.
- **v0.0.17 (2026-11-27).** The workflow now picks one of three routes from the request. An audit request runs all five steps. A request to capture what a session learned goes to the capture steps, with no scores and no audit report. A request to rotate lab notes runs only the rotation, shows the move as a diff first and writes after approval. Before this version, a capture or a rotation request got a full audit first.
- **v0.0.16 (2026-11-25).** The lab-notes prompt the skill proposes is now a visible line under the heading, and a rotation keeps that line under the section header. The line also tells every session to leave secrets, machine paths and client names out of a lab note. Claude Code strips block-level HTML comments from a CLAUDE.md when it loads the file, so the old comment did not reach Claude through the loaded file. The audit now reads each CLAUDE.md from disk, offers the visible form where it finds an instruction inside a comment, and leaves the STRUCTURE markers as they are. The report gives the token estimate with imports included, and the line count against the target of under 200 lines. Knowledge Compression rewards what Claude cannot work out from the repo, such as the exact test, typecheck and lint commands, gotchas and decisions. The audit never lists a STRUCTURE block as a removal candidate, and the block never raises that score. Capability Declarations rewards which tool to prefer and when, and a plain list of skills or MCP servers counts against it, since Claude already sees them. A CLAUDE.md with no style rules gets plain, concrete instructions a reader can check, simple words, short sentences, and no em dashes in new content. The description no longer claims project context, project memory or system prompts, and it names its limits: pvc-end-set closes a work set and `/init` writes a first CLAUDE.md. Discovery also finds `AGENTS.md`. With `AGENTS.md` and no CLAUDE.md, the skill says that Claude Code already reads `AGENTS.md`, offers to audit it on the same four pillars, and offers a new CLAUDE.md only with the import of `AGENTS.md` as its first line. When both exist and the CLAUDE.md does not import `AGENTS.md`, it notes that `AGENTS.md` does not load by default and offers the import line. Before a first CLAUDE.md or CLAUDE.local.md in a project that relies on `AGENTS.md`, the skill warns, and offers an `@AGENTS.md` import line for a new CLAUDE.md or the Project instructions setting for a CLAUDE.local.md alone. The structure sync steps moved to `references/structure-sync.md`, read only when a CLAUDE.md holds a `STRUCTURE:BEGIN` marker, and SKILL.md drops the sections that repeat its description and rules. The steps say to use the Glob and Grep tools where Claude has them, the default on Windows, and to search one file name at a time with Glob, which returns at most 100 files. The lab-notes header no longer writes a skill version into your CLAUDE.md, and the version moved into the frontmatter. The README compares the skill with `/init`, `/doctor`, `/doctor prompt-audit` and auto memory, says a new skill folder shows up in the running session, and says the `name` field sets the command.
- **v0.0.15 (2026-11-24).** SKILL.md drops its `allowed-tools` line, which let the skill write files and run shell commands with no prompt. Claude Code now asks first, by the rules of your own settings. Before the first write to `CLAUDE.local.md`, the skill checks that git ignores the file. When git does not, it adds the file to `.gitignore`, and it keeps private detail out of the file when git tracks it already. The lab-notes count now uses `grep -Ec`: plain grep read the pattern wrong and counted zero. A rotation moves every entry past the latest 10 and raises the header count by the number moved. Discovery finds every CLAUDE.md, LABS.md and CLAUDE.local.md with no cap of 20, and skips `node_modules` and `.git`. The structure listing tells files from folders and handles a root with no folders. `LABS.md` sits in the same folder as its CLAUDE.md, also when that CLAUDE.md is in a subfolder.
- **v0.0.13 (2026-09-22).** README shows the cream banner `assets/pvc-claude-md-banner.jpg` full width under the title, the card art on a sheet in its own paper color with the PVC five-bar mark bottom right; the social preview matches it with the mark bottom left. Docs only, no change to what the skill does.
- **v0.0.12 (2026-09-04).** Corrected the release-zip install note: a project or personal skill takes its command name from the directory name, so an unrenamed folder shows up as `/pvc-claude-md-0.0.11`. The earlier note said it would not load.
- **v0.0.11 (2026-09-04).** Install note for the GitHub release zip. The zip unpacks to `pvc-claude-md-<version>`, and a project or personal skill takes its command name from the directory name, so the Install section now says to rename the folder to `pvc-claude-md` before placing it. First GitHub Release (tag v0.0.10) published the same day.
- **v0.0.10 (2026-09-01).** TRADEMARK.md now names this repository's URL for issues and as the version of record, and states that the artwork in `assets/` is covered by the code license while the names and logo stay trademarks. Also: the score label for 1 is now "weak" in the scale text to match the audit report, and the lab-notes header string is quoted the same way everywhere.
- **v0.0.9 (2026-09-01).** TRADEMARK.md contact and update clauses now point at the Pro Vibe Coding repository on GitHub instead of "this repository", so the policy reads right inside an installed copy.
- **v0.0.8 (2026-09-01).** README artwork and social preview now use the bar-chart art from the Pro Vibe Coding brand kit.
- **v0.0.7 (2026-09-01).** Public release under Apache 2.0. The members-only LICENSE is replaced by the Apache License 2.0, a NOTICE file is added, SKILL.md carries an SPDX header, and TRADEMARK.md is rewritten for public use. First Pro Vibe Coding skill published on the public GitHub organization. Also: the personal memory file is now named correctly as `CLAUDE.local.md`, session learnings get a strip-private-detail rule, a STRUCTURE block no longer raises the Pillar 1 score, and the LABS.md header template is literal text.
- **v0.0.6 (2026-07-24).** Claude 5 era scoring for Pillar 1. Knowledge Compression now rewards non-obvious knowledge only: facts Claude discovers by exploring the repo count against the score and get flagged as removal candidates in the audit. Follows Anthropic's context-engineering guidance for the Claude 5 generation models.
- **v0.0.5 (2026-07-11).** Structure sync now follows the block's home folder. Step folders can live inside a dotfolder (for example `.pvc_content_pipeline/`): the listing step reads the block to find where the folders live instead of always listing the project root.
- **v0.0.4 (2026-07-11).** Project structure sync. When CLAUDE.md carries a `STRUCTURE:BEGIN`/`STRUCTURE:END` block, each run checks it against the real folder listing, regenerates it on drift, and flags mismatches in the linked diagrams file. New `Structure:` line in the audit report.
- **v0.0.3 (2026-06-05).** Release packaging. README rewritten to the Standard Readme spec with all four install paths, a usage output sample, a comparison table, and known limitations. Added the members-only LICENSE and TRADEMARK.md.
- **v0.0.2 (2026-05-22).** Lab notes rotation. `CLAUDE.md` keeps the latest 10; older entries archive to `LABS.md`. New triggers: "rotate lab notes", "compact lab notes", "archive old lab notes".
- **v0.0.1 (2026-05-19).** Initial release. 4-pillar audit, targeted improvement, session-learning capture.

## License

This project is licensed under the [Apache License 2.0](LICENSE). See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details.

"Pro Vibe Coding" and "PVC" are trademarks. The license does not grant rights to use them. See [TRADEMARK.md](TRADEMARK.md).

Claude and Claude Code are trademarks of Anthropic. This project is independent. It is not affiliated with, endorsed by, or sponsored by Anthropic.
