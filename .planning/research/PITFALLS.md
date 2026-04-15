# Pitfalls Research
**Domain:** LLM-powered personal knowledge base wiki
**Researched:** 2026-04-15

---

## Critical Pitfalls

### P1. Hallucinated Cross-References and Fabricated Claims
**Area:** LLM content generation
**Severity:** Critical

The LLM will occasionally generate wikilinks to pages that do not exist, cite sources that were never ingested, or state facts not present in any source. Because the wiki is trusted as a "compiled" artifact, hallucinations become laundered into the knowledge base — they look indistinguishable from real extracted knowledge once they're sitting in a wiki page.

**Warning signs:**
- Broken `[[wikilinks]]` that lead nowhere (Obsidian shows them in a different color)
- Claims in wiki pages that cannot be traced to any source in `raw/`
- The LLM confidently referencing "as noted in [[Page That Doesn't Exist]]"
- Orphan pages that were created from hallucinated rather than real entities

**Prevention strategy:**
- Every claim in a wiki page must cite its source file (e.g., `(source: [[raw/article-name]])`)
- The lint operation must check that all wikilinks resolve to existing pages
- The lint operation must check that cited source files actually exist in `raw/`
- Consider a frontmatter field `sources: [list]` on every wiki page — verifiable by automation
- Keep ingest conversational: review summaries before they are committed to the wiki

**Phase to address:** Schema design (define citation conventions) and lint operation (enforce them)

---

### P2. Silent Inconsistency Across Pages
**Area:** Wiki maintenance
**Severity:** Critical

When the LLM updates one page but fails to update related pages, the wiki develops contradictions. Example: a source is ingested that corrects a date. The LLM updates the entity page but not the timeline page, the overview page, or two other pages that mention the same date. The wiki now contains both the old and new date with no indication of conflict.

**Warning signs:**
- The same fact appears with different values on different pages
- A page's "last updated" timestamp is much older than pages it references
- Grep for a specific claim returns conflicting statements across files
- Users notice discrepancies while browsing that the LLM didn't flag

**Prevention strategy:**
- The ingest workflow must explicitly list all pages touched and all pages that reference the updated entity
- Use a "ripple check" step: after updating a page, search the wiki for all other mentions of the changed fact
- The lint operation should compare claims across pages that reference the same entity
- Frontmatter `related: [list]` fields create a verifiable dependency graph
- Keep a `contradictions.md` page where the LLM logs unresolved conflicts

**Phase to address:** Ingest workflow design (ripple check) and lint operation (cross-page consistency)

---

### P3. Context Window Overflow During Ingest
**Area:** LLM content generation / Scale
**Severity:** Critical

A single ingest operation may require the LLM to: read the new source, read the index, read 5-15 existing wiki pages, understand the schema, and produce updates to multiple files. For a long source document or a wiki with hundreds of pages, this easily exceeds the LLM's effective context window. When the context overflows, the LLM silently drops information — it doesn't error, it just gets worse.

**Warning signs:**
- Ingest operations that produce shallow or incomplete summaries compared to earlier ingests
- The LLM "forgetting" to update pages it clearly should have updated
- Updates that repeat information already present instead of synthesizing with it
- The LLM asking to see files it was already shown in the same session

**Prevention strategy:**
- Design the ingest workflow as multiple sequential steps, not one massive prompt
- Step 1: Read source, produce summary (source + schema in context)
- Step 2: Read index, identify pages to update (summary + index in context)
- Step 3: For each page to update, read it individually and produce the update
- Never ask the LLM to hold the entire wiki in context at once
- Keep index.md as a lightweight lookup table, not a full-content dump
- Set a source size limit (e.g., <15,000 words) and chunk larger sources

**Phase to address:** Schema design (workflow steps) and ingest operation (chunked processing)

---

### P4. Index Drift — The Index Stops Reflecting Reality
**Area:** Wiki maintenance / Scale
**Severity:** Critical

index.md is the LLM's primary navigation tool. If it falls out of sync with actual wiki contents — missing pages, stale summaries, broken links — the LLM's ability to find and update relevant pages degrades. Every subsequent ingest and query becomes less accurate because the LLM is working from a wrong map.

**Warning signs:**
- Pages exist in the wiki directory that are not listed in index.md
- index.md lists pages that have been deleted or renamed
- One-line summaries in index.md no longer match the page's actual content
- The LLM fails to find obviously relevant pages during query operations

