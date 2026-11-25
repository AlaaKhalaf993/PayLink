# Stack Decision

**Status:** Accepted. It matches the repository skeleton (`pom.xml`, `Dockerfile`, `.github/workflows/ci.yml`).
**Decision:** Java + Spring Boot (fixed for the whole project, per the brief)

## Reasons

1. **Exact money handling.** `BigDecimal` gives exact decimal arithmetic, so no floating point is used for money anywhere (brief, section 2.6).
2. **Concurrency control.** Spring Data JPA supports pessimistic and optimistic locking. This is what we need so that two simultaneous payments never drive a wallet balance negative (hard case 4).
3. **OpenAPI out of the box.** `springdoc-openapi` generates the OpenAPI documentation required for the 20+ REST endpoints.
4. **Modular monolith friendly.** Java packages map directly to modules, and interfaces plus Spring dependency injection fit the hexagonal style and the Adapter/Strategy/State patterns.
5. **Mature testing and CI tooling.** JUnit 5, Mockito, Testcontainers and JaCoCo (coverage) all integrate well with GitHub Actions and Docker.
6. **Easy path to events in Phase 2.** Spring application events first, then RabbitMQ or Kafka through Spring AMQP/Kafka.

## Proposed toolchain

| Concern | Choice |
|---|---|
| Language | Java 21 (LTS) |
| Framework | Spring Boot 4.x (`pom.xml` uses 4.2.0) |
| Build | Maven with the wrapper (`mvnw`) |
| Database | PostgreSQL |
| Migrations | Flyway (SQL files in repository) |
| API docs | springdoc-openapi |
| Tests | JUnit 5, Mockito, Testcontainers |
| Coverage | JaCoCo (reported in CI, target 70% on domain logic) |
| Lint | Checkstyle (already in `pom.xml`, runs in `verify`) |
| Containers | Docker + docker compose |
| CI/CD | GitHub Actions |
| Broker (Phase 2) | RabbitMQ (alternative: Kafka) |

## Points the team must confirm

- [ ] Everyone is comfortable with Java and Spring Boot
- [x] Maven (the repository uses `mvnw`)
- [x] Java 21 in `pom.xml` and in CI

## Still to add in Phase 2 (not in `pom.xml` yet)

Flyway, springdoc-openapi, JaCoCo (coverage gate of 70%), Testcontainers and the message broker.
