# pvc-claude-md

<p align="center"><img src="assets/pvc-claude-md.jpg" width="360" alt="pvc-claude-md"></p>

![version](https://img.shields.io/badge/version-v0.0.9-blue) ![type](https://img.shields.io/badge/Claude%20Code-skill-7C5CFF) ![license](https://img.shields.io/badge/license-Apache%202.0-green)

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
2. **Missing capability declarations.** Claude does not know which skills or MCP servers are available, so it reinvents work you already automated.
3. **No memory of past failures.** The same dead ends get hit again because nothing recorded them.

This skill addresses all three in one pass instead of three separate habits you have to remember.

## What it does

- **Audits** an existing `CLAUDE.md` against 4 pillars and returns a scored report (0 to 12).
- **Suggests targeted diffs** for whatever pillar is missing or weak, kept minimal.
- **Captures session learnings** as dated lab notes so the next session inherits them.
- **Rotates lab notes** so `CLAUDE.md` holds the latest 10 while older history archives to `LABS.md` (v0.0.2+).
- **Syncs the project structure block** when `CLAUDE.md` carries `STRUCTURE:BEGIN`/`STRUCTURE:END` markers: every run compares the block (mermaid folder diagram, folder notes) against the real folder listing and regenerates it on drift (v0.0.4+).

## The 4 pillars

Each pillar scores 0 to 3: 0 = missing, 1 = minimal, 2 = adequate, 3 = strong.

| Pillar | Covers |
|---|---|
| Knowledge Compression | Project name, purpose, tech stack, key files, commands, audience |
| User Preferences | Style rules, conventions, voice, banned vocabulary |
| Capability Declarations | MCP servers, skills, scripts, integrations |
| Lab Notes | Running log of what worked, what did not, gotchas |

## Install

This is a Claude Code skill. Pick the path that matches where you want it available.

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
            ├── assets/          (README artwork)
            ├── REQUIREMENTS.md
            ├── LICENSE
            ├── NOTICE
            └── TRADEMARK.md
```

### Personal (all your projects)

Copy the `pvc-claude-md/` folder to `~/.claude/skills/` for user-global access across every project.

### Polyrepo / monorepo

Place it once at `.claude/skills/pvc-claude-md/` in the directory you open Claude Code from. The skill is available whenever you launch Claude Code from that root. Availability depends on where you launch Claude Code, not on which nested package is your current directory.

### As part of a plugin

If you bundle PVC skills as a Claude Code plugin, include `pvc-claude-md/` in the plugin's `skills/` directory. Claude Code auto-discovers skills in that folder when the plugin loads.

### Verify

Start a new Claude Code session (or reload skills), then ask: "What skills are available?" Confirm `pvc-claude-md` appears. Or type a trigger like "audit my CLAUDE.md".

## Usage

Trigger it with plain language. Any of these work:

- "audit my CLAUDE.md"
- "review my CLAUDE.md" or "check my CLAUDE.md"
- "improve my CLAUDE.md"
- "update CLAUDE.md with what we learned"
- "is my CLAUDE.md good?"
- "PVC CLAUDE.md"
- "rotate lab notes", "compact lab notes", or "archive old lab notes"

### Example

```
You: audit my CLAUDE.md
```

Agent response:

```markdown
## CLAUDE.md Audit Report
**File:** ./CLAUDE.md
**Token estimate:** ~380 tokens
**Lab notes:** 0 in CLAUDE.md, no LABS.md exists
**Structure:** no STRUCTURE markers

| Pillar | Score | Status |
|--------|-------|--------|
| Knowledge Compression  | 0 | Missing |
| User Preferences        | 3 | Strong  |
| Capability Declarations | 0 | Missing |
| Lab Notes               | 0 | Missing |

**Overall: 3/12**

### Recommendations
- Add project name, purpose, and key files (Knowledge Compression)
- Declare available skills and MCP servers (Capability Declarations)
- Start a Lab notes section so future sessions inherit lessons
```

It then shows the proposed diffs and asks before writing. It only edits files you approve:

```diff
### Update: ./CLAUDE.md
+ ## Capabilities
+ - Skills: pvc-claude-md, pvc-readme-writer, pvc-license
```

## Lab notes rotation

`CLAUDE.md` is loaded into every session's context, so a long lab-notes log quietly taxes every prompt. From v0.0.2, the skill keeps the latest 10 entries in `CLAUDE.md` and archives older ones to a sibling `LABS.md`.

- Adding the 11th entry moves the oldest to the bottom of `LABS.md`.
- The section header tracks the count: `**N older entries archived in [LABS.md](LABS.md).**`
- When the 11th entry triggers a move, the skill reports it: `### Rotation: oldest entry moves to LABS.md`.
- `LABS.md` is append-only and frozen. The skill does not rewrite entries already there.
- Searching history? Grep both files: `grep -n "pattern" CLAUDE.md LABS.md`

## How it compares

Here is how this skill compares with the two common alternatives, so you can tell before installing whether you need it.

| | pvc-claude-md | Built-in `/init` | Editing by hand |
|---|---|---|---|
| Scores quality | Yes, 4 pillars, 0 to 12 | No | No |
| Proposes targeted diffs | Yes | Generates a file from scratch | You do it |
| Captures session learnings | Yes, dated lab notes | No | If you remember |
| Caps context cost | Yes, rotates to LABS.md | No | No |
| Best for | Keeping an existing CLAUDE.md current | First-time file creation | Small one-off tweaks |

## Output rubric

Every audit report the skill produces aims to pass on:

- **Accurate scoring.** Each pillar judged against what the file actually contains.
- **Minimal diffs.** Additions are brief, no padding, nothing already obvious from the code.
- **Voice match.** Follows the target project's own style rules, with no em dashes in new content.
- **Rotation integrity.** `CLAUDE.md` never exceeds 10 lab notes, and the archive count stays accurate.

## Known limitations

- Token estimates are approximate, not a tokenizer count.
- Rotation assumes the `### YYYY-MM-DD: title` lab-note format. Free-form notes may not rotate cleanly.
- It edits only files you approve, so a fully unattended run still pauses for confirmation.

## Changelog

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
