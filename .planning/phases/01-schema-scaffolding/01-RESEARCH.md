# Phase 1: Schema & Scaffolding — Research

**Researched:** 2026-04-15
**Phase:** 01-schema-scaffolding
**Objective:** What do we need to know to PLAN this phase well?

---

## 1. CLAUDE.md Schema Design

### What Sections Should CLAUDE.md Contain?

The current CLAUDE.md is auto-generated GSD boilerplate (project summary, stack, conventions, architecture, workflow enforcement). Phase 1 must replace or augment this with wiki-specific schema content that satisfies SCHM-01: "CLAUDE.md schema defines wiki conventions, page templates, editorial rules, and operation workflows."

The architecture research (ARCHITECTURE.md §Schema Design) already defines seven sections. Cross-referencing with the decisions in 01-CONTEXT.md, the final CLAUDE.md needs:

| # | Section | Purpose | Source Decision |
|---|---------|---------|-----------------|
| 1 | **Quick Start** | Orient a fresh Claude Code session in 30 seconds — what this wiki is, how to operate on it | D-02 (self-sufficient for fresh session), specifics note |
| 2 | **Directory Conventions** | Map of raw/, wiki/, index.md, log.md, what goes where | D-13, D-14 |
| 3 | **Page Templates** | Frontmatter schema + body structure for each of 5 page types | D-03, D-06, D-07, D-08 |
| 4 | **Naming & Linking Rules** | Kebab-case filenames, wikilink format with display names, tag conventions | D-04, D-05 |
| 5 | **Operation Workflows** | Step-by-step instructions for ingest, query, lint | ARCHITECTURE.md §4 |
| 6 | **Editorial Rules** | Citation requirements, contradiction handling, immutability of raw/ | P1, P2 mitigations |
| 7 | **Index Format** | Exact format of index.md entries, category structure | D-09, D-10 |
| 8 | **Log Format** | Exact format of log.md entries, access pattern (tail, not full read) | D-11, D-12, P8 mitigation |

### How Detailed Should Page Templates Be?

**Finding:** Per P6 (over-engineered schema), templates should be minimal but precise. The research recommends:
- Mandatory frontmatter: `title`, `type`, `sources` (array), `tags` (array), `created` (ISO date), `updated` (ISO date) — per D-06
- Optional frontmatter varies by type — per D-07
- Body sections defined as heading names only (not paragraph-level templates)
- Include one concrete example per page type — P15 mitigation (session amnesia)

**Key tension:** D-08 says no `wiki_*` prefix for custom fields (keep it simple at this scale), but P10 warns about Obsidian plugin conflicts. Resolution: use unprefixed fields now, document the escape hatch in CLAUDE.md ("if Dataview conflicts arise, prefix custom fields with `wiki_`").

### Five Page Type Templates

Based on D-03 and ARCHITECTURE.md:

1. **Source Summary** (`type: source`) — Frontmatter adds `source_path`. Body: Key Takeaways, Detailed Notes, Related Pages.
2. **Entity** (`type: entity`) — Frontmatter adds `aliases`, `category` (person/org/tool/place). Body: Overview, Key Details, Appearances in Sources, Related Entities.
3. **Concept** (`type: concept`) — Frontmatter adds `related` (array). Body: Definition, Key Points, Sources, Related Concepts.
4. **Synthesis** (`type: synthesis`) — Body: Overview, Analysis, Sources, Related Pages.
5. **Query** (`type: query`) — Frontmatter adds `question` (string). Body: Answer, Sources Cited, Related Pages.

### Operation Workflows to Define

Three workflows needed (from ARCHITECTURE.md data flows):

1. **Ingest** — The most complex. Must include the multi-step design from P3 mitigation: (a) read source + schema, (b) read index to understand wiki state, (c) create source summary, (d) create/update entity pages individually, (e) create/update concept pages, (f) ripple check for consistency (P2), (g) update index.md, (h) append log.md. Must emphasize: never hold full wiki in context.

2. **Query** — Read index first, drill into relevant pages, synthesize answer with wikilink citations, optionally persist as wiki page, append log.

3. **Lint** — Check broken wikilinks, orphan pages, missing index entries, frontmatter compliance, contradictions. This workflow is defined in schema but not implemented until Phase 4.

