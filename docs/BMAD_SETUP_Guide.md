# BMAD Setup Guide — BMAD Agents for GitHub Copilot (BFSI)

The single guide to setting up and using the BMAD agents: Part A gets your repository ready, Part B walks through one
full SDLC cycle agent by agent.

- One-page list of every agent, command, input, output and example prompt: [BMAD_REFERENCE.md](BMAD_REFERENCE.md)
- Team decks: [BMAD_Greenfield.pptx](BMAD_Greenfield.pptx) (new app) · [BMAD_Brownfield.pptx](BMAD_Brownfield.pptx) (existing app)
- Example config: [bmad.config.example.yaml](bmad.config.example.yaml)

```
/onboard (existing code only) → /pm → /ba → /arch → /dev → /review → /qa → /doc
```

---

# Part A — Setup

## A1. Choose your path

| | Greenfield (new application) | Brownfield (existing application) |
|---|---|---|
| Starting point | Empty repo or a skeleton | Working code, maybe little documentation |
| First agent | `bmad-pm` | `bmad-onboard` |
| Rulebook | You write `AGENTS.md` first | Onboard drafts module `AGENTS.md` files from the code |
| Main early risk | Vague domain rules | Undocumented behaviour, legacy debt, low coverage |
| Setup time | About half a day | About half a day, plus reviewing onboarding drafts |

## A2. Prerequisites

| Item | Check |
|---|---|
| VS Code with GitHub Copilot Chat | Chat opens in **Agent** mode (Ask and Edit modes cannot run tools) |
| Org Copilot policy | Custom agents allowed; the **Run Subagent** tool appears in the chat tools picker |
| Git | `git --version` |
| GitHub CLI | `gh auth status` shows you are signed in (needed for PRs and reviews) |
| jq | `jq --version` |
| Bash | Git Bash on Windows: Terminal → Select Default Profile → Git Bash |
| Stack toolchain | e.g. `java -version` and `mvn -v`, or `node -v` and `npm -v` |
| Python 3 + PyYAML (optional) | Only for the API-spec parse check; without it the check shows WARN |

Atlassian or JIRA access is **not** required.

## A3. Install the kit

```bash
KIT=/path/to/kit-repo
APP=/path/to/your-app
mkdir -p "$APP/.github" "$APP/docs/bmad"
cp -r "$KIT/.github/agents" "$KIT/.github/prompts" "$KIT/.github/bmad" "$KIT/.github/instructions" "$APP/.github/"
cp "$KIT/.github/copilot-instructions.md" "$APP/.github/"
cp "$KIT/docs/BMAD_SETUP_Guide.md" "$KIT/docs/BMAD_REFERENCE.md" "$APP/docs/"
cp -r "$KIT/docs/bmad/export" "$APP/docs/bmad/"
grep -qxF '.bmad/' "$APP/.gitignore" 2>/dev/null || echo '.bmad/' >> "$APP/.gitignore"
```

Do not copy the kit repository's own application files (its `ci.yml`, `PULL_REQUEST_TEMPLATE.md` or root `CLAUDE.md`).

**Check:** reload VS Code. The agent dropdown lists the `bmad-*` agents, and typing `/` shows `/pm /ba /arch /dev
/review /qa /doc /onboard /to-jira-csv /to-confluence`.

## A4. Fill the project profile

Edit `.github/bmad/bmad.config.yaml` (fictional full example: [bmad.config.example.yaml](bmad.config.example.yaml)).

| Section | What to put | Tip |
|---|---|---|
| `project.name`, `project.key` | App name; short uppercase key, e.g. `TRD` | Ids become `TRD-1`, `TRD-2` … |
| `project.subdomains` | `capital-markets`, `payments`, `lending` … | Agents load only these domain-pack sections |
| `flow.*` | Keep defaults at first | `auto_loop: false`, `review_changes: true`, `review_granularity: step`, `critic: true` |
| `work.dir` | `docs/bmad` | PRDs, stories, designs, QA reports go here |
| `tracker.type` | `none` | Switch to `jira` only if you get Atlassian MCP access |
| `export.jira` | Your JIRA issue-type names | Used by `/to-jira-csv` |
| `rulebook.*` | Keep defaults | Add headings agents must never edit to `protected_headings` |
| `modules[]` | Name, path, stack, port, schema per deployable unit | Brownfield: `/onboard service:all` proposes it |
| `git.*` | Base branch, protected branches, branch pattern | Must match your branch protection |
| `api.*` | Base path, spec file, envelopes, money JSON type, roles | `money_json_type: string` recommended |
| `runtime.*` | Local or test URLs, login endpoint, env var **names** for credentials | Never production; never credential values |
| `quality.coverage_min_line_pct` | Agreed coverage floor | Brownfield: agree a realistic baseline first |

