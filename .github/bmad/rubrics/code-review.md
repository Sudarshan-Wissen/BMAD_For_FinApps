# Rubric — code-review 12-category

Applied by `bmad-critic` to the PR diff at the reviewed head SHA, with the story and design as context.

## Part 1 — Code categories C1–C12

Apply every item of `.github/bmad/rubrics/dev.md` (C1 Money and numerics through C12 BFSI lifecycle integrity)
exactly as written there, with the module's stack pack §6 rows and the project rulebook. Categories C1, C5, C11 and
C12 are zero-tolerance. Report each item in `items` with its dev id (for example `C1.3`).

## Part 2 — PR-level items CR1–CR8

Mark each item PASS, FAIL or N/A. Report each FAIL as a finding with evidence (`path:line` and quote, or an
`absence` stating where you searched).

- **CR1 — Traceability.** The PR names its ticket, and every changed module is one the story or design names.
- **CR2 — Design scope.** The diff implements the approved design (the `CONTEXT` design file): the same endpoints,
  fields, events, error codes and tests. Nothing the design did not ask for. If the facts file says
  `design: missing`, judge against the story's acceptance criteria and report the missing design as a major finding.
- **CR3 — No unrelated changes.** No drive-by refactors, formatting-only churn, renamed files or dependency bumps
  outside the story's scope.
- **CR4 — Tests at the reviewed SHA.** The build, tests and coverage checks ran on the reviewed SHA (facts), and
  the tests in the diff exercise the changed behaviour, including the BFSI boundary cases dev C10.3 lists. A test
  that asserts nothing meaningful is a finding.
- **CR5 — CI evidence.** CI check-runs on the reviewed SHA are green (check C8). A WARN for a run in progress is
  reported as a major finding to re-run once CI finishes.
- **CR6 — Protected paths.** No change to `.github/**`, `.vscode/**`, rulebook files, CI files or the stack pack's
  protected configuration, unless the facts say the PR carries the label `git.policy_change_label` (default `bmad-policy-change`). Even then, flag every
  such change for human review (major).
- **CR7 — Contract compatibility.** No other module is left using a removed or renamed
  endpoint, field, topic, queue, event schema or header, unless the change is backward compatible (dev C5.5).
- **CR8 — Reversibility.** The change can be rolled back by reverting the PR: migrations are additive or come with
  a documented rollback, and no irreversible data transformation runs without an approved plan.

## Verdict

FAIL if any blocker or major finding exists, any zero-tolerance item FAILs, or any CR item FAILs.
