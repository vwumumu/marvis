---
phase: 1
plan: 1
title: "CLAUDE.md Wiki Schema"
wave: 1
depends_on: []
files_modified:
  - CLAUDE.md
requirements_addressed:
  - SCHM-01
  - SCHM-02
  - SCHM-03
  - SCHM-04
  - SRCM-02
  - INDX-03
  - LOGM-02
autonomous: true
estimated_context_cost: "medium"
---

## Objective

Replace the GSD boilerplate sections in CLAUDE.md with a complete wiki schema that a fresh Claude Code session can read and immediately understand how to operate on the wiki. The schema defines: directory conventions, page templates with YAML frontmatter, naming and linking rules, operation workflows (ingest, query, lint), editorial rules, index format, and log format. This is the foundational artifact — all subsequent phases reference it.

## must_haves

- [ ] CLAUDE.md contains a Quick Start section that orients a fresh session in 30 seconds
- [ ] CLAUDE.md defines the directory structure (raw/ immutable, wiki/ LLM-maintained)
- [ ] CLAUDE.md contains 5 page type templates (source, entity, concept, synthesis, query) with YAML frontmatter
- [ ] Frontmatter schema includes mandatory fields: title, type, sources, tags, created, updated
- [ ] Wikilink format documented: `[[kebab-case-filename|Display Name]]` with Obsidian-compatible shortest-path resolution
- [ ] Ingest, Query, and Lint operation workflows are defined step-by-step
- [ ] Editorial rules include: mandatory source citations, contradiction handling, raw/ immutability
- [ ] Index format documented with entry format and category structure
- [ ] Log format documented with `## [YYYY-MM-DD] operation | subject` pattern
- [ ] Schema instructs LLM to use `tail -50 log.md` instead of reading entire log

## Tasks

<task id="1">
<title>Write complete wiki schema into CLAUDE.md</title>
<type>execute</type>
<read_first>
- CLAUDE.md (understand current GSD boilerplate structure and markers)
- .planning/phases/01-schema-scaffolding/01-CONTEXT.md (implementation decisions D-01 through D-15)
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (schema design details, frontmatter spec, template designs)
- .planning/research/ARCHITECTURE.md (data flows, schema design sections)
- .planning/research/PITFALLS.md (P1 citation rules, P3 context window, P5 wikilink format, P6 minimal schema, P8 log access)
</read_first>
<action>
Replace the content of CLAUDE.md while preserving GSD markers. Insert the wiki schema sections between the GSD `project-end` marker and the GSD `stack-start` marker. The complete wiki schema content to insert is:

```markdown
---

# Marvis Wiki Schema

## Quick Start

This is a personal knowledge base wiki maintained by Claude Code. The wiki compiles knowledge from source documents into interlinked markdown pages — like a personal Tolkien Gateway where cross-references are pre-built and synthesis reflects everything ingested.

**How it works:**
1. The user drops source documents into `raw/`
2. Claude Code ingests them: reading the source, creating/updating wiki pages, updating the index, logging the operation
3. The wiki grows as a compounding artifact — each new source enriches existing pages
4. The user browses in Obsidian; Claude Code answers questions from wiki content

**Before any operation**, read this schema fully. It contains all conventions you need.

## Directory Conventions

```
marvis/                     # Vault root
├── CLAUDE.md               # This file — wiki schema and conventions
├── index.md                # Wiki index — categorized catalog of all pages
├── log.md                  # Operations log — chronological, append-only
├── raw/                    # Immutable source documents (NEVER modify)
│   ├── articles/           # Web articles
│   ├── papers/             # Academic papers
│   ├── notes/              # Personal notes
│   ├── books/              # Book chapters/excerpts
│   └── assets/             # Images referenced by sources
├── wiki/                   # LLM-maintained wiki pages
│   ├── sources/            # Source summary pages (one per ingested source)
│   ├── entities/           # Entity pages (people, orgs, tools, places)
│   ├── concepts/           # Concept pages (ideas, frameworks, themes)
│   ├── synthesis/          # Cross-cutting analyses and comparisons
│   └── queries/            # Persisted answers to user questions
├── .obsidian/              # Obsidian config
├── .planning/              # Project planning (excluded from Obsidian)
└── .git/                   # Version history
```

**Critical rule:** Files in `raw/` are immutable. Never modify, rename, or delete files in `raw/`. The user controls this directory. The LLM reads from it during ingest but never writes to it.

## Page Templates

All wiki pages use YAML frontmatter and Obsidian-compatible `[[wikilinks]]`.

### Mandatory Frontmatter (all page types)

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Human-readable page title |
| `type` | enum | `source`, `entity`, `concept`, `synthesis`, `query` |
| `sources` | list | Raw source paths that contributed to this page |
| `tags` | list | Lowercase kebab-case tags (no `#` prefix in frontmatter) |
| `created` | date | ISO 8601 date (YYYY-MM-DD) |
| `updated` | date | ISO 8601 date (YYYY-MM-DD) |