Check: `grep -n "TODO" .github/bmad/bmad.config.yaml` prints nothing.

## A5. Create the rulebook

**Greenfield**
1. `cp .github/bmad/templates/AGENTS.md AGENTS.md` and fill it with the architect and a domain expert. Focus on
   **State Machine**, **Domain Rules** (precision, rounding, calendars, cut-offs, settlement cycle, allocation residual
   rule), **Roles and Entitlements**, **Data Classification** and **Out of Scope**.
2. Create a skeleton per module (build file, empty app, one passing test) so Dev's build and coverage checks run.

**Brownfield**
1. Copy the template to `AGENTS.md` and fill what the team already knows.
2. Run `/onboard service:all`: keep the proposed `modules` list, review each drafted `{module}/AGENTS.md` (written
   only from scanned facts, with a **Known Tech Debt** list), and review each root-rulebook table update.
3. Agree the coverage baseline; turn the tech-debt items into backlog later through `/pm`.

## A6. Verify

| Check | How | Expected |
|---|---|---|
| Config complete | `grep -n TODO .github/bmad/bmad.config.yaml` | No output |
| Scratch folder ignored | `git check-ignore -v .bmad/x` | Prints the `.gitignore` line |
| GitHub CLI | `gh auth status` | Signed in |
| Subagents work | Ask `bmad-bfsi-advisor`: "What is the settlement cycle for US equities?" | Answer with rule ids and a compliance note |
| Agents read config | `/pm "test requirement"`, then reply `3` at the first question | No config or rulebook errors |

## A7. Organisation controls

1. **Branch protection** on `develop` and `main`: PRs required, at least one approval, required CI, no force push.
2. **CODEOWNERS** for the kit and rulebooks:
   ```
   /.github/              @your-org/platform-architects
   /AGENTS.md             @your-org/platform-architects
   **/AGENTS.md           @your-org/module-owners
   ```
3. **Optional Copilot code review** through a repository ruleset; it reads `copilot-instructions.md` and
   `instructions/bfsi-code.instructions.md`.
4. **Named reviewers:** a qualified reviewer validates designs, code and documents before delivery; Compliance reviews
   the BFSI domain pack for your markets.

## A8. Your first week

| Day | Greenfield | Brownfield |
|---|---|---|
| 1 | Install the kit, fill the config, draft `AGENTS.md` | Install the kit, fill the config, start `AGENTS.md` |
| 2 | Review `AGENTS.md`; create module skeletons | `/onboard service:all`; review each module rulebook |
| 3 | `/pm` for the first epic; `/ba` to split it | `/pm` for one small, low-risk change |
| 4 | `/arch` and `/dev` for the first story | `/ba`, `/arch`, `/dev` for that change |
| 5 | `/review`, `/qa`, `/doc`; tune the flow settings | `/review`, `/qa`, `/doc`; plan tech-debt stories |

---

# Part B — Using the agents

## B1. How every agent works

```
1 Start-up     config → rulebook → BFSI rules → only the needed domain-pack sections → stack pack (code roles)
2 Work         one change ──► REVIEW pause (you: keep / revise / discard)        … repeat per step
3 Checks       deterministic: headings, placeholders, build, tests, coverage, BFSI greps
4 Critic       independent read-only subagent applies the stage rubric + BFSI rules
5 Decision     "1 next agent · 2 continue loop · 3 stop"   (automatic if auto_loop: true)
6 Gate         on pass, the gate is written into the story/epic file (another REVIEW pause)
7 Hand-off     the button under the reply opens the next agent
```

| You see | Reply | Effect |
|---|---|---|
| `REVIEW n — …` | `keep` · `revise: <notes>` · `discard` | Accept · rework and ask again · restore files |
| `1 next agent · 2 continue loop · 3 stop` | `1` · `2` · `3` | Move on (open items recorded) · fix findings · stop |
| `ACTION — … Proceed? (yes/no)` | `yes` · `no` | Allow or block a push, PR, review post, rulebook or JIRA write |
| A plan (BA split, QA test plan) | `ok` or changes | Approve or adjust before work starts |

Every agent: never pushes to protected branches, merges, force-pushes or edits CI files; asks before every push, PR,
review post, rulebook edit or JIRA write; uses synthetic data only; refuses URLs containing `prod`; records
legal, tax and regulatory questions for a qualified human; never weakens a test, check or threshold.

## B2. The cycle, step by step

