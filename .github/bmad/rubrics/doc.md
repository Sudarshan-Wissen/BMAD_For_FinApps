# Rubric — doc completeness

Applied by `bmad-critic` to the documentation diff written by the Docs agent, with the merged code diff as context.
Judge only what the merged PR introduced; gaps that existed before it are reported as `minor` debt, not FAIL.
Mark each item PASS, FAIL or N/A, and report each FAIL as a finding with evidence.

## API spec

- D1 Every endpoint the merged PR added or changed has a path entry in the module's API spec, with every status code
  the code returns.
- D2 Every new or changed response field appears in the corresponding component schema, with the right name,
  type and required flag.
- D3 Every new error status and error code has a documented response using the error envelope schema.
- D8 Field types are correct: amounts per `api.money_json_type` (never `type: number`) with a currency field; ids
  `format: uuid` (or the rulebook's id format); instants `format: date-time`; business dates `format: date`; enums
  list every accepted value (BFSI-MON-07, BFSI-TIME-01, -02).

## Module rulebook

- D4 Every event, topic or queue the PR added (produced or consumed) is documented.
- D5 Every new cache key pattern is documented, with TTL and invalidation.
- D6 Every new environment variable or configuration key is documented, without its value.
- D7 Every new dependency is documented with its purpose.

## BFSI documentation integrity

- D9 Lifecycle states and transitions changed by the PR are reflected in the rulebook's state table, including
  rejected transitions and terminal states (BFSI-STATE-01, -02).
- D10 New monetary fields document currency source, scale and rounding mode (BFSI-MON-03, -04, -05).
- D11 New fields above internal classification are marked, masked in examples, and spec examples contain no
  real-looking personal or financial data (BFSI-SEC-02, -03, -08).
- D12 New state-changing operations document their idempotency key header or field (BFSI-ID-01).

## Accuracy and scope

- D13 The documentation matches the merged code: no endpoint, field, code, event or behaviour is documented that the
  code does not have, and none the code has is contradicted.
- D14 The root rulebook changed only for an architecture-level change (new module, shared event, new pattern,
  root-owned lifecycle), and no heading in `rulebook.protected_headings` was edited.
