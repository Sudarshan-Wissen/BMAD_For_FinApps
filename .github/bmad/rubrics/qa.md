# Rubric — qa evidence

Applied by `bmad-critic` to the QA report `{work.dir}/{id}-qa.md` written by the QA agent. Context: the story and
the check results. The checks have already measured completeness and format (every
criterion has a result, statuses agree with verdicts, envelopes, money type, screenshots, replay). Judge what it cannot.

Mark each item PASS, FAIL or N/A. Report each FAIL as a finding with the result `id` and a quote from the results file.

## Q1 — Proof per acceptance criterion
- Q1.1 Each criterion is proven by an assertion on the field and value it names, not only by the status code
- Q1.2 The request actually exercises the criterion (right endpoint, method, role and input), not a nearby case
- Q1.3 UI assertions check the behaviour the criterion describes (value, state, navigation), not only that an element exists

## Q2 — Persistence and events
- Q2.1 For each mutating criterion, the database row shown matches what the criterion changes (entity id, new state, key fields)
- Q2.2 The event shown is on the expected topic or queue and its payload matches the entity and state (BFSI-STATE-03)

## Q3 — Edge cases are real
- Q3.1 Each edge case exercises the real condition (for example 401 with no token, not a wrong URL; a genuinely
  invalid transition, not a malformed request)
- Q3.2 No criterion or edge case is SKIP or N/A without a stated precondition that is actually missing

## Q4 — BFSI evidence
- Q4.1 Money: amounts carry a currency, use the representation `api.money_json_type` requires and the currency's scale;
  invalid amounts are rejected (BFSI-MON-03, -04, -07)
- Q4.2 Idempotency: the replay returned the original result and created nothing new (BFSI-ID-01, BFSI-TEST-03)
- Q4.3 Lifecycle: the rejected transition left the state unchanged (BFSI-STATE-01, -02)
- Q4.4 Entitlement: a caller without the entitlement was denied server-side (BFSI-SEC-04)
- Q4.5 Time: boundary evidence uses the right time zone and calendar, or the SKIP reason is valid (BFSI-TIME-03, -04)
- Q4.6 Audit: each state change has an audit record with actor, UTC time and correlation id (BFSI-AUD-01, -02)

## Q5 — Evidence hygiene
- Q5.1 Only observable evidence: no root cause, component name or fix suggestion anywhere in the results
- Q5.2 Only synthetic data; no token, password, real name, account or card number in any command or response (BFSI-SEC-02, -08)
- Q5.3 No call targets a production environment