### Source Summary Template

File location: `wiki/sources/<kebab-case-title>.md`

```yaml
---
title: "As We May Think"
type: source
sources:
  - raw/articles/as-we-may-think.md
source_path: raw/articles/as-we-may-think.md
tags:
  - information-retrieval
  - memex
created: 2026-04-15
updated: 2026-04-15
---
```

```markdown
# As We May Think

## Key Takeaways
- Bulleted list of 3-7 main points from the source
- Each takeaway cites the source: (source: [[as-we-may-think-source|As We May Think]])

## Detailed Notes
Longer prose summarizing the source content. Every claim must trace back to the source document.

## Related Pages
- [[vannevar-bush|Vannevar Bush]] — author of the article
- [[memex|Memex]] — key concept introduced in the article
```

### Entity Template

File location: `wiki/entities/<kebab-case-name>.md`

```yaml
---
title: "Vannevar Bush"
type: entity
sources:
  - raw/articles/as-we-may-think.md
aliases:
  - V. Bush
category: person
tags:
  - scientist
  - information-retrieval
created: 2026-04-15
updated: 2026-04-15
---
```

```markdown
# Vannevar Bush

## Overview
Brief description of the entity (2-3 sentences). Cite the source for each claim.

## Key Details
- **Role:** Director of the Office of Scientific Research and Development (source: [[as-we-may-think-source|As We May Think]])
- Key factual details as a bulleted list, each with source citation

## Appearances in Sources
- [[as-we-may-think-source|As We May Think]] — authored the article proposing the Memex

## Related Entities
- [[memex|Memex]] — concept he proposed
```

### Concept Template

File location: `wiki/concepts/<kebab-case-name>.md`

```yaml
---
title: "Memex"
type: concept
sources:
  - raw/articles/as-we-may-think.md
related:
  - "[[knowledge-compounding|Knowledge Compounding]]"
tags:
  - information-retrieval
  - knowledge-management
created: 2026-04-15
updated: 2026-04-15
---
```

```markdown
# Memex

## Definition
One-paragraph definition of the concept. Cite the originating source.

## Key Points
- Bulleted list of important aspects, each citing its source
- Focus on what makes this concept distinct and important

## Sources
- [[as-we-may-think-source|As We May Think]] — original proposal of the Memex concept

## Related Concepts
- [[knowledge-compounding|Knowledge Compounding]] — modern realization of the Memex vision
```

### Synthesis Template

File location: `wiki/synthesis/<kebab-case-title>.md`

```yaml
---
title: "Evolution of Knowledge Management Tools"
type: synthesis
sources:
  - raw/articles/as-we-may-think.md
  - raw/articles/personal-knowledge-management.md
tags:
  - knowledge-management
  - historical-overview
created: 2026-04-15
updated: 2026-04-15
---
```

```markdown
# Evolution of Knowledge Management Tools

## Overview
Brief statement of what this synthesis covers and why it exists.

## Analysis
Multi-paragraph analysis drawing from multiple sources. Every claim cites its source wiki page using wikilinks.

## Sources
- [[as-we-may-think-source|As We May Think]] — historical foundation
- [[personal-knowledge-management-source|Personal Knowledge Management]] — modern approaches

## Related Pages
- [[vannevar-bush|Vannevar Bush]] — key figure in the history
- [[memex|Memex]] — foundational concept
```

### Query Template

File location: `wiki/queries/<kebab-case-question>.md`

```yaml
---
title: "How did the Memex influence modern knowledge tools?"
type: query
sources:
  - raw/articles/as-we-may-think.md
question: "How did the Memex influence modern knowledge tools?"
tags:
  - knowledge-management
  - memex
created: 2026-04-15
updated: 2026-04-15
---
```

```markdown
# How did the Memex influence modern knowledge tools?

## Answer
Multi-paragraph answer synthesized from wiki pages. Every claim includes a wikilink citation to the wiki page it came from.

## Sources Cited
- [[memex|Memex]] — definition and original proposal
- [[vannevar-bush|Vannevar Bush]] — author context

## Related Pages
- [[knowledge-compounding|Knowledge Compounding]] — related concept
```

## Naming & Linking Rules

