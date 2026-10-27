# BFSI Domain Rules — the rule pack every BMAD agent applies

These rules apply to every artifact (PRD, story, design, code, tests, docs) in a Banking, Financial Services or
Insurance application. The critic cites them by id in findings (`"rule": "BFSI-MON-01"`).

- **Severity** is the default for a violation. The project rulebook may raise it, never lower a non-negotiable one.
- **Non-negotiable (NN)** rules can be relaxed only by a written, approved exception in the project rulebook that
  names the rule id.
- Rules are technology-neutral. Stack packs say how each is checked in a given language.
- This pack is engineering guidance, not legal or regulatory advice. Where a rule mentions a regulation, a
  qualified compliance professional confirms how it applies to the project.

## MON — Money and numbers

| Id | Rule | Severity |
|---|---|---|
| BFSI-MON-01 (NN) | Monetary amounts, prices, rates, FX rates, quantities of fractional instruments and fees use an exact decimal type (e.g. `BigDecimal`, `decimal`, `NUMERIC`). Never binary floating point (`float`, `double`, JS `number` for arithmetic). | blocker |
| BFSI-MON-02 (NN) | Decimals are constructed from strings or integers, never from a float literal or float value. | blocker |
| BFSI-MON-03 (NN) | Every amount travels with its ISO 4217 currency code. No bare amount crosses a module boundary, an API or a table. | blocker |
| BFSI-MON-04 | Scale follows the currency's minor units (e.g. JPY 0, USD 2, BHD 3) or a documented instrument precision. Prices and rates may carry more decimals than amounts; the precision is stated in the rulebook. | major |
| BFSI-MON-05 (NN) | Every rounding states its mode explicitly (e.g. HALF_EVEN for banking, HALF_UP where a rule requires it) and happens at a documented step, once. No implicit rounding by the type or the database. | blocker |
| BFSI-MON-06 | Amount equality uses value comparison (`compareTo` = 0), not scale-sensitive equality. | major |
| BFSI-MON-07 | Amounts in JSON are strings (or integers in minor units) as `api.money_json_type` states, never JSON floats. | major |
| BFSI-MON-08 | Currency conversion records the rate, the rate source, the rate timestamp and the direction. | major |

## TIME — Time, dates and calendars

| Id | Rule | Severity |
|---|---|---|
| BFSI-TIME-01 (NN) | Instants are stored and exchanged in UTC (ISO 8601 with `Z` or an offset). Local market or branch time is derived for display and for cut-off logic only. | blocker |
| BFSI-TIME-02 | Business dates (trade date, value date, settlement date, booking date, accrual date) are date-only types, separate from instants. | major |
| BFSI-TIME-03 | Cut-offs, market hours and end-of-day use the named time zone of the market or entity (e.g. `America/New_York`, `Europe/London`, `Asia/Kolkata`), never a fixed offset, so daylight saving is handled. | major |
| BFSI-TIME-04 | Business-day arithmetic (T+n settlement, payment value dates, maturity roll) uses a holiday calendar per market or currency and a stated roll convention (Following, Modified Following, Preceding). | major |
| BFSI-TIME-05 | Time is read from an injectable clock so cut-off, expiry and end-of-day logic is testable. | minor |

## ID — Identity, idempotency and reference data

| Id | Rule | Severity |
|---|---|---|
| BFSI-ID-01 (NN) | Every externally triggered, state-changing request (order submission, payment initiation, transfer, allocation, booking) carries an idempotency key or client reference. A replay returns the original result and creates nothing new. | blocker |
| BFSI-ID-02 | Duplicate detection rules (same key fields within a window) are stated in the design and tested. | major |
| BFSI-ID-03 | Instrument, party and venue identifiers use standard schemes and are validated: ISIN (12 characters, check digit), CUSIP (9), SEDOL (7), FIGI, LEI (20, ISO 17442), MIC (ISO 10383), BIC (8 or 11), IBAN (check digits), ISO 3166 country codes. | major |
| BFSI-ID-04 | Internal ids are opaque (UUID or equivalent), never sequential numbers exposed in APIs. | minor |

## STATE — Lifecycles and state machines

| Id | Rule | Severity |
|---|---|---|
| BFSI-STATE-01 (NN) | Lifecycle entities (order, execution, allocation, payment, loan, claim) move only through the transitions the rulebook defines. Transitions are validated in one place. | blocker |
| BFSI-STATE-02 (NN) | Terminal states are immutable. A correction is a new compensating record (cancel/correct, reversal, bust), never an in-place edit. | blocker |
| BFSI-STATE-03 | Each transition is persisted atomically with its outbound event (transactional outbox or equivalent), so no state change is lost or published twice. | major |
| BFSI-STATE-04 | Concurrent updates to the same balance, position, limit or order use optimistic or pessimistic locking. A lost update is a defect. | major |
| BFSI-STATE-05 | Message consumers are idempotent: they check the current state before applying an event and ignore duplicates and out-of-order stale events. | major |

## LEDGER — Balances, positions and books

| Id | Rule | Severity |
|---|---|---|
| BFSI-LEDGER-01 (NN) | Money movements are double-entry: every posting has balanced debits and credits per currency. Balances are derived from postings or reconciled to them. | blocker |
| BFSI-LEDGER-02 | Available balance, ledger (book) balance, holds/reservations and uncleared funds are distinct and named. | major |
| BFSI-LEDGER-03 | Positions distinguish trade-date and settlement-date quantities. | major |
| BFSI-LEDGER-04 | Every book of record has a reconciliation (internal vs counterparty, custodian, nostro or scheme) with breaks reported, not silently corrected. | major |

## TRADE — Capital-markets lifecycle

