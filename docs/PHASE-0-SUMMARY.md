# Phase 0 Summary — Foundation

**Completed:** 17 October 2026
**Last verified:** 26 October 2026
**Next phase:** Phase 1 — Server Core

---

## What Phase 0 delivered

| Area | What exists | Evidence |
|---|---|---|
| Repository | Public GitHub repo, Apache-2.0 license, README, CONTRIBUTING, CHANGELOG | `LICENSE`, `README.md` |
| Build | Maven multi-module parent + 4 modules | `pom.xml`, `mvn clean install` |
| Runtime baseline | Java 25, Spring Boot 4.1.2, Spring Framework 7 | Parent POM |
| Infrastructure | PostgreSQL 16, Redis 7, RabbitMQ 3.13 with health checks and volumes | `docker-compose.yml` |
| CI | Build, tests, JaCoCo, Trivy, dependency scan | `.github/workflows/build.yml` |
| Decisions | ADR-001 RabbitMQ over Kafka, ADR-002 client-side aggregation | `adr/` |
| Documentation | Setup, quickstart, phase checklist, next steps | `docs/` |

## Module map

```
sunsetguard-parent
├── sunsetguard-server               # API, workers, dashboard (Phase 1+)
├── sunsetguard-spring-boot-starter  # client library (Phase 2)
├── sunsetguard-cli                  # CI check (Phase 6)
└── sunsetguard-demo-orders          # monitored demo API (Phase 2)
```

All four modules compile today. Only the parent POM contains behaviour; the
module POMs declare dependencies for the code that arrives in later phases.

## Verified environment

| Tool | Verified version | Note |
|---|---|---|
| JDK | 25.0.2 | `JAVA_HOME` must point here; the machine default may be older |
| Maven | 3.10.0 | Uses `JAVA_HOME`, not `PATH` |
| Docker | 29.8.2 | Compose v2 |
| PostgreSQL | 16 | Container, port 5432 |
| Redis | 7 | Container, port 6379 |
| RabbitMQ | 3.13 | Container, ports 5672 and 15672 |

### Reproduce the verification

```powershell
$env:JAVA_HOME = "C:\Program Files\Java\jdk-25.0.2"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"

java -version          # 25.x
mvn -v                 # Java version 25.x
docker compose up -d
docker compose ps      # all healthy
mvn clean install      # BUILD SUCCESS
```

## What Phase 0 deliberately did not build

- No `@SpringBootApplication` class, controllers, entities, or migrations.
- No filter, aggregator, or sender in the starter.
- No RabbitMQ queues or consumers.
- No dashboard templates.
- No CLI commands or GitHub Action.

This is why the server cannot start yet. `mvn spring-boot:run` becomes meaningful
in Phase 1.

## Exit criteria — met

- [x] `docker compose up -d` starts all three services healthy.
- [x] `mvn clean install` succeeds on JDK 25.
- [x] CI runs on every push and pull request.
- [x] Two ADRs explain the two riskiest choices.
- [x] A new reader can find setup, scope, and build order in `docs/`.

## Where to go next

1. Read [NEXT-STEPS.md](NEXT-STEPS.md) for the ordered plan.
2. Start Phase 1 with the Flyway schema.
3. Keep the phase exit test as the gate; do not start Phase 2 early.

---

_Phase 0 is complete. The project is now judged by working software, not by
structure._
