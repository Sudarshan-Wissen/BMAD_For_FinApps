---
id: {KEY}-{n}
type: story
epic: {KEY}-{n}
status: todo
gates: []
depends_on: []
points: {1|2|3|5|8}
---
# {Verb} {METHOD} {api_path}/{resource} — {outcome}   (non-API: {Verb} {job|consumer|screen|report} {name} — {outcome})

## User Story
As a {role from the rulebook / api.roles}
I want {capability}
So that {business outcome}

## Context
- Module: {name} — {path} — port {port or none}
- Call path: {client} → {gateway if any} → {module}
- Response fields and sources: {field → source}
- Calls to other modules or systems: {… or None}
- Lifecycle: {entity, states and transitions added, used or rejected — or None}

## Acceptance Criteria
- [ ] [API] {METHOD} {path} returns {status} with data.{field}={value}
- [ ] [API] unknown {id} returns 404 with error.code={CODE}
- [ ] [API] invalid {input} returns 400 (422 for a business rule) with error.code={CODE}
- [ ] [DB|MSG|UI|BATCH] {exact target} — expect {value}
- [ ] {each BFSI pattern below that applies}

BFSI patterns: money returned as `api.money_json_type` with currency and correct scale · same idempotency key replayed
→ same result, nothing new · each allowed transition works and a disallowed one returns 409/422 with state unchanged ·
another role or owner gets 403 (or 404) · one audit record per change (actor, action, before/after, UTC time) ·
just before / after the cut-off and on a holiday (state the time zone) · control unavailable → rejected or held ·
touched invariants (allocations sum to the block with the residual rule, fills ≤ order qty, postings balance,
settlement = trade date + cycle on the market calendar).

## BFSI Considerations
Money: {fields, currency, scale, rounding mode and step — or N/A — reason}
Time and calendars: {business date, cut-off and time zone, calendar, roll rule — or N/A — reason}
Idempotency: {key and replay behaviour — or N/A — reason}
Lifecycle: {allowed / rejected transitions, terminal states — or N/A — reason}
Entitlements: {who may, who may not, record ownership — or N/A — reason}
Audit: {what is recorded — or N/A — reason}
Regulated data: {classification, masking — or N/A — reason}
Controls fail closed: {which check, behaviour when unavailable — or N/A — reason}
Open compliance questions: {question — owner — or None}

## Out of Scope
- {…}

## Tech Constraints
Module rulebook: {path} · API spec: {path or none yet} · Branch: {feature_branch_pattern} from {base} · Depends on: {ids or None}

## Test Data
{synthetic records or fixtures needed for every AC}
