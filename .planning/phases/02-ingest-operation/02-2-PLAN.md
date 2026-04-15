---
phase: 2
plan: 2
title: "Second Ingest — Test Incremental Integration"
wave: 2
depends_on: [1]
files_modified:
  - raw/articles/as-we-may-think-summary.md
  - wiki/sources/as-we-may-think-summary.md
  - wiki/entities/vannevar-bush.md
  - wiki/concepts/memex.md
  - wiki/concepts/associative-trails.md
  - index.md
  - log.md
requirements_addressed:
  - NGST-05
autonomous: true
estimated_context_cost: "medium"
---

## Objective

Perform the second ingest to test incremental integration (NGST-05 — the core differentiator D1). Ingest a summary of Vannevar Bush's "As We May Think" article, which shares entities (Vannevar Bush) and concepts (Memex) with the first ingest. Verify that existing entity and concept pages are UPDATED with new information rather than duplicated.

## must_haves

- [ ] Source file exists at raw/articles/as-we-may-think-summary.md
- [ ] Source summary page wiki/sources/as-we-may-think-summary.md created with correct frontmatter
- [ ] wiki/entities/vannevar-bush.md is UPDATED (not recreated) — sources list now includes both sources, updated date changes, new information integrated
- [ ] wiki/concepts/memex.md is UPDATED (not recreated) — sources list now includes both sources, new details added
- [ ] At least one new page created from the second source (e.g., associative-trails concept)
- [ ] Every claim in updated pages cites the correct source
- [ ] index.md updated with new pages; existing entries' summaries updated if needed
- [ ] log.md appended with second ingest entry
- [ ] Git commit captures all second-ingest changes

## Tasks

<task id="1">
<title>Create second test source document</title>
<type>execute</type>
<action>
Create raw/articles/as-we-may-think-summary.md — a concise summary of Vannevar Bush's 1945 essay "As We May Think" published in The Atlantic. This serves as a test source that overlaps with the first ingest's entities and concepts.

The content should cover:
- Vannevar Bush authored the essay in 1945 while serving as Director of the Office of Scientific Research and Development
- The essay proposed the Memex — a mechanized desk that stores books, records, and communications on microfilm
- The Memex would allow users to create "associative trails" — linked paths between documents, a concept that prefigured hyperlinks
- Bush argued that the growing volume of scientific knowledge was outpacing humanity's ability to organize and access it
- The essay influenced Douglas Engelbart, Ted Nelson (who coined "hypertext"), and the development of the World Wide Web
- The essay was published in The Atlantic Monthly, July 1945

This is a created summary, not an actual article copy. It provides enough factual content to test entity/concept updates.
</action>
<acceptance_criteria>
- File exists: raw/articles/as-we-may-think-summary.md
- Content mentions Vannevar Bush, Memex, associative trails
</acceptance_criteria>
</task>

<task id="2">
<title>Create source summary for second source</title>
<type>execute</type>
<read_first>
- CLAUDE.md (Source Summary Template)
- raw/articles/as-we-may-think-summary.md
</read_first>
<action>
Create wiki/sources/as-we-may-think-summary.md following the Source Summary Template.

Frontmatter:
- title: "As We May Think — Summary"
- type: source
- sources: [raw/articles/as-we-may-think-summary.md]
- source_path: raw/articles/as-we-may-think-summary.md
- tags: [information-retrieval, memex, historical]
- created/updated: 2026-04-15
</action>
<acceptance_criteria>
- File exists: wiki/sources/as-we-may-think-summary.md
- Grep for "type: source" returns match
- Grep for "raw/articles/as-we-may-think-summary.md" in sources
</acceptance_criteria>
</task>

<task id="3">
<title>Update existing entity and concept pages (incremental integration)</title>
<type>execute</type>
<read_first>
- CLAUDE.md (editorial rules, especially incremental integration rule)
- wiki/entities/vannevar-bush.md (existing — needs update)
- wiki/concepts/memex.md (existing — needs update)
- raw/articles/as-we-may-think-summary.md (new source)
</read_first>
<action>
Update existing pages with new information from the second source:

1. **wiki/entities/vannevar-bush.md** — ADD:
   - New source to frontmatter sources list: raw/articles/as-we-may-think-summary.md
   - Update `updated` date
   - Add detail: Director of the Office of Scientific Research and Development
   - Add "As We May Think" to Appearances in Sources
   - Cite the new source for new claims

2. **wiki/concepts/memex.md** — ADD:
   - New source to frontmatter sources list
   - Update `updated` date
   - Add detail: mechanized desk with microfilm storage
   - Add detail about associative trails
   - Cite the new source for new claims

Do NOT recreate these pages. Read the existing content and integrate new information.
</action>
<acceptance_criteria>
- vannevar-bush.md frontmatter sources list contains BOTH source paths
- memex.md frontmatter sources list contains BOTH source paths
- New claims cite [[as-we-may-think-summary|As We May Think]]
- Old claims still cite [[llm-wiki-concept|LLM Wiki]]
- No information from the first source was lost
</acceptance_criteria>
</task>

<task id="4">
<title>Create new concept page from second source</title>
<type>execute</type>
<read_first>
- CLAUDE.md (Concept Template)
- raw/articles/as-we-may-think-summary.md
</read_first>
<action>
Create wiki/concepts/associative-trails.md — a concept unique to the second source.

- title: "Associative Trails"
- type: concept
- sources: [raw/articles/as-we-may-think-summary.md]
- tags: [information-retrieval, memex, hypertext]
- related: memex, knowledge-compounding
</action>
<acceptance_criteria>
- File exists: wiki/concepts/associative-trails.md
- Grep for "type: concept" returns match
- Source citations present
</acceptance_criteria>
</task>

<task id="5">
<title>Update index.md and log.md</title>
<type>execute</type>
<read_first>
- index.md (current state after first ingest)
- log.md (current state after first ingest)
- CLAUDE.md (Index Format, Log Format)
</read_first>
<action>
1. Add to index.md:
   - Under Sources: entry for as-we-may-think-summary
   - Under Concepts: entry for associative-trails
   - Update summaries for vannevar-bush and memex if they changed significantly

2. Append to log.md:
   ```
   ## [2026-04-15] ingest | As We May Think — Summary
   Ingested raw/articles/as-we-may-think-summary.md. Pages created: [[as-we-may-think-summary|As We May Think]] (source), [[associative-trails|Associative Trails]] (concept). Pages updated: [[vannevar-bush|Vannevar Bush]] (entity), [[memex|Memex]] (concept).
   ```
</action>
<acceptance_criteria>
- Grep for "as-we-may-think-summary" in index.md returns match
- Grep for "associative-trails" in index.md returns match
- Grep for "ingest | As We May Think" in log.md returns match
</acceptance_criteria>
</task>

<task id="6">
<title>Git commit all second-ingest changes</title>
<type>execute</type>
<action>
Stage and commit all changes from the second ingest.
Commit message: `ingest: As We May Think — created 2 pages, updated 2 pages`
</action>
<acceptance_criteria>
- git log shows commit with "ingest: As We May Think" in message
- git status shows clean working tree
</acceptance_criteria>
</task>
