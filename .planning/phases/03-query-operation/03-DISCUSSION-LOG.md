# Phase 3: Query Operation - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-15
**Phase:** 03-Query Operation
**Areas discussed:** Query mechanism, answer persistence, citation format
**Mode:** Auto (all decisions auto-selected)

---

## Query Mechanism

| Option | Description | Selected |
|--------|-------------|----------|
| Pure manual | User asks Claude Code a question, Claude follows CLAUDE.md query workflow | ✓ |
| Search-assisted | Build MiniSearch index, query it first | |
| CLI tool | Build `src/query.ts` tool | |

**User's choice:** [auto] Pure manual (recommended default — wiki has <20 pages, index.md is sufficient for navigation)
**Notes:** At current scale, Claude Code reads index.md, identifies relevant pages, reads them, and synthesizes an answer. No search infrastructure needed yet.

---

## Answer Persistence

| Option | Description | Selected |
|--------|-------------|----------|
| Offer to persist | After answering, offer to save as wiki/queries/ or wiki/synthesis/ page | ✓ |
| Always persist | Every query answer becomes a wiki page | |
| Never persist | Answers stay in chat only | |

**User's choice:** [auto] Offer to persist (recommended default — per llmwiki.md: "good answers can be filed back into the wiki")
**Notes:** For testing, we will persist one query answer to verify the full workflow.

---

## Claude's Discretion

- Which wiki pages to read for a given query
- Level of detail in the answer
- Whether to suggest follow-up questions

## Deferred Ideas

None