**Important for planning:** Phase 1 only needs to *define* these workflows in CLAUDE.md. Phases 2-4 *implement* them. The workflows must be written clearly enough that a future Claude Code session can follow them without having read this research.

### GSD Boilerplate Coexistence

The current CLAUDE.md has GSD-managed sections (delimited by `<!-- GSD:*-start -->` / `<!-- GSD:*-end -->` comments). The wiki schema content must coexist with these. Options:
- **Option A:** Place wiki schema in a dedicated section outside GSD markers
- **Option B:** Replace GSD architecture/conventions sections with wiki-specific content

**Recommendation:** Option A is safer — keep GSD markers intact, add wiki schema sections above or below them. The GSD workflow enforcement section is still useful for project management.

---

## 2. Obsidian Compatibility

### YAML Frontmatter Best Practices

Obsidian has specific frontmatter requirements:

- **`tags` field:** Must be a YAML list, not a string. Obsidian renders tags in the tag pane. Format: `tags: [knowledge-management, tool]` or multi-line list. No `#` prefix in frontmatter (the `#` is only for inline tags).
- **`aliases` field:** Obsidian-native. Allows pages to be found by alternative names. Must be a YAML list.
- **`title` field:** Not natively used by Obsidian (Obsidian uses the filename as title), but useful for the LLM and Dataview queries. No harm including it.
- **`created` / `updated` dates:** Use ISO 8601 format (`YYYY-MM-DD`). Obsidian's Properties view renders these as dates if formatted correctly.
- **YAML quoting:** Values containing colons, brackets, or other YAML special characters must be quoted. The LLM must be instructed to quote defensively.
- **Property types:** Obsidian 1.4+ has a "Properties" feature that infers types from frontmatter. Consistent types across pages prevent confusion (e.g., `sources` must always be a list, never sometimes a string).

### Wikilink Format Requirements

Per D-04, D-05, and P5:

- **Format:** `[[filename]]` or `[[filename|Display Name]]` — Obsidian resolves by filename search across the vault
- **No `.md` extension:** Obsidian convention omits the extension in wikilinks
- **Path-relative vs. shortest-path:** Obsidian defaults to "shortest path" resolution (just the filename, no directory prefix). If two files have the same name in different directories, full path is needed. With kebab-case naming and distinct page types, collisions are unlikely.
- **Avoid heading links:** `[[page#heading]]` breaks when headings change (P5). Use only if the heading is stable.
- **Case sensitivity:** macOS filesystem is case-insensitive but case-preserving. Linux is case-sensitive. Kebab-case (D-04) avoids this entirely.

**Planning implication:** The schema must document the exact wikilink format with examples. The index.md entries need to use wikilinks that Obsidian can resolve.

### Obsidian Configuration for Excluding .planning/

Obsidian has multiple mechanisms:

1. **File exclusion filter** (Settings > Files & Links > Excluded files): Glob patterns that hide files from the file explorer and search. This is the simplest approach.
2. **`.obsidian/app.json`** — The `"userIgnoreFilters"` key accepts an array of glob patterns: `[".planning"]`. These files are hidden from file explorer, search, and graph view.
3. **Dotfile convention:** Obsidian already ignores directories starting with `.` in search and graph view by default on most platforms. Since `.planning/` starts with a dot, it may already be excluded. However, this behavior is not guaranteed across all Obsidian versions.

**Recommendation for scaffolding:** Create `.obsidian/app.json` with explicit exclusion of `.planning/`. This is the most reliable approach and survives Obsidian version changes.

Minimal `.obsidian/app.json`:
```json
{
  "userIgnoreFilters": [".planning/"]
}
```

### Graph View Optimization

- **Tags as colors:** Obsidian Graph View can color nodes by tag. If page types are also tags (e.g., `#source`, `#entity`, `#concept`), the graph shows natural clusters.
- **Folder-based grouping:** Graph View can group by folder. Since wiki pages are in `wiki/sources/`, `wiki/entities/`, etc., folder-based grouping gives automatic visual clustering.
- **Link density:** P13 warns about graph hairball. The schema should advise meaningful links only — don't link every mention, link the first/most relevant mention.

