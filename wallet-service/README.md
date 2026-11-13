# p2p-payments

A PayPal-style peer-to-peer payments backend: users, wallets, top-ups and transfers.
Built as a learning project. Payments are simulated; no real money is involved.

![CI](https://github.com/<you>/p2p-payments/actions/workflows/ci.yml/badge.svg)

## Status
- [x] M0 · Project setup
- [ ] M1 · Core money movement (ledger, transactions)
- [ ] M2 · Idempotency and concurrency
- [ ] M3 · Kafka and the transactional outbox
- [ ] M4 · Authentication (Spring Security, JWT)
- [ ] M5 · Frontend
- [ ] M6 · Docker and Kubernetes
- [ ] M7 · Azure deployment

## Tech stack
Java 25 · Spring Boot 4 · Maven · PostgreSQL 18 · GitHub Actions

## Running locally
Prerequisites: JDK 25, Docker. You don't need to install Maven; the wrapper (`mvnw`) is included.

    docker compose -f deploy/docker-compose.yml up -d
    cd wallet
    ./mvnw spring-boot:run
    curl localhost:8080/actuator/health   # → {"status":"UP"}

## Running tests
    cd wallet && ./mvnw verify

## Project structure
    wallet/   Spring Boot app, owns the database
    deploy/           Docker Compose (Kubernetes manifests later)
    docs/             Design decisions

## Design decisions
See [docs/decisions.md](docs/decisions.md).