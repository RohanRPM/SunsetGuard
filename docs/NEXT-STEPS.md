# SunsetGuard — Next Steps

**Last updated:** 26 October 2026
**Current state:** Phase 0 is complete. The build, Docker stack, CI, and ADRs
exist. There is still **no runtime Java code**.
**Target:** a working MVP demo at the end of week 8, and v0.1.0 at the end of
week 12, at roughly 10 hours per week.

> This document is the build order. The product thinking lives in the README and
> the two ADRs. If this document and an older note disagree, this document wins.

---

## How to use this plan

1. Work on **one phase at a time**. Do not start the next phase until the exit
   test passes.
2. Commit after every small task. A commit should be understandable on its own.
3. Write a three-line journal at the end of each session: what I learned, what
   is blocking me, what is next.
4. For every big choice, write a short ADR in `adr/`. Two already exist.
5. Keep a visible checklist. The phases are designed to give you something you
   can demo, not just something you can describe.

**The rule that protects the timeline:** if a task does not help the current
phase's exit test, it waits.

---

## Part A — Your first seven days

These tasks finish the foundation and remove every environment surprise before
you write application code.

### Day 1 — Make the toolchain honest

Your machine's default Java may be older than the project. Check it:

```powershell
java -version
mvn -v
```

If either says 21 or 17, set JDK 25 for the session:

```powershell
$env:JAVA_HOME = "C:\Program Files\Java\jdk-25.0.2"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
mvn -v
```

Make it permanent through Windows Environment Variables. Then run:

```powershell
docker compose up -d
docker compose ps
mvn clean install
```

**Done when:** `mvn -v` reports Java 25, all three containers are healthy, and
the build ends with `BUILD SUCCESS`.

### Day 2 — Prove the pipeline around the code

- Push the current branch.
- Open the Actions tab and confirm the build is green.
- Turn on branch protection for `main`: require the CI check and one review
  (you can review your own pull requests, but the habit matters).
- Create a GitHub Project board with columns: Backlog, This week, Doing, Done.

**Done when:** a trivial pull request shows a green check before merge.

### Day 3 — Do the one-hour competitor search

Spend one focused hour. Record what you find in
`docs/COMPETITIVE-NOTES.md`:

```text
Search terms:
- "deprecated endpoint usage tracker"
- "API sunset monitor"
- "API deprecation tool open source"
- site:github.com "deprecated endpoints" dashboard
```

For each close match, write: name, link, what it does, what it does **not** do,
and whether it changes your angle. Do not skip this because it feels like
procrastination; it is the evidence for your "why this project" answer.

**Done when:** the note has at least five entries and a one-paragraph conclusion.

### Day 4 — Settle the four setup questions

| Question | Recommended answer | Why |
|---|---|---|
| Keep the name SunsetGuard? | Yes, until launch | It is descriptive and easy to say. Check GitHub and Maven Central before publishing. |
| Dashboard: React or Thymeleaf? | Thymeleaf + htmx | No frontend build, no second deployment, more backend time. |
| Starter publishing: GitHub Packages or Maven Central? | GitHub Packages first | It works without signing setup. Move to Central only if you have time after v0.1.0. |
| Do you have a real API to monitor? | Use the demo API now; ask later | Do not use employer code or data. A personal side project is enough. |

### Day 5 — Turn Phase 1 into issues

Create these issues and put them in "This week":

1. Add the Spring Boot main class and package structure.
2. Create the Flyway `V1__init.sql` schema.
3. Add JPA entities and repositories.
4. Add API-key creation, hashing, and authentication.
5. Add project and endpoint controllers.
6. Add validation and Problem Details error handling.
7. Add pagination.
8. Add OpenAPI documentation.
9. Add Testcontainers integration tests.
10. Add the tenant-isolation test and the curl exit script.

### Day 6–7 — Start Phase 1

Begin with the schema, not the controllers. The schema is the part every later
phase depends on.

---

