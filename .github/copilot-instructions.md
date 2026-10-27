# Copilot instructions — BMAD agentic SDLC (BFSI)

- Project profile: `.github/bmad/bmad.config.yaml`. Rulebook: `AGENTS.md` (root) and each module's `AGENTS.md`; read
  them before writing or reviewing code. BFSI rules: `.github/bmad/domain/bfsi-rules.md`.
- Stage agents: `bmad-pm → bmad-ba → bmad-arch → bmad-dev → bmad-code-review → bmad-qa → bmad-doc`, plus
  `bmad-onboard` for existing code and `bmad-bfsi-advisor` for domain questions. Slash commands: `/pm /ba /arch /dev
  /review /qa /doc /onboard`; exports `/to-jira-csv /to-confluence`. They follow `.github/bmad/protocol.md`.

Non-negotiable in every change: exact decimals with currency and explicit rounding for money (never float/double);
UTC instants and business dates with named time zones and holiday calendars; idempotency keys; forward-only
lifecycles with immutable terminal states; append-only audit records; fail-closed controls; allocations sum to the
block and postings balance; server-side entitlements; parameterised queries; no secrets, PII or card data in code,
tests, logs or commits; synthetic test data only.

Never commit to or push protected branches. Ask before any push, PR, or external write. Do not decide legal, tax or
regulatory questions; flag them for a qualified human.
