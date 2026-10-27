# Rubric — arch DoR

Applied by `bmad-critic` to the technical design `{work.dir}/{id}-design.md`, with the story as context.
Mark each item PASS, FAIL or N/A. N/A is allowed only where the design states
`N/A — {reason}` in the matching block and the reason holds. Report each FAIL as a finding with evidence.

## Definition of Ready

- A1 The story has a clear user story (As a / I want / So that).
- A2 Every acceptance criterion is verifiable: it maps to an observable assertion (HTTP status and body, an event
  published, a stored state, or a UI outcome).
- A3 The module boundary is clear: each change has one owning module from config, and the design names it (D1).
- A4 No blocked-by ticket is open (from the supplied facts).
- A5 Nothing in the design depends on an item under the rulebook's out-of-scope heading.
- A6 The interface is confirmed: method and path, or message, job or page, for every acceptance criterion (D2).
- A7 Request and response shapes are designed: every field named and typed (D2, D4).
- A8 An error code is defined for every failure path in the acceptance criteria (D2 error table).
- A9 The schema change is scoped to the module's own schema, or explicitly not needed (D3).
- A10 Outbound calls are identified with timeouts, bounded retries and a breaker or fallback (D11).
- A11 The test strategy lists at least 3 cases per layer, including error paths (D19).

## Design Completeness Rule (hard stop)

- A12 The exact field used for every ownership or identity check is named (D15).
- A13 The id domain of every identifier is stated, and every comparison is between ids of the same domain (D15).
- A14 Entitlements per role in `api.roles` for every endpoint: who is allowed, who is blocked, which records (D15).
- A15 Forwarded headers or claims are listed with their meaning, or `none` with a reason (D15).

## BFSI design integrity

- A16 Lifecycle delta is complete and consistent with the rulebook: allowed and rejected transitions, terminal states,
  corrections as new records (D6; BFSI-STATE-01, -02, BFSI-TRADE-05).
- A17 Money is specified: currency, scale, rounding mode and step, JSON representation; allocations sum to the block
  with a deterministic residual rule (D7; BFSI-MON-*, BFSI-TRADE-03, -04).
- A18 Time is specified: instants vs business dates, named time zone for cut-offs, calendars and roll convention,
  settlement or value date rule (D8; BFSI-TIME-*, BFSI-TRADE-06, BFSI-PAY-02).
- A19 Idempotency and duplicate detection are specified for every state-changing entry point (D9; BFSI-ID-01, -02).
- A20 Consistency is specified: transaction boundary, outbox or equivalent, locking, idempotent consumers
  (D10; BFSI-STATE-03, -04, -05).
- A21 Controls apply where needed and fail closed (D14; BFSI-TRADE-01, BFSI-PAY-01, BFSI-RISK-01, -02).
- A22 Audit records are specified for every state change and privileged action (D17; BFSI-AUD-01, -02).
- A23 Every new field is classified, with masking and encryption where required (D16; BFSI-SEC-02, -03, -07).
- A24 Reconciliation, reporting and regulatory impact are stated, and every regulatory question is flagged for
  compliance confirmation rather than answered (D18; BFSI-LEDGER-04).
- A25 Resilience is specified: dead-letter handling and safe replay for messaging, restartable batch steps
  (D11, D12; BFSI-RES-*).

## Domain soundness (judge's own check, reported under the closest item)

The critic also reports, under the closest item, any design choice that would break a domain invariant even when
every block is filled in. Examples: allocating at fill prices when the rulebook requires the average price,
settling on a market holiday, releasing an unscreened payment, double-posting on a retry, or editing a terminal
record.
