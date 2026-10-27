# Stack Pack — Java / Spring Boot

Used by modules with `stack: java-spring` in `bmad.config.yaml`. The project rulebook overrides any default here.

## 1. Detection

A module is this stack if `{module_path}/pom.xml` or `{module_path}/build.gradle(.kts)` exists and the build
declares `spring-boot`.

```bash
[ -f "{module_path}/pom.xml" ] && grep -q "spring-boot" "{module_path}/pom.xml" && echo "STACK=java-spring BUILD=maven"
ls "{module_path}"/build.gradle* >/dev/null 2>&1 && grep -q "spring-boot" "{module_path}"/build.gradle* && echo "STACK=java-spring BUILD=gradle"
```

## 2. Toolchain check

```bash
java -version 2>&1 | head -1          # must match the version in the rulebook or the build file
mvn -v 2>/dev/null | head -1 || ./gradlew --version 2>/dev/null | head -3
```

## 3. Build, test and coverage commands

| Purpose | Maven | Gradle |
|---|---|---|
| Clean verify one module | `mvn -B -q clean verify -pl {module_path} -am` | `./gradlew :{module}:clean :{module}:check :{module}:jacocoTestReport` |
| Unit tests only | `mvn -B -q test -pl {module_path} -am` | `./gradlew :{module}:test` |
| Test reports | `{module_path}/target/surefire-reports/*.xml` | `{module_path}/build/test-results/test/*.xml` |
| Coverage CSV | `{module_path}/target/site/jacoco/jacoco.csv` | `{module_path}/build/reports/jacoco/test/jacocoTestReport.csv` |

Changed modules since `{base}`:

```bash
CHANGED=$( { git diff --name-only {base} --; git ls-files -o --exclude-standard; } | sort -u )
# keep the modules whose path prefixes a changed file:  for each modules[].path, grep -q "^{path}/" <<< "$CHANGED"
```

Test failures (0 required):

```bash
grep -ho 'failures="[0-9]*"\|errors="[0-9]*"' {module_path}/target/surefire-reports/*.xml | grep -v '="0"' | wc -l
```

Line coverage from the JaCoCo CSV (columns 8 LINE_MISSED, 9 LINE_COVERED):

```bash
awk -F, 'NR>1{m+=$8;c+=$9} END{if(m+c==0){print "0.0"}else{printf "%.1f\n", 100*c/(m+c)}}' {module_path}/target/site/jacoco/jacoco.csv
```

The result must be at least `quality.coverage_min_line_pct`.

## 4. Protected configuration (agents may not change it)

`pom.xml` and `build.gradle*` sections for test, coverage (JaCoCo rules and excludes), Surefire/Failsafe, Checkstyle,
SpotBugs, PMD or ArchUnit; `src/test/resources/**` logging config; `.mvn/**`; `gradle/wrapper/**`. A new dependency
needs a Hard Stop and a rulebook update first.

## 5. Implementation layer order (Dev agent)

Default package layout `{base_package}.{controller|service|repository|domain|dto.request|dto.response|config|messaging.producer|messaging.consumer|exception|mapper}`.
The rulebook may define another layout; follow it.

1. Domain: entities, enums, value objects (Money, quantities)
2. Migration: Flyway `V{next}__{description}.sql` or Liquibase changeset, own schema only
3. DTOs: request and response classes; never expose entities
4. Mapper (MapStruct or manual)
5. Repository: query methods
6. Service: business logic, transactions, state transitions, outbox
7. Controller: thin; validation annotations; delegates to the service; standard envelope
8. Exceptions and the global handler (`@RestControllerAdvice`)
9. Config: HTTP clients with timeouts, Resilience4j, messaging, cache
10. Tests: unit tests for services and rules, `@WebMvcTest` for controllers, consumer and producer tests

## 6. Rubric additions per category (dev and code-review)

| Category | Java / Spring checks |
|---|---|
| 1 Money & numerics | `BigDecimal` for money, built with `new BigDecimal(String)` or `BigDecimal.valueOf(long)`; `compareTo` for equality; `setScale(n, RoundingMode.X)` explicit; `@Column(precision=…, scale=…)` on amounts; quantities as `Long`/`Integer` or `BigDecimal` for fractional units |
| 2 Naming & structure | classes PascalCase with layer suffix (`…Controller`, `…Service`, `…Repository`, `…Request`, `…Response`, `…Mapper`, `…Exception`, `…Config`, `…Producer`, `…Consumer`); constants UPPER_SNAKE_CASE; snake_case tables and columns |
| 3 Framework conventions | Lombok per rulebook (`@RequiredArgsConstructor`, `@Slf4j`; immutable responses); `@Enumerated(EnumType.STRING)`, never ORDINAL; `@Transactional` on service methods that write |
| 4 Dependency injection | constructor injection only; no field `@Autowired` |
| 5 Architecture boundaries | no business logic in controllers; services return DTOs, never entities; no access to another module's schema; no Spring `ApplicationEvent` for cross-module integration if the rulebook mandates messaging |
| 7 Persistence | own schema only; no `DROP` / `TRUNCATE` in migrations; migrations are additive and re-runnable where possible; optimistic locking `@Version` on balances, positions, orders |
| 8 Logging | SLF4J; log ids and states; no PII, card data, tokens or amounts tied to a person at INFO or above |
| 9 Errors | custom exceptions extend `RuntimeException` and carry an error code; global handler maps to the error envelope; no stack traces in responses |
| 10 Testing | JUnit 5 + Mockito; `@ExtendWith(MockitoExtension.class)` or `@WebMvcTest`; descriptive `should…When…` names; tests for 2xx, 400, 404, 409/422 paths |
| 11 Security | `@PreAuthorize` or equivalent per endpoint; ownership checks in the service; Spring Data or `JdbcTemplate` with parameters, never string-built SQL |

