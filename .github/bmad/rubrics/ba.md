# Rubric — ba story 12-item

Applied by `bmad-critic` to the stories `{work.dir}/{id}.md` written by the BA agent, with the PRD as context.
Mark each item PASS, FAIL or N/A. Report each FAIL as a finding with `path:line` evidence or an absence.

## Story quality
- S1 User story format: "As a / I want / So that" with a specific role from the rulebook, and a real business outcome.
- S2 Verifiable ACs: every AC is checkable by one observation. `[API]` ACs give method, path, status code, and the
  response field and value or error code. Other ACs name the exact target (table and column, topic or queue and event,
  screen element, job output) and the expected value.
- S3 Single module boundary: the story lives entirely in one module; other modules appear only as calls or
  dependencies.
- S4 Story points: one of 1, 2, 3, 5, 8, and plausible for the ACs.
- S5 Blocked-by links set, or explicitly "None", and consistent with the dependencies in `FACTS`.
- S6 No out-of-scope items anywhere in the story.
- S7 Tech constraints complete: module, port, rulebook instruction, API spec, branch pattern, PR target, blocked-by.
- S8 Seed or synthetic test data identified, enough for Dev and QA to set up every AC. No real customer data.
- S9 AI Context complete: module and port, call path, response field sources, external calls, lifecycle entity.

## BFSI
- S10 BFSI acceptance criteria present for every consideration that applies: money and currency with scale and
  rounding (BFSI-MON-*), idempotent replay for state-changing requests (BFSI-ID-01), allowed and rejected transitions
  (BFSI-STATE-01/02), entitlement 403 cases (BFSI-SEC-04), audit record (BFSI-AUD-01), cut-off and calendar
  boundaries (BFSI-TIME-03/04), fail-closed controls (BFSI-TRADE-01, BFSI-PAY-01). Every "N/A" has a reason that
  holds.
- S11 Domain correctness: terms, states and flows agree with the domain pack and rulebook; touched invariants
  (allocation sums and residuals, fill limits, balanced postings, settlement and value dates, day-count conventions)
  are stated correctly and testably; nothing traces to a requirement that does not exist.
- S12 Regulated data and compliance: the data classification is stated and plausible; masking needs are noted for
  restricted fields; regulatory points are framed as questions with an owner, not asserted as settled.
