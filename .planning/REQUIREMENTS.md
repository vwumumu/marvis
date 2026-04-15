# Requirements: Marvis — LLM Wiki

**Defined:** 2026-04-15
**Core Value:** The wiki is a persistent, compounding knowledge artifact — cross-references are pre-built, contradictions pre-flagged, and synthesis reflects everything ingested.

## v1 Requirements

### Schema & Structure

- [ ] **SCHM-01**: CLAUDE.md schema defines wiki conventions, page templates, editorial rules, and operation workflows
- [ ] **SCHM-02**: Directory structure separates raw sources (immutable) and wiki pages (LLM-maintained)
- [ ] **SCHM-03**: All wiki pages have YAML frontmatter (title, type, sources, tags, created, updated)
- [ ] **SCHM-04**: Wiki pages use Obsidian-compatible `[[wikilinks]]` for cross-referencing

### Source Management

- [ ] **SRCM-01**: Raw sources directory exists with subdirectories by type (articles, papers, notes, books)
- [ ] **SRCM-02**: Raw sources are immutable — the system never modifies files in raw/
- [ ] **SRCM-03**: Assets directory for locally-downloaded images referenced by sources

### Ingest Operation

- [ ] **NGST-01**: User can trigger ingestion of a single source document
- [ ] **NGST-02**: Ingest creates a source summary page in wiki/sources/
- [ ] **NGST-03**: Ingest creates or updates entity pages in wiki/entities/ for key people, organizations, tools mentioned
- [ ] **NGST-04**: Ingest creates or updates concept pages in wiki/concepts/ for key ideas and themes
- [ ] **NGST-05**: Ingest updates existing wiki pages when new source adds relevant information (incremental integration — D1)
- [ ] **NGST-06**: Ingest updates index.md with new and modified pages
- [ ] **NGST-07**: Ingest appends operation entry to log.md
- [ ] **NGST-08**: Every claim in a wiki page cites its source(s) to prevent hallucination (P1 mitigation)

### Query Operation

- [ ] **QERY-01**: User can ask questions and get answers synthesized from wiki pages (not raw sources)
- [ ] **QERY-02**: Query answers include citations with `[[wikilinks]]` to source wiki pages
- [ ] **QERY-03**: Valuable query answers can be filed as new wiki pages in wiki/queries/ or wiki/synthesis/
- [ ] **QERY-04**: Query operation appends entry to log.md

### Lint Operation

- [ ] **LINT-01**: Lint checks for broken `[[wikilinks]]` (links that don't resolve to existing pages)
- [ ] **LINT-02**: Lint detects orphan pages (zero inbound links from other wiki pages)
- [ ] **LINT-03**: Lint identifies entities/concepts frequently mentioned but lacking their own page
- [ ] **LINT-04**: Lint checks index.md completeness (all wiki pages listed, no stale entries)
- [ ] **LINT-05**: Lint reports contradictions between pages citing different sources
- [ ] **LINT-06**: Lint appends operation entry to log.md

### Index & Navigation

- [ ] **INDX-01**: index.md catalogs every wiki page with link, one-line summary, and category
- [ ] **INDX-02**: index.md is organized by page type (sources, entities, concepts, synthesis, queries)
- [ ] **INDX-03**: index.md is the LLM's primary navigation tool for query and lint operations

### Logging

- [ ] **LOGM-01**: log.md is append-only with parseable format: `## [YYYY-MM-DD] operation | subject`
- [ ] **LOGM-02**: Every operation (ingest, query, lint) appends an entry with timestamp, operation type, and summary

### Version Control

- [ ] **GITM-01**: The entire wiki is a git repository
- [ ] **GITM-02**: Each ingest operation produces a git commit capturing all changes

## v2 Requirements

### Advanced Lint

- **LINT-10**: Lint detects stale claims superseded by newer sources
- **LINT-11**: Lint suggests new sources to investigate for data gaps

### Search Enhancement

- **SRCH-01**: Full-text search via MiniSearch when wiki exceeds ~50 pages
- **SRCH-02**: Hybrid search via qmd when wiki exceeds ~200 pages

### Multi-Format Output

- **MFMT-01**: Generate Marp slide decks from wiki content
- **MFMT-02**: Dataview-compatible frontmatter for dynamic tables in Obsidian

### Schema Evolution

- **SCHM-10**: Schema co-evolution — CLAUDE.md adapts conventions based on usage patterns

## Out of Scope

| Feature | Reason |
|---------|--------|
| Embedding-based RAG | Undermines core thesis; index.md sufficient at target scale |
| Custom web UI | Obsidian is the IDE; building UI is wasted effort |
| Multi-user collaboration | Scope explosion; this is a personal knowledge base |
| Automated/scheduled ingestion | Quality requires human curation; batch ingest = garbage in (P11) |
| Chat UI | Claude Code already is the chat interface |
| LLM fine-tuning | The wiki IS the memory; fine-tuning is redundant |
| Plugin/extension system | YAGNI; premature abstraction |
| Mobile app | Desktop-first; Obsidian Mobile exists if needed later |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| SCHM-01 | Phase 1 | Pending |
| SCHM-02 | Phase 1 | Pending |
| SCHM-03 | Phase 1 | Pending |
| SCHM-04 | Phase 1 | Pending |
| SRCM-01 | Phase 1 | Pending |
| SRCM-02 | Phase 1 | Pending |
| SRCM-03 | Phase 1 | Pending |
| NGST-01 | Phase 2 | Pending |
| NGST-02 | Phase 2 | Pending |
| NGST-03 | Phase 2 | Pending |
| NGST-04 | Phase 2 | Pending |
| NGST-05 | Phase 2 | Pending |
| NGST-06 | Phase 2 | Pending |
| NGST-07 | Phase 2 | Pending |
| NGST-08 | Phase 2 | Pending |
| QERY-01 | Phase 3 | Pending |
| QERY-02 | Phase 3 | Pending |
| QERY-03 | Phase 3 | Pending |
| QERY-04 | Phase 3 | Pending |
| LINT-01 | Phase 4 | Pending |
| LINT-02 | Phase 4 | Pending |
| LINT-03 | Phase 4 | Pending |
| LINT-04 | Phase 4 | Pending |
| LINT-05 | Phase 4 | Pending |
| LINT-06 | Phase 4 | Pending |
| INDX-01 | Phase 1 | Pending |
| INDX-02 | Phase 1 | Pending |
| INDX-03 | Phase 1 | Pending |
| LOGM-01 | Phase 1 | Pending |
| LOGM-02 | Phase 1 | Pending |
| GITM-01 | Phase 1 | Pending |
| GITM-02 | Phase 2 | Pending |

**Coverage:**
- v1 requirements: 32 total
- Mapped to phases: 32
- Unmapped: 0 ✓

---
*Requirements defined: 2026-04-15*
*Last updated: 2026-04-15 after initial definition*
