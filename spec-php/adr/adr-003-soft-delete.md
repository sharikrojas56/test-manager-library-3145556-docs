# ADR-003 · Books are soft‑deleted, not removed

- **Status:** accepted
- **Context:** a loan item points to a book. If the book is physically deleted, the history and report become broken.
- **Decision:** the book carries `deleted_at`. Deleting fills it; the catalog filters by `deleted_at IS NULL` in a single place (global filter). A book with open loans **cannot** be deleted (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a **partial** index on active books, so a soft‑deleted ISBN can be registered again.
- **Discarded alternatives:** physical deletion with cascade (destroys history); mark `active = false` (doesn't say when).
- **Consequences:** `loan_item.book_id` has a `RESTRICT` FK as a **last‑line barrier**: a manual `DELETE` or future code change fails noisily instead of corrupting the history. The cover binary is the only data physically deleted.
- **Verification:** querying a soft‑deleted book returns 404; a `DELETE` via `psql` on a book with items fails because of the FK.