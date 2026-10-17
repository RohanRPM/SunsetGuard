# SunsetGuard

> Know exactly who is still calling your deprecated API endpoints, so you can delete them safely instead of guessing.

[![Build Status](https://github.com/rohanmeshram/SunsetGuard/workflows/Build%20and%20Test/badge.svg)](https://github.com/rohanmeshram/SunsetGuard/actions)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4-brightgreen.svg)](https://spring.io/projects/spring-boot)

**Status**: 🚧 In Development - Phase 0 (Foundation)

---

## The Problem

Your team builds a new API endpoint and marks the old one as "deprecated". Six months later:

**Manager**: "Can we delete `/v1/users/{id}` yet?"  
**You**: "¯\\_(ツ)_/¯ Let me search the logs..."

Three hours of grepping later, you're still not sure if:
- That old mobile app version (5% of users) still calls it
- A partner company's integration depends on it
- Someone's nightly cron job will break

**Today's options:**
- ❌ Manually search raw access logs
- ❌ Build one-off dashboards
- ❌ Send emails and hope everyone responds
- ❌ Delete it and pray nothing breaks 🙏

## The Solution

SunsetGuard answers three questions automatically:

1. **Who still calls this endpoint?** (with app version, last seen date)
2. **How much, and is it declining?** (hourly/daily trends)
3. **Is it safe to delete now?** (plain-English verdict with confidence level)

Plus: **block pull requests** that delete endpoints still in use.

---

## Features

### MVP (Current Focus)
- ✅ Drop-in Spring Boot starter (one dependency, zero config)
- ✅ Consumer-aware tracking (knows WHO calls, not just HOW MANY)
- ✅ Automatic deprecation workflow (mark → track → notify → verdict)
- ✅ Safe-to-remove verdict with confidence levels
- ✅ Standard `Deprecation` and `Sunset` HTTP headers (RFC 8594, 9745)
- ✅ Notifications (Slack, Discord, email)
- ✅ Dashboard (Thymeleaf + htmx, no frontend build)
- ✅ CLI + GitHub Action (block unsafe deletions in PRs)

### Coming Soon
- 🔄 Prometheus metrics + Grafana dashboards
- 🔄 Consumer-to-team mapping
- 🔄 Weekly digest emails
- 🔄 OpenTelemetry support

---

## Quick Start

**Prerequisites**: Docker, JDK 21, Maven

```bash
# 1. Clone and start infrastructure
git clone https://github.com/rohanmeshram/SunsetGuard.git
cd SunsetGuard
docker compose up -d

# Wait ~30 seconds for services to be healthy
docker compose ps

# 2. Build the project
mvn clean install

# 3. Start the server
cd sunsetguard-server
mvn spring-boot:run

# 4. Open dashboard
# http://localhost:8080
```

**Services running:**
- PostgreSQL: `localhost:5432`
- Redis: `localhost:6379`
- RabbitMQ: `localhost:5672` (management UI at `localhost:15672`)
- SunsetGuard Server: `localhost:8080`

---

## How It Works

```
Your API (Spring Boot)
  |
  |-- SunsetGuard Starter (servlet filter)
  |     • Counts requests in memory (~10 sec windows)
  |     • Sends usage reports
  |
  v
SunsetGuard Server (Ingest API)
  • Validates API key
  • Rate limits (Redis token bucket)
  • Deduplicates (Redis SET NX)
  • Publishes to RabbitMQ queue
  |
  v
RabbitMQ Queue
  |
  v
Worker (Spring AMQP)
  • Upserts hourly rollups to PostgreSQL
  • Updates consumer registry
  • Manual acknowledgements (idempotent)
  |
  v
PostgreSQL (source of truth)
  |
  v
Verdict Engine (daily scheduled job)
  • Analyzes usage trends
  • Generates safe-to-remove verdicts
  • Sends notifications
  |
  v
Dashboard / CLI / GitHub Action
```

### Key Design Principles

1. **Fail-open**: If SunsetGuard is down, your API keeps working
2. **Count, don't log**: Aggregate in-memory, send totals (reduces traffic 100-10,000x)
3. **Idempotent processing**: Reports can be delivered multiple times safely
4. **Consumer-aware**: Tracks WHO calls, not just anonymous traffic
5. **Multi-tenant**: One server, many projects

See [ADR-001](adr/ADR-001-rabbitmq-over-kafka.md) and [ADR-002](adr/ADR-002-client-side-aggregation.md) for architectural decisions.

---

## Usage Example

### 1. Add the starter to your Spring Boot API

```xml
<dependency>
    <groupId>io.github.rohanmeshram</groupId>
    <artifactId>sunsetguard-spring-boot-starter</artifactId>
    <version>0.1.0</version>
</dependency>
```

### 2. Configure (application.yml)

```yaml
sunsetguard:
  server-url: http://localhost:8080
  api-key: ${SUNSETGUARD_API_KEY}
  service-name: orders-api
  consumer-identity:
    - header: X-Client-Id
    - jwt-claim: client_id
    - api-key-id: true
    - user-agent: true
```

### 3. Mark an endpoint as deprecated

```java
@RestController
@RequestMapping("/v1/orders")
public class OrdersController {
    
    @GetMapping("/{id}")
    @DeprecatedEndpoint(
        sunsetDate = "2027-03-01",
        replacement = "/v2/orders/{id}"
    )
    public Order getOrder(@PathVariable Long id) {
        // ...
    }
}
```

**Every response now includes standard headers:**

```http
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Mar 2027 00:00:00 GMT
Link: </v2/orders/123>; rel="successor-version"
```

### 4. Check the dashboard

Navigate to `http://localhost:8080/endpoints`

| Endpoint | State | Last Call | Verdict |
|----------|-------|-----------|---------|
| `GET /v1/orders/{id}` | 🟡 DEPRECATED | 2 min ago | 🔴 IN_USE |

Click for details:

**Consumers:**
| App | Version | Calls (7d) | Last Seen | Trend |
|-----|---------|------------|-----------|-------|
| android-app | 2.3 | 412 | 5 min ago | ↓ Declining |

**Verdict**: ❌ **Not safe to remove**
- Last call: 5 minutes ago
- Active consumers: 1 (android-app v2.3)
- Collecting data for: 90 days
- All 4 instances reported in last 24h

### 5. Block unsafe PR deletions

```yaml
# .github/workflows/api-safety-check.yml
- name: Check for removed endpoints
  uses: rohanmeshram/sunsetguard-action@v1
  with:
    sunsetguard-url: ${{ secrets.SUNSETGUARD_URL }}
    api-key: ${{ secrets.SUNSETGUARD_API_KEY }}
```

When someone opens a PR that removes `/v1/orders/{id}`:

> 🤖 **SunsetGuard**: This PR removes `GET /v1/orders/{id}`, but it's still in use:
> - **412 calls** in the last 7 days
> - **1 active consumer**: android-app v2.3
> - **Verdict**: ❌ Not safe to remove
> 
> [View details](http://localhost:8080/endpoints/123)

---

## Why Not Just Search Logs?

| Approach | Who calls? | Trend over time? | Confidence level? | Proactive alerts? | Works across services? |
|----------|------------|------------------|-------------------|-------------------|------------------------|
| **grep logs** | ❌ Maybe | ❌ Manual | ❌ No | ❌ No | ❌ No |
| **API Gateway** | ✅ Yes | ✅ Yes | ⚠️ Partial | ⚠️ Basic | ✅ Yes |
| **SunsetGuard** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |

**Advantages over API Gateway deprecation:**
- Works without forcing you to adopt a specific gateway
- Consumer-aware (knows app version, not just IP)
- Lifecycle management built-in (deprecate → sunset → remove workflow)
- PR checks that block unsafe deletions

---

## Architecture

### Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Language** | Java 21 | LTS, modern features (records, sealed types, virtual threads) |
| **Framework** | Spring Boot 3.4 | Industry standard, dependency injection, auto-configuration |
| **Database** | PostgreSQL 16 | ACID transactions, upserts, partitioning, proven at scale |
| **Cache** | Redis 7 | Token bucket, deduplication, HyperLogLog, distributed locks |
| **Queue** | RabbitMQ 3.13 | Lightweight (150MB), teaches core messaging patterns |
| **Dashboard** | Thymeleaf + htmx | Server-rendered, no build step, 100% backend focus |
| **Testing** | Testcontainers | Real databases in tests, not mocks |
| **Build** | Maven | Multi-module, standard in enterprise Java |

**Why RabbitMQ and not Kafka?** See [ADR-001](adr/ADR-001-rabbitmq-over-kafka.md). TL;DR: 8GB laptop, RabbitMQ uses 150MB vs Kafka's 600MB+. Kafka concepts are studied separately for interview prep.

### Database Schema

**Core tables:**
- `projects` - Multi-tenant isolation
- `api_keys` - Hashed, prefix for display
- `endpoints` - Route template, state machine, deprecation metadata
- `consumers` - Who calls what (app, version, owner contact)
- `usage_hourly` / `usage_daily` - Rollup counters (target of upserts)
- `verdict_history` - Audit trail of verdict changes

**Cardinality controls:**
- Cap unique endpoints per service per day (prevent buggy clients exploding table)
- Cap consumers per project (bucket overflow into `_other`)

---

## Project Structure

```
SunsetGuard/
├── sunsetguard-server/          # Main application + Thymeleaf dashboard
│   ├── src/main/java/
│   │   ├── api/                 # REST controllers
│   │   ├── domain/              # Entities, repositories
│   │   ├── service/             # Business logic, verdict engine
│   │   ├── worker/              # RabbitMQ consumers
│   │   └── config/              # Security, Redis, RabbitMQ setup
│   └── src/main/resources/
│       ├── db/migration/        # Flyway SQL scripts
│       └── templates/           # Thymeleaf HTML
├── sunsetguard-spring-boot-starter/  # Client library
│   ├── filter/                  # Servlet filter
│   ├── aggregator/              # In-memory counters
│   └── sender/                  # HTTP client with retry
├── sunsetguard-cli/             # Command-line tool
├── sunsetguard-demo-orders/     # Demo API to monitor
├── adr/                         # Architecture Decision Records
├── docs/                        # MkDocs documentation
├── docker-compose.yml           # Local infrastructure
└── pom.xml                      # Multi-module Maven
```

---

## Development Roadmap

| Phase | Weeks | Goal | Status |
|-------|-------|------|--------|
| **Phase 0** | 1 | Foundation (repo, Docker, CI) | 🚧 In Progress |
| **Phase 1** | 2-3 | Server core (projects, keys, endpoints) | ⏳ Not Started |
| **Phase 2** | 4-5 | Starter library (filter, aggregator) | ⏳ Not Started |
| **Phase 3** | 6-7 | Pipeline (ingest → RabbitMQ → PostgreSQL) | ⏳ Not Started |
| **Phase 4** | 8 | Verdict engine + notifications | ⏳ Not Started |
| **Phase 5** | 9 | Dashboard (Thymeleaf + htmx) | ⏳ Not Started |
| **Phase 6** | 10 | CLI + GitHub Action | ⏳ Not Started |
| **Phase 7** | 11 | Hardening (metrics, load tests, security) | ⏳ Not Started |
| **Phase 8** | 12 | Launch (docs, blog, v0.1.0 release) | ⏳ Not Started |
| **Phase 9** | 13 | Cloud deployment (Railway.app) | ⏳ Not Started |

**Target**: v0.1.0 by January 2027

---

## Contributing

This is currently a solo learning project, but contributions will be welcome after v0.1.0!

**If you want to help:**
1. ⭐ Star the repo
2. 🐛 Open issues for bugs or ideas
3. 📖 Improve documentation
4. 🔍 Review PRs

See [CONTRIBUTING.md](CONTRIBUTING.md) (coming in Phase 8).

---

## License

Apache License 2.0 - see [LICENSE](LICENSE)

This license:
- ✅ Allows commercial use
- ✅ Allows modification and distribution
- ✅ Includes patent grant (protects contributors)
- ✅ Company-friendly

---

## Credits

**Author**: [Rohan Meshram](https://github.com/rohanmeshram)

**Learning Journey**: Building this project to:
- Practice backend system design at scale
- Learn Spring Boot internals deeply
- Understand messaging systems (RabbitMQ + Kafka theory)
- Master PostgreSQL optimization
- Experience full DevOps workflow (Docker, CI/CD, cloud deployment)
- Create a portfolio project that solves a real problem

**Inspiration**: Every team that's ever been scared to delete old code.

---

## Resources

- [Project Blueprint](START_HERE.md) - Complete 13-week plan
- [Architecture Decision Records](adr/) - Why we chose what
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [PostgreSQL Performance](https://www.postgresql.org/docs/current/performance-tips.html)

---

**Questions?** Open an issue or reach out on [Twitter/X](https://twitter.com/your-handle)

**Status**: 🚧 Phase 0 - Setting up foundation (Oct 2026)
