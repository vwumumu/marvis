---
phase: 2
plan: 1
title: "First Ingest — llmwiki.md"
wave: 1
depends_on: []
files_modified:
  - raw/articles/llm-wiki-concept.md
  - wiki/sources/llm-wiki-concept.md
  - wiki/entities/vannevar-bush.md
  - wiki/entities/obsidian.md
  - wiki/concepts/knowledge-compounding.md
  - wiki/concepts/retrieval-augmented-generation.md
  - wiki/concepts/memex.md
  - wiki/concepts/incremental-integration.md
  - index.md
  - log.md
requirements_addressed:
  - NGST-01
  - NGST-02
  - NGST-03
  - NGST-04
  - NGST-06
  - NGST-07
  - NGST-08
  - GITM-02
autonomous: true
estimated_context_cost: "medium"
---

## Objective

Perform the first ingest operation: copy the llmwiki.md idea document into raw/articles/, then follow the CLAUDE.md ingest workflow to create wiki pages (source summary, entity pages, concept pages), update index.md, append to log.md, and commit all changes. This tests the entire ingest pipeline end-to-end and populates the wiki with its first content.

## must_haves

- [ ] Source file exists at raw/articles/llm-wiki-concept.md (copy of ../llmwiki.md)
- [ ] Source summary page wiki/sources/llm-wiki-concept.md with correct frontmatter and citations
- [ ] Entity pages created: vannevar-bush.md, obsidian.md (at minimum)
- [ ] Concept pages created: knowledge-compounding.md, retrieval-augmented-generation.md, memex.md, incremental-integration.md (at minimum)
- [ ] Every claim in every wiki page cites its source (NGST-08)
- [ ] All pages have valid YAML frontmatter with 6 mandatory fields
- [ ] All wikilinks use format [[kebab-case|Display Name]] and resolve to existing files
- [ ] index.md updated with entries for all new pages under correct categories
- [ ] log.md appended with ingest entry in correct format
- [ ] Git commit captures all changes with descriptive message

## Tasks

<task id="1">
<title>Copy source document to raw/articles/</title>
<type>execute</type>
<read_first>
- ../llmwiki.md (the source document to ingest)
</read_first>
<action>
Copy ../llmwiki.md to raw/articles/llm-wiki-concept.md. Do not modify the content — raw sources are immutable.
</action>
<acceptance_criteria>
- File exists: raw/articles/llm-wiki-concept.md
- Content matches ../llmwiki.md exactly
</acceptance_criteria>
</task>

<task id="2">
<title>Create source summary page</title>
<type>execute</type>
<read_first>
- CLAUDE.md (Source Summary Template, editorial rules)
- raw/articles/llm-wiki-concept.md (the source to summarize)
</read_first>
<action>
Create wiki/sources/llm-wiki-concept.md following the Source Summary Template from CLAUDE.md.

Frontmatter:
- title: "LLM Wiki — A Pattern for Building Personal Knowledge Bases Using LLMs"
- type: source
- sources: [raw/articles/llm-wiki-concept.md]
- source_path: raw/articles/llm-wiki-concept.md
- tags: [knowledge-management, llm, wiki, personal-knowledge-base]
- created/updated: 2026-04-15

