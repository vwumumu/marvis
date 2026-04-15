# Research Summary
**Project:** Marvis — LLM-powered personal knowledge base wiki
**Date:** 2026-04-15

---

## 1. Recommended Stack

**Runtime:** Node.js 22 LTS with TypeScript 6, ESM-only (`"type": "module"`). Run via `tsx` (no build step needed).

**Core libraries (Phase 1 — lean start):**
- **gray-matter** — frontmatter extraction (read path)
- **yaml v2** — YAML serialization (write path)
- **fast-glob** — file discovery
- **zod** — schema/frontmatter validation
- **simple-git** — programmatic git for commit-on-ingest

**Add later when needed:**
- **remark/unified ecosystem** — when AST-level markdown manipulation is required (wikilink extraction, section rewriting)
- **MiniSearch** — when wiki exceeds ~50 pages and grep/index scanning slows down
- **qmd** — when wiki exceeds ~200 pages and needs hybrid search with LLM re-ranking

**Dev tools:** tsx (TS execution), vitest (testing), Biome (lint + format).

**Key constraint:** The remark/unified ecosystem is ESM-only since v13+. All new dependencies support ESM. CJS libraries (gray-matter, simple-git) work via Node's interop.

**Explicitly rejected:** Vector databases, LLM orchestration frameworks (LangChain etc.), databases (SQLite etc.), web frameworks, embedding models, heavy build tooling. The LLM is Claude Code itself — it is the caller, not a library dependency.

---

## 2. Table Stakes Features

Eight features required for minimum viability:

| # | Feature | Complexity | Notes |
|---|---------|-----------|-------|
| 1 | Source ingestion with summary generation | Low | Single LLM call + file writes. The core interaction. |
| 2 | Cross-referencing via `[[wikilinks]]` | Medium | Risk of over/under-linking. Needs prompt iteration. |
| 3 | Search/navigation via index.md | Low | Sufficient at target scale (~100 sources). |
| 4 | Consistent page structure | Low | Schema + templates in CLAUDE.md. |
| 5 | Source preservation (immutability) | Low | Directory convention, no code needed. |
| 6 | Operation logging | Low | Append-only log.md. |
| 7 | Readable output in Obsidian | Low | Obsidian-compatible markdown with YAML frontmatter. |
| 8 | Version history | Low | Git. Already decided. |

**Critical path for MVP:** 4 -> 5 -> 1 -> 2 -> 3 -> 6

---

## 3. Key Differentiators

What separates Marvis from NotebookLM, RAG systems, and manual note-taking:

| # | Differentiator | Value | Complexity |
|---|---------------|-------|-----------|
| D1 | **Incremental integration** — new sources update existing pages across the wiki, not just index | The entire value proposition. Without this, it is just a summarizer. | **High** (1-2 weeks) |
| D2 | **Compounding queries** — good answers become permanent wiki pages with links and citations | Knowledge compounds instead of vanishing into chat history. | Medium (2-3 days) |
| D3 | **Contradiction/staleness detection** (lint) | No consumer KB does this. Unique to the persistent-wiki pattern. | **High** (1-2 weeks) |
| D4 | **Zero human writing burden** | User curates sources and reviews output. LLM handles all writing. | Ongoing quality problem |
| D5 | **Schema co-evolution** via CLAUDE.md | Wiki structure adapts to the domain over time. | Low-Medium |
| D6 | **Multi-format output** (Marp slides, Dataview tables, graph view) | Structured frontmatter enables programmatic queries. | Low-Medium |
| D7 | **Transparent knowledge graph** — plain markdown, no opaque database | Human-auditable at every level. Portable across tools and LLMs. | Architectural choice |

**Tier 1 (must nail):** D1 — incremental integration defines the product.
**Tier 2 (strong):** D2 + D3 — make the wiki compound over time.
**Tier 3 (polish):** D5 + D6 — nice but not why someone chooses this.

**Anti-features (deliberately not building):** Embedding-based RAG, custom web UI, multi-user collaboration, automated/scheduled ingestion, chat UI (Claude Code already is the interface), fine-tuning, plugin system, permission models.

---

## 4. Architecture Summary

### Components

1. **Raw Source Store** (`raw/`) — Immutable source documents. User writes, LLM reads. Never modified by the system.
2. **Wiki Layer** (`wiki/`) — LLM-maintained interlinked markdown pages. Five page types: source summaries, entities, concepts, synthesis, query-derived.
3. **Index** (`index.md`) — Categorized catalog of all wiki pages. The LLM's primary navigation tool. Eliminates need for RAG at moderate scale.
4. **Log** (`log.md`) — Chronological append-only operation record. Parseable format with `## [YYYY-MM-DD] operation | subject` entries.
5. **Schema** (`CLAUDE.md`) — Conventions, page templates, workflows, editorial rules. The single most important file for wiki quality.
6. **LLM Agent** (Claude Code) — The engine. Reads schema/sources/index/log/wiki. Writes to wiki/index/log. Never writes to raw sources.

### Data Flows

