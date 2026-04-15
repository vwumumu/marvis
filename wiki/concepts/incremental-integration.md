---
title: "Incremental Integration"
type: concept
sources:
  - raw/articles/llm-wiki-concept.md
related:
  - "[[knowledge-compounding|Knowledge Compounding]]"
  - "[[retrieval-augmented-generation|Retrieval-Augmented Generation]]"
tags:
  - knowledge-management
  - core-concept
created: 2026-04-15
updated: 2026-04-15
---

# Incremental Integration

## Definition

Incremental integration is the key mechanism of the LLM Wiki pattern: when a new source is added, the LLM doesn't just index it for later retrieval — it reads the source, extracts key information, and **integrates it into existing wiki pages**. Entity pages are updated, topic summaries are revised, contradictions with existing content are noted, and the evolving synthesis is strengthened (source: [[llm-wiki-concept|LLM Wiki]]).

## Key Points

- This is the core differentiator from [[retrieval-augmented-generation|RAG]]: knowledge is compiled once and kept current, not re-derived on every query (source: [[llm-wiki-concept|LLM Wiki]])
- A single ingest may touch 10-15+ wiki pages as the LLM updates entity pages, concept pages, and cross-references across the wiki (source: [[llm-wiki-concept|LLM Wiki]])
- The process preserves existing knowledge: when new information contradicts old claims, both are kept with citations — the LLM never silently overwrites (source: [[llm-wiki-concept|LLM Wiki]])
- Incremental integration is what makes the wiki a [[knowledge-compounding|compounding artifact]]: each source builds on everything that came before (source: [[llm-wiki-concept|LLM Wiki]])
- The LLM handles the bookkeeping — summarizing, cross-referencing, filing, and maintenance — that humans typically abandon in manual knowledge bases (source: [[llm-wiki-concept|LLM Wiki]])

## Sources

- [[llm-wiki-concept|LLM Wiki]] — defines incremental integration as the key differentiator of the pattern

## Related Concepts

- [[knowledge-compounding|Knowledge Compounding]] — the outcome that incremental integration enables
- [[retrieval-augmented-generation|Retrieval-Augmented Generation]] — the non-incremental alternative
