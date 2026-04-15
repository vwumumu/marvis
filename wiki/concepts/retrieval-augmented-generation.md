---
title: "Retrieval-Augmented Generation"
type: concept
sources:
  - raw/articles/llm-wiki-concept.md
related:
  - "[[knowledge-compounding|Knowledge Compounding]]"
tags:
  - llm
  - information-retrieval
created: 2026-04-15
updated: 2026-04-15
---

# Retrieval-Augmented Generation

## Definition

Retrieval-Augmented Generation (RAG) is the conventional approach to using LLMs with document collections. The user uploads files, the LLM retrieves relevant chunks at query time, and generates an answer. The LLM rediscovers knowledge from scratch on every question — there is no accumulation (source: [[llm-wiki-concept|LLM Wiki]]).

## Key Points

- RAG is the dominant paradigm: NotebookLM, ChatGPT file uploads, and most document QA systems use this approach (source: [[llm-wiki-concept|LLM Wiki]])
- The fundamental limitation: ask a subtle question that requires synthesizing five documents, and the LLM must find and piece together the relevant fragments every time (source: [[llm-wiki-concept|LLM Wiki]])
- Nothing is built up across queries — each interaction starts from the raw documents (source: [[llm-wiki-concept|LLM Wiki]])
- The LLM Wiki pattern explicitly contrasts with RAG by replacing retrieval-at-query-time with [[incremental-integration|incremental compilation]] into a persistent wiki (source: [[llm-wiki-concept|LLM Wiki]])
- At moderate scale (~100 sources, ~hundreds of pages), a well-maintained index.md eliminates the need for embedding-based RAG infrastructure entirely (source: [[llm-wiki-concept|LLM Wiki]])

## Sources

- [[llm-wiki-concept|LLM Wiki]] — describes RAG as the approach the LLM Wiki pattern contrasts with

## Related Concepts

- [[knowledge-compounding|Knowledge Compounding]] — the alternative paradigm where knowledge accumulates
- [[incremental-integration|Incremental Integration]] — the mechanism that replaces RAG's retrieve-every-time approach