## Part B — Phase 1: Server core (weeks 2–3, about 20 hours)

**Goal:** a secure REST server that manages projects, API keys, and endpoints.
No RabbitMQ and no starter yet.

### Build order

1. **Main class and packages.** Create
   `io.github.rohanmeshram.sunsetguard.SunsetGuardApplication` in
   `sunsetguard-server`. Use this package layout:
   `api`, `domain`, `repository`, `service`, `config`, `security`, `error`.
2. **Flyway schema.** Add `src/main/resources/db/migration/V1__init.sql` with
   `projects`, `api_keys`, `service_instances`, `endpoints`, `consumers`,
   `usage_hourly`, `usage_daily`, `processed_reports`, `verdict_history`,
   `notification_log`, `users`, and `audit_log`. Every tenant table carries
   `project_id`. Add the unique constraints from the README.
3. **Entities and repositories.** Map only what Phase 1 needs: project, API key,
   endpoint, and user. Use `LocalDateTime` in UTC and `GenerationType.IDENTITY`
   for simple identities.
4. **API-key security.** Generate a random key, show it once, store
   `SHA-256(key)`, keep the first eight characters as a display prefix, and
   compare hashes with a constant-time comparison. Put the filter in
   `security/`.
5. **Project and key endpoints.** `POST /v1/projects`,
   `POST /v1/projects/{id}/api-keys`, and
   `DELETE /v1/projects/{id}/api-keys/{keyId}`. Protect management endpoints
   with a temporary HTTP Basic admin user in Phase 1; the dashboard form login
   replaces it in Phase 5. The API key authenticates ingestion, not management.
6. **Endpoint registry.** Create/list/get/update endpoints. The unique key is
   `(project_id, service_name, http_method, route_template)`.
7. **Errors and validation.** One `@RestControllerAdvice` that returns RFC 9457
   Problem Details. Validate request bodies. Never return a stack trace.
8. **Pagination.** Use `page` and `size`, cap `size` at 100, and return a stable
   sort order.
9. **OpenAPI.** Add springdoc and check `/swagger-ui.html`.
10. **Tests.** Testcontainers PostgreSQL for repositories and the API. One test
    must prove project A cannot read project B's data. One test must prove a
    revoked key is rejected.

### Commands you will use

```powershell
# run only the server module
mvn -pl sunsetguard-server -am spring-boot:run

# run its tests
mvn -pl sunsetguard-server test

# rebuild everything
mvn clean install
```

### Exit test

```powershell
# Management calls use the temporary Basic admin user (Phase 1).
# Ingestion will use X-SunsetGuard-Key from Phase 3.

# 1. Create a project and capture its id
curl -u admin:change-me -X POST http://localhost:8080/v1/projects `
  -H "Content-Type: application/json" `
  -d '{"name":"demo"}'

# 2. Create a key and copy the plaintext value (shown once)
curl -u admin:change-me -X POST http://localhost:8080/v1/projects/1/api-keys

# 3. Create an endpoint
curl -u admin:change-me -X POST http://localhost:8080/v1/projects/1/endpoints `
  -H "Content-Type: application/json" `
  -d '{"service":"orders-api","method":"GET","route":"/v1/orders/{id}"}'

# 4. List endpoints
curl -u admin:change-me http://localhost:8080/v1/projects/1/endpoints
```

**Done when:** the curl flow works, the tenant-isolation test passes, and
`/swagger-ui.html` lists the endpoints.

### What to learn this phase

- Spring MVC request lifecycle and the filter chain.
- `@ConfigurationProperties`, `@Transactional`, and bean scopes.
- JPA mapping, lazy loading, and the N+1 problem (create one on purpose, then
  fix it with a fetch join).
- Flyway naming and repeatable migrations.
- Testcontainers with a real PostgreSQL container.
- Constant-time comparison and why a fast hash is wrong for passwords but
  acceptable for high-entropy API keys.

### Common mistakes to avoid

