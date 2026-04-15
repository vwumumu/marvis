<!-- GSD:project-start source:PROJECT.md -->
## Project

**Marvis — LLM Wiki**

A personal knowledge base system powered by LLMs. Instead of traditional RAG (retrieve-and-generate on every query), the LLM incrementally builds and maintains a persistent, interlinked wiki of markdown files. When new sources are added, the LLM reads, extracts, and integrates knowledge into existing wiki pages — updating entities, revising summaries, flagging contradictions, and strengthening the evolving synthesis. The wiki is a compounding artifact that gets richer with every source ingested and every question asked.

**Core Value:** **The wiki is a persistent, compounding knowledge artifact** — cross-references are pre-built, contradictions pre-flagged, and synthesis reflects everything ingested. Knowledge is compiled once and kept current, not re-derived on every query.

### Constraints

- **Format**: Obsidian-compatible markdown with YAML frontmatter and `[[wikilinks]]`
- **Storage**: Local filesystem, git-tracked
- **Interface**: Claude Code as the LLM interface, Obsidian as the browsing interface
- **Sources**: Text-based (markdown, text files); images handled separately via Obsidian attachments
- **Scale**: Designed for moderate scale (~100 sources, ~hundreds of pages); index-based navigation, no vector DB
<!-- GSD:project-end -->

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

**Critical rule:** Files in `raw/` are immutable. NEVER modify, rename, or delete files in `raw/`. The user controls this directory. The LLM reads from it during ingest but never writes to it.

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

~~~yaml
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
~~~

~~~markdown
# As We May Think

## Key Takeaways
- Bulleted list of 3-7 main points from the source
- Each takeaway cites the source: (source: [[as-we-may-think-source|As We May Think]])

## Detailed Notes
Longer prose summarizing the source content. Every claim must trace back to the source document.

## Related Pages
- [[vannevar-bush|Vannevar Bush]] — author of the article
- [[memex|Memex]] — key concept introduced in the article
~~~

### Entity Template

File location: `wiki/entities/<kebab-case-name>.md`

~~~yaml
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
~~~

~~~markdown
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
~~~

### Concept Template

File location: `wiki/concepts/<kebab-case-name>.md`

~~~yaml
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
~~~

~~~markdown
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
~~~

### Synthesis Template

File location: `wiki/synthesis/<kebab-case-title>.md`

~~~yaml
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
~~~

~~~markdown
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
~~~

### Query Template

File location: `wiki/queries/<kebab-case-question>.md`

~~~yaml
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
~~~

~~~markdown
# How did the Memex influence modern knowledge tools?

## Answer
Multi-paragraph answer synthesized from wiki pages. Every claim includes a wikilink citation to the wiki page it came from.

## Sources Cited
- [[memex|Memex]] — definition and original proposal
- [[vannevar-bush|Vannevar Bush]] — author context

## Related Pages
- [[knowledge-compounding|Knowledge Compounding]] — related concept
~~~

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
~~~markdown
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
~~~

### Index Rules
- Update index.md on every ingest and every query-filing operation
- The LLM reads index.md first during query and lint operations to navigate the wiki (this is the primary navigation tool)
- At ~150 pages, consider splitting into sub-indexes linked from the main index
- HTML comments under each category header describe what goes there

## Log Format

`log.md` is append-only. Each entry starts with a parseable header.

### Entry Format
~~~markdown
## [YYYY-MM-DD] operation | Subject
Brief description. Pages created: [[page1]], [[page2]]. Pages updated: [[page3]].
~~~