**Planning implication:** No special scaffolding needed for Graph View. The directory structure (D-14) and tag conventions naturally produce good graph visualization. Document this in CLAUDE.md so the LLM makes intentional linking decisions.

---

## 3. Directory Scaffolding

### Exact Directory Structure

From D-13, D-14, and ARCHITECTURE.md:

```
marvis/                          # Vault root (already exists)
├── CLAUDE.md                    # Schema (exists, needs wiki content)
├── index.md                     # Wiki index (create)
├── log.md                       # Operations log (create)
├── raw/                         # Immutable sources (create)
│   ├── articles/                # Web articles
│   ├── papers/                  # Academic papers
│   ├── notes/                   # Personal notes
│   ├── books/                   # Book chapters/excerpts
│   └── assets/                  # Downloaded images
├── wiki/                        # LLM-maintained pages (create)
│   ├── sources/                 # Source summary pages
│   ├── entities/                # Entity pages
│   ├── concepts/                # Concept pages
│   ├── synthesis/               # Cross-cutting analyses
│   └── queries/                 # Persisted query answers
├── .obsidian/                   # Obsidian config (create)
│   └── app.json                 # Exclusion patterns
├── .planning/                   # Project planning (exists)
└── .git/                        # Version history (exists)
```

### Placeholder Files and .gitkeep Patterns

Git does not track empty directories. Options:

1. **`.gitkeep`** — Convention (not a git feature) where an empty file named `.gitkeep` is placed in each empty directory. Pros: directories exist after clone. Cons: extra files visible in Obsidian.
2. **No placeholders** — Create directories only when first needed (during ingest). Pros: clean. Cons: the scaffolding phase doesn't actually create the directory structure in git.
3. **README.md placeholders** — A small `README.md` in each directory explaining what goes there. Pros: self-documenting. Cons: adds noise to the wiki.

**Recommendation:** Use `.gitkeep` for `raw/` subdirectories (user-facing, needs to exist for them to drop files in). For `wiki/` subdirectories, `.gitkeep` is also appropriate since the directory structure is part of the schema convention and should exist from day one. Obsidian will show `.gitkeep` files but they're harmless and can be excluded via `.obsidian/app.json` if desired.

**Add to exclusion filter:** `[".planning/", ".gitkeep"]` in `userIgnoreFilters`.

### Files to Create

| File | Content | Purpose |
|------|---------|---------|
| `index.md` | Category headers, no entries | Navigation scaffold |
| `log.md` | Header + initial creation entry | Operation log |
| `.obsidian/app.json` | Exclusion config | Hide .planning/ from Obsidian |
| `raw/articles/.gitkeep` | Empty | Preserve directory in git |
| `raw/papers/.gitkeep` | Empty | Preserve directory in git |
| `raw/notes/.gitkeep` | Empty | Preserve directory in git |
| `raw/books/.gitkeep` | Empty | Preserve directory in git |
| `raw/assets/.gitkeep` | Empty | Preserve directory in git |
| `wiki/sources/.gitkeep` | Empty | Preserve directory in git |
| `wiki/entities/.gitkeep` | Empty | Preserve directory in git |
| `wiki/concepts/.gitkeep` | Empty | Preserve directory in git |
| `wiki/synthesis/.gitkeep` | Empty | Preserve directory in git |
| `wiki/queries/.gitkeep` | Empty | Preserve directory in git |

---

## 4. Index Format

### Design Goals

Per D-09, D-10, INDX-01, INDX-02, INDX-03:
- Organized by page type categories matching wiki/ subdirectories
- Each entry: wikilink + one-line summary
- Machine-parseable (consistent format for the LLM to update programmatically)
- Starts empty with category headers only (populated by ingest operations)
- Serves as the LLM's primary navigation tool (INDX-03)

### Proposed Format

```markdown
# Wiki Index

> This index catalogs every wiki page. It is the primary navigation tool for both the LLM and human readers. Updated on every ingest operation.

## Sources
<!-- Source summary pages — one per ingested source -->

## Entities
<!-- Entity pages — people, organizations, tools, places -->

## Concepts
<!-- Concept pages — ideas, frameworks, patterns, themes -->

## Synthesis
<!-- Cross-cutting analyses, comparisons, timelines -->

## Queries
<!-- Persisted answers to user questions -->
```