- Returning entities directly from controllers. Use response records.
- Trusting a `projectId` from the request body instead of the authenticated key.
- Storing the raw API key "just for debugging".
- Adding RabbitMQ early because it feels like progress. It is Phase 3.
- Writing the dashboard now. It is Phase 5.

### Interview checkpoint

Be able to answer: How does the API key filter work? Why SHA-256 here and BCrypt
for passwords? Where is tenant isolation enforced? What does an N+1 query look
like and how did you fix it? Why is the route template stored instead of the raw
URL?

---

## Part C — Phase 2: The starter (weeks 4–5, about 20 hours)

**Goal:** add one dependency to the demo API and see real usage data arrive.

### Build order

1. **Servlet filter.** Capture method, matched route template, status, and
   consumer identity. Never read the request body.
2. **Route template.** Use the pattern Spring already matched. Test static
   resources, error pages, and 404s so they do not create junk endpoints.
3. **Consumer identity strategies.** Header, JWT claim, API-key id, then
   User-Agent. Implement them behind one interface and try them in a
   configurable order.
4. **Aggregator.** `ConcurrentHashMap` plus `LongAdder`, keyed by
   `(method, route, consumer)`. A snapshot must not block request threads.
5. **Flush loop.** A background thread every ten seconds, a bounded queue, and a
   sender with retry, exponential backoff, and jitter.
6. **Fail-open behaviour.** If the server is down, requests continue. If the
   buffer is full, drop the oldest window and increment a counter.
7. **Headers.** A `@DeprecatedEndpoint` annotation adds `Deprecation` and
   `Sunset` headers (RFC 9745 and RFC 8594).
8. **Auto-configuration.** `AutoConfiguration.imports` and
   `@ConfigurationProperties` for `sunsetguard.server-url`,
   `sunsetguard.api-key`, and `sunsetguard.service-name`.
9. **Demo API and traffic generator.** A small v1/v2 orders controller and a
   script that calls it at a steady rate.

### Exit test

- Start the demo API with the starter.
- Generate 1,000 requests.
- Confirm one report per window arrives at the server (temporarily log it).
- Stop the server. Confirm the demo API still answers and latency is unchanged.
- Measure overhead with and without the filter. Write the number in the README.

**Done when:** the demo reports traffic, the host app survives the server being
down, and the aggregator and extractors have unit tests.

### What to learn

Filters vs interceptors, thread safety, `LongAdder` vs `AtomicLong`, bounded
queues, backoff with jitter, Spring Boot auto-configuration, and virtual threads.

### Interview checkpoint

Why aggregate on the client? What happens when the buffer is full? How do you
find the route template? How do you keep the filter off the critical path? What
is the cost of the ten-second window?

---

## Part D — Phase 3: The pipeline (weeks 6–7, about 20 hours)

**Goal:** reports flow from ingest to RabbitMQ to the worker to PostgreSQL,
safely and exactly once in effect.

### Build order

1. **Ingest endpoint.** Validate size and shape, return `202 Accepted`, and use
   `413` and `429` correctly.
2. **Rate limiter.** Redis token bucket with a Lua script so check-and-decrement
   is atomic.
3. **Deduplication.** `SET NX` on the report id with a TTL, backed by the
   `processed_reports` unique key.
4. **Publish.** Send to the `usage-reports` queue only after validation. Do not
   write rollups on the request thread.
5. **RabbitMQ topology.** Durable queue, manual acknowledgements, prefetch
   limit, retry with backoff, and a dead-letter exchange/queue.
6. **Worker.** One transaction per report: upsert hourly rows, register new
   endpoints and consumers within caps, update `last_seen`, and ack only after
   commit.
7. **API-key cache.** Cache-aside in Redis with a short TTL and explicit
   invalidation on revoke.
8. **Unique callers.** HyperLogLog per endpoint per day, with a documented
   error margin.
9. **Instance registry.** Record `service_instances.last_report_at`; the verdict
   engine needs it.