**Prevention strategy:**
- The lint operation must diff the filesystem against index.md entries
- Every ingest operation must update index.md as its final step (not optional, not "if needed")
- index.md should be regeneratable from scratch: a lint "rebuild index" command
- Keep index.md machine-parseable (consistent format, no prose between entries)
- Consider a simple script that lists all .md files not in index.md as a pre-lint check

**Phase to address:** Schema design (index format) and lint operation (index validation)

---

### P5. Wikilink Format Fragility with Obsidian
**Area:** Obsidian compatibility
**Severity:** Critical

Obsidian's wikilink resolution has specific behaviors that the LLM must match exactly, or links break silently. Key issues: Obsidian resolves `[[Page Name]]` by searching all vault files for a matching filename (case-insensitive on macOS, case-sensitive on Linux). If two files have similar names, Obsidian may resolve to the wrong one. Pipe aliases `[[Page Name|display text]]` work differently from standard markdown links. Heading links `[[Page Name#Section]]` break if the heading changes.

**Warning signs:**
- Links that appear valid in raw markdown but don't resolve in Obsidian
- Links that resolve to the wrong page (silent mismatch)
- Heading links `[[Page#Section]]` that break after page edits
- Different behavior when the vault is opened on macOS vs Linux

**Prevention strategy:**
- Establish strict naming conventions: kebab-case filenames, no special characters
- Wikilinks must use exact filenames (without .md extension, matching Obsidian convention)
- Avoid heading links (`[[Page#Section]]`) unless the section heading is stable
- The lint operation should verify all wikilinks resolve correctly
- Document the exact wikilink format in the schema with examples
- Test on the actual target OS early

**Phase to address:** Schema design (naming conventions, link format)

---

## Common Mistakes

### P6. Over-Engineered Schema That the LLM Cannot Follow
**Area:** Schema design
**Severity:** Medium

Designing an elaborate schema with many page types, complex frontmatter requirements, strict taxonomies, and detailed formatting rules. The LLM will follow it inconsistently — sometimes perfectly, sometimes partially, sometimes not at all. The more complex the schema, the more the LLM drifts from it over time and across sessions.

**Warning signs:**
- Frontmatter fields that are empty or missing on many pages
- Pages that don't match their declared type's template
- The LLM producing pages that look right but violate subtle schema rules
- You spending more time fixing schema compliance than reading wiki content

**Prevention strategy:**
- Start with the minimal viable schema: page title, sources list, one-line summary, body, wikilinks
- Add complexity only when you hit a real problem that requires it
- Every schema rule must be enforceable by lint — if you can't check it, don't require it
- Prefer conventions that are natural for markdown (headings, lists, links) over custom structures
- Test the schema by having the LLM generate 5 pages and checking compliance before committing

**Phase to address:** Schema design (start minimal, iterate)

---

### P7. Orphan Pages and Dead Zones
**Area:** Wiki maintenance
**Severity:** Medium

As the wiki grows, pages accumulate that have no inbound links — nothing points to them. They become invisible: the LLM won't find them via index browsing, users won't discover them via link-following, and they rot silently. Conversely, some pages become over-linked hubs where every new ingest adds another link, making them noisy and hard to read.

**Warning signs:**
- Obsidian graph view shows isolated nodes (orphans)
- Pages with zero inbound links (checkable by grepping for `[[PageName]]` across all files)
- Hub pages where the link section is longer than the actual content
- Important entities that have no dedicated page (mentioned in text but never filed)

**Prevention strategy:**
- The lint operation should report pages with zero inbound links
- The lint operation should report entities mentioned 3+ times across pages that lack their own page
- Set a soft cap on links per page section (e.g., "Related" sections with 20+ links need restructuring)
- The ingest workflow should check: "Does this source mention entities that deserve their own page?"

**Phase to address:** Lint operation

---

### P8. Log File Grows Unbounded and Becomes Useless
**Area:** Scale
**Severity:** Medium

log.md is append-only by design, but it has no rotation or summarization strategy. After 100+ ingests, it becomes a massive file that the LLM cannot usefully read (context window) and that provides no signal (too much noise). The LLM may try to read the entire log to understand history and waste its context budget.

**Warning signs:**
- log.md exceeds ~500 lines
- The LLM reads log.md during operations but doesn't extract useful information from it
- You never actually look at log.md yourself
- Ingest operations slow down because the LLM is processing log history

**Prevention strategy:**
- Design log.md with a parseable format from day one (consistent prefixes, dates, types)
- The LLM should only read the last N entries of the log, not the entire file (use `tail`)
- Consider periodic log rotation: archive old entries to `log-archive/2026-Q1.md`
- Alternatively, keep log.md short and maintain a separate `stats.md` with aggregate counts
- The schema should explicitly say: "Never read the entire log. Use `tail -50 log.md`."