When populated, entries follow this format:
```markdown
## Sources
- [[source-filename|Source Title]] — One-line summary (YYYY-MM-DD)
```

### Design Decisions for Planning

1. **Wikilink format in index:** Use `[[filename|Display Title]]` per D-05. The filename is the kebab-case version; the display title is human-readable.
2. **No path prefix in wikilinks:** Obsidian resolves by shortest path. `[[vannevar-bush|Vannevar Bush]]` resolves to `wiki/entities/vannevar-bush.md` without needing the full path. This only works if filenames are unique across the vault — the kebab-case convention (D-04) plus distinct page names makes this very likely.
3. **Date in source entries:** Source summaries include the ingest date for chronological context. Other page types omit it.
4. **HTML comments as section markers:** The `<!-- ... -->` comments help the LLM identify empty sections and understand what goes where, without rendering in Obsidian.
5. **No YAML frontmatter on index.md:** The index is a navigation file, not a wiki page. Adding frontmatter would complicate it unnecessarily. Keep it as plain markdown.

### Scaling Considerations

Per P9, the flat index works up to ~150 pages. Planning should document the migration path:
- At ~150 pages: split into sub-indexes (e.g., `index-entities.md`, `index-concepts.md`) with the main index linking to them
- At ~200 pages: introduce MiniSearch or qmd for programmatic search
- This is a v2 concern but the format should not preclude it

---

## 5. Log Format

### Design Goals

Per D-11, D-12, LOGM-01, LOGM-02:
- Append-only
- Parseable format: `## [YYYY-MM-DD] operation | subject`
- Every operation (ingest, query, lint) appends an entry
- Brief description paragraph under each entry

### Proposed Format

```markdown
# Operations Log

> Chronological record of all wiki operations. Append-only. Parse with: `grep "^## \[" log.md`

## [2026-04-15] create | Wiki Initialization
Established wiki schema, directory structure, index, and log. Created CLAUDE.md with page templates, operation workflows, and editorial rules.
```

### Design Decisions for Planning

1. **Initial entry:** Per D-12, the first log entry records the wiki creation event. This entry is created as part of the Phase 1 scaffolding.
2. **Operation types:** `create`, `ingest`, `query`, `lint`, `update` (for manual corrections or schema changes).
3. **Entry body:** One paragraph (2-4 sentences). Lists pages created/updated with wikilinks where relevant. Example for a future ingest:
   ```
   ## [2026-04-16] ingest | As We May Think (Vannevar Bush)
   Ingested article on Vannevar Bush's Memex vision. Created [[as-we-may-think|As We May Think]], [[vannevar-bush|Vannevar Bush]], [[memex|Memex]]. Updated [[knowledge-compounding|Knowledge Compounding]] with new historical context.
   ```
4. **Access pattern:** Per P8, the schema must instruct the LLM to use `tail -50 log.md` instead of reading the full log. Only the recent entries matter for operational context.
5. **No frontmatter:** Like index.md, log.md is an operational file, not a wiki page.
6. **Rotation plan:** Document in CLAUDE.md that at ~500 entries, old entries should be archived. This is a v2 concern but should be noted.

---

## 6. Git Foundation

### Current State

The marvis directory is already a git repository with 5 commits (planning artifacts). Phase 1 needs to ensure GITM-01 is satisfied: "The entire wiki is a git repository."

### What Phase 1 Must Do

1. The git repo already exists — no `git init` needed.
2. All scaffolding files (index.md, log.md, CLAUDE.md, directory structure, .obsidian/app.json) must be committed.
3. The commit message should follow the project's existing pattern (conventional commits: `docs(01): ...` or similar).
4. `.gitignore` considerations: Should `.obsidian/` be committed? Obsidian creates a `workspace.json` that changes constantly. Best practice: commit `.obsidian/app.json` (our configuration) but gitignore `.obsidian/workspace.json` and other volatile files.

### .gitignore Recommendations

```
# Obsidian workspace (changes on every session)
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# Obsidian cache
.obsidian/cache

# OS files
.DS_Store
```

Note: `.obsidian/app.json` should NOT be gitignored — it contains our exclusion configuration.

---

## 7. Implementation Considerations

### Ordering Within Phase 1

The tasks have a natural dependency order:

