# BMAD Protocol — every BMAD agent follows this

Runs in VS Code, GitHub Copilot Chat, **Agent mode**. Shell snippets are bash (Git Bash or WSL on Windows).

## 1. Start-up (keep it cheap)

1. `START=$(date +%s)`. Read `.github/bmad/bmad.config.yaml`. If a value this role needs is `TODO`, stop and list it.
2. Read the root rulebook (`rulebook.root`) and, if the work names a module, its module rulebook. If one is missing,
   say so once and continue (suggest the onboard agent).
3. Read `.github/bmad/domain/bfsi-rules.md`. Read `bfsi-domain-pack.md` **only the sections** the work touches
   (find them with `grep -n '^##' .github/bmad/domain/bfsi-domain-pack.md`, then read that line range).
4. Code roles: read the stack pack `.github/bmad/stacks/{stack}.md` of each module involved.

Project rulebook > stack pack > agent defaults. BFSI rules marked NN can be relaxed only by a rulebook exception
naming the rule id.

## 2. Token discipline

- Never read a file twice in one run; never read a whole large file when a heading range or `grep -n` will do.
- Pipe command output through `tail -40`, `grep` or `wc -l`; never paste whole logs or files into chat.
- Keep chat replies short: what changed, what was checked, the decision needed. Details belong in files.
- The critic gets file paths, never pasted content.

## 3. Work items (no tracker needed)

All artifacts are Markdown files in `work.dir` (default `docs/bmad`), committed with the code:

| File | Written by | Holds |
|---|---|---|
| `{id}-prd.md` | PM | PRD for an epic |
| `{id}.md` | BA | one story (front matter below) |
| `{id}-design.md` | Arch | technical design for a story |
| `{id}-qa.md` | QA | acceptance evidence |
| `backlog.md` | PM, BA | index: `| id | type | title | status | gates |` |

Story and epic front matter:

```yaml
---
id: PAY-7            # {project.key}-{n}; n = 1 + highest n in work.dir
type: story          # epic | story
epic: PAY-3
status: todo         # todo | in-progress | in-review | done
gates: []            # prd-approved · dor-ready · arch-approved · code-review-approved · qa-passed
depends_on: []
points: 3
---
```

A gate is added only when the stage passed its checks **and** you kept the artifact. Agents read gates from these
files, never from memory.

**JIRA is optional.** Only if `tracker.type: jira` and Atlassian tools are present in this chat, the agent also mirrors
the change (create issue, comment, label, transition), each behind an approval (§6). If the tools are missing, say so
once and continue with files only.

## 4. Change review — after every change

When `flow.review_changes` is true, after each step (`review_granularity: step`) or each file (`file`), stop and print:

```
REVIEW {n} — {role} — {step name}
  changed: {path} — {one-line summary}   (one line per file)
  checks : {what was already verified, or "not yet"}
Review the diff in the editor (Keep / Undo), then reply:  keep | revise: <notes> | discard
```

End your turn and wait. **keep** → continue. **revise** → apply the notes, then show the review block again.
**discard** → restore the files (`git checkout -- <tracked files>`; delete files you created) and ask how to proceed.
Never batch several steps into one review, and never continue without a reply.

## 5. Checks, critic and the loop

Each attempt: **produce/fix → change review (§4) → checks → critic (if `flow.critic`) → decision**.

- **Checks** are the deterministic commands listed in the agent (grep, build, tests, coverage). Report each as
  `PASS | FAIL | WARN (tool missing)` with the decisive output line. Never weaken a check, test, threshold or rule to
  make it pass.
- **Critic** runs only when all checks pass. Write any facts it cannot read itself into `.bmad/review/{id}-facts.md`.
  Run the `bmad-critic` subagent with only: `ARTIFACT` path(s), `RUBRIC` file `.github/bmad/rubrics/{rubric}.md`,
  `RULEBOOK` paths, `DOMAIN` (`bfsi-rules.md`, plus the domain-pack sections in scope), `STACK` (code only), and from
  attempt 2 `PRIOR_FINDINGS` (ids, rule, location). Never pass your reasoning. Before using a `location` finding,
  confirm the quote is at that line; drop it if not.
- **Scorecard** (one block): `Attempt n/max · Checks PASS/FAIL (k) · Critic PASS/FAIL (k findings) · open items`.

**Decision**

| Situation | `auto_loop: false` (default) | `auto_loop: true` |
|---|---|---|
| Attempt failed, attempts left | Ask `1 next agent (open items recorded) · 2 continue loop (fix k items) · 3 stop` and wait | Fix the items and re-check automatically |
| Attempt failed, none left | Escalate: list open items with evidence, and stop | Same |
| Stage passed | Write the gate (§3, it is a change → §4), then ask `1 next agent · 2 continue loop (another improvement pass) · 3 stop` | Write the gate, then offer the hand-off button |

"Next agent" means the hand-off button under the reply (or the next prompt in `README.md`). Moving on with open
items records them in the artifact under `## Open items`.

## 6. Approvals — always asked, whatever the mode

Ask before each: `git push`, `gh pr create`, `gh pr review`, any JIRA or Confluence write, any edit to a rulebook file,
and any change outside the role's paths. Print `ACTION — target — exactly what will be written — Proceed? (yes/no)`,
end the turn, act only on "yes". Never: push to or commit on `git.protected_branches`, merge, force-push, rewrite
history, or edit `.github/workflows`, CI or infrastructure files. Local commits on a feature branch need no approval.
If `gh` is not signed in, print the commands for the user instead.

## 7. BFSI data handling

Synthetic data only. If an input contains what looks like real PII, card or account numbers, or a secret, stop using
it, say where it is, and remind the user an exposed secret must be rotated. Read credentials only from the env vars
named in config and never print them. Never target a URL containing `prod`. Legal, tax and regulatory questions are
recorded as open questions for a qualified human, never answered as final.

## 8. End of stage

Print: `BMAD {ROLE} — {id} — {PASSED | ESCALATED | STOPPED}`, the files written, the gate set, open items, the next
agent, and the time (`e=$(( $(date +%s)-START )); echo "$((e/60))m $((e%60))s"`).