10. **Cardinality caps.** Maximum new endpoints per service per day and maximum
    consumers per project; overflow goes to `_other`.

### Exit test

- Send 10,000 reports.
- Kill the worker halfway through.
- Restart it.
- Verify the totals are exactly right: nothing lost, nothing double-counted.
- Send the same report twice; verify the second is ignored.
- Publish a malformed message; verify it lands in the dead-letter queue.

**Done when:** the numbers survive a crash and duplicates change nothing.

### What to learn

Exchanges, queues, routing keys, manual ack, prefetch, `x-death`, and dead
lettering. Redis `INCR`, `EXPIRE`, Lua, cache-aside, stampede. PostgreSQL
`INSERT ... ON CONFLICT DO UPDATE`, batching, and transaction boundaries.

### Interview checkpoint

Why at-least-once instead of exactly-once? What happens if the worker dies after
the database commit but before the ack? Why is the report id the idempotency
key? Why is the queue in front of the database? What is your back-pressure
story?

---

## Part E — Phase 4: Verdicts and nudges (week 8, about 10 hours)

**Goal:** the MVP demo works end to end.

### Build order

1. **Endpoint states.** `ACTIVE`, `DEPRECATED`, `SUNSET_SCHEDULED`,
   `READY_TO_REMOVE`, `REMOVED`, with explicit allowed transitions.
2. **Verdict engine.** A pure class: usage history, coverage, and instance
   heartbeats in; verdict plus reasons out. No database calls inside it.
3. **Confidence.** `SAFE`, `PROBABLY_SAFE`, `IN_USE`, `UNKNOWN`. Low coverage
   can never produce `SAFE`.
4. **Scheduler.** Daily job with ShedLock so only one instance runs it.
5. **Notifications.** One interface, Slack/Discord webhook first, email second,
   with a cooldown from `notification_log`.
6. **Retention.** Merge old hourly rows into daily rows and delete expired data.

### Exit test

Generate traffic, mark an endpoint deprecated, stop the traffic, run the
scheduler, and watch the verdict change. Receive a real Slack message. Record a
two-minute demo video now.

---

## Part F — Phase 5: Dashboard (week 9, about 10 hours)

**Goal:** a person can complete the whole flow without curl.

### Build order

1. Form login with Spring Security; BCrypt password hashing; secure cookies.
2. Endpoints list: state, last call, verdict badge.
3. Endpoint detail: consumers, trend table, verdict reasons, deprecate form.
4. Settings: create/revoke keys, notification channels.
5. htmx on one or two interactions, such as refreshing the consumer table.

**Exit test:** a new user logs in, creates a key, deprecates an endpoint, and
reads the verdict without touching the API directly.

**Do not** add a JavaScript framework. The point is backend depth.

---

## Part G — Phase 6: CLI and GitHub Action (week 10, about 10 hours)

**Goal:** a pull request that removes an in-use endpoint gets a comment with
evidence.

1. Picocli `check` command calling `GET /v1/check`.
2. Exit code `1` for unsafe, `0` for safe, `2` for an error.
3. Composite Action: generate OpenAPI for base and head, diff removed
   operations, call the server for each, comment once.
4. Least-privilege `GITHUB_TOKEN`; no secrets in logs.
5. WireMock tests for the HTTP contract.

**Exit test:** a test PR receives an automatic, accurate comment and the check
fails.

---

## Part H — Phase 7: Hardening (week 11, about 10 hours)

**Goal:** real numbers and a documented failure story.

1. Micrometer metrics: accepted/rejected reports, queue depth, upsert latency,
   dropped reports in the starter.
2. Grafana dashboard saved as JSON in the repo.
3. Structured JSON logs with a correlation id.
4. Gatling or k6 load test; record throughput, p50/p95/p99, and hardware.
5. Chaos tests: stop Redis, then RabbitMQ, then PostgreSQL. Write down what
   happens and what the user sees.
