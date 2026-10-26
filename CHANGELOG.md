# Changelog

All notable changes to SunsetGuard will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed
- Upgraded the build to Spring Boot 4.1.2 and Spring Framework 7.
- Upgraded Lombok to 1.18.48, JaCoCo to 0.8.16, springdoc to 3.2.0, and
  Testcontainers to 1.21.4.
- Removed the unused version properties and the explicit PostgreSQL version so
  the Spring Boot BOM owns dependency versions.
- Corrected the documentation to match the real Phase 0 state: foundation
  built, runtime code not started.
- Added `docs/NEXT-STEPS.md`, the ordered build plan for Phases 1–8.
- Removed the unbuilt "Phase 9 cloud deployment" from the roadmap; hosting is
  optional and local-first.

### Phase 0 - Foundation (Week 1)
- Initial project structure
- Maven multi-module setup
- Docker Compose with PostgreSQL, Redis, RabbitMQ
- GitHub Actions CI workflow
- Architecture Decision Records (ADR-001, ADR-002)
- Development environment documentation

## [0.1.0-SNAPSHOT] - In Development

Initial development version. See [README.md](README.md) for roadmap.

---

## Version Format

- **Major.Minor.Patch** (e.g., 1.2.3)
- **Major**: Breaking changes (e.g., API contract changes)
- **Minor**: New features, backward compatible
- **Patch**: Bug fixes, backward compatible
- **-SNAPSHOT**: Development version

## Categories

- **Added**: New features
- **Changed**: Changes to existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Removed features
- **Fixed**: Bug fixes
- **Security**: Security improvements
