---
phase: 4
plan: 1
title: "Lint Operation — Full Wiki Health Check"
wave: 1
depends_on: []
files_modified:
  - log.md
requirements_addressed:
  - LINT-01
  - LINT-02
  - LINT-03
  - LINT-04
  - LINT-05
  - LINT-06
autonomous: true
estimated_context_cost: "medium"
---

## Objective

Execute the lint workflow on the current wiki. Check for broken wikilinks, orphan pages, missing entity pages, index completeness, frontmatter compliance, and contradictions. Fix any issues found, report results, log the operation.

## must_haves

- [ ] Broken wikilink check completed (LINT-01)
- [ ] Orphan page check completed (LINT-02)
- [ ] Missing entity/concept page check completed (LINT-03)
- [ ] Index completeness check completed (LINT-04)
- [ ] Contradiction check completed (LINT-05)
- [ ] log.md has lint entry (LINT-06)
- [ ] Git commit captures any fixes

## Tasks

<task id="1">
<title>Check broken wikilinks</title>
<type>execute</type>
<action>
Grep all wiki pages for [[wikilinks]], extract target filenames, verify each resolves to an existing file.
</action>
</task>

<task id="2">
<title>Check orphan pages</title>
<type>execute</type>
<action>
For each wiki page, check if any other wiki page links to it. Report pages with zero inbound links.
</action>
</task>

<task id="3">
<title>Check missing entity/concept pages</title>
<type>execute</type>
<action>
Scan wiki pages for entities/concepts mentioned frequently but lacking their own page.
</action>
</task>

<task id="4">
<title>Check index completeness</title>
<type>execute</type>
<action>
Compare wiki/ filesystem contents against index.md entries. Report pages not in index and stale index entries.
</action>
</task>

<task id="5">
<title>Check frontmatter compliance</title>
<type>execute</type>
<action>
Verify all wiki pages have the 6 mandatory frontmatter fields with correct types.
</action>
</task>

<task id="6">
<title>Check contradictions</title>
<type>execute</type>
<action>
Compare claims across pages referencing the same entity/concept. Flag discrepancies.
</action>
</task>

<task id="7">
<title>Fix issues and log</title>
<type>execute</type>
<action>
Fix any issues found, append lint entry to log.md, commit changes.
</action>
</task>