6. OWASP Top 10 pass; secrets never logged; Trivy scan green.
7. `EXPLAIN` every slow query; add the missing index.
8. A short "what breaks first at 100x traffic" note.

**Exit test:** the README has measured numbers and a failure table, not
adjectives.

---

## Part I — Phase 8: Launch (week 12, about 10 hours)

1. README quickstart under five minutes, with a demo GIF.
2. Architecture diagram and "why not just search logs?" comparison.
3. Docs site (MkDocs Material) and 8–10 labelled good-first-issues.
4. `v0.1.0` release, GHCR image, starter artifact.
5. Blog post with the real story, including mistakes.
6. Two friends follow the quickstart; fix everything that confuses them.

**Exit test:** a stranger can run it without your help.

---

## Part J — The 12-week calendar

| Week | Phase | What you can show on Friday |
|---|---|---|
| 1 | 0 | Green CI, healthy containers, competitor notes |
| 2 | 1 | Projects, keys, schema, one passing isolation test |
| 3 | 1 | Endpoint CRUD, OpenAPI, curl exit script |
| 4 | 2 | Filter and aggregator reporting to a stub |
| 5 | 2 | Demo API reporting; latency number recorded |
| 6 | 3 | Ingest, Redis limits, RabbitMQ publish |
| 7 | 3 | Worker upserts; crash test and DLQ proven |
| 8 | 4 | Verdict changes; Slack message; demo video |
| 9 | 5 | Login and dashboard |
| 10 | 6 | PR comment and failing check |
| 11 | 7 | Load test numbers, Grafana, chaos notes |
| 12 | 8 | v0.1.0 release and blog post |

### Weekly rhythm

- 3 hours learning: official docs first, one video only if stuck.
- 7 hours building: small commits, tests with the code.
- 30 minutes review: journal, board, next week's first task.

### Git workflow

```text
main
└── phase-1/server-core
    ├── feat/project-api
    ├── feat/api-key-auth
    └── test/tenant-isolation
```

- One branch per task or small group.
- Commit messages: `feat: add project creation endpoint`,
  `fix: reject revoked keys`, `docs: record load test numbers`.
- Merge with a pull request even when working alone; CI must be green.

---

## Part K — When you fall behind

Cut in this order:

1. Email notifications — keep the webhook.
2. Dashboard — use Swagger UI and the API.
3. GitHub Action — keep the CLI.
4. Daily rollups — keep hourly.
5. htmx — plain form submits.

**Never cut:** the starter, the ingest pipeline, idempotency, the verdict
engine, the tests, and the README.

If you are more than a week behind, reduce scope, not sleep. The project is
supposed to be finished, not maximal.

---

## Part L — Definition of done (every phase)

A phase is done only when all of these are true:

- [ ] The exit test passes from a clean checkout.
- [ ] New behaviour has tests, including one failure case.
- [ ] `mvn clean install` passes on JDK 25.
- [ ] CI is green on `main`.
- [ ] The README or an ADR explains the choice.
- [ ] You can answer three interview questions about it without notes.
- [ ] The journal entry is written.

---

## Part M — Cost and risk guardrails

- **Spend:** zero. Local Docker, free GitHub, free public CI.
- **Hosting:** not a phase. A demo video is enough; free tiers change and can
  expire.
- **Memory:** keep Docker at 4 GB, 2 CPUs. Leave Prometheus and Grafana off
  until Phase 7.
- **Scope:** anything not on this page is a stretch goal. Write it in the
  backlog and close the tab.
- **Employer:** use only your own code, data, and time. Never use client names
  or internal APIs.

---

## Part N — First task, right now

Do this before anything else:

```powershell
$env:JAVA_HOME = "C:\Program Files\Java\jdk-25.0.2"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
java -version
mvn -v
mvn clean install
docker compose up -d
docker compose ps
```

Then open Phase 1, issue 1, and create the main class.

The project does not become real when the dashboard looks good. It becomes real
when one report from one service lands in PostgreSQL exactly once. Protect that
milestone.
