---
title: "LLM Wiki — A Pattern for Building Personal Knowledge Bases Using LLMs"
type: source
sources:
  - raw/articles/llm-wiki-concept.md
source_path: raw/articles/llm-wiki-concept.md
tags:
  - knowledge-management
  - llm
  - wiki
  - personal-knowledge-base
created: 2026-04-15
updated: 2026-04-15
---

# LLM Wiki — A Pattern for Building Personal Knowledge Bases Using LLMs

## Key Takeaways

- The LLM incrementally builds and maintains a persistent wiki rather than using RAG to re-derive answers on every query (source: [[llm-wiki-concept|LLM Wiki]])
- The wiki is a **persistent, compounding artifact** — cross-references are pre-built, contradictions are pre-flagged, and synthesis reflects everything ingested (source: [[llm-wiki-concept|LLM Wiki]])
- The architecture has three layers: raw sources (immutable), the wiki (LLM-maintained), and the schema (co-evolved by user and LLM) (source: [[llm-wiki-concept|LLM Wiki]])
- Three core operations drive the system: **ingest** (process new sources into wiki pages), **query** (answer questions from wiki content), and **lint** (health-check for consistency and completeness) (source: [[llm-wiki-concept|LLM Wiki]])
- Two navigation files support the system: **index.md** (content-oriented catalog) and **log.md** (chronological operation record) (source: [[llm-wiki-concept|LLM Wiki]])
- The pattern applies broadly: personal development, research, book reading, business knowledge, competitive analysis (source: [[llm-wiki-concept|LLM Wiki]])
- The concept is related in spirit to [[vannevar-bush|Vannevar Bush]]'s [[memex|Memex]] (1945) — a personal, curated knowledge store with associative trails between documents (source: [[llm-wiki-concept|LLM Wiki]])

## Detailed Notes

The core insight is that most LLM-document interactions follow the RAG pattern: upload files, retrieve relevant chunks at query time, generate an answer. This works but involves rediscovering knowledge from scratch on every question. Nothing accumulates (source: [[llm-wiki-concept|LLM Wiki]]).

The LLM Wiki pattern inverts this. Instead of retrieving at query time, the LLM **compiles knowledge incrementally** into a persistent wiki. When a new source is added, the LLM reads it, extracts key information, and integrates it into existing wiki pages — updating entity pages, revising topic summaries, noting contradictions, and strengthening the evolving synthesis (source: [[llm-wiki-concept|LLM Wiki]]).

The human's role is to curate sources, direct analysis, ask good questions, and think about what it all means. The LLM handles the bookkeeping that makes a knowledge base useful over time — the summarizing, cross-referencing, filing, and maintenance that humans typically abandon (source: [[llm-wiki-concept|LLM Wiki]]).

In practice, the workflow uses [[obsidian|Obsidian]] as the browsing interface and an LLM agent (such as Claude Code) as the writing engine. The LLM makes edits; the user browses results in real time via Obsidian's graph view and page navigation (source: [[llm-wiki-concept|LLM Wiki]]).

A key technical insight: at moderate scale (~100 sources, ~hundreds of pages), a well-maintained index.md eliminates the need for embedding-based [[retrieval-augmented-generation|RAG]] infrastructure. The LLM reads the index first to find relevant pages, then drills into them (source: [[llm-wiki-concept|LLM Wiki]]).

## Related Pages

- [[vannevar-bush|Vannevar Bush]] — proposed the Memex, the intellectual predecessor to the LLM Wiki pattern
- [[obsidian|Obsidian]] — the browsing interface for the wiki
- [[knowledge-compounding|Knowledge Compounding]] — the core thesis of the LLM Wiki pattern
- [[retrieval-augmented-generation|Retrieval-Augmented Generation]] — the conventional approach that the LLM Wiki contrasts with
- [[memex|Memex]] — the historical concept that prefigured the LLM Wiki
- [[incremental-integration|Incremental Integration]] — the key mechanism that makes the wiki a compounding artifact
