<!-- SPDX-License-Identifier: Apache-2.0 -->

# Project Structure Sync

Read this only when a CLAUDE.md holds a `STRUCTURE:BEGIN` marker. Some projects keep an auto-managed structure block in CLAUDE.md between that marker and a `STRUCTURE:END` marker:

```markdown
<!-- STRUCTURE:BEGIN ... -->
[folder overview, usually a mermaid diagram plus a link to the project's diagrams file]
<!-- STRUCTURE:END -->
```

When the markers exist, every run of this skill refreshes the block.

## How to sync

1. Find where the mapped folders live before listing anything. Read the block: if it names a home folder for the steps (for example `.pipeline/`), list inside that folder (`ls -ap`: folders end in `/`, files do not). Otherwise list the project root (`ls -d */ 2>/dev/null`; no output means no folders). Skip infrastructure dotfolders like `.claude` and `.git`, but never skip a dotfolder the block itself names as the home. These listings are Bash commands; in a session without Bash, get the same listing with the PowerShell tool
2. Compare that listing against the folder names inside the block (mermaid node labels and any folder lists must match the real folder names exactly)
3. In sync: leave the block untouched, report `Structure: in sync` in the audit
4. Drifted (folder added, renamed, or removed): regenerate the mermaid and folder notes inside the markers from the real listing, report `Structure: drifted (N changes)` with a diff. Node labels stay identical to the exact folder names
5. If the block links a diagrams file (for example `docs/diagrams/pipeline.md`), check its node labels against the same listing and propose updating it in the same pass so CLAUDE.md and the diagram never disagree

Structure sync edits follow the same approval rule as everything else in this skill: show the diff, ask before writing. A regenerated block goes into Step 4's proposed updates like any other diff.

## Effect on Pillar 1

The block feeds Pillar 1 (Knowledge Compression) in one direction only. A structure block is a folder listing, so an in-sync block does not raise the score. A drifted block misleads Claude, so it caps the pillar at weak until regenerated.

The block is the one exception to the removal-candidate rule of Pillar 1: keep it, never list it as a removal candidate, and never let it raise the score.
