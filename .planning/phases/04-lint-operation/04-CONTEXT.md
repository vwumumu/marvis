# Phase 4: Lint Operation - Context

**Gathered:** 2026-04-15
**Status:** Ready for planning

<domain>
## Phase Boundary

Implement the lint operation: Claude Code checks the wiki for broken wikilinks, orphan pages, missing entity pages, index completeness, frontmatter compliance, and contradictions. Report findings, fix what can be fixed, and log the operation.

</domain>

<decisions>
## Implementation Decisions

- **D-01:** Lint is a manual LLM-driven process following the CLAUDE.md lint workflow.
- **D-02:** Report findings in chat, fix obvious issues, log the operation.
- **D-03:** Test by running lint on the current wiki state (10 pages across 5 types).

</decisions>

<canonical_refs>
## Canonical References

- `CLAUDE.md` — Lint workflow (10 steps)
- `.planning/REQUIREMENTS.md` — LINT-01..06
- `index.md` — Page inventory to check against filesystem
- All wiki pages in wiki/sources/, wiki/entities/, wiki/concepts/, wiki/queries/

</canonical_refs>

---

*Phase: 04-lint-operation*
*Context gathered: 2026-04-15*
