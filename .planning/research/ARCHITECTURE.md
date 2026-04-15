# Architecture Research
**Domain:** LLM-powered personal knowledge base wiki
**Researched:** 2026-04-15

## System Components

### 1. Raw Source Store
The immutable layer. A directory of user-curated source documents (markdown, text, clipped articles, PDFs converted to markdown). The LLM reads from this layer but never modifies it. Each source file is the ground truth — all wiki content traces back here.

**Responsibilities:**
- Hold original source material in markdown format
- Serve as the authority when wiki claims need verification
- Store associated assets (images, diagrams) in a dedicated subdirectory

**Boundaries:** This component has no dependencies. It is purely passive — written to by the user (or Obsidian Web Clipper), read by the LLM during ingest and when answering queries that need source-level detail.

### 2. Wiki Layer
The LLM-maintained knowledge artifact. A directory of interlinked markdown pages that the LLM creates, updates, and maintains. The user reads this layer through Obsidian but does not write to it (except optionally to correct errors or add annotations).

**Page types:**
- **Source summaries** — one per ingested source, containing key takeaways, metadata, and links to entity/concept pages
- **Entity pages** — pages for specific people, organizations, tools, books, etc.
- **Concept pages** — pages for ideas, frameworks, patterns, themes
- **Synthesis pages** — comparisons, analyses, timelines, overviews that span multiple sources
- **Query-derived pages** — answers to user questions that were valuable enough to persist

**Responsibilities:**
- Maintain interlinked, up-to-date knowledge across all pages
- Use `[[wikilinks]]` for cross-referencing
- Include YAML frontmatter for metadata (tags, dates, source count, page type)
- Keep pages consistent — when a new source updates an entity, all pages referencing that entity should reflect the update

**Boundaries:** Written exclusively by the LLM. Read by both the user (via Obsidian) and the LLM (during query and lint operations). Depends on raw sources for content, and on the schema for conventions.

### 3. Index (index.md)
A content-oriented catalog of all wiki pages. This is the LLM's primary navigation tool — the equivalent of a table of contents that the LLM reads first to decide which pages to drill into.

**Structure:**
- Organized by category (sources, entities, concepts, syntheses)
- Each entry: `[[page link]]` + one-line summary + optional metadata
- Updated on every ingest operation

**Boundaries:** Written by the LLM. Read by both the LLM (as the first step of query operations) and the user. This is the component that eliminates the need for embedding-based RAG at moderate scale. At ~100 sources / ~hundreds of pages, a well-maintained index gives the LLM enough information to find relevant pages without vector search.

### 4. Log (log.md)
A chronological, append-only record of all operations performed on the wiki.

**Structure:**
- Each entry starts with a parseable prefix: `## [YYYY-MM-DD] operation | subject`
- Operation types: ingest, query, lint, update
- Entries include what was done and which pages were touched

