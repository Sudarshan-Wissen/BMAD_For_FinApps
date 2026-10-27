# BMAD Agentic SDLC for GitHub Copilot — BFSI

Copilot custom agents that take a requirement from PRD to tested, reviewed, documented code for Banking and Financial
Services apps. No JIRA or Atlassian needed: work items are Markdown files in your repo.

New to the kit? Start with [docs/BMAD_SETUP_Guide.md](../../docs/BMAD_SETUP_Guide.md) (setup and step-by-step use) and [docs/BMAD_REFERENCE.md](../../docs/BMAD_REFERENCE.md) (one-page list of agents and commands). Team decks: [docs/BMAD_Greenfield.pptx](../../docs/BMAD_Greenfield.pptx), [docs/BMAD_Brownfield.pptx](../../docs/BMAD_Brownfield.pptx).

## How a stage works

```
agent makes a change ──► you review it in the editor (Keep / Undo) and reply keep | revise | discard
        ▼ (repeat per step)
deterministic checks (build, tests, coverage, BFSI greps) ──► independent critic (read-only subagent)
        ▼
auto_loop false (default): agent asks  1 next agent · 2 continue loop · 3 stop
auto_loop true           : agent fixes and re-checks on its own, then offers the next agent
```

Pushes, PRs, PR reviews and rulebook edits always ask first, in both modes.

## Files

```
.github/
  copilot-instructions.md          always-on rules (short)
  instructions/bfsi-code.instructions.md   BFSI coding rules for source files and Copilot code review
  agents/
    bmad-pm  bmad-ba  bmad-arch  bmad-dev  bmad-code-review  bmad-qa  bmad-doc  bmad-onboard   stage agents
    bmad-critic                    read-only judge (subagent only)
    bmad-bfsi-advisor              domain Q&A (trading, allocation, settlement, payments, lending…)
  prompts/  pm ba arch dev review qa doc onboard   slash commands that start each agent
            to-jira-csv  to-confluence         exports for JIRA CSV import and Confluence paste (no MCP)
  bmad/
    bmad.config.yaml               the ONLY project-specific file (incl. the flow toggles)
    protocol.md                    shared rules every agent follows
    domain/                        bfsi-rules.md (rule ids) · bfsi-domain-pack.md (domain knowledge)
    stacks/                        java-spring · react-typescript · _template (add your stack)
    rubrics/                       the critic's checklist per stage
    templates/                     AGENTS.md · module-AGENTS.md · prd · story · design
```

## Pipeline

| Command | Agent | Writes | Gate set on the story |
|---|---|---|---|
| `/pm "requirement"` | bmad-pm | `docs/bmad/{id}-prd.md` | prd-approved (on the epic) |
| `/ba epic:{id}` | bmad-ba | `docs/bmad/{id}.md` per story | dor-ready |
| `/arch {id}` | bmad-arch | `docs/bmad/{id}-design.md` | arch-approved |
| `/dev {id}` | bmad-dev | code + tests, branch, PR | status in-review |
| `/review pr:{n}` | bmad-code-review | PR review comment | code-review-approved |
| `/qa {id}` | bmad-qa | `docs/bmad/{id}-qa.md`, E2E spec | qa-passed |
| `/doc {id}` | bmad-doc | API spec + module AGENTS.md, docs PR | — |
| `/onboard service:all` | bmad-onboard | module AGENTS.md files, `modules` in config | — |

After each stage, the **hand-off button** under the reply opens the next agent.

## Setup (once per project)

1. Copy `.github/agents`, `.github/prompts`, `.github/bmad`, `.github/instructions` and `.github/copilot-instructions.md`.
2. Add `.bmad/` to `.gitignore` (scratch files for the critic).
3. Fill every `TODO` in `bmad.config.yaml`. Choose the flow: `auto_loop`, `review_changes`, `review_granularity`, `critic`.
4. Copy `templates/AGENTS.md` to the repo root and fill it. For existing code, run `/onboard service:all`.
5. Use VS Code Copilot Chat in **Agent mode**, with the Run Subagent tool enabled (needed for the critic and advisor).
6. Tools on your machine: `git`, `gh` (signed in, for PRs), `jq`, bash (Git Bash on Windows), and your stack's toolchain.

## Optional JIRA

Set `tracker.type: jira`, add the Atlassian MCP server in `.vscode/mcp.json`
(`{"servers":{"atlassian":{"type":"http","url":"https://mcp.atlassian.com/v2/mcp"}}}`), and add `'atlassian/*'` to the
`tools:` line of the stage agents. Agents then also mirror work items to JIRA, asking before each write.

## Keeping token use low

- One agent per stage; only the critic and advisor run as subagents. Set `flow.critic: false` to skip the critic.
- Agents read only the domain-pack sections they need and trim command output.
- `review_granularity: step` pauses once per logical step instead of once per file.
- For long pipelines, start a new chat per stage with its slash command. Agents read state from the `docs/bmad`
  files, so nothing is lost and earlier stages don't stay in context.

All agent output is assistive. A qualified reviewer validates designs, code and documents before delivery, and
Compliance confirms any regulatory point.
