# Phase 2: Ingest Operation - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-15
**Phase:** 02-Ingest Operation
**Areas discussed:** Ingest trigger mechanism, Test source selection, Incremental integration strategy, Ripple check depth
**Mode:** Auto (all decisions auto-selected)

---

## Ingest Trigger Mechanism

| Option | Description | Selected |
|--------|-------------|----------|
| Pure manual | User tells Claude Code "ingest this file", Claude follows CLAUDE.md workflow | ✓ |
| TypeScript CLI | Build `src/ingest.ts` script that automates parts of the pipeline | |
| Claude Code skill | Create a `/ingest` skill command | |

**User's choice:** [auto] Pure manual (recommended default — Claude Code IS the engine per architecture; no premature tooling per P6)
**Notes:** The CLAUDE.md already defines a 10-step ingest workflow. Claude Code follows it step by step. Automation tooling (TypeScript scripts for validation, index rebuild) is deferred to Phase 4 (Lint) or later when patterns emerge from real usage. At moderate scale (~100 sources), manual LLM-driven ingest is sufficient and preserves the review step that prevents P11 (garbage-in-garbage-out).

---

## Test Source Selection

| Option | Description | Selected |
|--------|-------------|----------|
| llmwiki.md as first source | Ingest the project's own idea document — meta but content-rich | ✓ |
| External article | Find a real knowledge management article | |
| Synthetic test document | Create a minimal test document | |

**User's choice:** [auto] llmwiki.md as first source (recommended default — already available, content-rich, tests all page types)
**Notes:** llmwiki.md describes the LLM Wiki concept with named entities (Vannevar Bush, Memex, etc.), concepts (knowledge compounding, RAG), and cross-references. It is an ideal first source. A second source will be needed to test incremental integration (NGST-05).

---

## Second Test Source

| Option | Description | Selected |
|--------|-------------|----------|
| As We May Think (Vannevar Bush) | Classic article that overlaps with llmwiki.md entities | ✓ |
| Another knowledge management article | Broader topic overlap | |
| Create a synthetic overlapping doc | Controlled test | |

**User's choice:** [auto] As We May Think reference (recommended default — direct entity overlap with llmwiki.md for testing NGST-05)
**Notes:** If the actual article isn't available, a concise summary/excerpt placed in raw/articles/ will suffice to test incremental integration. The key is that it mentions entities (Vannevar Bush, Memex) already created from the first source.

---

## Ripple Check Depth

| Option | Description | Selected |
|--------|-------------|----------|
| One hop | Check only pages directly linked to updated entities/concepts | ✓ |
| Two hops | Also check pages linked from those pages | |
| Full scan | Grep entire wiki for mentions | |

**User's choice:** [auto] One hop (recommended default — per P3 context window management; deeper checks are lint territory)
**Notes:** At early wiki size, one-hop ripple check is sufficient. Full consistency checking is the lint operation's job (Phase 4). Keeping ingest focused prevents context window overflow (P3).

---

## Claude's Discretion

- Specific entity/concept extraction decisions during ingest
- Summary length and detail level
- Tag selection for new pages
- Whether to create synthesis pages from a single source (generally: no)

## Deferred Ideas

None