- **Ingest:** Read source -> read schema -> read index -> create/update wiki pages (may touch 10-15+ files) -> update index -> append log
- **Query:** Read index -> read relevant wiki pages (not raw sources) -> synthesize answer -> optionally persist as wiki page -> append log
- **Lint:** Read index -> systematic checks (orphans, dead links, stale claims, missing pages, inconsistencies, frontmatter compliance) -> report/fix -> append log

### Directory Structure

```
marvis/                          # Obsidian vault root / git repo root
+-- CLAUDE.md                    # Schema
+-- index.md                     # Wiki index
+-- log.md                       # Operations log
+-- raw/                         # Immutable sources
|   +-- articles/, papers/, notes/, books/, assets/
+-- wiki/                        # LLM-maintained pages
|   +-- sources/, entities/, concepts/, synthesis/, queries/
+-- .planning/                   # Project planning (excluded from Obsidian)
+-- .obsidian/                   # Obsidian config
+-- .git/                        # Version history
```

---

## 5. Critical Risks

### Severity: Critical

| # | Pitfall | Core Risk | Key Mitigation |
|---|---------|-----------|----------------|
| P1 | **Hallucinated cross-references** | Fabricated links/claims get laundered into the wiki as "compiled knowledge" | Mandatory source citations on every claim; lint checks all wikilinks resolve and cited sources exist |
| P2 | **Silent inconsistency** | LLM updates one page but misses related pages, creating contradictions | "Ripple check" step in ingest; cross-page consistency lint |
| P3 | **Context window overflow** | Long sources + many wiki pages exceed effective context; LLM silently degrades | Multi-step ingest (summary -> identify pages -> update each individually); never hold full wiki in context |
| P4 | **Index drift** | Index falls out of sync; all subsequent operations work from a wrong map | Mandatory index update on every ingest; lint can rebuild index from scratch |
| P5 | **Wikilink format fragility** | Links that look valid but don't resolve in Obsidian (case sensitivity, duplicate names) | Strict kebab-case naming; avoid heading links; lint verifies resolution |

### Severity: Medium

| # | Pitfall | Key Mitigation |
|---|---------|----------------|
| P6 | Over-engineered schema | Start minimal; add rules only when needed; every rule must be lint-enforceable |
| P7 | Orphan pages / dead zones | Lint reports zero-inbound pages and frequently-mentioned entities without pages |
| P8 | Unbounded log growth | Parseable format; LLM reads only last N entries (`tail`); periodic rotation |
| P9 | Query quality degradation at scale | Categorized index from day one; plan hierarchical index at ~150 pages; add qmd at ~200 |
| P10 | Frontmatter conflicts with Obsidian plugins | Use Obsidian-standard fields; prefix custom fields (`wiki_*`); test with Dataview early |
| P11 | Ingest without review (garbage in, garbage out) | One source at a time; review before commit; `reviewed: false` frontmatter field |

### Severity: Low

P12 (git history bloat), P13 (graph view unreadable), P14 (source format variability), P15 (session amnesia — mitigate by updating CLAUDE.md after every convention change).

---

## 6. Build Order Recommendation

```
Phase 1: Schema (CLAUDE.md)           -- Foundation. All else depends on conventions defined here.
    |
Phase 2: Directory Structure           -- Create raw/, wiki/, index.md, log.md scaffolding.
    |
Phase 3: Ingest Operation              -- Primary value creation. Test with one real source.
    |
Phase 4: Query Operation               -- Read wiki to answer questions. Depends on populated content.
    |
Phase 5: Lint Operation                -- Maintenance. Useful once wiki has enough content to drift.
    |
Phase 6: Schema Refinement (ongoing)   -- Co-evolve CLAUDE.md based on real usage patterns.
```

**Critical path is Phases 1-3.** Once ingest works, the wiki is usable — users can browse directly in Obsidian even without formal query/lint workflows.

**Effort estimates:**
- Phases 1-2: 1-2 days (schema design + scaffolding)
- Phase 3: 1-2 days for basic ingest; 1-2 weeks to nail incremental integration (D1)
- Phase 4: 1-2 days
- Phase 5: 1-2 weeks (contradiction detection is hard)
- Phase 6: Ongoing

---

## 7. Open Questions

1. **Ingest review model** — Should the LLM present summaries for approval before writing, or write and let the user review in Obsidian? Affects workflow speed vs. error prevention (P11).

2. **Commit granularity** — One git commit per ingest operation, or one commit per session? Affects history usefulness vs. noise (P12).

3. **Source size limit** — What is the maximum source document size before chunking is required? Suggested threshold: ~15,000 words (P3).

4. **Index scaling strategy** — At what page count should the system switch from flat index to hierarchical sub-indexes, and when to introduce search tooling? Suggested: hierarchical at ~150 pages, qmd at ~200 (P9).

5. **Schema complexity starting point** — How many frontmatter fields and page types to start with? Research recommends minimal (title, sources, tags, type) and adding only when lint can enforce it (P6).

6. **Contradiction handling** — When sources conflict, should the wiki keep both claims with citations (recommended), or flag for human resolution? This affects both ingest workflow and lint behavior.

7. **Log rotation strategy** — At what size should log.md be archived? Suggested: ~500 lines or quarterly rotation (P8).

8. **Cross-platform compatibility** — Will the vault be used on both macOS and Linux? Affects wikilink case sensitivity requirements (P5).
