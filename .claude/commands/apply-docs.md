# Apply Docs Command

Apply a proposed-edits file to the actual documentation files.

## Usage
```
/apply-docs [proposed-file]
/apply-docs latest
```

- Pass a specific file path from `docs-updates/`
- Pass `latest` to auto-select the most recently modified pending proposed file

## Process

### Step 1 — Read the proposed file
Parse every `<!-- EDIT:N -->` block. Extract:
- `TARGET` — file to edit
- `AFTER_HEADING` — insertion point
- `ACTION` — what to do
- `REASON` — for logging
- Content between `---` and `<!-- /EDIT:N -->`

### Step 2 — Pre-flight check
For each edit:
- Verify `TARGET` file exists. If not, skip and log a warning.
- Verify `AFTER_HEADING` exists in the file. If not, skip and log a warning.
- Check that the content to insert is not already present (idempotency guard). If it is, skip silently.

### Step 3 — Apply each edit
Based on `ACTION`:

**`insert_section`** — Insert the content block immediately after the `AFTER_HEADING` line and one blank line. Preserve existing content below.

**`update_section`** — Find the section between `AFTER_HEADING` and the next same-level heading. Replace only the matched content block (identified by version/date tag). Do not touch other content in the section.

**`prepend_warning`** — Insert the content immediately before the first line of the `AFTER_HEADING` section body (after the heading itself). Used for breaking-change and deprecation notices.

### Step 4 — Update tracking
In `docs-updates/.applied.json`, find the entry matching this proposed file and update:
```json
{
  "status": "applied",
  "applied_at": "YYYY-MM-DD",
  "edits_applied": N,
  "edits_skipped": M
}
```
Also prepend `status: applied` to the proposed file's front matter.

### Step 5 — Report
```
✓ Applied N edits, skipped M

  Applied:
    context/competitor-analysis.md — inserted "What's New — v2.5" after "## DataForSEO"
    CLAUDE.md — inserted "What's New" after "### Data Integrations"

  Skipped:
    .claude/commands/publish-draft.md — heading "## Process" not found

  Proposed file updated: docs-updates/proposed-2026-05-05-wordpress-5-8.md (status → applied)
```

## Safety rules
- Never deletes existing content — only inserts or prepends
- Never edits files not listed in the proposed file
- Never applies an already-applied proposed file (checks `status` in front matter)
- Skips rather than errors when a heading is missing — logs it clearly
