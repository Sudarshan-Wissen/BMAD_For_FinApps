# Rubric — pm PRD 13-item

Applied by `bmad-critic` to `{work.dir}/{id}-prd.md` written by the PM agent.
Mark each item PASS, FAIL or N/A. Report each FAIL as a finding with `path:line` evidence or an absence.

## Product quality
- P1 Problem owner identified: a specific role from the rulebook (or `system`), not "users".
- P2 Gap stated: what the role cannot do today is specific and observable, not vague.
- P3 Business impact stated: a concrete risk, inefficiency, financial loss or compliance exposure.
- P4 Solution summary: at least one paragraph describing the proposed behaviour, with no technical design.
- P5 Success criteria: at least two measurable outcomes, each verifiable by an API call, query, report or visible
  UI change.
- P6 Out of scope present: cites the rulebook's out-of-scope heading where applicable.
- P7 Single owning domain: the feature maps to one owning domain or module from the rulebook; any others appear only
  as dependencies.
- P8 Dependencies mapped: blocked-by and enables are present, or explicitly "None".
- P9 No out-of-scope item in P1, P2 or P3.
- P10 P1 scope is plausible for one delivery increment; anything larger is flagged for a split.

## BFSI
- P11 Regulatory and control impact complete and plausible. Each line has a value or a reasoned N/A. Audit,
  maker-checker, reconciliation and data classification match what the feature actually does (for example, a
  feature that changes limits, fees or reference data needs maker-checker, BFSI-RISK-02; one that moves money needs a
  ledger reconciliation, BFSI-LEDGER-04). Regulatory statements are framed as orientation, and uncertain points are
  listed as open compliance questions, not asserted.
- P12 Domain correctness: terminology, lifecycle states and flows agree with the domain pack and the rulebook (for
  example, allocations follow execution, settlement dates follow the market calendar, payments are screened before
  release). A control success criterion exists when section 4 is not all N/A.
- P13 Revision fit (`epic:` mode only, else N/A): the revised PRD addresses every point of the escalation or change
  request in `FACTS`, and keeps unaffected content unchanged.
