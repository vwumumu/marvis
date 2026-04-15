# Features Research
**Domain:** LLM-powered personal knowledge base wiki
**Researched:** 2026-04-15

## Table Stakes

Features users expect from any personal knowledge base. Without these, the system is unusable or abandoned within a week.

### 1. Source ingestion with summary generation
- User adds a document, system produces a readable summary page
- This is the minimum viable interaction — if adding sources doesn't produce visible value immediately, users won't add a second source
- **Complexity:** Low. Single LLM call, single file write. The hard part is deciding what to extract, not the mechanics.

### 2. Cross-referencing and linking
- Wiki pages link to each other via `[[wikilinks]]` or equivalent
- Users expect to click from a concept to its entity page, from a summary to related summaries
- Without links the wiki is just a folder of files — no better than a messy notes app
- **Complexity:** Medium. The LLM must know what pages exist (via index) and decide which links to create. Risk of over-linking (noise) or under-linking (missed connections).

### 3. Search / navigation
- Users must be able to find things. At minimum: an index page organized by category. At scale: full-text search.
- The index.md approach in Marvis is sufficient for ~100 sources but this is a ceiling that must be acknowledged
- **Complexity:** Low for index-based. Medium-high if adding search tooling (qmd, embeddings, etc).

### 4. Consistent page structure
- Every wiki page should follow a predictable format (frontmatter, sections, links)
- Users learn to scan pages quickly when structure is consistent; inconsistency erodes trust
- **Complexity:** Low. This is a schema/prompt engineering problem. The CLAUDE.md convention handles it.

### 5. Source preservation (immutability)
- Raw sources must never be modified by the system
- Users need confidence that the LLM's interpretation layer doesn't corrupt their originals
- **Complexity:** Low. Architectural convention enforced by directory structure and instructions.

### 6. Operation logging
- Users need a record of what happened: what was ingested, when, what changed
- Without this, the wiki feels like a black box — pages change and you don't know why
- **Complexity:** Low. Append-only log file. The discipline is in making entries useful (timestamps, affected pages, source references).

### 7. Readable output in a familiar interface
- The wiki must be browsable in a tool users already know (Obsidian, VS Code, a file browser)
- Building a custom UI is a trap — it adds months of work and competes with polished existing tools
- **Complexity:** Low. Obsidian-compatible markdown is the right call. The format constraints (YAML frontmatter, wikilinks) are well-documented.

### 8. Version history
- Users need the ability to see what changed and roll back mistakes
- The LLM will occasionally produce bad updates; recovery must be trivial
- **Complexity:** Low. Git provides this for free. The key decision is granularity — commit per operation vs. commit per session.

## Differentiators

Features that separate Marvis from NotebookLM, standard RAG systems, Notion AI, and manual note-taking.

### D1. Incremental integration (not just retrieval)
- **What:** When a new source arrives, the LLM doesn't just index it — it updates existing pages across the wiki. Entity pages get new facts. Concept pages get revised. Contradictions get flagged. A single ingest can touch 10-15 files.
- **Why it's different:** NotebookLM and RAG systems treat sources as static retrieval targets. They never update a synthesis based on new information — they re-derive everything at query time. Marvis compiles knowledge once and keeps it current.
- **Complexity:** High. This is the hardest feature. The LLM must: (a) understand what already exists, (b) decide what needs updating, (c) make surgical edits without breaking existing content, (d) maintain consistency across many files. The quality of this operation defines the system.

### D2. Compounding queries (answers become wiki pages)
- **What:** When you ask a question and get a good answer, that answer can be filed as a new wiki page — with links, citations, and its own place in the index. Your explorations become permanent knowledge.
- **Why it's different:** In ChatGPT/NotebookLM, good answers vanish into chat history. In Marvis, they compound. A comparison table you asked for last month is still there, linked from the relevant entity pages.
- **Complexity:** Medium. The mechanics are simple (write a page, update index). The hard part is deciding which answers are worth filing and where they belong in the wiki structure.

### D3. Contradiction and staleness detection (lint)
- **What:** The system can audit itself — finding where newer sources contradict older claims, where pages have gone stale, where cross-references are missing, where concepts are mentioned but lack their own page.
- **Why it's different:** No consumer knowledge base does this. Notion AI can summarize but can't tell you "page X says the opposite of page Y based on sources from different dates." This is a unique capability of the persistent-wiki pattern.
- **Complexity:** High. Requires the LLM to read many pages in context and make nuanced judgments about consistency. False positives (flagging non-contradictions) will erode trust. Needs careful prompt engineering and possibly human-in-the-loop confirmation.

