# Phase 0 Checklist: Foundation

**Week**: 1 of 13  
**Time Budget**: ~10 hours  
**Status**: 🚧 In Progress

---

## Goals
✅ Repository with everything building and running with `docker compose up`  
✅ Clean project structure  
✅ CI/CD pipeline  
✅ Documentation foundation

---

## Tasks

### Repository Setup
- [x] Create GitHub repository `SunsetGuard`
- [x] Apache-2.0 License
- [x] .gitignore (Java, Maven, Docker, IDE)
- [x] Initial commit
- [ ] Push to GitHub (remote not configured yet)
- [ ] Set repository description and topics

### Maven Multi-Module Structure
- [x] Root `pom.xml` with parent configuration
- [x] `sunsetguard-server` module (main application)
- [x] `sunsetguard-spring-boot-starter` module (client library)
- [x] `sunsetguard-cli` module (command-line tool)
- [x] `sunsetguard-demo-orders` module (demo API)
- [ ] Verify build works: `mvn clean compile`

### Docker Compose Infrastructure
- [x] `docker-compose.yml` created
- [x] PostgreSQL 16 service (port 5432)
- [x] Redis 7 service (port 6379)
- [x] RabbitMQ 3.13 with management UI (ports 5672, 15672)
- [x] Prometheus commented out (enable in Phase 7)
- [x] Grafana commented out (enable in Phase 7)
- [ ] Start services: `docker compose up -d`
- [ ] Verify all services healthy: `docker compose ps`
- [ ] Test RabbitMQ UI: http://localhost:15672

### GitHub Actions CI
- [x] `.github/workflows/build.yml` created
- [x] Build job (Java 21, Maven)
- [x] Test job
- [x] Coverage report (JaCoCo)
- [x] Dependency scanning (Trivy)
- [ ] Push to trigger first CI run
- [ ] Fix any CI failures

### Architecture Decision Records
- [x] `adr/ADR-001-rabbitmq-over-kafka.md`
- [x] `adr/ADR-002-client-side-aggregation.md`
- [ ] Review and understand both decisions (interview prep)

### Documentation
- [x] `README.md` - Project overview, quickstart, architecture
- [x] `docs/SETUP.md` - Complete environment setup guide
- [x] `CONTRIBUTING.md` - Contribution guidelines
- [x] `CHANGELOG.md` - Version history
- [ ] Review all documentation for clarity

---

## Environment Setup (Your Machine)

### Prerequisites to Install
- [ ] **Java 21 (JDK)** - [Download Eclipse Temurin](https://adoptium.net/temurin/releases/?version=21)
  - Verify: `java --version`
  - Should show: openjdk version "21.x.x"
  
- [ ] **Maven 3.9+** - [Download from Apache](https://maven.apache.org/download.cgi)
  - Install to: `C:\Program Files\Apache\maven`
  - Add to PATH
  - Verify: `mvn --version`
  
- [ ] **Docker Desktop** - [Download for Windows](https://www.docker.com/products/docker-desktop/)
  - Configure: 4GB RAM, 2 CPUs
  - Verify: `docker --version` and `docker compose version`
  
- [ ] **IDE** - IntelliJ IDEA Community or VS Code with Java extensions

### First-Time Verification
```bash
# 1. Check all tools installed
java --version
mvn --version
docker --version
git --version

# 2. Build the project
mvn clean install -DskipTests

# 3. Start infrastructure
docker compose up -d

# 4. Check services are healthy
docker compose ps

# 5. Open RabbitMQ Management UI
# http://localhost:15672
# Username: sunsetguard
# Password: sunsetguard_dev_password
```

---

## Learning Objectives for Week 1

### Maven
- [ ] Understand multi-module project structure
- [ ] Know what a parent POM does
- [ ] Understand dependency management vs dependencies
- [ ] Learn Maven lifecycle: `clean`, `compile`, `test`, `package`, `install`

### Docker Compose
- [ ] Understand service definitions
- [ ] Know how networks work (all services on `sunsetguard-network`)
- [ ] Understand volumes (data persistence)
- [ ] Learn health checks (why they matter)
- [ ] Practice: `docker compose up/down/logs/ps`

### GitHub Actions
- [ ] Understand workflow triggers (push, pull_request)
- [ ] Know what a job and step are
- [ ] Learn about action marketplace (setup-java, checkout)
- [ ] Understand caching (speeds up builds)

### RabbitMQ Basics
- [ ] Open management UI at http://localhost:15672
- [ ] Explore: Queues, Exchanges, Connections
- [ ] Understand that you'll create queues in Phase 3
- [ ] Read: [RabbitMQ in 5 minutes](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)

---

## Definition of Done

Phase 0 is complete when:

1. ✅ All files committed to Git
2. ⏳ Repository pushed to GitHub
3. ⏳ `docker compose up -d` starts all services
4. ⏳ All containers show "healthy" status
5. ⏳ RabbitMQ management UI accessible
6. ⏳ `mvn clean compile` succeeds (even with empty modules)
7. ⏳ GitHub Actions CI runs and passes (green checkmark)
8. ⏳ You can explain each container's purpose in one sentence

**Test yourself:**
- "What does PostgreSQL do in this project?" → _Stores endpoint usage data, consumer registry, verdicts_
- "What does Redis do?" → _Caching API keys, rate limiting, deduplication, distributed locks_
- "What does RabbitMQ do?" → _Message queue between ingest API and database workers_
- "Why RabbitMQ and not Kafka?" → _Lower memory footprint (150MB vs 600MB), teaches same concepts_

---

## Time Tracking

| Task | Estimated | Actual | Notes |
|------|-----------|--------|-------|
| Maven setup | 2h | | Multi-module structure, dependencies |
| Docker Compose | 2h | | Service definitions, testing |
| GitHub Actions | 1h | | Workflow file, trigger test |
| ADRs | 2h | | Write and review decisions |
| Documentation | 2h | | README, SETUP.md, CONTRIBUTING.md |
| Environment setup | 1h | | Install tools, verify |
| **Total** | **10h** | | |

---

## Blockers / Issues

_Document any problems here:_

- [ ] Maven not found in PATH → See [docs/SETUP.md](docs/SETUP.md)
- [ ] Docker Desktop won't start → Check WSL 2, restart computer
- [ ] Port conflicts → Change ports in docker-compose.yml
- [ ] _Add yours here_

---

## Next Steps (Phase 1 Preview)

After Phase 0 is complete, you'll start **Phase 1: Server Core** (Weeks 2-3):
- Flyway database migrations
- JPA entities and repositories
- REST controllers for projects and API keys
- Spring Security with API key authentication
- Testcontainers tests

**Before moving to Phase 1:**
- [ ] Complete all Phase 0 tasks
- [ ] Record learnings in a note (what confused you, what clicked)
- [ ] Take a screenshot of RabbitMQ UI for your portfolio
- [ ] Read Spring Boot docs: [Getting Started](https://spring.io/guides/gs/spring-boot/)

---

## Resources for This Phase

- [Maven Multi-Module Guide](https://maven.apache.org/guides/mini/guide-multiple-modules.html)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [GitHub Actions Quickstart](https://docs.github.com/en/actions/quickstart)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)

---

**Started**: 2026-10-17  
**Target Completion**: 2026-10-24 (1 week)  
**Status**: 🚧 Foundation in progress
