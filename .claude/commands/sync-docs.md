# Sync Docs Command

Fetch a changelog URL, extract every changed widget/component, and either update existing doc files or create new ones from a standard template. Always asks user permission before writing anything.

## Usage
```
/sync-docs [changelog-url]
```

Docs folder: `docs/` (created automatically if it doesn't exist)

---

## Full Process

### PHASE 1 — Fetch & Parse Changelog

Fetch the changelog URL using WebFetch.

Extract every change item. For each item identify:
- **Widget/component name** — the specific thing that changed
- **Change type** — `Improvement` | `Bug Fix` | `Breaking` | `Deprecated` | `Removed`
- **Change description** — plain English, one sentence
- **Version** and **date** from the page

Print a clean parsed list so the user can see what was found:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fetched: https://roadmap.wdesignkit.com/updates/v2-2-8
Version: v2.2.8  |  Date: 2026-05-05
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Found 8 changes:

  Improvement  Tooltip            Content enhanced for better clarity
  Improvement  Nexter Blocks      Icon updated
  Improvement  License Page       Responsive UI optimised
  Improvement  White Label        Flow logic refined; validation uses white_label meta
  Bug Fix      Number Controller  Default value issue resolved
  Bug Fix      Widget Builder     Name label inconsistencies + input box fixed
  Bug Fix      Storage Page       Responsive issue resolved
  Bug Fix      Product Logo       Display issue fixed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### PHASE 2 — Check Which Docs Exist

List all files currently in `docs/`. For each widget found in the changelog:
- Check if `docs/[widget-name-slug].md` already exists
- **Exists** → will UPDATE the `## Changelog` section of that file
- **Does not exist** → will CREATE a new file from the standard template

Print the status:
```
  Tooltip            →  docs/tooltip.md             [CREATE]
  Nexter Blocks      →  docs/nexter-blocks.md        [CREATE]
  License Page       →  docs/license-page.md         [CREATE]
  White Label        →  docs/white-label.md          [CREATE]
  Number Controller  →  docs/number-controller.md    [CREATE]
  Widget Builder     →  docs/widget-builder.md       [CREATE]
  Storage Page       →  docs/storage-page.md         [CREATE]
  Product Logo       →  docs/product-logo.md         [CREATE]
```

---

### PHASE 3 — Generate Doc Content

For each widget, prepare the full content using the doc-sync-agent.

**If CREATE:** generate a complete doc file using the standard widget template (see below), with the `## Changelog` section pre-filled with this version's changes.

**If UPDATE:** generate only the new changelog entry block to insert into the existing file's `## Changelog` section.

---

### PHASE 4 — Show Preview & Ask Permission

Show the user exactly what will be created/updated. **Do NOT write any files yet.**

For CREATE items, show the full file preview:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[CREATE] docs/tooltip.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Tooltip

> Last updated: v2.2.8 — 2026-05-05

## Overview
[Describe what the Tooltip widget does — fill this in]

## Getting Started
[How to add this widget to your page — fill this in]

## Content
[Content settings: text, labels — fill this in]

## Layout
[Layout options: position, spacing — fill this in]

## Style
[Style options: colors, typography, icons — fill this in]

## Settings
[Configuration options — fill this in]

## Advanced
[Advanced options — fill this in]

## Changelog

### v2.2.8 — 2026-05-05
> **Improvement:** Tooltip content enhanced for better clarity.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[CREATE] docs/number-controller.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Number Controller
...same structure...
### v2.2.8 — 2026-05-05
> **Bug Fix:** Default value issue in the Number Controller resolved.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then ask:
```
Ready to create 8 doc files in docs/

  yes          → create all 8 files
  no           → cancel, nothing is written
  1,3,5        → create only those (refer to widget names or numbers)
  skip 2,4     → create all except those
```

**Wait for the user's response before writing anything.**

---

### PHASE 5 — Create / Update Files

Apply only what the user approved.

**For CREATE:**
Write the full file to `docs/[widget-name-slug].md` using the standard template.

**For UPDATE:**
Insert the new version block at the TOP of the `## Changelog` section (newest version first):
```markdown
## Changelog

### v2.2.8 — 2026-05-05
> **Improvement:** Tooltip content enhanced for better clarity.

### v2.2.7 — [previous date]   ← existing entries stay below
...
```

---

### PHASE 6 — Write Sync Log

Append to `docs-updates/CHANGELOG.md` (create if missing):

```markdown
## v2.2.8 — 2026-05-05
Source: https://roadmap.wdesignkit.com/updates/v2-2-8

### Created
- docs/tooltip.md
- docs/nexter-blocks.md
- docs/license-page.md
- docs/white-label.md
- docs/number-controller.md
- docs/widget-builder.md
- docs/storage-page.md
- docs/product-logo.md
```

---

### PHASE 7 — Final Report

```
✓ Created 8 doc files in docs/
✓ Sync log written to docs-updates/CHANGELOG.md

Next steps:
  Open each file in docs/ and fill in the placeholder sections
  (Overview, Getting Started, Content, Layout, Style, Settings, Advanced)
  Future runs of /sync-docs will update the Changelog section automatically.
```

---

## Standard Widget Doc Template

Every newly created doc file uses this exact structure:

```markdown
# [Widget Name]

> Last updated: [version] — [date]

## Overview

[Describe what the [Widget Name] does and when to use it — fill this in]

## Getting Started

[How to add this widget to your page. Any prerequisites — fill this in]

## Content

[Content settings: text, labels, tooltips, media — fill this in]

## Layout

[Layout options: position, alignment, spacing, size — fill this in]

## Style

[Style settings: colors, typography, borders, shadows, icons — fill this in]

## Settings

[Configuration options and controls — fill this in]

## Advanced

[Advanced options, custom CSS, responsive controls — fill this in]

## Changelog

### [version] — [date]
> **[Change type]:** [Change description]
```

---

## Flags
- `--force` — re-process a URL already in the sync log
- `--dry-run` — run Phases 1–4 only (preview without creating files)