### D4. LLM as sole maintainer (zero human writing burden)
- **What:** The human never writes wiki pages. They curate sources, ask questions, and review output. The LLM handles all summarizing, cross-referencing, filing, and bookkeeping.
- **Why it's different:** Obsidian/Notion/Roam require the user to do the writing. "I'll organize my notes later" is a promise no one keeps. Marvis eliminates the maintenance burden that kills every personal wiki.
- **Complexity:** Low in mechanics, high in trust-building. Users must trust the LLM's output enough to stop wanting to edit it manually. This is a UX/quality problem, not a technical one.

### D5. Schema co-evolution (CLAUDE.md as living configuration)
- **What:** The wiki's conventions, page formats, and workflows are defined in a schema file that evolves as the wiki grows. The LLM follows the schema, and user and LLM refine it together over time.
- **Why it's different:** Most tools have fixed schemas. Marvis adapts its structure to the domain — a research wiki has different conventions than a book companion wiki or a personal development tracker.
- **Complexity:** Low-medium. The file itself is simple. The challenge is knowing when to evolve the schema vs. when stability matters more.

### D6. Multi-format output from wiki content
- **What:** The wiki's structured content can be rendered as slide decks (Marp), data tables (Dataview), graph visualizations (Obsidian graph view), and potentially charts or other formats.
- **Why it's different:** RAG systems return text. Marvis's structured, frontmatter-rich pages enable programmatic queries and alternative presentations of the same knowledge.
- **Complexity:** Low-medium. Leverages Obsidian plugins. The LLM just needs to produce compatible formats (Marp syntax, proper YAML frontmatter).

### D7. Transparent knowledge graph (human-readable)
- **What:** The entire knowledge base is plain markdown files on disk. There's no opaque database, no proprietary format, no vendor lock-in. Users can read every page, inspect every link, and understand the full structure.
- **Why it's different:** NotebookLM's knowledge graph is invisible. Vector databases are opaque. Marvis's knowledge is human-auditable at every level — you can grep it, diff it, read it in any text editor.
- **Complexity:** Low. This is an architectural choice, not a feature to build. The discipline is in keeping markdown files clean and well-structured.

## Anti-Features

Things to deliberately NOT build, with reasoning.

### A1. DO NOT build embedding-based RAG
- **Why not:** The entire point of Marvis is that knowledge is compiled into wiki pages, not re-derived at query time via vector similarity search. Adding RAG undermines the core value proposition. At target scale (~100 sources, ~hundreds of pages), the index file is sufficient for navigation. If scale demands search later, use a simple tool like qmd rather than building RAG infrastructure.
- **Risk of building it:** Architectural complexity explosion. Two retrieval paths to maintain. Users stop trusting the wiki because "the RAG finds things the wiki missed."

### A2. DO NOT build a custom web UI
- **Why not:** Obsidian is a mature, extensible markdown reader with graph view, Dataview, community plugins, and mobile apps. Building a custom frontend means competing with a polished product on every dimension — and losing. Every hour spent on UI is an hour not spent on wiki quality.
- **Risk of building it:** 6+ months of frontend work. Constant maintenance. Users compare it unfavorably to Obsidian anyway.

### A3. DO NOT build real-time collaboration / multi-user
- **Why not:** This is a personal knowledge base. Multi-user support means: access control, conflict resolution, notification systems, permission models, shared vs. private pages. Each adds massive complexity. The wiki is a git repo — collaboration comes free via branches/PRs if ever needed.
- **Risk of building it:** Scope explosion. The system tries to be Notion and fails at being a great personal tool.

### A4. DO NOT build automated/scheduled ingestion
- **Why not:** The human curating sources is a feature, not a bug. Automated ingestion (RSS feeds, email inboxes, Slack streams) floods the wiki with low-quality content and removes the human judgment that makes the knowledge base valuable. The user should deliberately choose what enters the system.
- **Risk of building it:** Wiki quality collapses. Signal-to-noise ratio drops. The system becomes a dump rather than a curated knowledge base.

### A5. DO NOT build chat-based conversational UI
- **Why not:** Claude Code already is the conversational interface. Building a chat UI on top is redundant. The interaction model is: talk to the LLM in Claude Code, browse results in Obsidian. Adding another chat interface fragments the experience.
- **Risk of building it:** Users now have two places to type questions and neither is great.

### A6. DO NOT build fine-tuning or model training
- **Why not:** The wiki IS the model's "memory" — it's a structured, human-readable knowledge store that any LLM can read. Fine-tuning couples the knowledge to a specific model version and makes it opaque. The wiki's portability across models is a strength.
- **Risk of building it:** Vendor lock-in. Knowledge becomes opaque. Update cycles become model retraining cycles.

