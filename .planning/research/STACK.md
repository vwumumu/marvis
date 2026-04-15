# Stack Research

**Domain:** LLM-powered personal knowledge base wiki (Marvis)
**Researched:** 2026-04-15
**Confidence:** High — all libraries are mature, actively maintained, and well-documented. The remark/unified ecosystem is the clear standard for programmatic markdown in Node.js.

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

The remark/unified ecosystem is **ESM-only** since v13+ (unified) and v15+ (remark). This is the most important ecosystem constraint:

- Use `"type": "module"` in `package.json`
- Use `tsx` for development (handles ESM natively)
- `gray-matter` is CJS but works fine when imported from ESM via Node's interop
- `simple-git` is CJS but also works via ESM interop
- All newer libraries (zod, yaml, commander 14+) support ESM

### Why gray-matter + remark-frontmatter (both)

These solve different problems:
- **gray-matter**: Use when you need to quickly extract frontmatter data from a file (e.g., reading tags, dates, source counts during index building). Fast, simple API: `matter(fileContent)` returns `{ data, content }`.
- **remark-frontmatter**: Use when you need to process the full markdown AST (e.g., extracting wikilinks, rewriting headings). Without this plugin, remark would try to parse the YAML block as markdown content.

In practice, most Marvis operations will use `gray-matter` for reads and `remark` + `remark-frontmatter` for writes/transforms.

### Why MiniSearch over alternatives

| Library | Verdict | Reason |
|---|---|---|
| **MiniSearch** | Recommended | Tiny, fast, BM25 scoring, actively maintained, trivial API. Perfect for ~hundreds of documents |
| FlexSearch | Pass | More features but larger, more complex API, documentation is sparse |
| Fuse.js | Pass | Fuzzy string matching, not full-text search. No relevance ranking for documents |
| Lunr | Pass | Unmaintained since 2023; MiniSearch is the modern replacement |
| qmd | Later | Overkill at launch scale; add when wiki exceeds ~200 pages |

### Markdown Round-Trip Fidelity

A critical requirement: the system must read markdown, modify it, and write it back without corrupting formatting. The `remark-parse` + `remark-stringify` pipeline handles this well, but be aware:
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
The project explicitly scopes to ~100 sources and ~hundreds of wiki pages. Index-based navigation (index.md + MiniSearch) is sufficient and far simpler. Vector search adds infrastructure complexity (embedding model, vector store, similarity thresholds) that provides no benefit at this scale. If the wiki grows to thousands of pages, revisit — but even then, qmd (local, on-device) is preferable to a hosted vector DB.

### LLM orchestration frameworks (LangChain, LlamaIndex, CrewAI, AutoGen)
Claude Code IS the LLM interface. The system does not call an LLM API — it is invoked BY an LLM. These frameworks add layers of abstraction around API calls that are irrelevant when the LLM is already the runtime environment. Using them would mean wrapping Claude Code inside another LLM orchestration layer, which is architecturally backwards.

### Databases (SQLite, PostgreSQL, MongoDB)
The wiki is a directory of markdown files tracked by git. Introducing a database creates a synchronization problem (DB state vs. file state) and breaks the "it's just a git repo" property that makes Obsidian browsing and version history work. All structured queries should go through frontmatter (via gray-matter + zod) and search (via MiniSearch).

### Heavy build tooling (Webpack, Vite, Rollup, esbuild bundling)
This is a CLI tool / library, not a web application. `tsx` runs TypeScript directly. If distribution becomes important later, `tsup` (esbuild-based) can produce a single bundle, but that is a post-v1 concern.

### React / web frameworks
There is no web UI. Obsidian is the browsing interface. Building a custom frontend would violate the project's explicit out-of-scope constraint.

### Embedding models / sentence-transformers
Same reasoning as vector databases. The wiki's navigation is index-based. Adding an embedding pipeline for ~hundreds of pages is complexity without proportional benefit.

## Minimal Starting Dependencies

For phase 1 (ingest + index), the actual `package.json` dependencies are lean:

```
dependencies:
  gray-matter        ^4.0     # frontmatter parsing
  yaml               ^2.8     # YAML serialization (writing frontmatter)
  fast-glob          ^3.3     # file discovery
  zod                ^4.3     # schema validation
  simple-git         ^3.36    # git operations (optional: commit-on-ingest)

devDependencies:
  typescript         ^6.0     # type checking
  tsx                ^4.21    # run TS directly
  vitest             ^4.1     # testing
  @types/node        ^22      # Node.js type defs
```

Add remark/unified ecosystem only when you need AST-level markdown manipulation (e.g., extracting wikilinks from existing pages, rewriting sections programmatically). For the initial ingest workflow — where the LLM reads a source and writes new markdown — string-level operations with gray-matter may be all you need.

Add MiniSearch when the wiki has enough pages that grep/index scanning becomes slow (~50+ pages).

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
