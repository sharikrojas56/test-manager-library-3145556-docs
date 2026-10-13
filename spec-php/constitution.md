# Constitution — Manager Library (PHP/Laravel)

These are the project’s non-negotiable principles. They are not style or preference: every article includes **how it is verified**, because a principle that nobody can verify is only an intention. A change that violates an article is rejected without considering the rest.

---

## Article I — The domain depends on nothing

`app/Domain/` imports no other application layer or third-party package: only PHP’s standard library; no Eloquent, no Laravel HTTP classes, and no validation framework.

**Why:** this is the only way to test invariants without a database, server, or container, and to replace the ORM without changing rules.
**Verification:** an architecture test prevents the domain from depending on `Application`, `Infrastructure`, or `Interface` namespaces.

## Article II — The core declares its ports; the outside world implements them

The application layer only knows the domain. Everything it needs from outside (repositories, clock, password hashing, cover storage, token generation) is expressed as a **port that the application layer defines**. No framework or ORM type appears in a port signature.

**Verification:** architecture tests prevent `Application` from importing `Infrastructure`. Application tests run with fakes and without referencing `Infrastructure`.

## Article III — One composition root

Only the Laravel service providers, in `app/Providers/`, bind ports to their adapters. Infrastructure is not instantiated outside that composition root.
**Verification:** no infrastructure class is instantiated outside service providers; this is checked with an architecture test.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends books and does nothing else | A service with two reasons to change is split |
| **OCP** | Replacing cover storage (local disk → S3) does not touch domain or application code | The new adapter only changes a provider binding |
| **LSP** | Test fakes substitute the real adapter without changing expectations | The same test passes against the fake and the real implementation |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementation leaves methods unused |
| **DIP** | The consumer defines the port | Ports live in the application layer |

## Article V — Migrations own the schema

The `infra` repository starts an **empty** database engine. All DDL (tables, indexes, constraints, extensions, and seed data) belongs in the application migrations (Laravel migrations). See [ADR-001](adr/adr-001-schema-ownership.md).
**Verification:** `infra` contains no `CREATE TABLE` statement.

## Article VI — Invariants the database can enforce are enforced by the database

A rule that exists only in code protects the application, not the data. Every invariant expressible by the engine (`CHECK`, `UNIQUE`, FK) is declared there. Anything not enforced is marked **pending** in [`data-model.md`](data-model.md); it is not promised.
**Verification:** the queries in `data-model.md` §10 match what the document declares.

## Article VII — Derived values are calculated, not stored

A loan’s status (`open`, `overdue`, `closed`) and late days are calculated. There is no column for either.
**Verification:** `data-model.md` lists neither a `status` nor a `days_late` column.

## Article VIII — Frozen values do not follow the catalog

A loan item stores the title, ISBN, and category at the instant of lending. No report reads them from the live catalog.
**Verification:** a test renames a book after lending it and verifies that the report retains the old title ([ADR-004](adr/adr-004-aggregated-and-frozen-report.md)).

## Article IX — No secret enters the repository or a log

Passwords, precomputed hashes, and the token key all come from the environment. The real `.env` is in `.gitignore`; only `.env.example` is versioned.
**Verification:** `gitleaks` or pattern scanning in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The database engine wins

If the data document and the database differ, the document is broken and must be corrected.
The migration schema is the authoritative implementation of database constraints.
The data document records that implementation and its deliberate pending constraints.
Application code must not treat a conflicting document as an excuse to bypass the database.
**Verification:** migration review confirms every documented enforced constraint.

## Article XI — Language

Code, columns, and routes are in English. Documentation and user-facing messages are in English. All specification documents have been translated from Spanish to English to ensure consistency across implementations.

## Article XII — Nothing is complete without evidence

A [`tasks.md`](tasks.md) task is complete when its **evidence** exists and can be executed, not when “it has already been programmed.”
