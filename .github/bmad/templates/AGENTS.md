# {Project name} — Project Rulebook (AGENTS.md)

Copy this file to the repository root as `AGENTS.md` (or the path in `rulebook.root`) and fill every `{…}`.
Copilot reads it automatically; every BMAD agent reads it before working. It is the single source of truth for this
project's architecture, standards and domain rules. Changes go through a reviewed PR; BMAD agents edit it only
behind the RULEBOOK Write Guard.

Keep these headings exactly as written. BMAD agents locate sections by heading text, never by number.

## Project Overview

- System: {what it does, for whom}
- BFSI sub-domains: {capital markets | retail banking | payments | lending | cards | wealth | insurance}
- Jurisdictions and regimes in scope: {e.g. SEC/FINRA, MiFID II, RBI, DORA}. Owner for regulatory questions: {Compliance contact role}
- Environments: {local, test, UAT}. AI agents never target production.

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| {Language} | {…} | {…} |

A new dependency needs this table updated first, in a reviewed PR.

## Modules

| Module | Path | Port | Owns | Must never |
|---|---|---|---|---|
| {module} | {path} | {port} | {data and behaviour it owns} | {boundaries} |

One schema per module. No module reads or writes another module's schema. Cross-module access only via {REST | messaging}.

## Code Structure and Conventions

{Package / folder layout, naming, layer suffixes, DI style, DTO rules. The stack pack gives defaults; state only what differs.}

## API Standards

- Base path: `{api.base_path}`; resources are plural kebab-case nouns.
- Success envelope: `{…}`. Error envelope: `{…}` with a machine-readable code.
- Money in JSON: {string | integer minor units}. Timestamps: ISO 8601 UTC.
- Idempotency: header `{Idempotency-Key}` required on {which} requests; replay window {n}.

## Roles and Entitlements

| Role | May | May not |
|---|---|---|
| {ROLE} | {…} | {…} |

Authorisation is enforced server-side per endpoint and per record.

## State Machine

{For each lifecycle entity: states, terminal states, allowed transitions, who triggers each (USER/SYSTEM), and the
event published. Any transition not listed is forbidden.}

## Validation Rules

{Syntactic and semantic validation, in order, with the HTTP status and error code for each failure.}

## Domain Rules

{Project-specific money precision per currency or instrument, rounding modes and steps, calendars and cut-off
times, settlement cycles, allocation method and residual rule, limits and their owners, maker-checker scope.
BFSI rules in `.github/bmad/domain/bfsi-rules.md` apply in full unless an exception below names the rule id.}

### Rule exceptions

| Rule id | Exception | Approved by | Date |
|---|---|---|---|

## Messaging

| Topic / queue | Producer | Consumers | Payload | Dead-letter |
|---|---|---|---|---|

Consumers are idempotent; retries {n} with backoff {…}; dead-letter naming `{…}`.

## Cache Keys

| Key pattern | Module | TTL | Invalidation |
|---|---|---|---|

## Data Classification

| Data | Class (public / internal / confidential / restricted) | Where stored | Masking in logs and UI |
|---|---|---|---|

## Audit and Records

{Audit record fields, storage, retention periods per record type, who can read them.}

## Security

{Authentication, token lifetime, secret storage, encryption, logging restrictions.}

## Testing

- Line coverage at least {n}% per module (mirrors `quality.coverage_min_line_pct`).
- Required test types: {unit, integration, contract, E2E}.

## Git Workflow

| Branch | Created by | Merges to | Approvals |
|---|---|---|---|

## Out of Scope

{Features and changes the agents must not ticket, design or build.}

## Document Control

| Field | Value |
|---|---|
| Owner | {role, not a personal name} |
| Last updated | {date} |
