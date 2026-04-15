# Phase 1: Schema & Scaffolding - Context

**Gathered:** 2026-04-15
**Status:** Ready for planning

<domain>
## Phase Boundary

Establish the CLAUDE.md wiki schema (conventions, page templates, operation workflows, editorial rules), the directory structure (raw/ and wiki/ with subdirectories), the index.md navigation catalog, the log.md operation record, and the git foundation. This phase creates the conventions that all subsequent operations (ingest, query, lint) will follow.

</domain>

<decisions>
## Implementation Decisions

### Schema Design
- **D-01:** Start with a minimal schema in CLAUDE.md. Only include rules that can be enforced by lint. Add complexity incrementally as real usage patterns emerge (per pitfall P6).
- **D-02:** CLAUDE.md must be self-sufficient for a fresh Claude Code session — a new session should be able to read CLAUDE.md and understand all wiki conventions without prior context (per success criterion 3).
- **D-03:** Schema defines 5 wiki page types: source summaries, entities, concepts, synthesis, and queries. Each type has a distinct template.

### Naming Convention
- **D-04:** All wiki page filenames use kebab-case (e.g., `vannevar-bush.md`, `knowledge-compounding.md`). This avoids cross-platform case sensitivity issues (per pitfall P5) and matches URL-friendly conventions.
- **D-05:** Wikilinks use the display name format: `[[vannevar-bush|Vannevar Bush]]` for human-readable display in Obsidian while keeping filenames consistent.

### Frontmatter Schema
- **D-06:** Mandatory frontmatter fields for all wiki pages: `title`, `type`, `sources` (array), `tags` (array), `created` (ISO date), `updated` (ISO date).
- **D-07:** Optional fields vary by page type. Entity pages add `aliases` (array). Source summary pages add `source_path` (relative path to raw source). Concept pages add `related` (array of related concept links).
- **D-08:** Frontmatter uses Obsidian-compatible YAML. Custom fields are not prefixed (no `wiki_*` prefix) — keep it simple at this scale (per pitfall P10, prefix only if Obsidian plugin conflicts arise).

### Index Design
- **D-09:** index.md is organized by page type categories matching the wiki/ subdirectory structure: Sources, Entities, Concepts, Synthesis, Queries. Each entry has a wikilink and one-line summary.
- **D-10:** index.md starts empty (with category headers only) and is populated by ingest/query operations. The initial scaffold has the structure but no entries.

### Log Design
- **D-11:** log.md uses the format `## [YYYY-MM-DD] operation | subject` with a brief description paragraph under each entry. Parseable with `grep "^## \[" log.md`.
- **D-12:** The initial log entry records the wiki creation event.

### Directory Structure
- **D-13:** The vault root is the marvis/ directory. CLAUDE.md, index.md, and log.md live at the root. raw/ and wiki/ are top-level directories.
- **D-14:** raw/ has subdirectories: articles/, papers/, notes/, books/, assets/. wiki/ has subdirectories: sources/, entities/, concepts/, synthesis/, queries/.
- **D-15:** .planning/ is excluded from Obsidian via .obsidian/excluded-patterns or similar mechanism.

### Claude's Discretion
- Template formatting details (heading levels, section ordering within pages)
- Specific tag taxonomy (let it emerge organically from ingested sources)
- Log entry verbosity level (keep it concise but informative)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Definition
- `.planning/PROJECT.md` — Core value, constraints, key decisions
- `.planning/REQUIREMENTS.md` — Full v1 requirements with IDs (SCHM-01..04, SRCM-01..03, INDX-01..03, LOGM-01..02, GITM-01)

### Research
- `.planning/research/ARCHITECTURE.md` — Detailed directory structure, component boundaries, data flow
- `.planning/research/PITFALLS.md` — P5 (wikilink format fragility), P6 (over-engineered schema) directly relevant
- `.planning/research/STACK.md` — Technology choices and ESM constraints

### Idea Document
- `../llmwiki.md` — Original idea document describing the LLM Wiki pattern

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — greenfield project, no existing code in marvis/

### Established Patterns
- None — this phase establishes the foundational patterns

### Integration Points
- Obsidian vault configuration (.obsidian/) will need to be compatible with the directory structure
- Git repository already initialized

</code_context>

<specifics>
## Specific Ideas

- The schema should reference the Tolkien Gateway example from llmwiki.md as a north star for the kind of interlinked wiki this system aims to produce
- Obsidian Graph View should show meaningful clusters by page type once content is added
- The CLAUDE.md schema should include a "Quick Start" section so a fresh Claude Code session can immediately understand how to operate on the wiki

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-schema-scaffolding*
*Context gathered: 2026-04-15*