**Phase to address:** Schema design (log format and access patterns)

---

### P9. Query Quality Degrades as Wiki Grows
**Area:** Scale / Workflow
**Severity:** Medium

At 10 pages, the LLM can effectively read the index and find what it needs. At 100 pages, the index itself becomes long enough that the LLM may miss relevant entries. At 300+ pages, index-based navigation breaks down entirely — the LLM either reads a truncated index or reads the whole thing but can't effectively scan it.

**Warning signs:**
- Query answers that miss obviously relevant wiki pages
- The LLM citing only recently ingested pages, not older relevant ones
- Answers that are shallower than what the wiki actually contains
- The index file exceeding ~200 lines

**Prevention strategy:**
- Design the index with categories/sections from the start (not a flat list)
- Consider a hierarchical index: top-level index links to category indexes
- Set a threshold (e.g., 150 pages) where you introduce search tooling (qmd or similar)
- The schema should define when to split the index into sub-indexes
- Monitor answer quality: periodically ask questions you know the wiki can answer well

**Phase to address:** Schema design (index structure) with a planned migration path at scale

---

### P10. Frontmatter Conflicts with Obsidian Plugins
**Area:** Obsidian compatibility
**Severity:** Medium

Obsidian plugins (especially Dataview) read and sometimes write YAML frontmatter. If the LLM's frontmatter schema conflicts with plugin expectations — wrong types, reserved field names, invalid YAML — pages may display incorrectly or plugins may error. Common issues: `tags` must be a list not a string, `date` must be ISO format, `title` with colons needs quoting, `aliases` has specific Obsidian behavior.

**Warning signs:**
- Dataview queries returning unexpected results or errors
- Obsidian showing raw YAML instead of rendered frontmatter
- Pages where frontmatter silently gets corrupted after Obsidian edits
- Tags not appearing in Obsidian's tag pane

**Prevention strategy:**
- Use only Obsidian-standard frontmatter fields: `tags` (list), `aliases` (list), `date` (ISO), `cssclasses`
- Custom fields should be prefixed to avoid collisions (e.g., `wiki_sources` not `sources`)
- Always quote string values that contain colons, brackets, or other YAML special characters
- Test frontmatter with Dataview early: create a test query page
- Document the exact frontmatter schema with valid examples in CLAUDE.md

**Phase to address:** Schema design (frontmatter specification)

---

### P11. Ingest Without Review Creates Garbage-In-Garbage-Out
**Area:** Workflow
**Severity:** Medium

Batch-ingesting many sources without reviewing the LLM's output leads to compounding errors. A bad summary gets cross-referenced by three other pages. A misidentified entity gets its own page. A nuance the LLM missed becomes the canonical version of a fact. Because each ingest builds on previous wiki state, early errors propagate.

**Warning signs:**
- Pages that feel "off" when you read them but you're not sure when the error was introduced
- Entity pages that conflate two different things with similar names
- The wiki's narrative drifting from what the sources actually say
- Corrections requiring edits to many pages (sign of early-stage error propagation)

**Prevention strategy:**
- Ingest one source at a time, especially early on when the wiki's foundations are being set
- Review summaries before they are committed (the LLM should present, not just write)
- After every ingest, browse the updated pages in Obsidian and spot-check
- Keep a "review queue" — newly created/updated pages that haven't been human-verified
- Frontmatter field `reviewed: false` that gets flipped to `true` after human review

**Phase to address:** Workflow design (ingest review step)

---

## Minor Risks

### P12. Git History Bloat from Frequent Small Commits
**Area:** Workflow
**Severity:** Low

If every LLM operation creates a git commit, the repo accumulates thousands of small commits quickly. This is not a functional problem at moderate scale, but it makes `git log` noisy and `git diff` between meaningful states harder.

**Warning signs:**
- Hundreds of commits per week
- Commit messages that are not informative ("Updated wiki pages")
- Difficulty finding when a specific change was made

**Prevention strategy:**
- Commit at the operation level (one commit per ingest, per query-with-filing, per lint pass)
- Use descriptive commit messages: "ingest: [source name] - updated N pages"
- Consider squashing commits periodically if history gets noisy
- This is a minor concern — don't over-optimize early

**Phase to address:** Workflow design (commit strategy)

---

### P13. Obsidian Graph View Becomes Unreadable
**Area:** Obsidian compatibility / Scale
**Severity:** Low

