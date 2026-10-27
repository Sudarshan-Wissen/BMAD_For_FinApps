---
description: BFSI coding rules applied whenever Copilot writes or reviews source, test, SQL or API files
applyTo: "**/*.{java,kt,scala,cs,py,go,ts,tsx,js,jsx,sql,yaml,yml}"
---

# BFSI coding rules (summary of `.github/bmad/domain/bfsi-rules.md`)

Apply these to every line you write or review in this repository. Cite the rule id when you flag a violation.
The project rulebook (`AGENTS.md`) may add rules; it never removes a non-negotiable one without a written exception.

- **BFSI-MON-01/02/03/05/06:** exact decimals for money, prices, rates and fees, built from strings; currency with every
  amount; explicit rounding mode; value comparison for equality. No `float`, `double`, JS `number` arithmetic,
  `parseFloat` or `toFixed` on money.
- **BFSI-MON-07:** money in JSON as strings, or integer minor units, as the project's config states.
- **BFSI-TIME-01..05:** UTC instants; business dates as dates; named time zones for cut-offs; holiday calendars for
  T+n; an injectable clock.
- **BFSI-ID-01, BFSI-STATE-01..05:** idempotency keys on state-changing requests; one place validates transitions;
  terminal states immutable; state change and event atomic (outbox); locking on balances and positions; idempotent
  consumers.
- **BFSI-LEDGER-01, BFSI-TRADE-02/03/06:** balanced double entry; fills never exceed order quantity; allocations sum
  to the block with a deterministic residual rule; settlement dates from the market calendar.
- **BFSI-TRADE-01, BFSI-PAY-01:** risk, compliance and sanctions controls fail closed.
- **BFSI-AUD-01/02:** append-only audit records for state changes and privileged actions; correlation id end to end.
- **BFSI-SEC-01..08:** no secrets; no PII, card data or person-linked balances in logs, errors or metrics; never store
  CVV or PIN; server-side authorisation per record; parameterised queries; safe error messages; synthetic test data.
- **BFSI-RES-01..03:** timeouts, bounded retries and circuit breakers on outbound calls; dead-letter with idempotent
  replay; restartable batch jobs.
- **BFSI-TEST-01..04:** boundary tests for money and rounding, both sides of cut-offs, replay tests for idempotency,
  a test for every allowed and rejected transition.

In code review, report each violation with the file and line, the rule id, and the required fix. Do not report style
preferences that no rule states.