### A7. DO NOT build complex permission/sharing models
- **Why not:** It's files on disk. Share them however you share files (git, Synology Drive, Dropbox, email). Adding application-level sharing is reinventing wheels that already spin.

### A8. DO NOT build a plugin system
- **Why not:** Obsidian already has a plugin system. The LLM's behavior is configured via CLAUDE.md, not plugins. Adding plugin architecture means maintaining an API contract, handling plugin conflicts, and supporting third-party code. The system should stay simple: files, LLM, schema.

## Feature Dependencies

```
Source ingestion (1)
  ├── requires: Consistent page structure (4)
  ├── requires: Source preservation (5)
  ├── requires: Operation logging (6)
  ├── enables: Cross-referencing (2)
  └── enables: Incremental integration (D1)

Cross-referencing (2)
  ├── requires: Search/navigation via index (3)
  └── enables: Contradiction detection (D3)

Search/navigation (3)
  ├── requires: Consistent page structure (4)
  └── enables: Compounding queries (D2)

Incremental integration (D1)
  ├── requires: Source ingestion (1)
  ├── requires: Cross-referencing (2)
  ├── requires: Search/navigation (3)
  └── enables: Contradiction detection (D3)

Compounding queries (D2)
  ├── requires: Search/navigation (3)
  ├── requires: Cross-referencing (2)
  └── requires: Operation logging (6)

Contradiction detection (D3)
  ├── requires: Incremental integration (D1)
  ├── requires: Cross-referencing (2)
  └── requires: Consistent page structure (4)

Schema co-evolution (D5)
  └── independent (but improves all other features)

Multi-format output (D6)
  ├── requires: Consistent page structure (4)
  └── requires: YAML frontmatter conventions

Version history (8)
  └── independent (git provides this)
```

**Critical path for MVP:** Consistent page structure (4) -> Source preservation (5) -> Source ingestion (1) -> Cross-referencing (2) -> Search/navigation (3) -> Operation logging (6)

**Critical path for differentiation:** MVP + Incremental integration (D1) -> Compounding queries (D2) -> Contradiction detection (D3)

## Complexity Assessment

| Feature | Complexity | Effort | Risk | Notes |
|---------|-----------|--------|------|-------|
| Source ingestion (1) | Low | 1-2 days | Low | Single LLM call + file writes. Well-understood pattern. |
| Cross-referencing (2) | Medium | 2-3 days | Medium | Over/under-linking risk. Needs iteration on prompt quality. |
| Search/navigation (3) | Low | 1 day | Low | Index.md is sufficient at target scale. |
| Consistent page structure (4) | Low | 1 day | Low | Schema definition in CLAUDE.md. One-time design. |
| Source preservation (5) | Low | <1 day | Low | Directory convention. No code needed. |
| Operation logging (6) | Low | <1 day | Low | Append-only file. Trivial. |
| Readable output (7) | Low | <1 day | Low | Obsidian-compatible markdown. Format constraints are simple. |
| Version history (8) | Low | <1 day | Low | Git. Already decided. |
| **Incremental integration (D1)** | **High** | **1-2 weeks** | **High** | **Core differentiator. Hardest feature. LLM must decide what to update across many files without breaking consistency. Quality here defines the product.** |
| Compounding queries (D2) | Medium | 2-3 days | Medium | Mechanics are simple. Deciding what to file and where is the judgment call. |
| Contradiction detection (D3) | High | 1-2 weeks | High | Requires nuanced LLM reasoning across many pages. False positive rate is the key metric. |
| Zero writing burden (D4) | N/A | Ongoing | Medium | Quality problem, not a feature to build. Improves as prompts improve. |
| Schema co-evolution (D5) | Low-Med | 1-2 days | Low | Living document. Evolves naturally through use. |
| Multi-format output (D6) | Low-Med | 2-3 days | Low | Leverages Obsidian plugins. LLM produces compatible syntax. |
| Transparent knowledge graph (D7) | Low | 0 days | Low | Architectural choice, not a feature. Already decided. |

**Summary:** The table stakes features are all low complexity and can be built in the first phase. The differentiators split into two tiers:
- **Tier 1 (must nail):** Incremental integration (D1) is the entire value proposition. If this doesn't work well, the system is just a fancy summarizer. This deserves the most iteration time.
- **Tier 2 (strong differentiators):** Compounding queries (D2) and contradiction detection (D3) are what make the wiki compound over time. They can ship after D1 is solid.
- **Tier 3 (nice to have):** Multi-format output (D6) and schema co-evolution (D5) add polish but aren't why someone chooses this over NotebookLM.