### Filenames
- All wiki page filenames use **kebab-case**: `vannevar-bush.md`, `knowledge-compounding.md`
- No spaces, no uppercase, no special characters
- Descriptive names: `spaced-repetition.md` not `learning-technique-1.md`

### Wikilinks
- Format: `[[filename|Display Name]]` — e.g., `[[vannevar-bush|Vannevar Bush]]`
- Omit the `.md` extension (Obsidian convention)
- Use shortest-path resolution (filename only, no directory prefix) — Obsidian resolves by searching the vault
- Avoid heading links `[[page#heading]]` — headings change and break links
- Link meaningfully: link the first/most relevant mention, not every mention

### Tags
- Lowercase kebab-case: `knowledge-management`, `information-retrieval`
- No `#` prefix in YAML frontmatter (the `#` is only for inline tags)
- Let domain tags emerge organically from ingested content
- Page-type tags are implied by the `type` frontmatter field — do not duplicate as tags

## Operation Workflows

### Ingest Workflow

When the user asks to ingest a source:

1. **Read schema**: Read this file (CLAUDE.md) for conventions
2. **Read the source**: Read the raw source file completely
3. **Read index**: Read `index.md` to understand current wiki state and identify existing pages that may need updating
4. **Create source summary**: Create a new page in `wiki/sources/` following the Source Summary template
5. **Create/update entity pages**: For each key person, organization, tool, or place mentioned:
   - Check if an entity page already exists (search `wiki/entities/`)
   - If it exists: read the existing page, integrate new information, update the `sources` list and `updated` date
   - If new: create a page in `wiki/entities/` following the Entity template
6. **Create/update concept pages**: For each key idea, framework, or theme:
   - Check if a concept page already exists (search `wiki/concepts/`)
   - If it exists: read the existing page, integrate new information, update the `sources` list and `updated` date
   - If new: create a page in `wiki/concepts/` following the Concept template
7. **Ripple check**: Search the wiki for pages that reference updated entities/concepts. Verify consistency. If the new source contradicts existing content, note both claims with citations on the relevant page — never silently overwrite.
8. **Update index**: Add new page entries and update summaries for modified pages in `index.md`
9. **Append log**: Add an entry to `log.md` following the Log Format
10. **Git commit**: Commit all changes with message: `ingest: <source title> — created N pages, updated M pages`

**Critical:** Never hold the entire wiki in context at once. Process entity and concept pages one at a time. Read only what you need for each step.

### Query Workflow

When the user asks a question:

1. **Read schema**: Read this file (CLAUDE.md) for conventions
2. **Read index**: Read `index.md` to identify relevant pages
3. **Read relevant pages**: Read the specific wiki pages (not raw sources) that contain the answer
4. **Synthesize answer**: Produce an answer citing wiki pages with `[[wikilinks]]`
5. **Offer to persist**: If the answer is substantial, offer to save it as a page in `wiki/queries/` or `wiki/synthesis/`
6. **Append log**: Add an entry to `log.md`

**Critical:** Query answers come from wiki pages, not raw sources. The wiki is the compiled knowledge. Only consult raw sources if the user needs to verify a specific claim.

### Lint Workflow

When the user asks to lint/health-check the wiki:

1. **Read schema**: Read this file (CLAUDE.md) for conventions
2. **Read index**: Read `index.md` for the complete page inventory
3. **Check broken wikilinks**: Scan all wiki pages for `[[links]]` that don't resolve to existing files
4. **Check orphan pages**: Find pages with zero inbound wikilinks from other wiki pages
5. **Check missing pages**: Find entities/concepts mentioned frequently but lacking their own page
6. **Check index completeness**: Verify every wiki page is listed in `index.md` and no stale entries exist
7. **Check frontmatter**: Verify all pages have required frontmatter fields with correct types
8. **Check contradictions**: Compare claims across pages that reference the same entity/concept
9. **Report findings**: Present issues with specific fix suggestions
10. **Append log**: Add an entry to `log.md`

## Editorial Rules

1. **Mandatory source citations**: Every claim in a wiki page must cite its source. Use format: `(source: [[source-page|Source Title]])`. No unsourced assertions.
2. **Contradiction handling**: When sources disagree, keep both claims with citations. Never silently overwrite. Format: "Source A states X (source: [[a]]). However, Source B states Y (source: [[b]])."
3. **Raw source immutability**: Never modify files in `raw/`. Read only.
4. **Incremental integration**: When a new source adds information about an existing entity or concept, update the existing page — do not create a duplicate page.
5. **YAML quoting**: Quote frontmatter string values that contain colons, brackets, or other YAML special characters.
6. **Consistent types**: `sources` and `tags` are always YAML lists, never strings. Even for a single value, use list format.
7. **Update timestamps**: When modifying any page, update the `updated` frontmatter field to today's date.
8. **Obsidian plugin compatibility**: If Dataview plugin conflicts arise with custom frontmatter fields, prefix them with `wiki_` (e.g., `wiki_sources`). Default: no prefix.

