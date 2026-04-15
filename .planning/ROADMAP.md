# Roadmap: Marvis

**Version:** v1
**Created:** 2026-04-15
**Phases:** 4
**Requirements:** 32 mapped

## Progress

| # | Phase | Status | Requirements |
|---|-------|--------|-------------|
| 1 | Schema & Scaffolding | Pending | SCHM-01, SCHM-02, SCHM-03, SCHM-04, SRCM-01, SRCM-02, SRCM-03, INDX-01, INDX-02, INDX-03, LOGM-01, LOGM-02, GITM-01 |
| 2 | Ingest Operation | Pending | NGST-01, NGST-02, NGST-03, NGST-04, NGST-05, NGST-06, NGST-07, NGST-08, GITM-02 |
| 3 | Query Operation | Pending | QERY-01, QERY-02, QERY-03, QERY-04 |
| 4 | Lint Operation | Pending | LINT-01, LINT-02, LINT-03, LINT-04, LINT-05, LINT-06 |

## Phase Details

### Phase 1: Schema & Scaffolding
**Goal:** Establish the CLAUDE.md schema, directory structure, index, log, and git foundation so all subsequent operations have a consistent convention to follow.
**Requirements:** SCHM-01, SCHM-02, SCHM-03, SCHM-04, SRCM-01, SRCM-02, SRCM-03, INDX-01, INDX-02, INDX-03, LOGM-01, LOGM-02, GITM-01
**Success Criteria:**
1. Opening the vault in Obsidian shows a navigable index.md with page type categories and no broken links
2. Placing a file in raw/ and confirming the system never modifies it (immutability convention holds)
3. CLAUDE.md contains page templates, frontmatter schema, and operation workflows that a fresh Claude Code session can follow without prior context
4. log.md exists with parseable `## [YYYY-MM-DD] operation | subject` format and the vault is a git repository with at least one commit
**Dependencies:** None
**UI hint:** no

### Phase 2: Ingest Operation
**Goal:** Enable the core value-creation loop: user provides a source document, the LLM reads it and writes/updates wiki pages (summary, entities, concepts), updates the index, logs the operation, and commits.
**Requirements:** NGST-01, NGST-02, NGST-03, NGST-04, NGST-05, NGST-06, NGST-07, NGST-08, GITM-02
**Success Criteria:**
1. After ingesting a single source, a source summary page exists in wiki/sources/ with YAML frontmatter and citations back to the raw source
2. Entity and concept pages are created or updated with new information, and every claim cites its source (no unsourced assertions)
3. Ingesting a second source that mentions an existing entity updates the existing entity page (incremental integration, not just new page creation)
4. index.md reflects all new and modified pages after each ingest, and log.md has a new timestamped entry
5. A git commit captures all changes from the ingest operation in a single commit
**Dependencies:** Phase 1
**UI hint:** no

### Phase 3: Query Operation
**Goal:** Allow the user to ask questions answered from wiki content (not raw sources), with cited wikilinks, and optionally persist valuable answers as new wiki pages.
**Requirements:** QERY-01, QERY-02, QERY-03, QERY-04
**Success Criteria:**
1. Asking a question about ingested content returns an answer synthesized from wiki pages, not from raw source files directly
2. Every query answer includes `[[wikilinks]]` citations to the wiki pages used, and these links resolve in Obsidian
3. A valuable query answer can be filed as a new page in wiki/queries/ or wiki/synthesis/ with proper frontmatter and index entry
**Dependencies:** Phase 2 (requires populated wiki content to query against)
**UI hint:** no

### Phase 4: Lint Operation
**Goal:** Provide automated health checks that detect broken links, orphan pages, missing entity pages, index drift, and contradictions — maintaining wiki integrity as content grows.
**Requirements:** LINT-01, LINT-02, LINT-03, LINT-04, LINT-05, LINT-06
**Success Criteria:**
1. Running lint on a wiki with a deliberately broken wikilink reports the broken link with the source page and target
2. Orphan pages (zero inbound links) and frequently-mentioned-but-missing entities are identified in the lint report
3. Index completeness check catches pages not listed in index.md and stale entries for deleted pages
4. Lint operation appends a structured entry to log.md summarizing findings
**Dependencies:** Phase 2 (requires wiki content to lint); Phase 3 recommended but not required
**UI hint:** no

---
*Created: 2026-04-15*
*Last updated: 2026-04-15*
