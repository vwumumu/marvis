# Phase 1: Schema & Scaffolding - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-15
**Phase:** 01-Schema & Scaffolding
**Areas discussed:** Schema depth, Wikilink naming, Index structure, Frontmatter fields
**Mode:** Auto (all decisions auto-selected)

---

## Schema Depth

| Option | Description | Selected |
|--------|-------------|----------|
| Minimal start | Only include rules that lint can enforce; add complexity incrementally | ✓ |
| Comprehensive | Define all possible fields, rules, and edge cases upfront | |
| Middle ground | Core rules comprehensive, edge cases deferred | |

**User's choice:** [auto] Minimal start (recommended default — per pitfall P6)
**Notes:** Research strongly recommends starting minimal. Every rule must be lint-enforceable.

---

## Wikilink Naming Convention

| Option | Description | Selected |
|--------|-------------|----------|
| kebab-case | `vannevar-bush.md` — URL-friendly, avoids case issues | ✓ |
| Title Case | `Vannevar Bush.md` — natural but case-sensitive on some OS | |
| Original casing | Match whatever is natural — inconsistent but flexible | |

**User's choice:** [auto] kebab-case (recommended default — per pitfall P5)
**Notes:** Cross-platform safety is the primary driver. Obsidian handles display names via aliases.

---

## Index Structure

| Option | Description | Selected |
|--------|-------------|----------|
| Categorized by type | Sections for Sources, Entities, Concepts, Synthesis, Queries | ✓ |
| Flat alphabetical | Single list, alphabetically sorted | |
| Tag-based grouping | Group by tags rather than page type | |

**User's choice:** [auto] Categorized by type (recommended default — matches directory structure)
**Notes:** Aligns with wiki/ subdirectory structure. Natural for both LLM navigation and human browsing.

---

## Frontmatter Fields

| Option | Description | Selected |
|--------|-------------|----------|
| 6 mandatory fields | title, type, sources, tags, created, updated | ✓ |
| 4 minimal fields | title, type, created, updated | |
| 8+ comprehensive | Add aliases, related, confidence, word_count, etc. | |

**User's choice:** [auto] 6 mandatory fields (recommended default — per research summary)
**Notes:** Sources and tags are essential for citation tracking and Dataview compatibility.

---

## Claude's Discretion

- Template formatting details
- Tag taxonomy
- Log entry verbosity

## Deferred Ideas

None