### Log Rules
- **Never read the entire log.** Use `tail -50 log.md` to see recent entries.
- Operation types: `create`, `ingest`, `query`, `lint`, `update`
- Keep entries concise: 2-4 sentences
- Include wikilinks to pages created or updated
- At ~500 entries, archive old entries to `log-archive/YYYY.md`

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|---|---|---|---|
| **Node.js** | 22 LTS (or 24+) | Runtime | LTS stability; native ESM support; Claude Code runs in Node |
| **TypeScript** | ^6.0 | Type safety | Catches schema/frontmatter shape errors at compile time; essential for a system that manipulates structured markdown |
| **unified** | ^11.0 | Markdown processing pipeline | The standard AST-based text processing framework in Node.js; remark (markdown), rehype (HTML), and retext (prose) all build on it |
| **remark** | ^15.0 | Markdown parser/serializer | Built on unified; parses markdown to mdast (markdown AST) and back; the ecosystem standard |
| **remark-parse** | ^11.0 | Markdown -> AST | Core parser plugin for unified |
| **remark-stringify** | ^11.0 | AST -> Markdown | Core serializer plugin; round-trips markdown faithfully |
| **remark-frontmatter** | ^5.0 | YAML frontmatter in AST | Teaches remark to parse/preserve YAML frontmatter blocks; essential for Obsidian compatibility |
| **gray-matter** | ^4.0 | Frontmatter extraction | Fast, reliable extraction of YAML frontmatter to JS objects; simpler than remark when you just need the data. CJS but works fine via `tsx` or `import()` in ESM. Battle-tested (used by Metalsmith, Assemble, Hugo tooling) |
| **yaml** | ^2.8 | YAML parsing/serialization | When you need fine-grained YAML control (e.g., writing frontmatter back with specific formatting). Newer and more spec-compliant than `js-yaml` |
| **remark-wiki-link** | ^2.0 | Wikilink parsing | Parses `[[wikilinks]]` in markdown AST; necessary for Obsidian-style cross-references. Last updated Oct 2023 but stable/feature-complete |
| **MiniSearch** | ^7.2 | Full-text search | Tiny (8KB), fast, in-memory full-text search with BM25 scoring. Perfect for searching an index of ~hundreds of wiki pages. No external dependencies. Actively maintained (updated Sep 2025) |
### Supporting Libraries
| Library | Version | Purpose | Why Recommended |
|---|---|---|---|
| **simple-git** | ^3.36 | Git operations | Programmatic git from Node.js; needed for commit-on-ingest, diff tracking, history queries. Mature, well-maintained, promise-based API |
| **fast-glob** | ^3.3 | File discovery | Find all `.md` files in source/wiki directories. Faster than Node's native `fs.glob`; supports gitignore patterns |
| **zod** | ^4.3 | Schema validation | Validate frontmatter shapes, CLI input, config files. The standard runtime validation library in TypeScript |
| **vfile** | ^6.0 | Virtual file representation | Part of the unified ecosystem; carries file path, content, and messages (warnings/errors) through processing pipelines |
| **unist-util-visit** | ^5.1 | AST traversal | Walk mdast trees to find/modify nodes (e.g., extract all wikilinks from a page, find all headings) |
| **mdast-util-to-string** | ^4.0 | AST text extraction | Extract plain text from AST nodes; useful for building search indexes from page content |
| **remark-gfm** | ^4.0 | GitHub Flavored Markdown | Tables, strikethrough, task lists, autolinks. Obsidian supports GFM, so the parser should too |
| **commander** | ^14.0 | CLI framework | If building a `marvis` CLI tool for ingest/query/lint commands. Mature, zero-dependency, TypeScript support |
| **date-fns** | ^4.x | Date handling | For log timestamps, frontmatter dates. Tree-shakeable, no mutable globals (unlike Moment/Day.js) |
### Development Tools
| Tool | Version | Purpose | Why Recommended |
|---|---|---|---|
| **tsx** | ^4.21 | TypeScript execution | Run `.ts` files directly without a build step; uses esbuild under the hood. Essential for Claude Code workflow (edit-and-run, no compile step) |
| **vitest** | ^4.1 | Testing | Fast, TypeScript-native test runner. Compatible with Jest API but faster. Needed for testing ingest/lint logic |
| **Biome** | ^1.x | Linting + formatting | Single tool replaces ESLint + Prettier. Fast (Rust-based), zero-config for TypeScript. Lower maintenance than ESLint |
| **qmd** | latest | Markdown search engine | Local search over markdown files with BM25/vector hybrid search and LLM re-ranking. Has CLI and MCP server modes. Install when wiki outgrows index-based navigation (~200+ pages) |
| **Obsidian Dataview** | plugin | Dynamic wiki queries | Obsidian plugin that queries YAML frontmatter across pages; generates tables/lists. Validates that the LLM's frontmatter is well-structured |
| **Obsidian Graph View** | built-in | Wiki visualization | Shows wikilink topology; identifies orphan pages and hub pages. Essential for lint operations |
## Architecture Notes
### ESM vs CJS
- Use `"type": "module"` in `package.json`
- Use `tsx` for development (handles ESM natively)
- `gray-matter` is CJS but works fine when imported from ESM via Node's interop
- `simple-git` is CJS but also works via ESM interop
- All newer libraries (zod, yaml, commander 14+) support ESM
### Why gray-matter + remark-frontmatter (both)
- **gray-matter**: Use when you need to quickly extract frontmatter data from a file (e.g., reading tags, dates, source counts during index building). Fast, simple API: `matter(fileContent)` returns `{ data, content }`.
- **remark-frontmatter**: Use when you need to process the full markdown AST (e.g., extracting wikilinks, rewriting headings). Without this plugin, remark would try to parse the YAML block as markdown content.
### Why MiniSearch over alternatives
| Library | Verdict | Reason |
|---|---|---|
| **MiniSearch** | Recommended | Tiny, fast, BM25 scoring, actively maintained, trivial API. Perfect for ~hundreds of documents |
| FlexSearch | Pass | More features but larger, more complex API, documentation is sparse |
| Fuse.js | Pass | Fuzzy string matching, not full-text search. No relevance ranking for documents |
| Lunr | Pass | Unmaintained since 2023; MiniSearch is the modern replacement |
| qmd | Later | Overkill at launch scale; add when wiki exceeds ~200 pages |
### Markdown Round-Trip Fidelity
- remark normalizes some whitespace (e.g., inconsistent list indentation)
- For files the LLM wrote, this is fine (it wrote clean markdown)
- For raw source files, this is irrelevant (they are immutable/read-only)
- Configure `remark-stringify` options: `{ bullet: '-', emphasis: '*', strong: '**', listItemIndent: 'one' }` to match Obsidian defaults
## Alternatives Considered
| Alternative | Category | Why Not |
|---|---|---|
| **Python + python-frontmatter** | Runtime | Python has good markdown libraries but Claude Code's native environment is Node.js/TypeScript. Mixing runtimes adds complexity for no clear benefit at this scale |
| **mdx** | Markdown format | MDX adds JSX to markdown; irrelevant for a wiki browsed in Obsidian |
| **Markdoc (Stripe)** | Markdown processing | Designed for documentation sites with custom tags; overkill for wiki markdown. No wikilink support |
| **markdown-it** | Markdown parser | Good parser but no AST manipulation story. remark/unified has a complete ecosystem for transforms |
| **js-yaml** | YAML parsing | Older, less spec-compliant than `yaml` v2. Still works but `yaml` is the modern choice |
| **Ink (React for CLI)** | CLI framework | Marvis CLI is invoked by Claude Code, not interactively by humans. commander is simpler and sufficient |
| **Langchain / LlamaIndex** | LLM orchestration | Massive abstractions for a system where the "LLM" is Claude Code itself running shell commands. The LLM is the caller, not a library dependency |
| **SQLite / better-sqlite3** | Storage | Tempting for structured queries, but the wiki is git-tracked markdown. Adding a database creates a sync problem. The index file + MiniSearch covers search needs |
| **Elasticsearch / Typesense** | Search | Server-based search engines; absurd overhead for ~hundreds of local markdown files |
| **Chokidar** | File watching | File watching is unnecessary — Marvis operations are triggered explicitly by the user via Claude Code, not by file system events |
## What NOT to Use
### Vector databases (Pinecone, Chroma, Weaviate, Qdrant)
### LLM orchestration frameworks (LangChain, LlamaIndex, CrewAI, AutoGen)
### Databases (SQLite, PostgreSQL, MongoDB)
### Heavy build tooling (Webpack, Vite, Rollup, esbuild bundling)
### React / web frameworks
### Embedding models / sentence-transformers
## Minimal Starting Dependencies
## Sources
- unified ecosystem: https://unifiedjs.com/
- remark: https://github.com/remarkjs/remark
- remark-wiki-link: https://github.com/landakram/remark-wiki-link
- gray-matter: https://github.com/jonschlinkert/gray-matter
- MiniSearch: https://github.com/lucaong/minisearch
- simple-git: https://github.com/steveukx/git-js
- qmd: https://github.com/tobi/qmd
- Obsidian developer docs: https://docs.obsidian.md/
- yaml: https://github.com/eemeli/yaml
- Biome: https://biomejs.dev/
- tsx: https://github.com/privatenumber/tsx
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
