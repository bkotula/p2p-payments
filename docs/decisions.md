# Decisions

Short notes on what was chosen, what was rejected, and why. Newest first.

## 2026-11-16 · Scope extended: account deletion, payouts and AML

**Chose:** bring account deletion, payouts (withdrawals to a linked bank account) and simulated AML checks into scope. AML is split in two: limits checked synchronously inside wallet-service before money moves, and transaction monitoring in a new `aml-monitoring-service` that consumes Kafka events and raises alerts.

**Rejected:** keeping the original scope, which left withdrawals out and had no compliance features.

**Why:** a banking-sector story needs more than moving money between wallets. Deletion, payouts and AML bring up GDPR erasure vs record retention, KYC-based limits and asynchronous monitoring, all of which come up in fintech interviews. Details in [wallet-service/docs/requirements.md](../wallet-service/docs/requirements.md).

## 2026-11-16 · CI: one workflow, triggered on trunk pushes and PRs, one job per app

**Chose:** a single GitHub Actions workflow (`.github/workflows/pipeline.yaml`) that runs `./mvnw -B verify` on `push` to `trunk` and on every `pull_request`. Jobs are named after the app they build (`wallet-service`).

**Rejected:** running on every push to every branch (a branch with an open PR would be tested twice); a generic job name like `test`.

**Why:** PRs are checked before merging and `trunk` is checked after. A job named after its app shows at a glance which app broke once there are several, and gives branch protection a stable check name. Per-app path filters come in M7, when there is more than one app to build.

## 2026-11-13 · Credentials in the repo only for the local dev database

**Chose:** plain-text `wallet`/`wallet` credentials in `deploy/docker-compose.yaml` and `application.yaml`.

**Rejected:** a secrets setup (`.env` files, a vault) for local development.

**Why:** they only protect a throwaway database on a developer laptop, with no real data and no outside access. Real environments get credentials injected at runtime (Kubernetes Secrets in M6, Azure in M7), read through placeholders like `${DB_PASSWORD}`; no real credential is ever committed.

## 2026-11-13 · One database per service

**Chose:** each service owns its own database (`wallet` for wallet-service; notification-service gets its own in M3).

**Rejected:** one shared database that all services read and write.

**Why:** a service's tables are its private implementation. Other services get the data only through its API or its events. With a shared database, a schema change in wallet-service could silently break notification-service, and the services could no longer be deployed independently.

## 2026-11-13 · Throwaway database per test run (Testcontainers)

**Chose:** integration tests start a throwaway `postgres:18` container through Testcontainers (`@ServiceConnection`).

**Rejected:** an in-memory H2 database; a long-lived test database (for example in Docker Compose) reused across runs and developers.

**Why:** tests run against the same PostgreSQL version as development and production, so locking, SQL dialect and constraints behave for real, which the concurrency tests in M2 depend on. Every run starts from an empty database, so leftover data can't make tests flaky. `./mvnw verify` works locally and in CI with no setup beyond Docker. Within one run the container is shared by all test classes, so tests still clean up their own data (M1).

## 2026-11-13 · Flyway owns the schema

**Chose:** schema changes only through versioned Flyway SQL migrations in Git; Hibernate set to `ddl-auto: validate` (from M1).

**Rejected:** letting Hibernate create or update tables from the entities (`ddl-auto: update`).

**Why:** every environment (developer laptops, CI, production) reaches the same schema by applying the same migrations in the same order, and Flyway records which ones ran. Schema changes are code-reviewed like any other code. Hibernate's auto-update never drops anything, can't migrate data and isn't reviewed.

## 2026-11-13 · Maven with the Maven wrapper

**Chose:** Maven, run through the wrapper (`./mvnw`), which pins Maven 3.10.0 in `.mvn/wrapper/maven-wrapper.properties`.

**Rejected:** Gradle; requiring a locally installed Maven.

**Why:** Maven is the most common build tool in Java teams. The wrapper downloads the pinned version, so nobody needs Maven installed and every laptop and the CI runner build with the same version, which removes a source of "works on my machine" problems.

## 2026-11-13 · Monorepo

**Chose:** one repository (`p2p-payments`) holding all apps, with shared configuration at the root (`.gitignore`, `deploy/`, `docs/`, CI).

**Rejected:** one repository per service.

**Why:** easier to maintain for a solo learning project, and one repository to share with recruiters. Changes that span services, like an event format shared by producer and consumer, land in one PR and are tested together. Trade-off: per-app CI needs path filters (M7), and the repository grows with every app.
