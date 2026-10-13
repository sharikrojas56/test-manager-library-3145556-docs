# Specification — Manager Library

**What is built and why.** A system for a small library to manage its book catalog, members, and loans, and to see which books are borrowed most often.

It is governed by [`constitution.md`](constitution.md). The data model exists only in [`data-model.md`](data-model.md), and the HTTP contract exists only in [`api-contract.md`](api-contract.md).

---

## 1. Actors

| Actor | What they do | How they authenticate |
|---|---|---|
| **Administrator** (`admin`) | Maintains the book catalog and members; creates librarians; sees every report | Username and password → token |
| **Librarian** (`librarian`) | Records loans and returns; consults the catalog, members, and overdue items | Username and password → token |
| **Member** | A person who borrows books. **They do not access the system**: they are data, not a user | — |

The `admin` role is provisioned by deployment through environment variables. **Nobody grants it at runtime.**

## 2. Scope

**In scope:** book catalog with an optional cover, five fixed categories, members, loans with multiple items, per-item returns, overdue books, most-borrowed report, and role-based login.

**Out of scope, and not to be reopened:** fines and payments, reservations, renewals, email or SMS, bulk import, inter-branch loans, and book attributes beyond those in §3 (no publisher, year, or language).

## 3. User stories and acceptance criteria

### US-01 · Log in

As a user, I want to authenticate so that I can operate according to my role.

- **AC-01.1** Valid credentials return a token valid for 8 hours.
- **AC-01.2** Invalid credentials return 401 with a message that does **not** reveal whether the username or password failed.
- **AC-01.3** Every endpoint except login and `/health` requires a token. No token: 401. Insufficient role: 403.

### US-02 · Create librarians (admin)

- **AC-02.1** The admin creates a user with a username, password, and the `librarian` role.
- **AC-02.2** The username is stored lowercase and trimmed; it is unique. Duplicate: 409.
- **AC-02.3** Requesting the `admin` role through the API returns 422. The password is never returned or logged.

### US-03 · Manage books (admin)

- **AC-03.1** Create a book with title, author, ISBN, category, and total copies. Available copies start equal to total copies.
- **AC-03.2** The ISBN has 13 digits and is unique among active books. Duplicate: 409.
- **AC-03.3** Total copies must be ≥ 1. Changing the total adjusts available copies by the same difference and **never** leaves available copies < 0 (422).
- **AC-03.4** Uploading a cover (JPG/PNG, maximum 2 MB) stores an **opaque key**; without a cover the field is `null`, never an empty string.
- **AC-03.5** Deleting a book is a **soft delete**: it disappears from the catalog but remains in past loans. A book with open loans cannot be deleted (409).

### US-04 · Search books

- **AC-04.1** Partial-text search in title or author, category filter, active books only, ordered by title, paginated (`page`, `size`, maximum 50).
- **AC-04.2** The response includes the total result count.
- **AC-04.3** Requesting a nonexistent or deleted book by id returns 404.

### US-05 · Manage members

- **AC-05.1** The admin and librarian create members with name and document number. The document number is unique; duplicate: 409.
- **AC-05.2** Members can be listed and searched by name or document number, paginated.
- **AC-05.3** Members are not deleted.

### US-06 · Record a loan

- **AC-06.1** A loan belongs to **one** member, is recorded by the authenticated user, and contains 1 to 3 items.
- **AC-06.2** The same book cannot appear more than once in a loan (422).
- **AC-06.3** Each item subtracts **1** available copy in the same operation. If no copy is available: 409 and **nothing is recorded**.
- **AC-06.4** A member cannot have more than **3 unreturned books** in total (422 with the current count).
- **AC-06.5** A member with any **overdue** book cannot borrow another (409).
- **AC-06.6** The due date is `loaned_at + 14 days`. The item freezes the title, ISBN, and category name **at that instant**.
- **AC-06.7** A recorded loan cannot be edited or deleted.

### US-07 · Record a return

- **AC-07.1** **One item** is returned at a time. It sets `returned_at` and adds 1 to available copies.
- **AC-07.2** Returning an item that was already returned: 409. It **never** adds twice.
- **AC-07.3** Available copies never exceed total copies (the database guarantees this).
- **AC-07.4** When every item is returned, the loan appears as **closed**.

### US-08 · View and list loans

- **AC-08.1** Requesting a loan returns its items and its calculated status (`open`, `overdue`, `closed`).
- **AC-08.2** List by date range, status, and member, ordered by descending date, paginated.
- **AC-08.3** Status is **calculated**, not stored.

### US-09 · Overdue books

- **AC-09.1** Lists unreturned items with `due_at` before now: book, member, and late days.
- **AC-09.2** Ordered by highest late-day count first. Only `admin` and `librarian`.

### US-10 · Most-borrowed books report

- **AC-10.1** Given a date range, one row per **book and frozen category name**, with the number of times borrowed, ordered from highest to lowest.
- **AC-10.2** End date before start date: 422. A range without loans: empty list, not an error.
- **AC-10.3** The report is **not** broken down by librarian or member.
- **AC-10.4** A report for a closed period **does not change** if the book or its category is renamed later.
- **AC-10.5** Only `admin`.

## 4. Business rules

| # | Rule |
|---|---|
| BR-1 | Available copies are always between 0 and total copies. |
| BR-2 | Loan period: 14 days. Maximum 3 unreturned books per member. |
| BR-3 | A member with overdue items cannot borrow more until returning them. |
| BR-4 | Values frozen on the item (title, ISBN, category) do not follow the catalog. |
| BR-5 | Loans and returns are not deleted. Books use soft delete only. |
| BR-6 | Dates are UTC instants. User-facing messages are in English; code and columns are in English. |

## 5. Non-functional requirements

| # | Requirement | How it is measured |
|---|---|---|
| NFR-1 | Book search responds in less than 300 ms with 5,000 books | Load test with the `tool` seeder |
| NFR-2 | Two librarians lending the **last copy** simultaneously: one wins and the other receives 409 | Concurrent test |
| NFR-3 | The password never travels or is stored in plain text; it never appears in logs | Review logs and responses |
| NFR-4 | Every HTTP error uses the same body (see contract) | Contract test |
| NFR-5 | The system starts with one command and the database starts empty | `infra` README |
