---
title: "Knowledge Compounding"
type: concept
sources:
  - raw/articles/llm-wiki-concept.md
related:
  - "[[incremental-integration|Incremental Integration]]"
  - "[[memex|Memex]]"
  - "[[retrieval-augmented-generation|Retrieval-Augmented Generation]]"
tags:
  - knowledge-management
  - core-concept
created: 2026-04-15
updated: 2026-04-15
---

# Knowledge Compounding

## Definition

Knowledge compounding is the core thesis of the LLM Wiki pattern: the wiki is a **persistent, compounding artifact** that gets richer with every source ingested and every question asked. Cross-references are pre-built, contradictions are pre-flagged, and synthesis reflects everything that has been read (source: [[llm-wiki-concept|LLM Wiki]]).

## Key Points

- Contrasts with [[retrieval-augmented-generation|RAG]], where knowledge is re-derived from scratch on every query — nothing accumulates (source: [[llm-wiki-concept|LLM Wiki]])
- Each new source doesn't just get indexed — the LLM reads it, extracts information, and integrates it into existing wiki pages via [[incremental-integration|incremental integration]] (source: [[llm-wiki-concept|LLM Wiki]])
- The wiki keeps getting richer because the LLM handles the bookkeeping that humans typically abandon: updating cross-references, maintaining consistency, noting contradictions (source: [[llm-wiki-concept|LLM Wiki]])
- Query answers can be filed back into the wiki as new pages, so explorations compound just like ingested sources do (source: [[llm-wiki-concept|LLM Wiki]])
- The concept traces back to [[vannevar-bush|Vannevar Bush]]'s [[memex|Memex]] (1945) — a personal knowledge store where connections between documents are as valuable as the documents themselves (source: [[llm-wiki-concept|LLM Wiki]])

## Sources

- [[llm-wiki-concept|LLM Wiki]] — defines knowledge compounding as the core value proposition

## Related Concepts

- [[incremental-integration|Incremental Integration]] — the mechanism that enables compounding
- [[memex|Memex]] — the historical precedent for personal compounding knowledge stores
- [[retrieval-augmented-generation|Retrieval-Augmented Generation]] — the non-compounding alternative
