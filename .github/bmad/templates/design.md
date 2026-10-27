# Design — {id} — {story title}

Every block is filled or says `N/A — {reason}`. Use the stack pack §9 for stack-specific detail.

## D1 Scope and module boundary
Module: {owning module} · Other modules touched: {only as calls/events, or none}

## D2 Interface design
Per entry point: method + path (or message / job / page), auth, request fields (name, type, required, constraint),
response fields (name, type, source), and an error table `status | error.code | condition` covering every AC failure.

## D3 Data model and persistence
Schema: {module's own} · Migration: {file name} · DDL or `no schema change` · indexes · locking column.

## D4 Domain model
Entities, value objects (Money, Quantity), enums with values.

## D5 Business logic
Class / function and signature; numbered steps; error paths → exception and error code.

## D6 Lifecycle and state machine delta
Allowed transitions, rejected transitions (error code), terminal states, corrections as new records.

## D7 Money and precision
Currency source, scale per currency, rounding mode and the step where it happens, JSON representation;
allocation method and residual rule if allocation is touched.

## D8 Time, calendars and cut-offs
Instants vs business dates, named time zone, calendar, roll convention, settlement or value date rule.

## D9 Idempotency and duplicate detection
Key, storage, replay result, duplicate window.

## D10 Consistency and concurrency
Transaction boundary, outbox or equivalent, locking, idempotent consumers.

## D11 Outbound calls and resilience
Target, purpose, timeout, retries with backoff, circuit breaker, fallback (fail closed where it is a control).

## D12 Events and messaging
Produced and consumed topics or queues, payload fields, consumer group, retry, dead-letter, replay.

## D13 Caching
Key pattern, TTL, invalidation trigger — or N/A.

## D14 Controls
Limits, risk, compliance or sanctions checks applied; behaviour when unavailable.

## D15 Security and entitlements
Per role in `api.roles`: allowed / blocked, per-record ownership field, id domain of every identifier compared,
forwarded headers or claims.

## D16 Data classification
Each new field: class, masking in logs and UI, encryption.

## D17 Audit
Audit record per state change and privileged action: fields, storage, correlation id.

## D18 Reconciliation, reporting and regulatory impact
Books affected, reports affected, open compliance questions (orientation only).

## D19 Test strategy
At least 3 cases per layer, including error paths and the BFSI boundary, replay and transition tests.

## D20 API spec delta
Paths, operations and schemas to add or change in the API spec file — or no change.