With hundreds of pages and dense cross-references, Obsidian's graph view becomes a hairball — visually impressive but informationally useless. It stops being a useful navigation tool.

**Warning signs:**
- The graph is a single dense cluster with no visible structure
- You stop using graph view because it's not helpful
- New users can't orient themselves by looking at the graph

**Prevention strategy:**
- Use tags and folders to create visual clusters in the graph
- Avoid linking everything to everything — links should be meaningful, not exhaustive
- Use Obsidian's graph view filters (filter by tag, folder, or link depth)
- Consider a "map of content" (MOC) pattern: hub pages that link to clusters of related pages
- This is an aesthetic concern, not a functional one

**Phase to address:** Ongoing maintenance (link hygiene)

---

### P14. Source Format Variability Causes Inconsistent Extraction
**Area:** Workflow
**Severity:** Low

Web-clipped articles, PDFs converted to text, raw notes, and formatted markdown all look different. The LLM extracts information with varying quality depending on source format — clean markdown gives good results, OCR'd PDFs give poor results. Inconsistent source quality leads to inconsistent wiki quality.

**Warning signs:**
- Some source summaries are much better than others for no content-related reason
- The LLM struggling with or misinterpreting formatting artifacts
- Important content being missed because it was in a table, image caption, or sidebar

**Prevention strategy:**
- Standardize source format as much as possible (Obsidian Web Clipper helps)
- Clean up sources manually before ingest if they have obvious formatting issues
- The ingest workflow should note source quality/confidence in the summary
- For critical sources, verify the summary against the original

**Phase to address:** Workflow design (source preparation guidelines)

---

### P15. Session Amnesia Between Claude Code Sessions
**Area:** Workflow
**Severity:** Low

Each Claude Code session starts fresh. The LLM has no memory of previous sessions beyond what's in the wiki and schema. If the schema doesn't capture important conventions discovered during use, they're lost. If the LLM developed good habits through conversation in one session, it won't have them in the next.

**Warning signs:**
- Different sessions producing wiki pages with different styles
- Having to re-explain preferences to the LLM each session
- Conventions that "everyone knows" but aren't written in the schema
- Quality regression after sessions where the schema wasn't updated

**Prevention strategy:**
- Treat CLAUDE.md as a living document: update it whenever a new convention is established
- After each productive session, ask: "What should we add to the schema based on today?"
- The schema should include examples of good output, not just rules
- Pin down style decisions explicitly: tone, heading levels, link density, summary length

**Phase to address:** Schema design (iterative schema evolution process)

---

## Prevention Matrix

| # | Pitfall | Warning Sign | Prevention | Phase |
|---|---------|-------------|------------|-------|
| P1 | Hallucinated cross-references | Broken wikilinks, untraceable claims | Mandatory source citations, lint link checking | Schema + Lint |
| P2 | Silent inconsistency across pages | Same fact with different values on different pages | Ripple check on ingest, cross-page lint | Ingest + Lint |
| P3 | Context window overflow | Shallow summaries, missed updates | Multi-step ingest workflow, never full-wiki context | Schema + Ingest |
| P4 | Index drift | Missing pages in index, stale summaries | Index rebuild in lint, mandatory index update on ingest | Schema + Lint |
| P5 | Wikilink format fragility | Links not resolving in Obsidian | Strict naming conventions, OS-aware testing | Schema |
| P6 | Over-engineered schema | Inconsistent compliance, empty frontmatter | Start minimal, add rules only when needed | Schema |
| P7 | Orphan pages and dead zones | Isolated nodes in graph view | Lint for zero-inbound pages, entity page creation | Lint |
| P8 | Unbounded log growth | Log exceeds 500 lines, LLM reads entire log | Parseable format, tail-only access, rotation | Schema |
| P9 | Query quality degradation | Missed relevant pages, shallow answers | Categorized index, hierarchical index at scale | Schema + Scale migration |
| P10 | Frontmatter plugin conflicts | Dataview errors, raw YAML display | Use Obsidian-standard fields, test with plugins | Schema |
| P11 | Ingest without review | Error propagation, entity conflation | One-at-a-time ingest, human review step | Workflow |
| P12 | Git history bloat | Hundreds of uninformative commits | Operation-level commits, descriptive messages | Workflow |
| P13 | Graph view becomes unreadable | Dense hairball graph | Meaningful links only, MOC pattern, tags | Maintenance |
| P14 | Source format variability | Inconsistent summary quality | Standardize source format, pre-ingest cleanup | Workflow |
| P15 | Session amnesia | Style drift across sessions | Update CLAUDE.md after every convention change | Schema |