Content sections:
- Key Takeaways (5-7 bullet points covering core idea, architecture, operations, indexing, tips)
- Detailed Notes (prose summary of the document's main arguments)
- Related Pages (wikilinks to entity and concept pages created in subsequent tasks)
</action>
<acceptance_criteria>
- File exists: wiki/sources/llm-wiki-concept.md
- Grep for "type: source" returns match
- Grep for "raw/articles/llm-wiki-concept.md" returns match (source citation)
- Grep for "Key Takeaways" returns match
- Grep for "Related Pages" returns match
</acceptance_criteria>
</task>

<task id="3">
<title>Create entity pages</title>
<type>execute</type>
<read_first>
- CLAUDE.md (Entity Template, editorial rules)
- raw/articles/llm-wiki-concept.md (source for entity extraction)
</read_first>
<action>
Create entity pages for significant entities mentioned in the source:

1. **wiki/entities/vannevar-bush.md** — Historical figure who proposed the Memex (1945). Mentioned as intellectual predecessor to the LLM Wiki concept.
   - category: person
   - tags: [information-retrieval, memex, historical-figure]

2. **wiki/entities/obsidian.md** — Markdown-based knowledge management tool used as the browsing interface for the LLM Wiki.
   - category: tool
   - tags: [knowledge-management, markdown, tool]

Each entity page must:
- Follow the Entity Template from CLAUDE.md
- Have all 6 mandatory frontmatter fields plus aliases and category
- Cite the source for every claim: (source: [[llm-wiki-concept|LLM Wiki]])
- Include wikilinks to related entities and concepts
</action>
<acceptance_criteria>
- File exists: wiki/entities/vannevar-bush.md
- File exists: wiki/entities/obsidian.md
- All files have "type: entity" in frontmatter
- All files reference "raw/articles/llm-wiki-concept.md" in sources
- All claims include source citations
</acceptance_criteria>
</task>

<task id="4">
<title>Create concept pages</title>
<type>execute</type>
<read_first>
- CLAUDE.md (Concept Template, editorial rules)
- raw/articles/llm-wiki-concept.md (source for concept extraction)
</read_first>
<action>
Create concept pages for key ideas in the source:

1. **wiki/concepts/knowledge-compounding.md** — The core thesis: the wiki is a persistent, compounding artifact that gets richer with every source ingested.
   - tags: [knowledge-management, core-concept]
   - related: incremental-integration, memex

2. **wiki/concepts/retrieval-augmented-generation.md** — The conventional approach (RAG) that the LLM Wiki pattern contrasts with. Rediscovers knowledge from scratch on every query.
   - tags: [llm, information-retrieval]
   - related: knowledge-compounding

3. **wiki/concepts/memex.md** — Vannevar Bush's 1945 vision of a personal knowledge store with associative trails. The LLM Wiki is a modern realization.
   - tags: [information-retrieval, historical-concept]
   - related: knowledge-compounding, vannevar-bush

4. **wiki/concepts/incremental-integration.md** — The key differentiator: new sources update existing wiki pages rather than creating isolated entries. Knowledge is compiled once and kept current.
   - tags: [knowledge-management, core-concept]
   - related: knowledge-compounding

Each concept page must:
- Follow the Concept Template from CLAUDE.md
- Have all 6 mandatory frontmatter fields plus related
- Cite the source for every claim
- Include wikilinks to related concepts and entities
</action>
<acceptance_criteria>
- File exists: wiki/concepts/knowledge-compounding.md
- File exists: wiki/concepts/retrieval-augmented-generation.md
- File exists: wiki/concepts/memex.md
- File exists: wiki/concepts/incremental-integration.md
- All files have "type: concept" in frontmatter
- All files reference "raw/articles/llm-wiki-concept.md" in sources
- All claims include source citations
</acceptance_criteria>
</task>

<task id="5">
<title>Update index.md with new pages</title>
<type>execute</type>
<read_first>
- index.md (current state — empty categories)
- CLAUDE.md (Index Format section)
</read_first>
<action>
Add entries for all new wiki pages to index.md under the correct category headers:

## Sources
- [[llm-wiki-concept|LLM Wiki]] — A pattern for building personal knowledge bases using LLMs, with persistent wikis replacing RAG (2026-04-15)

## Entities
- [[vannevar-bush|Vannevar Bush]] — Scientist who proposed the Memex (1945), intellectual predecessor to the LLM Wiki
- [[obsidian|Obsidian]] — Markdown-based knowledge management tool used as the wiki browsing interface

## Concepts
- [[knowledge-compounding|Knowledge Compounding]] — Core thesis: the wiki is a persistent, compounding artifact
- [[retrieval-augmented-generation|Retrieval-Augmented Generation]] — Conventional LLM approach that rediscovers knowledge on every query
- [[memex|Memex]] — Vannevar Bush's 1945 vision of a personal knowledge store with associative trails
- [[incremental-integration|Incremental Integration]] — Key differentiator: new sources update existing pages, not create isolated entries
</action>
<acceptance_criteria>
- Grep for "llm-wiki-concept" in index.md returns match
- Grep for "vannevar-bush" in index.md returns match
- Grep for "obsidian" in index.md returns match
- Grep for "knowledge-compounding" in index.md returns match
- Grep for "retrieval-augmented-generation" in index.md returns match
- Grep for "memex" in index.md returns match
- Grep for "incremental-integration" in index.md returns match
</acceptance_criteria>
</task>

<task id="6">
<title>Append ingest entry to log.md</title>
<type>execute</type>
<read_first>
- log.md (current state)
- CLAUDE.md (Log Format section)
</read_first>
<action>
Append an ingest entry to log.md:

```markdown
## [2026-04-15] ingest | LLM Wiki — A Pattern for Building Personal Knowledge Bases Using LLMs

Ingested raw/articles/llm-wiki-concept.md. Pages created: [[llm-wiki-concept|LLM Wiki]] (source), [[vannevar-bush|Vannevar Bush]] (entity), [[obsidian|Obsidian]] (entity), [[knowledge-compounding|Knowledge Compounding]] (concept), [[retrieval-augmented-generation|RAG]] (concept), [[memex|Memex]] (concept), [[incremental-integration|Incremental Integration]] (concept). Pages updated: none (first ingest).
```
</action>
<acceptance_criteria>
- Grep for "ingest | LLM Wiki" in log.md returns match
- Grep for "llm-wiki-concept" in log.md returns match
- Entry follows the ## [YYYY-MM-DD] operation | subject format
</acceptance_criteria>
</task>

<task id="7">
<title>Git commit all first-ingest changes</title>
<type>execute</type>
<action>
Stage and commit all changes from the first ingest:
- raw/articles/llm-wiki-concept.md
- wiki/sources/llm-wiki-concept.md
- wiki/entities/vannevar-bush.md
- wiki/entities/obsidian.md
- wiki/concepts/knowledge-compounding.md
- wiki/concepts/retrieval-augmented-generation.md
- wiki/concepts/memex.md
- wiki/concepts/incremental-integration.md
- index.md
- log.md

Commit message: `ingest: LLM Wiki — created 7 pages, updated 0 pages`
</action>
<acceptance_criteria>
- git log shows commit with "ingest: LLM Wiki" in message
- git status shows clean working tree (no unstaged wiki files)
</acceptance_criteria>
</task>