## 7. Fast checks for BFSI non-negotiables (Dev and Code Review checks)

Run on the changed files only (`git diff --name-only {base} -- '*.java'`). A hit is a FAIL unless a rulebook
exception names the rule.

```bash
F=$(git diff --name-only {base} -- '*.java' | grep -v '/test/')
[ -z "$F" ] || grep -nE '\b(double|float|Double|Float)\b[^;]*(amount|price|rate|fee|balance|notional|value|cost|pnl|commission)' $F   # BFSI-MON-01
[ -z "$F" ] || grep -nE 'new BigDecimal\([0-9.]+[dDfF]?\)|BigDecimal\.valueOf\([0-9]*\.[0-9]+' $F                              # BFSI-MON-02
[ -z "$F" ] || grep -nE '\.setScale\([^,)]*\)' $F                                                                            # BFSI-MON-05 (no rounding mode)
[ -z "$F" ] || grep -nE 'BigDecimal[^;]*\.equals\(' $F                                                                        # BFSI-MON-06
[ -z "$F" ] || grep -nE 'EnumType\.ORDINAL|@Autowired\s+private' $F                                                          # conventions
[ -z "$F" ] || grep -nE '(password|secret|apiKey|token)\s*=\s*"[^"]+"' $F                                                     # BFSI-SEC-01
[ -z "$F" ] || grep -nE 'log\.(info|warn|error)\([^;]*(ssn|pan|cardNumber|accountNumber|dob|dateOfBirth|password|token)' $F  # BFSI-SEC-02
[ -z "$F" ] || grep -nE '"(SELECT|INSERT|UPDATE|DELETE)[^"]*"\s*\+' $F                                                       # BFSI-SEC-05
[ -z "$F" ] || grep -nE 'printStackTrace\(\)|java\.util\.Date|Calendar\.getInstance' $F                                      # errors / time
```

## 8. Onboarding discovery scans

```bash
M={module_path}
# J1 Endpoints
grep -rhn "@GetMapping\|@PostMapping\|@PutMapping\|@PatchMapping\|@DeleteMapping\|@RequestMapping" "$M/src/main" --include="*.java"
# J2 Message consumers (Kafka, RabbitMQ, JMS)
grep -rn "@KafkaListener\|@RabbitListener\|@JmsListener\|@SqsListener" "$M/src/main" --include="*.java" -A3
# J3 Message producers
grep -rhn "kafkaTemplate\.send\|rabbitTemplate\.convertAndSend\|jmsTemplate\.send\|StreamBridge" "$M/src/main" --include="*.java"
# J4 Entities
grep -rl "@Entity" "$M/src/main" --include="*.java"
# J5 Migrations in order
find "$M/src/main/resources" \( -name "V*.sql" -o -name "*changelog*" \) | sort
# J6 Cache usage
grep -rln "@Cacheable\|@CachePut\|@CacheEvict\|RedisTemplate\|StringRedisTemplate" "$M/src/main" --include="*.java"
# J7 Outbound HTTP calls
grep -rln "RestTemplate\|RestClient\|WebClient\|@FeignClient" "$M/src/main" --include="*.java"
# J8 Port and profiles
grep -rn "server.port\|SERVER_PORT\|spring.profiles" "$M/src/main/resources/" | head -5
# J9 Schedulers and batch
grep -rn "@Scheduled\|@EnableBatchProcessing\|JobBuilder" "$M/src/main" --include="*.java"
# J10 Money and time types in use (domain signal)
grep -rhoE "\b(BigDecimal|double|float|Instant|LocalDate|ZonedDateTime|java\.util\.Date)\b" "$M/src/main" --include="*.java" | sort | uniq -c
# J11 State enums (lifecycle signal)
grep -rln "enum .*\(Status\|State\)" "$M/src/main" --include="*.java"
```

Discovery report fields: stack and versions (from the build file), port, endpoints, API spec gaps, messages consumed
(topic or queue → group), messages produced (topic → trigger), entities, schema and migrations, cache keys, outbound
calls (module → endpoint → purpose), schedulers, money and time type usage, lifecycle enums and their values,
tech-debt markers.

## 9. Design template hints (Architect agent)

Entities with JPA annotations and Java types; migration file name; service method signatures returning DTOs; HTTP
client with timeouts and Resilience4j (circuit breaker, retry with backoff, bulkhead if needed); messaging with
consumer group, retry and dead-letter topic; cache key pattern, TTL and invalidation; tests per layer
(`…ServiceTest`, `…ControllerTest`, consumer and producer tests).
