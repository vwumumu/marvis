---
phase: 1
plan: 2
title: "Directory Structure, Index, Log, Obsidian Config, and Git Foundation"
wave: 1
depends_on: []
files_modified:
  - index.md
  - log.md
  - .obsidian/app.json
  - .gitignore
  - raw/articles/.gitkeep
  - raw/papers/.gitkeep
  - raw/notes/.gitkeep
  - raw/books/.gitkeep
  - raw/assets/.gitkeep
  - wiki/sources/.gitkeep
  - wiki/entities/.gitkeep
  - wiki/concepts/.gitkeep
  - wiki/synthesis/.gitkeep
  - wiki/queries/.gitkeep
requirements_addressed:
  - SCHM-02
  - SCHM-03
  - SCHM-04
  - SRCM-01
  - SRCM-02
  - SRCM-03
  - INDX-01
  - INDX-02
  - INDX-03
  - LOGM-01
  - LOGM-02
  - GITM-01
autonomous: true
estimated_context_cost: "low"
---

## Objective

Create the complete directory structure (raw/ with subdirectories, wiki/ with subdirectories), the index.md navigation catalog with category headers, the log.md operations log with the initial creation entry, the Obsidian configuration for excluding .planning/, the .gitignore for Obsidian volatile files, and commit everything as the foundation of the wiki git repository. After this plan, opening the vault in Obsidian shows a navigable index.md, the directory structure exists for all subsequent operations, and log.md has its first parseable entry.

## must_haves

- [ ] raw/ directory exists with subdirectories: articles/, papers/, notes/, books/, assets/
- [ ] wiki/ directory exists with subdirectories: sources/, entities/, concepts/, synthesis/, queries/
- [ ] All empty directories have .gitkeep files so git tracks them
- [ ] index.md exists at vault root with 5 category headers (Sources, Entities, Concepts, Synthesis, Queries) and descriptive HTML comments
- [ ] index.md has no YAML frontmatter (it is a navigation file, not a wiki page)
- [ ] log.md exists at vault root with the initial `## [2026-04-15] create | Wiki Initialization` entry
- [ ] log.md header includes the grep parse instruction: `grep "^## \[" log.md`
- [ ] .obsidian/app.json exists with userIgnoreFilters excluding .planning/ and .gitkeep
- [ ] .obsidian/app.json sets attachmentFolderPath to raw/assets
- [ ] .gitignore excludes .obsidian/workspace.json, .obsidian/workspace-mobile.json, .obsidian/cache, .DS_Store
- [ ] The vault is a git repository with at least one commit containing all scaffolding files

## Tasks

<task id="1">
<title>Create raw/ directory structure with .gitkeep files</title>
<type>execute</type>
<read_first>
- CLAUDE.md (verify directory conventions — raw/ subdirectories: articles, papers, notes, books, assets)
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 3: Directory Scaffolding, .gitkeep recommendations)
</read_first>
<action>
Create the following directories and empty .gitkeep files:

```bash
mkdir -p raw/articles raw/papers raw/notes raw/books raw/assets
touch raw/articles/.gitkeep raw/papers/.gitkeep raw/notes/.gitkeep raw/books/.gitkeep raw/assets/.gitkeep
```

Each .gitkeep is an empty file (0 bytes). Its sole purpose is to ensure git tracks the empty directory.
</action>
<acceptance_criteria>
- File exists: raw/articles/.gitkeep
- File exists: raw/papers/.gitkeep
- File exists: raw/notes/.gitkeep
- File exists: raw/books/.gitkeep
- File exists: raw/assets/.gitkeep
</acceptance_criteria>
</task>

<task id="2">
<title>Create wiki/ directory structure with .gitkeep files</title>
<type>execute</type>
<read_first>
- CLAUDE.md (verify directory conventions — wiki/ subdirectories: sources, entities, concepts, synthesis, queries)
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 3: Directory Scaffolding)
</read_first>
<action>
Create the following directories and empty .gitkeep files:

```bash
mkdir -p wiki/sources wiki/entities wiki/concepts wiki/synthesis wiki/queries
touch wiki/sources/.gitkeep wiki/entities/.gitkeep wiki/concepts/.gitkeep wiki/synthesis/.gitkeep wiki/queries/.gitkeep
```

Each .gitkeep is an empty file (0 bytes).
</action>
<acceptance_criteria>
- File exists: wiki/sources/.gitkeep
- File exists: wiki/entities/.gitkeep
- File exists: wiki/concepts/.gitkeep
- File exists: wiki/synthesis/.gitkeep
- File exists: wiki/queries/.gitkeep
</acceptance_criteria>
</task>

<task id="3">
<title>Create index.md with category headers</title>
<type>execute</type>
<read_first>
- CLAUDE.md (verify index format section — category structure and entry format)
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 4: Index Format — proposed format, design decisions)
</read_first>
<action>
Create `index.md` at the vault root with the following exact content:

```markdown
# Wiki Index

> This index catalogs every wiki page. It is the primary navigation tool for both the LLM and human readers. Updated on every ingest operation.

## Sources
<!-- Source summary pages — one per ingested source -->

## Entities
<!-- Entity pages — people, organizations, tools, places -->

## Concepts
<!-- Concept pages — ideas, frameworks, patterns, themes -->

## Synthesis
<!-- Cross-cutting analyses, comparisons, timelines -->

## Queries
<!-- Persisted answers to user questions -->
```

