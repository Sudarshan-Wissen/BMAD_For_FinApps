---
name: to-confluence
description: Convert BMAD Markdown (PRD, stories, design, QA report, any .md) into Confluence-ready files for copy-paste (no Atlassian MCP needed)
agent: agent
tools: ['read', 'search', 'edit']
argument-hint: "{id} | epic:{id} | path/to/file.md  [format:html|wiki|both]"
---

# Export to Confluence

Source: ${input:source:{id} | epic:{id} | path/to/file.md}
Format: ${input:format:html}

Read `.github/bmad/bmad.config.yaml` (only `project` and `work`). Do not call any external tool, do not run terminal
commands, and do not change the source files. Read only the files in scope.

## 1. Collect

- `{id}` of a story → `{id}.md`, plus `{id}-design.md` and `{id}-qa.md` if they exist, as one page.
- `{id}` of an epic or `epic:{id}` → `{id}-prd.md`, followed by a summary table of its stories (id, title, points,
  status, gates) and then each story in id order.
- A file path → that file only (for example `docs/BMAD_SETUP_Guide.md` or a module `AGENTS.md`).

## 2. Convert

| Markdown | `html` output | `wiki` output (Confluence wiki markup) |
|---|---|---|
| YAML front matter | a two-column "Page properties" table at the top (id, type, status, gates, points, depends on) | the same as a `||Key||Value||` table |
| `#` … `####` | `<h1>` … `<h4>` | `h1.` … `h4.` |
| `**bold**`, `*italic*`, `` `code` `` | `<strong>`, `<em>`, `<code>` | `*bold*`, `_italic_`, `{{code}}` |
| Lists, `- [ ]` tasks | `<ul>`/`<ol>`; tasks as `☐` / `☑` | `*` / `#`; tasks as `(x)` / `(/)` |
| Tables | `<table>` with a header row | `||h1||h2||` then `|c1|c2|` |
| Fenced code (incl. mermaid) | `<pre><code>` | `{code:language}…{code}` |
| `> quote` | `<blockquote>` | `{quote}…{quote}` |
| Links to repo files | plain text with the path | plain text with the path |

Keep headings, order and wording unchanged. Drop template placeholders that remain in braces. The `html` file is a
complete page with simple inline styles (readable fonts, bordered tables) and no scripts or external resources.

**Data check:** before writing, scan for apparent real customer names, account, card or government numbers,
credentials or tokens. If found, replace with `[REDACTED]` and list where.

## 3. Write

Into `{work.dir}/export/confluence/` (create it if needed), using the source name as `{name}`:
- `format:html` (default) → `{name}.html`
- `format:wiki` → `{name}.wiki.txt`
- `format:both` → both files

## 4. Report

Print only the file paths, any redactions, and how to paste:
- **html:** open the file in a browser (or VS Code's Live Preview), select all, copy, and paste into the Confluence
  page editor. Formatting, tables and code blocks carry over.
- **wiki:** in the Confluence editor use Insert → Markup → Confluence wiki (if your Confluence version offers it),
  paste the text, and insert.

Remind the user to publish only to an approved Confluence space.