| Id | Rule | Severity |
|---|---|---|
| BFSI-TRADE-01 (NN) | No order reaches execution without passing pre-trade risk and compliance checks (limits, restricted list, short-sale locate where required). A risk-service outage fails closed (reject or hold), never open. | blocker |
| BFSI-TRADE-02 | Fills are applied in order and the cumulative filled quantity never exceeds the order quantity. Average price is recomputed from fills with stated precision. | major |
| BFSI-TRADE-03 (NN) | Allocations of a block sum exactly to the block's executed quantity. The residual from rounding goes to an account chosen by a documented, deterministic rule. | blocker |
| BFSI-TRADE-04 | Each allocation carries its own price (average or per fill, as configured), commission, fees, taxes, settlement date and settlement instructions. | major |
| BFSI-TRADE-05 | Trade corrections, cancels and busts produce new records linked to the original; the original is never edited. | major |
| BFSI-TRADE-06 | Settlement date is computed from trade date, the market's settlement cycle and its holiday calendar. | major |
| BFSI-TRADE-07 | Self-trade, wash-trade and duplicate-order prevention rules are stated in the design and tested where the product allows crossing. | major |

## PAY — Payments

| Id | Rule | Severity |
|---|---|---|
| BFSI-PAY-01 (NN) | Payments are screened against sanctions lists before release. A screening outage holds the payment, never releases it. | blocker |
| BFSI-PAY-02 | Scheme cut-off times and value dating are applied in the scheme's time zone and calendar. | major |
| BFSI-PAY-03 | Status reports (e.g. ISO 20022 pacs.002, return and reject reason codes) map to internal states one to one, and unknown codes go to a manual queue. | major |
| BFSI-PAY-04 | Payment messages are validated against the scheme's schema and field rules before sending. | major |

## LEND — Lending and interest

| Id | Rule | Severity |
|---|---|---|
| BFSI-LEND-01 | Interest uses the product's day-count convention (ACT/360, ACT/365F, 30/360, ACT/ACT) and compounding rule, stated in the design. | major |
| BFSI-LEND-02 | Amortisation schedules reconcile: the sum of principal repayments equals the principal, and the final instalment absorbs rounding. | major |
| BFSI-LEND-03 | Delinquency (days past due) and provisioning stage are computed from dated events, never from a mutable counter. | major |

## RISK — Limits and controls

| Id | Rule | Severity |
|---|---|---|
| BFSI-RISK-01 | Limits and thresholds (credit, position, notional, fat-finger, velocity) are configuration with an owner, not literals in code. | major |
| BFSI-RISK-02 (NN) | Changes to limits, risk rules, fee schedules, rates and reference data follow maker-checker: the maker cannot approve their own change, and both are recorded. | blocker |
| BFSI-RISK-03 | A kill switch or trading halt exists for automated flows that can create financial exposure. | major |

## AUD — Audit and records

| Id | Rule | Severity |
|---|---|---|
| BFSI-AUD-01 (NN) | Every state change and every privileged action writes an append-only audit record: who (actor id and type USER/SYSTEM), what, when (UTC), before and after, and the correlation id. Audit records are never updated or deleted by the application. | blocker |
| BFSI-AUD-02 | A correlation or request id flows end to end across APIs, messages and logs. | major |
| BFSI-AUD-03 | Retention periods for records follow the rulebook. The application does not hard-delete records under retention. | major |

## SEC — Security and data protection

| Id | Rule | Severity |
|---|---|---|
| BFSI-SEC-01 (NN) | No secrets, keys, tokens, passwords or connection strings in code, configuration files, tests, fixtures, logs or documentation. They come from a secret store or environment. | blocker |
| BFSI-SEC-02 (NN) | No PII or financial identifiers in logs, metrics, traces, errors or exception messages: names, addresses, government ids, dates of birth, full account numbers, card numbers, balances tied to a person. Log ids and states. | blocker |
| BFSI-SEC-03 (NN) | Card data: never store CVV/CVC, PIN or full track data. Display at most the first 6 and last 4 digits of a card number. Prefer tokens. | blocker |
| BFSI-SEC-04 | Authorisation is enforced server-side for every endpoint and every record (ownership / entitlement check), not only by the UI or the gateway. | blocker |
| BFSI-SEC-05 | Queries are parameterised. No SQL, LDAP or command strings built by concatenating input. | blocker |
| BFSI-SEC-06 | Errors returned to clients carry a code and a safe message, never stack traces, SQL or internal hostnames. | major |
| BFSI-SEC-07 | Sensitive data is encrypted in transit (TLS) and at rest where the rulebook classifies it as confidential or restricted. | major |
| BFSI-SEC-08 | Test data is synthetic. No production extracts in the repository. | blocker |

## RES — Resilience

| Id | Rule | Severity |
|---|---|---|
| BFSI-RES-01 | Calls to other services have timeouts, bounded retries with backoff, and a circuit breaker or equivalent. Retries are safe because the target is idempotent (BFSI-ID-01). | major |
| BFSI-RES-02 | Failed messages go to a dead-letter destination with enough context to replay, and replay is idempotent. | major |
| BFSI-RES-03 | Batch and end-of-day jobs are restartable from a checkpoint and do not double-post on restart. | major |

## TEST — Evidence for financial logic

| Id | Rule | Severity |
|---|---|---|
| BFSI-TEST-01 | Money, rounding, allocation, interest and fee logic has boundary tests: zero, smallest unit, rounding ties, maximum values, negative values where they are invalid. | major |
| BFSI-TEST-02 | Cut-off, expiry and calendar logic has tests on both sides of the boundary, across a daylight-saving change and on a holiday. | major |
| BFSI-TEST-03 | Idempotency is tested by replaying the same request or message and asserting no second effect. | major |
| BFSI-TEST-04 | Every state transition and every rejected transition has a test. | major |