**Boundaries:** Append-only. Written by the LLM at the end of every operation. Read by the LLM (to understand recent context) and the user (to see the wiki's evolution timeline). Parseable with simple unix tools (`grep`, `tail`).

### 5. Schema (CLAUDE.md)
The configuration file that turns a generic LLM into a disciplined wiki maintainer. Defines conventions, page formats, workflows, and rules. Co-evolves with the wiki as the user and LLM discover what works.

**Boundaries:** Co-written by the user and LLM. Read by the LLM at the start of every session. This is the single most important file for wiki quality — it determines how consistently the LLM maintains the wiki across sessions.

### 6. LLM Agent (Claude Code)
The engine that performs all operations. Not a stored component but the active agent that reads the schema, processes sources, maintains the wiki, and answers queries.

**Boundaries:** Reads schema, raw sources, index, log, and wiki pages. Writes to wiki pages, index, and log. Never writes to raw sources. Operates through Claude Code's file editing capabilities.

## Data Flow

### Ingest Flow
```
User drops source file into raw/
    |
    v
User tells LLM: "ingest raw/source-file.md"
    |
    v
LLM reads CLAUDE.md (schema) for conventions
    |
    v
LLM reads the source file
    |
    v
LLM reads index.md to understand existing wiki state
    |
    v
LLM creates/updates wiki pages:
    1. Create source summary page (wiki/sources/source-file.md)
    2. Create or update entity pages mentioned in the source
    3. Create or update concept pages for ideas in the source
    4. Flag contradictions with existing wiki content
    5. Add/update cross-reference [[wikilinks]] across all touched pages
    |
    v
LLM updates index.md with new/modified page entries
    |
    v
LLM appends to log.md: "[date] ingest | Source Title — created N pages, updated M pages"
```

**Key property:** A single ingest may touch 10-15+ wiki pages. The LLM reads broadly (index, related pages) before writing, to ensure consistency.

### Query Flow
```
User asks a question
    |
    v
LLM reads CLAUDE.md (schema) for conventions
    |
    v
LLM reads index.md to identify relevant pages
    |
    v
LLM reads the relevant wiki pages (not raw sources — the wiki is the compiled knowledge)
    |
    v
LLM synthesizes an answer with [[wikilink]] citations to wiki pages
    |
    v
[Optional] If the answer is valuable, LLM creates a new wiki page for it
    |
    v
[Optional] LLM updates index.md if a new page was created
    |
    v
LLM appends to log.md: "[date] query | Question summary — cited N pages"
```

**Key property:** Queries read from the wiki layer, not raw sources. The wiki already contains the synthesis, cross-references, and contradiction flags. Raw sources are only consulted when the user needs to verify a specific claim.

### Lint Flow
```
User asks LLM to lint/health-check the wiki
    |
    v
LLM reads CLAUDE.md (schema) for conventions and quality standards
    |
    v
LLM reads index.md for complete page inventory
    |
    v
LLM systematically checks:
    1. Orphan pages (no inbound [[wikilinks]])
    2. Dead links (wikilinks pointing to non-existent pages)
    3. Stale claims (pages not updated since newer contradicting sources were ingested)
    4. Missing pages (entities/concepts mentioned but lacking dedicated pages)
    5. Inconsistencies (same fact stated differently on different pages)
    6. Frontmatter compliance (pages missing required YAML fields)
    7. Index completeness (pages that exist but aren't in index.md)
    |
    v
LLM produces a lint report (can be a wiki page or chat output)
    |
    v
LLM fixes issues or presents them for user decision
    |
    v
LLM appends to log.md: "[date] lint | Found N issues, fixed M"
```

**Key property:** Lint is the maintenance operation that keeps the wiki healthy as it grows. Without it, cross-references drift, orphan pages accumulate, and contradictions go unnoticed.

## Directory Structure

Designed for Obsidian compatibility. All markdown files use `[[wikilinks]]` and YAML frontmatter. The structure separates the three architectural layers cleanly while keeping everything in one Obsidian vault.

```
marvis/                          # Obsidian vault root / git repo root
|
+-- CLAUDE.md                    # Schema — LLM conventions, workflows, page formats
|
+-- index.md                     # Wiki index — categorized catalog of all wiki pages
+-- log.md                       # Operations log — chronological append-only record
|
+-- raw/                         # Layer 1: Raw sources (immutable)
|   +-- articles/                # Clipped web articles
|   +-- papers/                  # Academic papers / reports
|   +-- notes/                   # Personal notes, journal entries
|   +-- books/                   # Book chapters, excerpts
|   +-- assets/                  # Images, diagrams referenced by sources
|   +-- ...                      # User can add categories as needed
|
+-- wiki/                        # Layer 2: LLM-maintained wiki pages
|   +-- sources/                 # Source summary pages (one per ingested source)
|   +-- entities/                # Entity pages (people, orgs, tools, places)
|   +-- concepts/                # Concept/topic pages (ideas, frameworks, patterns)
|   +-- synthesis/               # Cross-cutting analyses, comparisons, timelines
|   +-- queries/                 # Persisted query answers
|
+-- .planning/                   # Project planning (not part of wiki content)
|   +-- PROJECT.md
|   +-- research/
|   +-- ...
|
+-- .obsidian/                   # Obsidian config (auto-generated)
+-- .git/                        # Git version history
```

**Design decisions:**

- **`raw/` and `wiki/` at vault root** — clean separation of immutable sources and LLM-maintained pages. Obsidian sees both and can link between them.
- **`CLAUDE.md` at vault root** — Claude Code reads this automatically. It is the schema layer.
- **`index.md` and `log.md` at vault root** — top-level navigation files, easy to find.
- **Subdirectories within `wiki/`** — categorized by page type. This helps both the LLM (knows where to file new pages) and the user (can browse by category in Obsidian). The subdirectories are a convention, not a hard rule — the LLM can create new categories as needed.
- **Subdirectories within `raw/`** — user-organized by source type. The LLM reads from here but the user controls the organization.
- **`.planning/` hidden from Obsidian** — project planning files are not wiki content. Obsidian can exclude dotfile directories.
- **Flat within each `wiki/` subdirectory** — no deep nesting. Page names are descriptive enough to be unique within their category. Cross-referencing happens through wikilinks, not directory hierarchy.

**Obsidian compatibility notes:**
- All wiki pages use `[[wikilinks]]` for cross-references (Obsidian's native link format)
- YAML frontmatter enables Dataview queries (e.g., list all pages tagged `#stale` or with source_count > 3)
- Graph view works naturally — pages are nodes, wikilinks are edges
- Attachment folder set to `raw/assets/` for downloaded images

## Schema Design

The `CLAUDE.md` schema is the most important design artifact. It transforms a generic LLM into a consistent wiki maintainer. Here is what it should contain:

### 1. Wiki Identity and Purpose
```markdown
# Marvis Wiki

## Purpose
[One paragraph describing what this wiki is about — the domain, the user's goals, what kind of knowledge is being accumulated.]
```

This orients the LLM at the start of every session. Without it, the LLM has no context for editorial judgment (what's important, what level of detail is appropriate, what tone to use).

### 2. Directory Conventions
```markdown
## Directory Structure
- `raw/` — immutable source documents. Never modify files here.
- `wiki/sources/` — one summary page per ingested source
- `wiki/entities/` — pages for people, organizations, tools, etc.
- `wiki/concepts/` — pages for ideas, frameworks, themes
- `wiki/synthesis/` — cross-cutting analyses and comparisons
- `wiki/queries/` — persisted answers to user questions
- `index.md` — categorized catalog of all wiki pages (update on every ingest)
- `log.md` — chronological operation log (append on every operation)
```

### 3. Page Format Templates
```markdown
## Page Formats

### Source Summary
---
type: source
title: [Title]
source_file: [[raw/path/to/source]]
date_ingested: YYYY-MM-DD
tags: [tag1, tag2]
---
# [Title]
## Key Takeaways
## Detailed Notes
## Related Pages

### Entity Page
---
type: entity
title: [Entity Name]
category: [person|org|tool|place|...]
source_count: N
last_updated: YYYY-MM-DD
tags: [tag1, tag2]
---
# [Entity Name]
## Overview
## Key Details
## Appearances in Sources
## Related Entities

### Concept Page
---
type: concept
title: [Concept Name]
source_count: N
last_updated: YYYY-MM-DD
tags: [tag1, tag2]
---
# [Concept Name]
## Definition
## Key Points
## Sources
## Related Concepts
```

Templates ensure consistency. The LLM knows exactly what sections to include, what frontmatter to set, and how to structure each page type.

### 4. Workflow Definitions
```markdown
## Workflows

### Ingest
When the user asks to ingest a source:
1. Read the source file completely
2. Read index.md to understand current wiki state
3. Create a source summary page in wiki/sources/
4. For each entity mentioned: create or update its page in wiki/entities/
5. For each concept discussed: create or update its page in wiki/concepts/
6. Add [[wikilinks]] to all related pages (bidirectional)
7. If new information contradicts existing wiki content, note the contradiction explicitly on the relevant page with both the old and new claims, citing sources
8. Update index.md with entries for all new/modified pages
9. Append to log.md

### Query
When the user asks a question:
1. Read index.md to find relevant pages
2. Read the relevant pages
3. Synthesize an answer citing wiki pages with [[wikilinks]]
4. If the answer is substantial, offer to persist it as a wiki page in wiki/queries/ or wiki/synthesis/
5. Append to log.md

### Lint
When the user asks to lint/health-check:
1. Read index.md for complete page inventory
2. Check for: orphan pages, dead links, stale content, missing pages, inconsistencies, frontmatter issues
3. Present findings with specific fix suggestions
4. Fix issues the user approves
5. Append to log.md
```

### 5. Editorial Rules
```markdown
## Editorial Rules
- Use [[wikilinks]] for all cross-references between wiki pages
- Every wiki page must have YAML frontmatter with at least: type, title, tags
- When updating a page, update the `last_updated` frontmatter field
- When information conflicts between sources, keep both claims with citations — do not silently overwrite
- Source summary pages link to the raw source file: [[raw/path/to/source]]
- Prefer specific, descriptive page names (e.g., "spaced-repetition" not "learning-technique-1")
- Use lowercase-kebab-case for file names
- Tags use lowercase, no spaces (e.g., #cognitive-science, #tool)
```

### 6. Index Format
```markdown
## Index Format
index.md is organized by category with this format:

## Sources
- [[wiki/sources/page-name]] — One-line summary (YYYY-MM-DD)

## Entities
- [[wiki/entities/page-name]] — One-line description

## Concepts
- [[wiki/concepts/page-name]] — One-line description

## Synthesis
- [[wiki/synthesis/page-name]] — One-line description
```

### 7. Log Format
```markdown
## Log Format
Each entry in log.md:

## [YYYY-MM-DD] operation | Subject
Brief description of what was done. Pages created: [[page1]], [[page2]]. Pages updated: [[page3]].
```

## Build Order

The build order reflects true dependencies — each component requires the previous ones to function.

### Phase 1: Schema (CLAUDE.md)
**Build first because:** Everything else depends on conventions defined here. Without the schema, the LLM has no instructions for how to create pages, maintain the index, or format the log. The schema is the foundation that makes all subsequent operations consistent.

**Deliverable:** A complete CLAUDE.md at the vault root with all sections described in Schema Design above.

**Depends on:** Nothing. This is the root dependency.

### Phase 2: Directory Structure + Scaffolding
**Build second because:** The ingest workflow needs directories to write into and navigation files to update.

**Deliverables:**
- Create `raw/` with initial subdirectories (`articles/`, `notes/`, `assets/`)
- Create `wiki/` with subdirectories (`sources/`, `entities/`, `concepts/`, `synthesis/`, `queries/`)
- Create empty `index.md` with category headers
- Create empty `log.md` with a header

**Depends on:** Schema (Phase 1) — directory names and file formats come from the schema.

### Phase 3: Ingest Operation
**Build third because:** This is the primary value-creating operation. Until you can ingest sources, the wiki has no content. Ingest exercises all the core mechanics: reading sources, creating pages, maintaining cross-references, updating the index, appending to the log.

**Deliverable:** A working ingest workflow. Test by ingesting one real source file and verifying:
- Source summary page created with correct frontmatter
- Entity/concept pages created or updated
- Wikilinks are bidirectional
- index.md updated
- log.md appended

**Depends on:** Schema (Phase 1), directory structure (Phase 2).

### Phase 4: Query Operation
**Build fourth because:** Once the wiki has content from ingest, you need to be able to query it. Query is simpler than ingest (it reads the wiki rather than modifying it heavily), but it depends on having a populated index to navigate.

**Deliverable:** A working query workflow. Test by asking a question that requires reading multiple wiki pages and verifying the answer cites sources with wikilinks.

**Depends on:** Schema (Phase 1), directory structure (Phase 2), at least one completed ingest (Phase 3) to have content to query.

### Phase 5: Lint Operation
**Build fifth because:** Lint is a maintenance operation — it only becomes useful once the wiki has enough content to develop inconsistencies. It's the least urgent of the three operations but the most important for long-term wiki health.

**Deliverable:** A working lint workflow. Test by intentionally introducing an orphan page, a dead link, and a missing index entry, then running lint to verify it catches them.

**Depends on:** Schema (Phase 1), directory structure (Phase 2), sufficient wiki content from multiple ingests (Phase 3).

### Phase 6: Iteration and Schema Refinement
**Build last because:** After using the system for a few ingest/query/lint cycles, patterns will emerge — page types that need different templates, workflows that need adjustment, editorial rules that need refinement. The schema co-evolves with usage.

**Deliverable:** Updated CLAUDE.md reflecting lessons learned from real usage. This is ongoing, not a one-time phase.

**Depends on:** All previous phases, plus actual usage experience.

### Dependency Graph
```
Phase 1: Schema (CLAUDE.md)
    |
    v
Phase 2: Directory Structure + Scaffolding (index.md, log.md, raw/, wiki/)
    |
    v
Phase 3: Ingest Operation
    |
    v
Phase 4: Query Operation
    |
    v
Phase 5: Lint Operation
    |
    v
Phase 6: Schema Refinement (ongoing)
```

The critical path is Phases 1-3. Once ingest works, the wiki is usable. Query and lint add value but aren't blockers for initial use — you can browse the wiki directly in Obsidian even without a formal query workflow.
