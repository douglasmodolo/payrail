# payrail — instructions for Claude

A payments system (accounts, balances, transfers, ledger) plus an AI copilot for fraud
analysts. This is a **learning** project: the goal is for Douglas to be able to explain and
defend every choice in an interview.

## Where things are

- **What to solve:** `docs/problems.md` (P1…P27, in order). This is the plan; there is no
  other.
- **What was decided and why:** `docs/adr/`. The latest ADR shows how far we have come.
- The **how** of each problem is not planned ahead: it comes out of discussing that problem.

## How we work

- **Douglas makes the design decisions.** Present the problem and the trade-offs, ask for
  his proposal first, then critique it with questions and failure/scale scenarios. Never
  hand over a finished design or choose for him.
- **He writes the domain code by hand.** Claude guides, questions and reviews; it does not
  write that code.
- **Claude may do side work** when asked: CI, Docker/compose, data generators, load-test
  scripts, configuration.
- No analogies to .NET/C#.
- No study-time rules.

## Workflow for each problem

1. He proposes a solution before researching (naive is fine).
2. Claude attacks it with failure and scale scenarios.
3. He records the decision in `docs/adr/NNNN-title.md` (template in `docs/adr/README.md`).
4. He implements it and proves it with the problem's "pronto quando" (done-when).
5. If the proof contradicts the decision, a new ADR explains what changed.

One session per problem. What carries over between sessions is the repo, not the chat.

One branch and one PR per problem (e.g. `p1-money-accounts`): the ADR, code and tests land
together. Small, frequent commits inside the branch. Until P5, small doc tweaks may go
straight to `main`; after P5, everything goes through a PR.

## Conventions

- Code, names, commits and ADRs in English. Conversation and `docs/problems.md` in
  Brazilian Portuguese.
- Fixed stack: Java 21, Spring Boot, PostgreSQL, Docker, GitHub Actions. Everything else is
  decided problem by problem.

## Commands

(fill in once there is a build — P3)