## Index Format

`index.md` is organized by page type categories. Each entry has a wikilink and one-line summary.

### Category Structure
```markdown
## Sources
- [[source-filename|Source Title]] — One-line summary (YYYY-MM-DD)

## Entities
- [[entity-filename|Entity Name]] — One-line description

## Concepts
- [[concept-filename|Concept Name]] — One-line description

## Synthesis
- [[synthesis-filename|Synthesis Title]] — One-line description

## Queries
- [[query-filename|Question]] — One-line answer summary
```

### Index Rules
- Update index.md on every ingest and every query-filing operation
- The LLM reads index.md first during query and lint operations to navigate the wiki (this is the primary navigation tool)
- At ~150 pages, consider splitting into sub-indexes linked from the main index
- HTML comments under each category header describe what goes there

## Log Format

`log.md` is append-only. Each entry starts with a parseable header.

### Entry Format
```markdown
## [YYYY-MM-DD] operation | Subject
Brief description. Pages created: [[page1]], [[page2]]. Pages updated: [[page3]].
```

### Log Rules
- **Never read the entire log.** Use `tail -50 log.md` to see recent entries.
- Operation types: `create`, `ingest`, `query`, `lint`, `update`
- Keep entries concise: 2-4 sentences
- Include wikilinks to pages created or updated
- At ~500 entries, archive old entries to `log-archive/YYYY.md`
```

**Important implementation notes:**
1. The above content goes AFTER the `<!-- GSD:project-end -->` comment and BEFORE the `<!-- GSD:stack-start -->` comment
2. Keep all existing GSD marker sections intact (project, stack, conventions, architecture, skills, workflow, profile)
3. The triple-backtick code blocks in the templates section need to use indented code blocks or a different delimiter to avoid nesting issues — use 4-space indented code blocks for the YAML frontmatter examples inside the templates, or use `~~~yaml` fenced blocks instead of triple backticks

**Actual edit approach:** Insert the wiki schema content between the GSD project-end and stack-start markers. Use `~~~yaml` for inner fenced blocks to avoid triple-backtick nesting.
</action>
<acceptance_criteria>
- Grep for "Quick Start" in CLAUDE.md returns a match
- Grep for "Directory Conventions" in CLAUDE.md returns a match
- Grep for "Page Templates" in CLAUDE.md returns a match
- Grep for "Source Summary Template" in CLAUDE.md returns a match
- Grep for "Entity Template" in CLAUDE.md returns a match
- Grep for "Concept Template" in CLAUDE.md returns a match
- Grep for "Synthesis Template" in CLAUDE.md returns a match
- Grep for "Query Template" in CLAUDE.md returns a match
- Grep for "Naming & Linking Rules" in CLAUDE.md returns a match
- Grep for "Operation Workflows" in CLAUDE.md returns a match
- Grep for "Ingest Workflow" in CLAUDE.md returns a match
- Grep for "Query Workflow" in CLAUDE.md returns a match
- Grep for "Lint Workflow" in CLAUDE.md returns a match
- Grep for "Editorial Rules" in CLAUDE.md returns a match
- Grep for "Mandatory source citations" in CLAUDE.md returns a match
- Grep for "Index Format" in CLAUDE.md returns a match
- Grep for "Log Format" in CLAUDE.md returns a match
- Grep for "tail -50 log.md" in CLAUDE.md returns a match
- Grep for "NEVER modify" in CLAUDE.md returns a match (raw/ immutability)
- Grep for "kebab-case" in CLAUDE.md returns a match
- Grep for "wikilinks" in CLAUDE.md returns a match
- Grep for "type: source" in CLAUDE.md returns a match
- Grep for "type: entity" in CLAUDE.md returns a match
- Grep for "type: concept" in CLAUDE.md returns a match
- Grep for "type: synthesis" in CLAUDE.md returns a match
- Grep for "type: query" in CLAUDE.md returns a match
- Grep for "GSD:project-start" in CLAUDE.md returns a match (GSD markers preserved)
- Grep for "GSD:workflow-start" in CLAUDE.md returns a match (GSD markers preserved)
</acceptance_criteria>
</task>
