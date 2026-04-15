# Phase 2: Ingest Operation - Context

**Gathered:** 2026-04-15
**Status:** Ready for planning

<domain>
## Phase Boundary

Implement the ingest operation: the user provides a source document in raw/, and Claude Code reads it, creates/updates wiki pages (source summary, entities, concepts), updates index.md, appends to log.md, and commits all changes. Test with two sources to verify incremental integration (updating existing pages when new sources add information).

</domain>

<decisions>
## Implementation Decisions

### Ingest Mechanism
- **D-01:** Ingest is a manual LLM-driven process. The user tells Claude Code to ingest a specific file, and Claude Code follows the 10-step workflow defined in CLAUDE.md. No TypeScript automation tooling is built in this phase.
- **D-02:** Each ingest operation processes one source document at a time, following the single-source-at-a-time principle (per P11 — prevents garbage-in-garbage-out).

### Test Sources
- **D-03:** The first test source is `llmwiki.md` (the project's own idea document), copied to `raw/articles/llm-wiki-concept.md`. It contains named entities, concepts, and cross-references suitable for testing all page types.
- **D-04:** The second test source will be a brief article/summary about Vannevar Bush's "As We May Think" concept, placed in `raw/articles/`. This tests incremental integration (NGST-05) because it shares entities (Vannevar Bush, Memex) with the first source.

### Ingest Workflow Details
- **D-05:** Ripple check goes one hop: after updating a page, check only pages that directly link to it. Deeper consistency checks are deferred to lint (Phase 4).
- **D-06:** Source summary pages are always new (one per source). Entity and concept pages are created if new, or updated if they already exist from a previous ingest.
- **D-07:** The git commit at the end of each ingest captures ALL changes from that ingest in a single commit, with message format: `ingest: <source title> — created N pages, updated M pages`.

### Quality Controls
- **D-08:** Every claim in a wiki page must cite its source using the format defined in CLAUDE.md. This is enforced by editorial discipline, not automation (per P1 mitigation).
- **D-09:** When a second source contradicts the first, both claims are preserved with citations — never silently overwrite (per P2 mitigation and CLAUDE.md editorial rules).
- **D-10:** After each ingest, the output is reviewed by browsing in Obsidian to verify correctness (per P11 — ingest with review prevents error propagation).

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Definition
- `.planning/PROJECT.md` — Core value, constraints, key decisions
- `.planning/REQUIREMENTS.md` — NGST-01..08, GITM-02

### Schema (Phase 1 output)
- `CLAUDE.md` — Wiki schema with page templates, operation workflows, editorial rules (the ingest workflow is defined here)
- `index.md` — Empty index with category headers (to be populated by ingest)
- `log.md` — Operations log with initial entry (to be appended by ingest)

### Research
- `.planning/research/ARCHITECTURE.md` — Ingest data flow, component boundaries
- `.planning/research/PITFALLS.md` — P1 (hallucination), P2 (silent inconsistency), P3 (context overflow), P11 (garbage-in-garbage-out)

### Idea Document
- `../llmwiki.md` — Original idea document (will be copied to raw/ as first test source)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `CLAUDE.md` — Complete ingest workflow already defined (10 steps)
- `index.md` — Empty scaffold ready for population
- `log.md` — Initial entry established; append pattern defined

### Established Patterns
- YAML frontmatter schema with 6 mandatory fields (from Phase 1)
- Page templates for 5 types (from Phase 1)
- Wikilink format: `[[kebab-case-filename|Display Name]]`
- kebab-case filenames

### Integration Points
- All wiki pages must validate against the frontmatter schema in CLAUDE.md
- index.md must be updated after every ingest
- log.md must have a new entry after every ingest
- Git commit must capture all ingest changes atomically

</code_context>

<specifics>
## Specific Ideas

- The first ingest of llmwiki.md should produce: 1 source summary, ~3-5 entity pages, ~3-5 concept pages
- Key entities from llmwiki.md: Vannevar Bush, Memex, Obsidian, Claude Code
- Key concepts from llmwiki.md: knowledge compounding, RAG (and why this project avoids it), incremental integration, wikilinks
- The second ingest tests the core differentiator D1 (incremental integration): existing entity pages get new information appended, not new duplicate pages created

</specifics>

<deferred>
## Deferred Ideas

- TypeScript validation scripts (deferred to Phase 4 / Lint)
- Automated frontmatter checking (deferred to Phase 4 / Lint)  
- Batch ingest tooling (explicitly out of scope per requirements)
- Source size chunking for large documents (not needed at current scale)

</deferred>

---

*Phase: 02-ingest-operation*
*Context gathered: 2026-04-15*
