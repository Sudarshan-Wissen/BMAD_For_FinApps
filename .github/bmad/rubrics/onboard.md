# Rubric — onboard

Applied by `bmad-critic` to the module rulebook draft `{module_path}/AGENTS.md` written by the Onboard agent.
Context: the discovery file `.bmad/review/{module}-discovery.md` and the module's source code. Section numbers refer to
`.github/bmad/templates/module-AGENTS.md`. Mark each item PASS, FAIL or N/A. Report each FAIL with `path:line`
evidence or an absence with the paths searched.

## O — Code versus rulebook (all modules)
- O1 Every endpoint in the code is in §3 with method, path, auth and roles; every path in the API spec file is in §3
- O2 Every outbound call is in §4 with its timeout, retry and failure handling
- O3 Every produced and consumed topic or queue is in §5 with its trigger or consumer group; idempotency is described
- O4 Every entity field and every migration (including ALTER changes) is reflected in §6
- O5 Cache and client storage in §7 match real usage; configured-but-unused is stated
- O6 Every TODO, FIXME, HACK and XXX from discovery is in §16
- O7 No invented facts: every claim traces to code, a migration or an explicit root-rulebook rule
- O8 Any violation of a root-rulebook "must never" or non-negotiable rule is recorded in §17

## U — UI modules only
- U1 Every route and page is in §3; every backend call is in §4 and resolves to a documented endpoint
- U2 No hard-coded API hosts; `any` and `console.*` in production code are listed in §16
- U3 Token storage is stated and matches the root rulebook; UI role guards match backend entitlements

## F — BFSI sections (all modules)
- F1 §8 lists every amount, price, rate and fee with type, scale and rounding; binary floating point is flagged (BFSI-MON-01)
- F2 §9 lists every status enum with values, enforced transitions and terminal states; reversible terminal states are flagged
- F3 §10 classifies every regulated field found; PII or card data in logs is flagged (BFSI-SEC-02, -03)
- F4 §11 states each control's behaviour when it or its dependency is unavailable; fail-open is flagged
- F5 §12 names the audit record per state change or flags its absence (BFSI-AUD-01); reconciliations listed or "None found"
- F6 §13 lists jobs, cut-offs and calendars with time zones; fixed offsets and non-restartable posting jobs are flagged
- F7 Terminology matches the domain pack or the rulebook glossary
