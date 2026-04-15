# Marvis — LLM Wiki

## What This Is

A personal knowledge base system powered by LLMs. Instead of traditional RAG (retrieve-and-generate on every query), the LLM incrementally builds and maintains a persistent, interlinked wiki of markdown files. When new sources are added, the LLM reads, extracts, and integrates knowledge into existing wiki pages — updating entities, revising summaries, flagging contradictions, and strengthening the evolving synthesis. The wiki is a compounding artifact that gets richer with every source ingested and every question asked.

## Core Value

**The wiki is a persistent, compounding knowledge artifact** — cross-references are pre-built, contradictions pre-flagged, and synthesis reflects everything ingested. Knowledge is compiled once and kept current, not re-derived on every query.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Three-layer architecture: raw sources (immutable), wiki (LLM-maintained markdown), schema (CLAUDE.md conventions)
- [ ] Ingest operation: process a new source → write summary → update index → update entity/concept pages → append to log
- [ ] Query operation: search wiki via index → read relevant pages → synthesize answer with citations → optionally file answer back as wiki page
- [ ] Lint operation: health-check wiki for contradictions, stale claims, orphan pages, missing cross-references, data gaps
- [ ] Index file (index.md): content-oriented catalog of all wiki pages with links, summaries, and metadata
- [ ] Log file (log.md): chronological append-only record of all operations (ingests, queries, lints)
- [ ] Obsidian-compatible markdown with wikilinks for cross-referencing
- [ ] Schema file (CLAUDE.md) defining wiki conventions, page formats, and workflows

### Out of Scope

- Embedding-based RAG infrastructure — index.md is sufficient at moderate scale (~100 sources)
- Real-time collaboration / multi-user — this is a personal knowledge base
- Web UI / custom frontend — Obsidian is the IDE, the LLM is the programmer
- Mobile app — desktop-first workflow
- Automated source ingestion — user curates and triggers ingestion manually

## Context

- Inspired by Vannevar Bush's Memex (1945) — private, actively curated knowledge store with associative trails
- The key insight: humans abandon wikis because maintenance burden grows faster than value; LLMs don't get bored and can touch 15+ files in one pass
- Target scale: ~100 sources, ~hundreds of wiki pages — no need for vector search at this scale
- Obsidian is the reading/browsing interface; Claude Code is the writing/maintenance engine
- The wiki is a git repo of markdown files — version history, branching, and collaboration come free
- Potential use cases: personal development, research deep-dives, book companions, business knowledge, competitive analysis

## Constraints

- **Format**: Obsidian-compatible markdown with YAML frontmatter and `[[wikilinks]]`
- **Storage**: Local filesystem, git-tracked
- **Interface**: Claude Code as the LLM interface, Obsidian as the browsing interface
- **Sources**: Text-based (markdown, text files); images handled separately via Obsidian attachments
- **Scale**: Designed for moderate scale (~100 sources, ~hundreds of pages); index-based navigation, no vector DB

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Obsidian-compatible markdown | Leverage existing ecosystem (graph view, Dataview, Marp) | — Pending |
| Index-based navigation over RAG | Simpler architecture, sufficient at target scale | — Pending |
| CLAUDE.md as schema | Co-evolves with wiki, native to Claude Code workflow | — Pending |
| Git-tracked wiki | Free version history, diffing, and branching | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-15 after initialization*
