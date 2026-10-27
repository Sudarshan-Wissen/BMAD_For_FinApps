# Rubric — dev 12-category

Applied by `bmad-critic` to the code diff written by the Dev agent, and reused by the Code Review agent
(rubric name `code-review 12-category`). Each category combines the generic items below with the matching row of
the module's stack pack (section 6) and the project rulebook. Categories 1, 5, 11 and 12 are zero-tolerance.

Mark each item PASS, FAIL or N/A. Report each FAIL as a finding with `path:line` evidence.

## C1 — Money and numerics (zero tolerance)
- C1.1 Exact decimal types for every amount, price, rate and fee (BFSI-MON-01)
- C1.2 Decimals built from strings or integers (BFSI-MON-02)
- C1.3 Currency travels with every amount (BFSI-MON-03); scale matches the currency or documented precision (BFSI-MON-04)
- C1.4 Explicit rounding mode at a documented step (BFSI-MON-05); value comparison for equality (BFSI-MON-06)
- C1.5 Money in JSON follows `api.money_json_type` (BFSI-MON-07)

## C2 — Naming and structure
- C2.1 Names and layout follow the rulebook and the stack pack
- C2.2 Database objects, API paths, JSON fields, message topics and cache keys follow the rulebook's conventions

## C3 — Framework conventions
- C3.1 Stack pack conventions for this framework are followed (stack pack §6, row 3)

## C4 — Dependency management and injection
- C4.1 No new dependency unless the brief lists it as approved
- C4.2 Stack pack injection and wiring conventions are followed

## C5 — Architecture boundaries (zero tolerance)
- C5.1 Thin entry points: no business logic in controllers or handlers
- C5.2 Persistence objects are never exposed outside the service layer
- C5.3 No access to another module's schema or internal storage; cross-module access only through the rulebook's
  allowed channels (API, messaging)
- C5.4 The implementation matches the approved design: endpoints, fields, events, error codes. A deviation is a
  finding, not a judgement call.
- C5.5 No other module is left using a removed or renamed contract

## C6 — API contract
- C6.1 Success and error envelopes match `api.success_envelope` and `api.error_envelope`
- C6.2 Paths, methods, status codes and field names match the design and the API spec file
- C6.3 Timestamps ISO 8601 UTC (BFSI-TIME-01); business dates as dates (BFSI-TIME-02)

## C7 — Persistence and migrations
- C7.1 Migrations are additive, ordered, own-schema only; no destructive statements without an approved plan
- C7.2 Concurrency control on balances, positions, limits and lifecycle entities (BFSI-STATE-04)
- C7.3 Required columns are non-null; amounts have explicit precision and scale

## C8 — Logging and data protection
- C8.1 No PII, card data, secrets or person-linked balances in logs, errors or metrics (BFSI-SEC-02, BFSI-SEC-03)
- C8.2 Correlation id propagated (BFSI-AUD-02); state transitions logged with ids and states only

## C9 — Error handling
- C9.1 Domain exceptions carry machine-readable error codes; a global handler maps them to the error envelope
- C9.2 No stack traces or internals in responses (BFSI-SEC-06); no generic exceptions thrown from business logic

## C10 — Testing
- C10.1 A test for every changed public service or business-rule method, and for every acceptance criterion
- C10.2 Entry-point tests cover success, validation failure (400), not found (404) and business-rule rejection (409/422)
- C10.3 BFSI boundary tests where the change touches money, rounding, allocation, interest or fees (BFSI-TEST-01);
  cut-offs, expiry or calendars (BFSI-TEST-02); idempotency (BFSI-TEST-03); state transitions (BFSI-TEST-04)
- C10.4 Existing UI end-to-end specs that asserted changed behaviour are updated in the same change
- C10.5 No test, assertion or coverage exclusion was deleted or weakened

## C11 — Security (zero tolerance)
- C11.1 No hard-coded secrets (BFSI-SEC-01); synthetic test data only (BFSI-SEC-08)
- C11.2 Server-side authorisation per endpoint and per record (BFSI-SEC-04)
- C11.3 Parameterised queries only (BFSI-SEC-05)

## C12 — BFSI lifecycle integrity (zero tolerance)
- C12.1 Only rulebook transitions; terminal states immutable; corrections as new records (BFSI-STATE-01, -02, BFSI-TRADE-05)
- C12.2 State change and outbound event are atomic; consumers idempotent (BFSI-STATE-03, -05)
- C12.3 Idempotency key honoured on state-changing requests (BFSI-ID-01)
- C12.4 Domain invariants hold where touched: double entry balanced (BFSI-LEDGER-01), allocations sum to the block
  (BFSI-TRADE-03), fills never exceed the order (BFSI-TRADE-02), settlement dates from calendars (BFSI-TRADE-06),
  interest and amortisation conventions (BFSI-LEND-*), sanctions hold (BFSI-PAY-01)
- C12.5 Controls fail closed (BFSI-TRADE-01, BFSI-PAY-01); outbound calls have timeouts and safe retries (BFSI-RES-01)
- C12.6 Audit record for every state change and privileged action (BFSI-AUD-01); maker-checker for configuration
  changes (BFSI-RISK-02)
