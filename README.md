# BMAD for FinApps — Agentic SDLC kit for GitHub Copilot (BFSI)

GitHub Copilot custom agents that take a Banking and Financial Services application from requirement to tested,
reviewed and documented code. Each stage runs a controlled loop: the agent changes a file, you review it, then
deterministic checks and an independent critic run. Pushes, PRs and other external writes always ask first.
No JIRA or Atlassian MCP is needed: work items are Markdown files in your repository, with CSV and Confluence exports.

```
/onboard (existing code) → /pm → /ba → /arch → /dev → /review → /qa → /doc
```

## Start here

| Document | Use it for |
|---|---|
| [docs/BMAD_SETUP_Guide.md](docs/BMAD_SETUP_Guide.md) | Setup (Part A) and step-by-step use of every agent (Part B) |
| [docs/BMAD_REFERENCE.md](docs/BMAD_REFERENCE.md) | One-page tables: agents, slash commands, inputs, outputs, example prompts |
| [docs/BMAD_Greenfield.pptx](docs/BMAD_Greenfield.pptx) | Team deck for a new application |
| [docs/BMAD_Brownfield.pptx](docs/BMAD_Brownfield.pptx) | Team deck for an existing application |
| [docs/bmad.config.example.yaml](docs/bmad.config.example.yaml) | A filled, fictional project profile |

## What to copy into your application

`.github/agents`, `.github/prompts`, `.github/bmad`, `.github/instructions`, `.github/copilot-instructions.md`, and the
`docs/` guides. Then fill `.github/bmad/bmad.config.yaml` and create `AGENTS.md` from `.github/bmad/templates/AGENTS.md`.

## Requirements

VS Code with GitHub Copilot Chat in Agent mode (Run Subagent tool enabled), `git`, `gh`, `jq`, bash (Git Bash on
Windows), and your stack's toolchain.

All agent output is assistive. A qualified reviewer validates designs, code and documents before delivery, and
Compliance confirms any regulatory point.
