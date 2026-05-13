# Changelog Command

Use this command to convert raw update notes into a structured changelog entry and flag which documentation may need updates.

## Usage
```
/changelog [input]
```

Where `[input]` is one of:
- A file path (e.g. `notes.txt`, `updates/may-week-1.md`) — the command reads the file
- A free-text dump pasted directly after the command — anything from a Slack export, meeting notes, PR descriptions, bullet list, or stream-of-consciousness recap
- Empty — the command will prompt you for the raw notes

## What This Command Does

1. Parses raw update notes into discrete change items
2. Categorizes each item using Keep-a-Changelog conventions (Added / Changed / Fixed / Removed / Deprecated / Security)
3. Rewrites each item as a polished, terse one-line entry
4. Scans the repo and flags documentation sections that likely need updates
5. Writes a dated changelog file to `changelogs/`
6. Appends a summary entry to the root `CHANGELOG.md` (creating it if missing)

## Process

### Step 1 — Ingest

If `[input]` is a path that exists, read the file. Otherwise treat the argument as the raw notes themselves. If no input is given, ask the user to paste their notes.

Strip noise (timestamps, names, formatting artifacts) but preserve technical detail.

### Step 2 — Refine via the changelog-refiner agent

Invoke the **changelog-refiner** agent (`.claude/agents/changelog-refiner.md`) with the raw notes. The agent returns:

- A list of discrete change items, each with: `category`, `summary` (one line), `detail` (optional 1–2 sentence expansion), `affected_paths` (best guess), `breaking` (bool)
- A doc-impact list mapping each change to specific files/sections that may need updates

### Step 3 — Doc-impact scan

For each refined change, check whether any of the following surfaces likely need an update. The agent does this by reading the relevant files (not guessing):

- **`CLAUDE.md`** — Setup, Commands list, Architecture, Python Analysis Pipeline, Data Integrations, Running Python Scripts, Content Pipeline, Context Files, WordPress Integration
- **`.claude/commands/*.md`** — if a command's behavior, args, or outputs changed
- **`.claude/agents/*.md`** — if an agent's role, inputs, or outputs changed
- **`context/*.md`** — brand voice, style guide, SEO guidelines, internal links map, features, competitor analysis, CRO best practices, AI citation targets, Reddit strategy
- **`data_sources/`** — if a Python module's API changed, flag any script that imports it
- **`README.md`** (if present at root)

The output marks each suggestion as **Likely**, **Possible**, or **Unlikely** so the human can triage quickly.

### Step 4 — Write outputs

**Per-batch file:** `changelogs/YYYY-MM-DD-[slug].md`

```markdown
# Changelog — YYYY-MM-DD — [slug]

## Summary
[One-paragraph human-readable recap of what changed and why it matters.]

## Changes

### Added
- [polished one-liner]
  - Detail: [optional expansion]
  - Affected: `path/to/file.py`

### Changed
- [polished one-liner]

### Fixed
- [polished one-liner]

### Removed / Deprecated / Security
- [as applicable; omit empty sections]

## Documentation Impact

| Doc | Section | Status | Suggested Update |
|---|---|---|---|
| `CLAUDE.md` | Commands | Likely | Add `/changelog` to the slash command list |
| `.claude/agents/changelog-refiner.md` | — | Likely | New agent — already created by this change |
| `README.md` | Quick Start | Possible | Mention the changelog workflow |

## Breaking Changes
[List any breaking changes with migration notes, or write "None."]

## Raw Notes (for traceability)
<details>
<summary>Original input</summary>

[verbatim raw input that was refined]

</details>
```

**Root `CHANGELOG.md`:** prepend a compact entry under a new dated heading, Keep-a-Changelog style:

```markdown
# Changelog

All notable changes to this project are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased] — YYYY-MM-DD

### Added
- [one-liner] — see `changelogs/YYYY-MM-DD-[slug].md`

### Changed
- [one-liner]

### Fixed
- [one-liner]
```

If a `## [Unreleased]` block already exists for today's date, merge into it instead of creating a duplicate.

### Step 5 — Report back

Print a short summary to the user:
- Path to the new per-batch file
- Number of changes per category
- Top 3 doc-impact items flagged as **Likely**
- Whether `CHANGELOG.md` was created or updated

## Output File Naming

- **Slug**: derive from the most prominent theme of the batch (e.g. `wordpress-publish-fix`, `cluster-command-rework`, `analytics-refresh`). Keep it under 5 words, kebab-case.
- **Date**: today's date in `YYYY-MM-DD`.
- **Collisions**: if a file with the same name already exists, suffix with `-2`, `-3`, etc.

## Categorization Rules

Use Keep-a-Changelog's six categories. Map ambiguous items as follows:

- New feature, command, agent, file, or capability → **Added**
- Behavior or output change to an existing thing → **Changed**
- Bug fix or regression patch → **Fixed**
- Anything pulled out of the codebase → **Removed**
- Still present but flagged for future removal → **Deprecated**
- Vulnerability patches, credential handling, auth → **Security**

When in doubt, prefer **Changed** over **Added**.

## Examples

### Example 1 — Pasted notes

User runs:
```
/changelog finished the new /changelog command today, also fixed a bug where publish-draft was double-encoding emoji in titles, and the editor agent now reads the brand-voice file before every pass
```

The command produces `changelogs/2026-05-04-changelog-system.md` with three entries (Added, Fixed, Changed) and flags `CLAUDE.md` Commands section + `.claude/agents/editor.md` as **Likely** doc updates.

### Example 2 — File input

```
/changelog updates/sprint-22.md
```

Reads the file and processes its contents the same way.

## Integration Notes

- **Does not touch git.** This command only reads/writes files in the working tree. Committing the changelog is the user's call.
- **Idempotent on re-run.** Running it twice on the same input creates a `-2` suffixed file rather than overwriting.
- **Safe with empty repo state.** If `CHANGELOG.md` doesn't exist, it gets created with the standard header.
