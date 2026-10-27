# Stack Pack — {Stack name}  (template)

Copy this file to `.github/bmad/stacks/{stack-id}.md` for a new stack (for example `dotnet`, `python-fastapi`,
`go`, `kotlin-spring`, `node-nestjs`). Set `stack: {stack-id}` on the module in `bmad.config.yaml`.
Every section is required; BMAD agents rely on each heading by name.

## 1. Detection
Commands that print `STACK={stack-id}` when the module uses this stack.

## 2. Toolchain check
Commands that print the versions and fail when the toolchain is missing.

## 3. Build, test and coverage commands
Clean verify for one module, unit tests, where test reports land, how to count failures, and a command that prints
the line-coverage percentage as a number.

## 4. Protected configuration
Files and sections an agent may never change (test, coverage, lint, dependency manifests).

## 5. Implementation layer order
The order in which the Dev agent adds or changes code, from domain to tests.

## 6. Rubric additions per category
One row per dev / code-review category (1 Money & numerics, 2 Naming & structure, 3 Framework conventions,
4 Dependency management, 5 Architecture boundaries, 6 API contract, 7 Persistence, 8 Logging, 9 Errors,
10 Testing, 11 Security, 12 BFSI lifecycle integrity) with the language-specific checks.

## 7. Fast checks for BFSI non-negotiables
`grep` commands over the changed files that detect violations of BFSI-MON-01/02/05, BFSI-SEC-01/02/05 and the
stack's own conventions. A hit is a check FAIL.

## 8. Onboarding discovery scans
Commands that list endpoints, message consumers and producers, persistence entities, migrations, cache use,
outbound calls, port, schedulers, money and time types and lifecycle enums, plus the discovery report fields.

## 9. Design template hints
What the Architect agent must specify for this stack.
