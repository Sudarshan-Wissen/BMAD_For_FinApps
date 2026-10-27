# Stack Pack — React / TypeScript

Used by modules with `stack: react-typescript` in `bmad.config.yaml`. The project rulebook overrides any default here.

## 1. Detection

```bash
[ -f "{module_path}/package.json" ] && grep -q '"react"' "{module_path}/package.json" && echo "STACK=react-typescript"
```

Package manager: `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, otherwise npm. Below, `$PM` is that manager and
`$X` is `npx`, `pnpm exec` or `yarn` accordingly.

## 2. Toolchain check

```bash
node -v && $PM -v
```

## 3. Build, test and coverage commands

| Purpose | Command (run in `{module_path}`) |
|---|---|
| Install | `$PM ci` (npm) / `pnpm install --frozen-lockfile` / `yarn install --immutable` |
| Type check | `$X tsc --noEmit` |
| Lint | `$PM run lint` (if the script exists) |
| Unit tests + coverage | `$X vitest run --coverage --coverage.reporter=json-summary` or `$X jest --coverage --coverageReporters=json-summary` |
| Coverage file | `coverage/coverage-summary.json` → `.total.lines.pct` |
| E2E (QA) | `$X playwright test {spec}` with the module's E2E folder (`e2e_dir` on the module in config, default `{module_path}/e2e`) |

```bash
jq '.total.lines.pct' {module_path}/coverage/coverage-summary.json   # must be >= quality.coverage_min_line_pct
```

## 4. Protected configuration

`vitest.config.*`, `jest.config.*`, `playwright.config.*` (except adding a spec file), `tsconfig*.json`,
`.eslintrc*` / `eslint.config.*`, coverage thresholds in `package.json`, lock files unless a dependency was approved.

## 5. Implementation order (Dev agent)

1. Types and API client (typed request and response, money as string)
2. State (query hooks or store slice)
3. Components and pages
4. Routing and guards (UI guards are convenience; the server enforces authorisation)
5. Tests: unit tests for logic and components; E2E specs for changed user journeys. If a story changes behaviour
   that existing E2E specs assert, update those specs in the same change.

## 6. Rubric additions per category

| Category | React / TypeScript checks |
|---|---|
| 1 Money & numerics | amounts stay strings or use a decimal library (`decimal.js`, `big.js`, `dinero.js`) for arithmetic; never `parseFloat`, `Number()` or `+` on money; formatting with `Intl.NumberFormat` and the currency code |
| 2 Naming & structure | components PascalCase; hooks `useX`; files follow the rulebook layout |
| 3 Framework conventions | no `any` in production code; strict mode on; no direct DOM manipulation |
| 8 Logging | no `console.*` in production code; no PII or tokens in client logs or analytics events |
| 10 Testing | Testing Library queries by role or label; E2E covers every changed acceptance criterion with a UI part |
| 11 Security | tokens never in `localStorage` if the rulebook forbids it; no `dangerouslySetInnerHTML` with untrusted data; no secrets in `VITE_*` / `NEXT_PUBLIC_*` variables |

## 7. Fast checks for BFSI non-negotiables (Dev and Code Review checks)

```bash
F=$(git diff --name-only {base} -- '{module_path}/src/*.ts' '{module_path}/src/*.tsx' | grep -vE '\.(test|spec)\.')
[ -z "$F" ] || grep -nE '(parseFloat|Number)\([^)]*(amount|price|rate|fee|balance|notional|total)' $F     # BFSI-MON-01
[ -z "$F" ] || grep -nE ':\s*any\b|as any\b' $F                                                          # conventions
[ -z "$F" ] || grep -nE 'console\.(log|warn|error|debug)' $F                                             # logging
[ -z "$F" ] || grep -nE '(apiKey|secret|password|token)\s*[:=]\s*["'"'"'][^"'"'"']+' $F                  # BFSI-SEC-01
```

## 8. Onboarding discovery scans

```bash
M={module_path}
grep -rhn "path=\|path:\|createBrowserRouter" "$M/src" --include="*.ts" --include="*.tsx" | sort -u   # R1 routes
grep -rhoE "['\"\`]/[a-zA-Z0-9_{}/\$-]*api[a-zA-Z0-9_{}/\$-]*" "$M/src" --include="*.ts" --include="*.tsx" | sort -u  # R2 API paths
grep -rln "createSlice\|createStore\|useQuery\|useMutation\|zustand" "$M/src"                         # R3 state
find "$M/src" -name "*.tsx" -path "*pages*" -o -name "*.tsx" -path "*routes*" | sort                 # R4 pages
grep -rhoE "(VITE|REACT_APP|NEXT_PUBLIC)_[A-Z0-9_]+" "$M/src" "$M"/.env* 2>/dev/null | sort -u        # R5 env vars
grep -rn ": any\|as any" "$M/src" --include="*.ts" --include="*.tsx" | grep -vE "\.(test|spec)\." | wc -l   # R6
grep -E '"(dev|start|build|test|lint)"' "$M/package.json"                                             # R7 scripts
grep -rln "console\.\(log\|warn\|error\)" "$M/src" | grep -vE "\.(test|spec)\." | head -20            # R8
grep -rn "parseFloat\|toFixed(" "$M/src" --include="*.ts" --include="*.tsx" | head -20                # R9 money handling signal
```

Discovery report fields: framework, router, state library, test runner, dev port, routes, backend paths called,
API client files, state stores, environment variables, `any` count, console calls, money-handling signals,
tech-debt markers.

## 9. Design template hints

Route, page and component tree; API client contract per endpoint; state and cache invalidation; role-based UI
guards; error and empty states; accessible forms with validation messages; E2E journeys per acceptance criterion.
