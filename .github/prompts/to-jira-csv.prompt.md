---
name: to-jira-csv
description: Export BMAD epics and stories from docs/bmad to a CSV file ready for JIRA bulk import (no Atlassian MCP needed)
agent: agent
tools: ['read', 'search', 'edit']
argument-hint: "epic:{id} | {id},{id} | status:todo | all"
---

# Export to JIRA CSV

Scope: ${input:scope:epic:{id} | {id},{id} | status:todo | all}

Read `.github/bmad/bmad.config.yaml` (only `project`, `work` and the optional `export.jira` block). Work only from the
files in `{work.dir}` (default `docs/bmad`). Do not call any external tool, do not run terminal commands, and do not
change any work item. Read only the files in scope.

## 1. Collect

- `epic:{id}` → the epic `{id}-prd.md` plus every story whose front matter has `epic: {id}`.
- `{id},{id}` → exactly those items (add their epics so the parent link works).
- `status:{s}` → every story with that status, plus their epics.
- `all` → every epic and story in `{work.dir}`.

Skip designs and QA reports. If nothing matches, say so and stop.

## 2. Build one row per item

Columns, in this order (header names as written; JIRA maps them in the import wizard):

| Column | Epic row | Story row |
|---|---|---|
| `Issue ID` | the BMAD id, e.g. `TRD-1` | the BMAD id |
| `Parent ID` | empty | the story's `epic` id, when that epic is in this file |
| `Issue Type` | `export.jira.epic_type` (default `Epic`) | `export.jira.story_type` (default `Story`) |
| `Summary` | the PRD `# title` | the story `# title` (max 255 characters) |
| `Epic Name` | same as Summary | empty |
| `Description` | Problem Statement, Proposed Solution, Success Criteria, Regulatory and Control Impact | User Story, Context, BFSI Considerations, Out of Scope, Tech Constraints, Test Data, `Depends on: …` |
| `Acceptance Criteria` | empty | every AC, one per line, `[TAG]` kept |
| `Story Points` | empty | front-matter `points` |
| `Priority` | empty | from the PRD backlog row for this story: P1 → `High`, P2 → `Medium`, P3 → `Low`; else empty |
| `Labels` | `bmad` | `bmad` (one extra `Labels` column per gate, e.g. `dor-ready`, `arch-approved`) |
| `BMAD Status` | front-matter `status` | front-matter `status` |

Description and Acceptance Criteria text is written in JIRA wiki markup: `h3. Heading`, `* bullet`, `||head||` and
`|cell|` for tables, `{code}` for code. Drop template placeholders that remain in braces. If
`export.jira.acceptance_criteria_in_description: true`, append the ACs to Description under `h3. Acceptance Criteria`
and leave that column empty.

## 3. Write the CSV

- RFC 4180, UTF-8. Quote **every** field, double any `"` inside a field, and keep line breaks inside quoted fields.
- Epics first, then stories in id order. Repeat the `Labels` header once per label column, the same number of
  columns on every row.
- **Data check:** before writing, scan every field for apparent real customer names, account, card or government
  numbers, credentials or tokens. If found, replace with `[REDACTED]` and list where.

Files (create the folder if needed):
- `{work.dir}/export/jira/{scope-slug}-{YYYYMMDD}.csv`
- `{work.dir}/export/jira/{scope-slug}-{YYYYMMDD}-import-guide.md` with the column → JIRA field mapping table, the row
  count per type, any redactions, and these steps:
  1. In JIRA: issue search ⋯ menu → **Import issues from CSV** (or admin: System → External System Import → CSV).
  2. Choose the file; encoding UTF-8; delimiter comma.
  3. Map `Issue ID` → Issue Id, `Parent ID` → Parent Id, `Issue Type`, `Summary`, `Epic Name` (company-managed
     projects), `Description`, `Acceptance Criteria` → your AC field (or skip), `Story Points` → your points field,
     `Priority`, `Labels`. Leave `BMAD Status` unmapped, or map it to Status if your workflow has matching names.
  4. Run a test import with the first two rows, then the full file.

## 4. Report

Print only: the file paths, the number of epics and stories, and any redactions or skipped items. Remind the user to
upload only to an approved JIRA instance.
