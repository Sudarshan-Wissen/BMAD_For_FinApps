# Exports for JIRA and Confluence

Generated files for teams without the Atlassian MCP server. Nothing here is sent anywhere automatically.

| Command (Copilot Chat, Agent mode) | Writes | Then |
|---|---|---|
| `/to-jira-csv epic:{id}` (or `{id},{id}`, `status:todo`, `all`) | `jira/{scope}-{date}.csv` and `jira/{scope}-{date}-import-guide.md` | JIRA → Import issues from CSV, map the columns as the guide says, test with two rows first |
| `/to-confluence {id}` (or `epic:{id}`, or any `.md` path; `format:html\|wiki\|both`) | `confluence/{name}.html` and/or `confluence/{name}.wiki.txt` | Open the HTML in a browser, copy all, paste into the Confluence editor (or paste the wiki text via Insert → Markup) |

The source of truth stays in `docs/bmad/`. Re-run the command after changes instead of editing exports by hand.
Upload or paste only into Wissen-approved Atlassian instances, and check the export for real client data first.
