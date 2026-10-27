# BMAD Reference — agents, commands and tools at a glance

**Setup check (one line, Git Bash, repo root):** expect 10 agents and 10 prompts.

```bash
ls .github/agents/*.agent.md | wc -l; ls .github/prompts/*.prompt.md | wc -l; grep -c TODO .github/bmad/bmad.config.yaml
```

Then open Copilot Chat in **Agent** mode: the dropdown lists the agents and `/` lists the commands. The last number
must be `0` (no `TODO` left in the config). Full setup: [BMAD_SETUP_Guide.md](BMAD_SETUP_Guide.md) Part A.

---

## 1. Agents (`.github/agents/`)

| Agent | What it does | Input | Output | Example prompt |
|---|---|---|---|---|
| `bmad-onboard` | Learns existing code and writes module rulebooks; flags BFSI tech debt | `service:{module}` or `service:all` | `{module}/AGENTS.md`, `modules` list in config, root rulebook table rows | `service:all` |
| `bmad-pm` | Turns a requirement into a BFSI-aware PRD (epic) | Requirement text, or `epic:{id}` to revise | `docs/bmad/{id}-prd.md`, backlog row, gate `prd-approved` | `Allocate block executions to client accounts at average price` |
| `bmad-ba` | Splits an epic into testable stories with BFSI acceptance criteria | `epic:{id}` or requirement text | `docs/bmad/{id}.md` per story, gate `dor-ready` | `epic:TRD-1` |
| `bmad-arch` | Writes the technical design D1–D20 for one story | Story id with `dor-ready` | `docs/bmad/{id}-design.md`, gate `arch-approved` | `TRD-2` |
| `bmad-dev` | Implements the story layer by layer with tests; raises the PR | Story id with `arch-approved` | Branch, commits, PR, story `in-review` | `TRD-2` |
| `bmad-code-review` | Reviews a PR against the rulebook, 12 code categories and BFSI rules | `pr:{n}` or branch | PR review comment, gate `code-review-approved` | `pr:42` |
| `bmad-qa` | Tests acceptance criteria and BFSI edge cases on a local or test system | Story id with `code-review-approved` | `docs/bmad/{id}-qa.md`, E2E spec, gate `qa-passed` | `TRD-2` |
| `bmad-doc` | Syncs the API spec and module rulebook with merged code | Story id or `pr:{n}` | Spec and `AGENTS.md` updates, docs PR | `pr:42` |
| `bmad-bfsi-advisor` | Answers domain questions (trading, allocation, settlement, payments, lending) | A question | Answer, rule ids, worked example, edge cases | `How should residual shares be allocated pro-rata across accounts?` |
| `bmad-critic` | Independent read-only judge, called automatically by stage agents | File paths (from the agent) | PASS/FAIL with `path:line` evidence | — (not user-invocable) |

## 2. Slash commands (`.github/prompts/`)

| Command | Starts | Input | Output | Example |
|---|---|---|---|---|
| `/onboard` | `bmad-onboard` | `service:{module}` · `service:all` | Module rulebooks | `/onboard service:all` |
| `/pm` | `bmad-pm` | Requirement · `epic:{id}` | PRD | `/pm "Add pro-rata allocation of block fills"` |
| `/ba` | `bmad-ba` | `epic:{id}` · requirement | Stories | `/ba epic:TRD-1` |
| `/arch` | `bmad-arch` | `{id}` | Design | `/arch TRD-2` |
| `/dev` | `bmad-dev` | `{id}` | Code, tests, PR | `/dev TRD-2` |
| `/review` | `bmad-code-review` | `pr:{n}` · branch | Review comment | `/review pr:42` |
| `/qa` | `bmad-qa` | `{id}` | QA report | `/qa TRD-2` |
| `/doc` | `bmad-doc` | `{id}` · `pr:{n}` | Docs PR | `/doc TRD-2` |
| `/to-jira-csv` | Built-in agent (read, search, edit only) | `epic:{id}` · `{id},{id}` · `status:{s}` · `all` | `docs/bmad/export/jira/*.csv` + import guide | `/to-jira-csv epic:TRD-1` |
| `/to-confluence` | Built-in agent (read, search, edit only) | `{id}` · `epic:{id}` · `.md` path; `format:html\|wiki\|both` | `docs/bmad/export/confluence/*.html` / `.wiki.txt` | `/to-confluence epic:TRD-1` |

The hand-off button under each stage agent's reply opens the next agent without typing a command.

## 3. Skills and auto-applied instructions

| Item | What it is | Applies when |
|---|---|---|
| Skills | None needed: the kit uses only custom agents and prompt files | — |
| `.github/copilot-instructions.md` | Short always-on rules: where things are, BFSI non-negotiables, approvals | Every Copilot chat in the repo |
| `.github/instructions/bfsi-code.instructions.md` | BFSI coding rules with rule ids | Copilot editing or reviewing source, SQL or YAML files (incl. Copilot code review) |
| MCP servers | None needed. Atlassian MCP is optional (`tracker.type: jira`) | Only if you enable it |

## 4. Your replies

| Agent shows | You reply |
|---|---|
| `REVIEW n — …` | `keep` · `revise: <notes>` · `discard` |
| `1 next agent · 2 continue loop · 3 stop` | `1` · `2` · `3` |
| `ACTION — … Proceed? (yes/no)` | `yes` · `no` |
| A plan (BA split, QA tests) | `ok` or your changes |

## 5. Switches (`.github/bmad/bmad.config.yaml`)

| Key | Default | Effect |
|---|---|---|
| `flow.auto_loop` | `false` | `false`: ask after each round. `true`: fix and re-check automatically |
| `flow.review_changes` | `true` | Pause for Keep/Undo review after each change |
| `flow.review_granularity` | `step` | `step`: per document or code layer. `file`: per file |
| `flow.critic` | `true` | Run the independent critic |
| `flow.max_attempts` | `3` | Rounds before escalation |
| `tracker.type` | `none` | `jira` mirrors items through the Atlassian MCP server |
| `export.jira.*` | `Epic`, `Story` | Issue-type names for `/to-jira-csv` |

## 6. Files

| Path | Holds |
|---|---|
| `.github/bmad/bmad.config.yaml` | The only project-specific settings |
| `AGENTS.md`, `{module}/AGENTS.md` | Project and module rulebooks |
| `.github/bmad/protocol.md` | Shared rules all agents follow |
| `.github/bmad/domain/` | BFSI rules (ids) and domain knowledge |
| `.github/bmad/stacks/` | Build, test, coverage and scan commands per technology |
| `.github/bmad/rubrics/` | Critic checklist per stage |
| `.github/bmad/templates/` | Rulebook, PRD, story and design templates |
| `docs/bmad/` | Work items: PRDs, stories, designs, QA reports, `backlog.md` |
| `docs/bmad/export/` | Generated JIRA CSV and Confluence files |
| `.bmad/` | Scratch files (git-ignored) |

## 7. Gates

`prd-approved` → `dor-ready` → `arch-approved` → `code-review-approved` → `qa-passed`. Status:
`todo` → `in-progress` → `in-review` → `done`.