### Step 0 — Onboard existing code · `/onboard service:all`
1. Detects modules from build files and proposes the `modules` list → review.
2. Scans each module (endpoints, messaging, entities, migrations, cache, outbound calls, schedulers) plus BFSI facts
   (money and time types, status enums, regulated fields, audit tables, TODOs).
3. Writes `{module}/AGENTS.md` from facts with `path:line` sources (17 sections) → review.
4. Lists BFSI tech debt: float money, PII in logs, missing idempotency or audit, fail-open controls.
5. Checks (all discovered items documented, nothing invented) → critic → syncs root Modules, Messaging, Cache Keys
   tables one at a time → review.

**Done when:** every module has a reviewed `AGENTS.md`. **Next:** PM.

### Step 1 — Requirement → PRD · `/pm "requirement"`
1. Reads the rulebook overview and backlog; may ask the BFSI advisor; asks up to 3 questions if vague.
2. Writes `docs/bmad/{id}-prd.md`: problem, solution, measurable success criteria, **regulatory and control impact**,
   out of scope, dependencies, backlog → review.
3. Checks (7 sections, no placeholders, ≥ 2 success criteria, control lines filled, module exists) → critic (13 items).
4. Adds `prd-approved` and a backlog row → review.

**Next:** BA.

### Step 2 — Epic → stories · `/ba epic:{id}`
1. Confirms `prd-approved`; proposes the split (`title — module — points — depends on`) → you reply `ok`.
2. Writes each story `docs/bmad/{id}.md` → review each: user story, context, tagged ACs (`[API] [DB] [MSG] [UI]
   [BATCH]`: happy path, 404, 400/422), **BFSI ACs** (money scale and currency, idempotent replay, rejected
   transitions, 403, audit record, cut-off and holiday, fail-closed, invariants), BFSI considerations, test data,
   points.
3. Checks → one critic call for all stories (12 items) → adds `dor-ready` → review.

**Next:** Architect, one story at a time.

### Step 3 — Story → design · `/arch {id}`
1. Confirms `dor-ready` and that dependencies are `done`; reads only the code it touches.
2. Writes `docs/bmad/{id}-design.md` with **D1–D20**: interface and error table, data model, lifecycle delta,
   money and rounding step, calendars and cut-offs, idempotency, consistency and outbox, resilience, controls,
   entitlements per role, data classification, audit, reconciliation and regulatory impact, tests, API spec delta
   → review.
3. **Design completeness hard stop:** ownership field, id domains, entitlements, forwarded headers, or hand back to BA.
4. Checks (20 blocks, every AC error code, own schema, next migration, roles, ≥ 3 tests per layer) → critic (25 items)
   → adds `arch-approved` → review.

**Next:** Dev.

### Step 4 — Design → code and PR · `/dev {id}`
1. Confirms gates and a clean tree; creates `feature/{id}-…`; sets `in-progress`.
2. Implements layer by layer (domain → migration → DTOs → mapper → repository → service → controller → exceptions →
   config → tests) → **review after each layer**, applying every BFSI rule.
3. Checks after a clean rebuild: build, 0 test failures, coverage ≥ minimum, test per changed class, BFSI greps, no
   protected-config change, no secrets → critic (12 categories; money, architecture, security, lifecycle are
   zero-tolerance).
4. Local commit → **approve** push → **approve** PR.

**Design gap:** stops and hands back to the Architect; never invents a workaround. **Next:** Code Review.

### Step 5 — PR → review · `/review pr:{n}`
1. Reads PR, story, design; counts rounds; checks out the head in a temporary worktree.
2. Same checks as Dev on the PR diff, CI green, scope matches the story → critic (12 categories + PR items).
3. **Approve** posting the review comment; on pass adds `code-review-approved` → review.

Never edits code or clicks Approve; humans approve and merge. **Next:** QA (or back to Dev for fixes).

### Step 6 — Acceptance testing · `/qa {id}`
1. Confirms `code-review-approved`; refuses `prod`; checks the app is up; logs in from env vars.
2. Plans tests per AC plus 404/400/401 and BFSI edge cases (replay, money format, rejected transition, other role
   denied, cut-off, audit record) → you reply `ok`.
3. Runs curl, Playwright (spec reviewed) and read-only DB queries; writes `docs/bmad/{id}-qa.md` → review.
4. Checks → critic → adds `qa-passed` (and `done` if merged) → review.

Failures are evidence only, never root-cause guesses. **Next:** Docs.

### Step 7 — Keep docs true · `/doc {id}`
1. Finds what the merged change touched; creates `docs/{id}-sync`.
2. Updates the API spec → review; updates the module `AGENTS.md` (never protected headings) → review.
3. Checks → critic → commit → **approve** push → **approve** docs PR.

