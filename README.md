# payrail

A payments system (accounts, balances, transfers, ledger) with an AI copilot for
fraud analysts, built from scratch as a hands-on lab for backend and distributed-systems
engineering.

The goal is learning: every design choice here is made deliberately, measured under load
or under failure, and recorded with its trade-offs.

- **What the system must solve:** [`docs/problems.md`](docs/problems.md)
- **Why each decision was made:** `docs/adr/` (Architecture Decision Records)

## Stack decided up front

Java 21 · Spring Boot · PostgreSQL · Docker · GitHub Actions. Everything else is decided
problem by problem.
