# Changelog — 2026-05-05 — changelog-system-launch

## Summary

Introduces a lightweight changelog refiner system that converts raw update notes into structured Markdown summaries and flags which documentation may need updates. The system is purely local — it never touches git — and is invoked via the new `/changelog` slash command, which delegates the heavy lifting to a new `changelog-refiner` agent.

## Changes

### Added
- Add `/changelog` slash command for converting raw notes into structured changelog entries
  - Detail: Accepts either a file path or pasted free-text, outputs a dated per-batch file under `changelogs/` and prepends a Keep-a-Changelog summary to root `CHANGELOG.md`.
  - Affected: `.claude/commands/changelog.md`
- Add `changelog-refiner` agent for parsing, categorizing, polishing, and doc-impact scanning
  - Detail: Categorizes items using Keep-a-Changelog (Added / Changed / Fixed / Removed / Deprecated / Security), produces concrete `suggested_update` strings, and reads candidate doc files to ground its suggestions instead of guessing.
  - Affected: `.claude/agents/changelog-refiner.md`
- Add `changelogs/` directory with README describing output format and guardrails
  - Affected: `changelogs/README.md`

### Changed
_None._

### Fixed
_None._

## Documentation Impact

| Doc | Section | Status | Suggested Update |
|---|---|---|---|
| `CLAUDE.md` | Commands (lines 17–34) | Likely | Add bullet: `` - `/changelog [input]` - Convert raw update notes into structured changelog entries in `changelogs/`, with doc-impact suggestions `` |
| `CLAUDE.md` | Architecture → Command-Agent Model (line 42) | Likely | Append `changelog-refiner.md` to the "Key agents" list |
| `CLAUDE.md` | Content Pipeline (lines 84–88) | Possible | Optionally mention `changelogs/` alongside `rewrites/`, `landing-pages/`, `audits/`, `repurposed/` as another output directory |
| `README.md` (root) | — | Possible | No `README.md` exists at repo root; if/when one is added, mention the changelog workflow |

## Breaking Changes

None.

## Raw Notes (for traceability)

<details>
<summary>Original input</summary>

Created the lightweight changelog refiner system today:
- New `/changelog` slash command at `.claude/commands/changelog.md` — accepts a file path or pasted free-text, runs the refiner agent, writes a per-batch file and updates root CHANGELOG.md
- New `changelog-refiner` agent at `.claude/agents/changelog-refiner.md` — parses raw notes into atomic items, categorizes via Keep-a-Changelog, polishes one-liners, scans CLAUDE.md / commands / agents / context / data_sources for doc-impact, returns structured output
- New `changelogs/` directory with a README explaining the format and guardrails
- System never touches git; doc updates remain manual

</details>