**Next:** PM for the next requirement.

## B3. Helper agents

| Agent | Use | Never |
|---|---|---|
| `bmad-bfsi-advisor` | Domain questions: allocation residuals, T+1, day counts, ISO 20022 messages. Gives rule ids, a worked example, edge cases, a compliance note | Edits files or gives final legal answers |
| `bmad-critic` | Runs inside each stage after checks pass; clean context; PASS/FAIL with `path:line` evidence | Sees the author's reasoning; edits anything |

## B4. JIRA and Confluence

**No MCP or extension (default):**

| Command | Output in `docs/bmad/export/` | Then |
|---|---|---|
| `/to-jira-csv epic:{id}` | `jira/{scope}-{date}.csv` + import guide | JIRA → Import issues from CSV; map columns; test with two rows first |
| `/to-confluence {id}` | `confluence/{name}.html` and/or `.wiki.txt` | Open the HTML in a browser, copy all, paste into Confluence |

**With the Atlassian MCP server (optional):** set `tracker.type: jira`, `tracker.site` and `tracker.project_key`; add
`.vscode/mcp.json` with `{"servers":{"atlassian":{"type":"http","url":"https://mcp.atlassian.com/v2/mcp"}}}` and sign in;
add `'atlassian/*'` to the `tools:` line of the stage agents. Agents then mirror items to JIRA, asking before each write.

## B5. Work-item lifecycle

| Status | Set by | Gates along the way |
|---|---|---|
| `todo` | PM / BA | `prd-approved` (epic) → `dor-ready` → `arch-approved` |
| `in-progress` | Dev at start | — |
| `in-review` | Dev when the PR is raised | `code-review-approved` → `qa-passed` |
| `done` | QA, when QA passed and the PR is merged | — |

Each agent checks the gates it needs and stops with a clear message if one is missing. Commit `docs/bmad/` with your
normal PRs.

## B6. Worked example

| Step | You type | What happens | You do |
|---|---|---|---|
| 1 | `/pm "Allocate block executions to client accounts at average price"` | PRD `TRD-1-prd.md` with control impact | Review, keep, reply `1` |
| 2 | Hand-off → BA | Proposes 3 stories (rule config, engine, API) | `ok`, review each story |
| 3 | Hand-off → Architect on `TRD-3` | Design: residual to largest account, 6-dp average price, HALF_EVEN, outbox event | Review, keep |
| 4 | Hand-off → Dev | Layer by layer; tests include "allocations sum to block" and replay | Review each layer; approve push and PR |
| 5 | `/review pr:42` | Checks + critic; review comment | Approve posting |
| 6 | Hand-off → QA | ACs + replay + rejected transition + 403 for another owner | `ok` plan, review report |
| 7 | Hand-off → Docs | API spec and module rulebook updated, docs PR | Review, approve PR |

---

# Reference

## Troubleshooting

| Symptom | Fix |
|---|---|
| Agents don't appear in the dropdown | Files must be in `.github/agents/` as `*.agent.md`; reload VS Code |
| A config value is `TODO` | Fill `bmad.config.yaml` |
| "Run Subagent tool not available" / critic doesn't run | Enable it in the tools picker; update VS Code; or set `flow.critic: false` |
| "Gate missing: arch-approved" | Run the stage it names |
| "Working tree not clean" (Dev) | Commit or stash first |
| `gh` errors | `gh auth login` (or the agent prints the commands for you) |
| Shell snippets fail on Windows | Switch the terminal to Git Bash |
| Coverage fails on legacy code | Agree a baseline in `quality.coverage_min_line_pct`; agents never lower it |
| Stack not supported | Copy `.github/bmad/stacks/_template.md` to `{stack}.md` and fill every section |
| Too many review pauses | `review_granularity: step` |
| Loop keeps asking | `auto_loop: true` |

## Keeping token use low

- One chat per stage (start each with its slash command); agents rebuild state from `docs/bmad/`.
- Keep `review_granularity: step` and reply briefly (`keep`, `1`).
- `flow.critic: false` for low-risk documentation changes.
- Keep rulebooks factual and short; agents read them on every run.

## Updating the kit

| Keep in your app | Replace when updating |
|---|---|
| `bmad.config.yaml`, `AGENTS.md` files, `docs/bmad/` work items, your own stack packs | `.github/agents/`, `.github/prompts/`, `protocol.md`, `rubrics/`, `templates/`, `domain/` (after Compliance review) |

Update through a normal PR so CODEOWNERS review applies.

---

All agent output is assistive. A qualified reviewer validates designs, code and documents before delivery, and
Compliance confirms any regulatory point.
