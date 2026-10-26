# SunsetGuard Quickstart (Phase 0)

**Status:** the foundation is built. Infrastructure, the Maven build, CI, and the
design documents work today. There is **no runtime Java code yet** — the server
starts in Phase 1. That is expected.

For the exact build order, read [docs/NEXT-STEPS.md](docs/NEXT-STEPS.md).

---

## What works today

| Thing | State |
|---|---|
| Maven multi-module build (Java 25, Spring Boot 4.1.2) | ✅ Works |
| Docker Compose: PostgreSQL 16, Redis 7, RabbitMQ 3.13 | ✅ Works |
| GitHub Actions CI (build, tests, Trivy) | ✅ Works |
| ADR-001 (RabbitMQ) and ADR-002 (client aggregation) | ✅ Written |
| Server, starter, CLI, demo API | POM only (no code yet) |

---

## 1. Check your tools

You need **JDK 25**, **Maven 3.9+**, and **Docker Desktop**.

```powershell
java -version      # must say 25.x
mvn -v             # "Java version" must say 25.x
docker --version
```

If `java -version` says 21 or 17, point `JAVA_HOME` at your JDK 25 first:

```powershell
$env:JAVA_HOME = "C:\Program Files\Java\jdk-25.0.2"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
mvn -v
```

To make it permanent: Windows Search → "Environment Variables" → set
`JAVA_HOME` to the JDK 25 folder, then add `%JAVA_HOME%\bin` to `Path`.

> Maven uses the JDK that `JAVA_HOME` points at, not necessarily the one on
> `PATH`. Always check `mvn -v` before blaming the build.

---

## 2. Start the infrastructure

```powershell
docker compose up -d
docker compose ps
```

Wait until PostgreSQL, Redis, and RabbitMQ show **healthy**.

| Service | Address | Credentials |
|---|---|---|
| PostgreSQL | `localhost:5432` | `sunsetguard` / `sunsetguard_dev_password` |
| Redis | `localhost:6379` | none in development |
| RabbitMQ | `localhost:5672` | `sunsetguard` / `sunsetguard_dev_password` |
| RabbitMQ UI | <http://localhost:15672> | same as above |

On an 8 GB laptop, keep Docker Desktop at about 4 GB of memory and 2 CPUs.
Prometheus and Grafana are intentionally not in the default Compose file; they
arrive in Phase 7.

---

## 3. Build everything

```powershell
mvn clean install
```

The first run downloads dependencies and takes several minutes. A successful
run ends with `BUILD SUCCESS` for all four modules.

---

## 4. Prove the baseline

```powershell
# 1. Build is reproducible
mvn clean install

# 2. Containers are healthy
docker compose ps

# 3. RabbitMQ answers
start http://localhost:15672

# 4. Git is clean after the build
git status
```

You are ready for Phase 1 when all four pass **and** CI is green on GitHub.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `release version 25 not supported` | Maven is using an older JDK | Set `JAVA_HOME` to JDK 25 and reopen the terminal |
| `Cannot connect to Docker daemon` | Docker Desktop is not running | Start Docker Desktop and wait for "running" |
| `Port 5432 already in use` | Another PostgreSQL is running | Stop it, or change the host port in `docker-compose.yml` |
| Containers restarting | Port conflict or low memory | `docker compose logs postgres` and raise Docker memory |
| Build fails in a module | Running from the wrong folder | Run `mvn clean install` from the repository root |

---

## What comes next

Phase 1 adds the first real code:

1. Flyway migrations for projects, API keys, and endpoints.
2. JPA entities and repositories.
3. REST controllers with validation and Problem Details errors.
4. API-key authentication and tenant-isolation tests.
5. OpenAPI documentation.

Then the starter (Phase 2), the RabbitMQ pipeline (Phase 3), verdicts
(Phase 4), the dashboard (Phase 5), the CLI and GitHub Action (Phase 6),
hardening (Phase 7), and launch (Phase 8).

**Do not start Phase 2 before Phase 1's exit test passes.** The pipeline is the
critical path; the dashboard is not.

---

_Updated: 2026-10-26_