1. **CLAUDE.md schema** (SCHM-01) — Must be written first because it defines the conventions that all other files follow.
2. **Directory structure** (SCHM-02, SRCM-01, SRCM-02, SRCM-03) — Create raw/ and wiki/ with subdirectories and .gitkeep files.
3. **.obsidian/app.json** — Obsidian configuration for excluding .planning/.
4. **.gitignore** — Exclude Obsidian volatile files.
5. **index.md** (INDX-01, INDX-02, INDX-03) — Empty scaffold with category headers.
6. **log.md** (LOGM-01, LOGM-02) — With initial creation entry.
7. **Git commit** (GITM-01) — Commit everything as the foundation commit.

### Frontmatter Schema Summary (for D-06, D-07)

| Field | Required | Type | All Pages | Notes |
|-------|----------|------|-----------|-------|
| `title` | Yes | string | Yes | Human-readable page title |
| `type` | Yes | enum | Yes | source, entity, concept, synthesis, query |
| `sources` | Yes | list | Yes | Array of raw source references |
| `tags` | Yes | list | Yes | Array of lowercase kebab-case tags |
| `created` | Yes | date | Yes | ISO 8601 (YYYY-MM-DD) |
| `updated` | Yes | date | Yes | ISO 8601 (YYYY-MM-DD) |
| `source_path` | No | string | source only | Relative path to raw source file |
| `aliases` | No | list | entity only | Alternative names for the entity |
| `category` | No | string | entity only | person, org, tool, place, etc. |
| `related` | No | list | concept only | Related concept wikilinks |
| `question` | No | string | query only | The original question asked |

### Success Criteria Verification Plan

| Criterion | How to Verify |
|-----------|---------------|
| 1. Navigable index.md in Obsidian with no broken links | Open vault in Obsidian, click index.md, verify category headers render. No entries = no broken links. |
| 2. raw/ immutability convention holds | Place a test file in raw/, run any operation, verify file is unchanged. (Convention-only at this phase — no code to test.) |
| 3. CLAUDE.md is self-sufficient for fresh session | Open a new Claude Code session in the marvis directory. Read only CLAUDE.md. Verify the session can describe how to ingest a source without additional context. |
| 4. log.md parseable, vault is git repo with commits | Run `grep "^## \[" log.md` to verify parseable format. Run `git log --oneline` to verify commits exist. |

---

## 8. Risks Specific to This Phase

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Schema too complex for LLM to follow consistently | Medium | High | Start minimal per P6. Include concrete examples. Test with a dry-run ingest after Phase 2. |
| GSD boilerplate in CLAUDE.md conflicts with wiki schema | Low | Medium | Keep GSD sections intact, add wiki sections separately. |
| Obsidian not properly excluding .planning/ | Low | Low | Test exclusion immediately after opening vault in Obsidian. |
| Wikilink format in index doesn't resolve in Obsidian | Low | Medium | Test with at least one real page after Phase 2 ingest. For scaffolding, empty index has no links to break. |
| .obsidian/app.json gets overwritten by Obsidian | Low | Medium | Obsidian merges settings, doesn't overwrite. Commit the file to git so it can be restored. |

---

## 9. Open Questions for Planning

1. **CLAUDE.md structure:** Should wiki schema content replace the GSD-generated architecture/conventions sections, or live in separate sections? (Recommendation: separate sections, preserve GSD markers.)

2. **Obsidian attachment folder:** Should `.obsidian/app.json` also set the attachment folder to `raw/assets/`? This affects where Obsidian saves pasted/downloaded images. (Recommendation: yes, include `"attachmentFolderPath": "raw/assets"` in app.json.)

3. **Tag taxonomy:** Should the schema define initial tag categories (e.g., `#source`, `#entity`, `#concept` matching page types) or let them emerge organically? (Recommendation: define page-type tags as mandatory; let domain tags emerge. Per D-03, page types are defined; per Claude's discretion, tag taxonomy can be organic.)

4. **Wikilink path strategy:** Use shortest-path (just filename) or relative paths in wikilinks? (Recommendation: shortest-path per Obsidian convention. Document this explicitly in CLAUDE.md.)

---

*Phase: 01-schema-scaffolding*
*Research completed: 2026-04-15*
