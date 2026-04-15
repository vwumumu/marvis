# Phase 3: Query Operation - Context

**Gathered:** 2026-04-15
**Status:** Ready for planning

<domain>
## Phase Boundary

Implement the query operation: the user asks a question, Claude Code reads index.md to find relevant wiki pages, reads those pages, synthesizes an answer with wikilink citations, and optionally persists the answer as a new wiki page. Test by asking a question that requires synthesizing from multiple wiki pages.

</domain>

<decisions>
## Implementation Decisions

- **D-01:** Query is a manual LLM-driven process. The user asks Claude Code a question, Claude follows the 6-step query workflow in CLAUDE.md.
- **D-02:** Answers come from wiki pages, not raw sources. The wiki is the compiled knowledge.
- **D-03:** After answering, Claude offers to persist substantial answers as wiki/queries/ or wiki/synthesis/ pages.
- **D-04:** For testing, ask a cross-cutting question that requires reading multiple wiki pages, then persist the answer.

</decisions>

<canonical_refs>
## Canonical References

- `CLAUDE.md` — Query workflow (6 steps), page templates
- `.planning/REQUIREMENTS.md` — QERY-01..04
- `index.md` — Navigation catalog (populated by Phase 2)
- Wiki pages in wiki/sources/, wiki/entities/, wiki/concepts/

</canonical_refs>

---

*Phase: 03-query-operation*
*Context gathered: 2026-04-15*
