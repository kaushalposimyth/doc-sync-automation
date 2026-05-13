# Doc Sync Automation

A Claude Code workspace that automatically keeps your documentation in sync with product changelogs.

## What It Does

1. **Paste a changelog URL** → system fetches and parses every change
2. **Matches each widget/component** to its doc file in `docs/`
3. **Finds the right section** (Content, Layout, Style, Settings, etc.)
4. **Shows you a preview** and asks permission before touching anything
5. **Applies approved changes** — inserts versioned notes into the correct sections
6. **Logs everything** to `docs-updates/CHANGELOG.md`

On first run: creates doc files from scratch (standard template with placeholder sections + pre-filled Changelog).
On future runs: updates only the `## Changelog` section of existing files.

## Commands

| Command | What it does |
|---|---|
| `/sync-docs [url]` | Main command — fetch changelog, propose updates, ask permission, apply |
| `/apply-docs [file or "latest"]` | Apply a pending proposed-edits file manually |
| `/changelog [notes or file]` | Convert your own raw update notes into a structured changelog entry |

## Folder Structure

```
docs/                        ← one .md file per widget/component
docs-updates/
  CHANGELOG.md               ← log of every sync run
  .applied.json              ← tracks which URLs have been processed
  proposed-*.md              ← proposed edits waiting for approval
changelogs/                  ← structured changelog entries from /changelog command
CHANGELOG.md                 ← root Keep-a-Changelog summary
.claude/
  commands/                  ← slash command definitions
  agents/                    ← specialist agents
```

## Standard Doc Template

Every new widget doc is created with these sections:

- **Overview** — what the widget does
- **Getting Started** — how to add it
- **Content** — text, labels, media settings
- **Layout** — spacing, alignment, position
- **Style** — colors, typography, icons
- **Settings** — configuration options
- **Advanced** — custom CSS, responsive controls
- **Changelog** — auto-updated version history ← only this section is written automatically

## Quick Start

```
# First time — creates all doc files from changelog
/sync-docs https://your-product.com/changelog/v1-0-0

# Future versions — updates existing docs
/sync-docs https://your-product.com/changelog/v1-1-0
```

## Requirements

- [Claude Code](https://claude.ai/code)
- Public changelog URL (HTML page, GitHub Releases, RSS — any format)
- No git, no database, no backend — pure markdown files

## Safety Guarantees

- Nothing is written until you approve
- Only inserts content — never deletes existing text
- Same URL run twice is blocked (use `--force` to override)
- Python files and live sites are never touched