No YAML frontmatter. No entries yet — only category headers with descriptive HTML comments. The index starts empty and is populated by ingest operations.
</action>
<acceptance_criteria>
- Grep for "# Wiki Index" in index.md returns a match
- Grep for "## Sources" in index.md returns a match
- Grep for "## Entities" in index.md returns a match
- Grep for "## Concepts" in index.md returns a match
- Grep for "## Synthesis" in index.md returns a match
- Grep for "## Queries" in index.md returns a match
- Grep for "primary navigation tool" in index.md returns a match
- Grep for "Source summary pages" in index.md returns a match
- index.md does NOT contain "---" (no frontmatter)
</acceptance_criteria>
</task>

<task id="4">
<title>Create log.md with initial creation entry</title>
<type>execute</type>
<read_first>
- CLAUDE.md (verify log format section — entry format, parseable header, operation types)
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 5: Log Format — proposed format, initial entry content)
</read_first>
<action>
Create `log.md` at the vault root with the following exact content:

```markdown
# Operations Log

> Chronological record of all wiki operations. Append-only. Parse with: `grep "^## \[" log.md`

## [2026-04-15] create | Wiki Initialization

Established wiki schema, directory structure, index, and log. Created CLAUDE.md with page templates, operation workflows, and editorial rules. Created index.md with category structure. Initialized raw/ and wiki/ directory trees.
```

No YAML frontmatter. The initial entry records the wiki creation event with today's date.
</action>
<acceptance_criteria>
- Grep for "# Operations Log" in log.md returns a match
- Grep for "Append-only" in log.md returns a match
- Grep for 'grep "\^## \\[" log.md' in log.md returns a match (parse instruction)
- Grep for "## \[2026-04-15\] create | Wiki Initialization" in log.md returns a match
- Grep for "Established wiki schema" in log.md returns a match
</acceptance_criteria>
</task>

<task id="5">
<title>Create .obsidian/app.json with exclusion config</title>
<type>execute</type>
<read_first>
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 2: Obsidian Compatibility — app.json format, exclusion patterns, attachment folder)
</read_first>
<action>
Create `.obsidian/app.json` with the following exact content:

```json
{
  "userIgnoreFilters": [
    ".planning/",
    ".gitkeep"
  ],
  "attachmentFolderPath": "raw/assets"
}
```

This configures Obsidian to:
1. Hide `.planning/` from file explorer, search, and graph view
2. Hide `.gitkeep` files from file explorer
3. Save pasted/downloaded images to `raw/assets/`

Create the `.obsidian/` directory first if it does not exist:
```bash
mkdir -p .obsidian
```
</action>
<acceptance_criteria>
- Grep for "userIgnoreFilters" in .obsidian/app.json returns a match
- Grep for ".planning/" in .obsidian/app.json returns a match
- Grep for ".gitkeep" in .obsidian/app.json returns a match
- Grep for "attachmentFolderPath" in .obsidian/app.json returns a match
- Grep for "raw/assets" in .obsidian/app.json returns a match
</acceptance_criteria>
</task>

<task id="6">
<title>Create .gitignore for Obsidian volatile files</title>
<type>execute</type>
<read_first>
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 6: Git Foundation — .gitignore recommendations)
</read_first>
<action>
Create `.gitignore` at the vault root with the following exact content:

```
# Obsidian workspace (changes on every session)
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# Obsidian cache
.obsidian/cache

# OS files
.DS_Store
```

This ensures:
- `.obsidian/app.json` IS committed (our configuration)
- Volatile Obsidian files are NOT committed
- macOS .DS_Store files are excluded
</action>
<acceptance_criteria>
- Grep for "workspace.json" in .gitignore returns a match
- Grep for "workspace-mobile.json" in .gitignore returns a match
- Grep for ".obsidian/cache" in .gitignore returns a match
- Grep for ".DS_Store" in .gitignore returns a match
- .gitignore does NOT contain "app.json" (app.json should be committed)
</acceptance_criteria>
</task>

<task id="7">
<title>Git commit all scaffolding files</title>
<type>execute</type>
<read_first>
- .planning/phases/01-schema-scaffolding/01-RESEARCH.md (section 6: Git Foundation — commit strategy)
- CLAUDE.md (verify all files that should exist are created)
</read_first>
<action>
Stage and commit all scaffolding files to git. The commit must include:

1. CLAUDE.md (updated with wiki schema)
2. index.md (new)
3. log.md (new)
4. .obsidian/app.json (new)
5. .gitignore (new)
6. All .gitkeep files in raw/ and wiki/ subdirectories

Commands:
```bash
git add CLAUDE.md index.md log.md .obsidian/app.json .gitignore
git add raw/articles/.gitkeep raw/papers/.gitkeep raw/notes/.gitkeep raw/books/.gitkeep raw/assets/.gitkeep
git add wiki/sources/.gitkeep wiki/entities/.gitkeep wiki/concepts/.gitkeep wiki/synthesis/.gitkeep wiki/queries/.gitkeep
git commit -m "feat(wiki): establish schema, directory structure, index, log, and git foundation

- CLAUDE.md: wiki schema with page templates, operation workflows, editorial rules
- index.md: categorized navigation catalog (Sources, Entities, Concepts, Synthesis, Queries)
- log.md: operations log with initial creation entry
- raw/: immutable source directories (articles, papers, notes, books, assets)
- wiki/: LLM-maintained page directories (sources, entities, concepts, synthesis, queries)
- .obsidian/app.json: exclude .planning/ and .gitkeep from Obsidian
- .gitignore: exclude Obsidian volatile files"
```

After committing, verify with:
```bash
git log --oneline -5
git status
```
</action>
<acceptance_criteria>
- Running `git log --oneline` shows a commit containing "establish schema" or "wiki" in its message
- Running `git status` shows a clean working tree (no untracked scaffolding files)
- Running `git show --stat HEAD` lists CLAUDE.md, index.md, log.md, .obsidian/app.json, .gitignore, and .gitkeep files
</acceptance_criteria>
</task>
