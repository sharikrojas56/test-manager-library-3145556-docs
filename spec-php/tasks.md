# Tasks — Manager Library (PHP/Laravel)

Each task finishes when its **evidence** exists and can be executed (article XII). Initial state of all: **pending**.

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` responds `{"status":"ok"}`; container starts with one command |
| T-02 | Initial migration: six tables, FKs, unique indexes, and five seeded categories | T-01 | Queries from `data-model.md` §10 return what is documented; no column with `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` (value objects) | T-01 | Unit tests: ISBN 13 digits, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`Withdraw`, `Restock`, soft delete) | T-03 | Tests: never < 0, never > total |
| T-05 | Domain: `Loan` (`AddItem`, `ReturnItem`, calculated status, 14 days, max 3) | T-04 | Tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (name normalization, valid role) | T-03 | `"Ana "` becomes `ana`; `admin` role via API rejected |
| T-07 | Ports and use cases for **authentication** (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | US-01 and US-02 pass against fakes |
| T-08 | Ports and use cases for **books** (create, search, edit, delete, cover) with fakes | T-04 | US-03 and US-04 pass against fakes |
| T-09 | Use case **lend books** (`LendBooksService`) with fakes | T-05, T-08 | US-06 complete: limits, overdue, no copies |
| T-10 | Use case **return** and **overdue** | T-09 | US-07 and US-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | Use case **most-borrowed report** and its read port | T-09 | US-10; old title preserved after renaming (ADR-004) |
| T-12 | Persistence adapter: book, member, user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository + version witness | T-12, T-09 | Two loans of the last copy: one 201, the other 409 (NFR-2) |
| T-14 | Security adapter: password hash and JWT token | T-07 | Password never appears in logs or responses (NFR-3) |
| T-15 | Cover adapter (disk) | T-08 | Upload 2 MB PNG ok; 3 MB → 422; no cover → `cover_key` null |
| T-16 | HTTP adapter: routes from [`api-contract.md`](api-contract.md), unified error body, role-based authorization | T-10, T-11, T-14, T-15 | Contract test covers all routes and all error codes from §2 |
| T-17 | Composition and startup (initial admin from environment) | T-16 | System starts with empty database and can log in as admin |
| T-18 | Move domain-only invariants to database: `ck_book_total_positive`, `ck_user_role` | T-12 | An `INSERT` via `psql` that violates them fails; `data-model.md` §4 updated |
| T-19 | Architecture tests (articles I, II, and III) | T-17 | `import-linter` contract and test in `tests/architecture/` passes and fails if prohibited import introduced |
| T-20 | `infra`: `docker compose up` brings up empty engine, API, and frontend | T-17 | One command; database starts without tables until migrations run |
| T-21 | `tool`: demonstration seeder **via the API** (≥ 40 books, 10 members, 15 loans, 3 overdue) | T-17 | After running it, `/loans/overdue` returns 3 items and report has rows |
| T-22 | `app`: login, catalog, members, lend, return, overdue, report | T-16 | End‑to‑end manual walkthrough |
| T-23 | `page`: static public presentation site | — | Opens without the API; correct on mobile and desktop |

## Definition of test completion

Whoever reaches T-17 with evidence has the **core** done. T-18 through T-23 are what separates good work from outstanding work. Performance is evaluated with SDD, not number of tasks: one task with evidence is worth more than three half‑done tasks.