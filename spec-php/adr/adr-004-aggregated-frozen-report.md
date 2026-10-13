# ADR-004 · Report is calculated in the engine and grouped by frozen values

- **Status:** accepted
- **Context:** the most‑borrowed report (US-10) crosses `loan` and `loan_item` over a range. If it read the title or category from the live catalog, renaming a book or category would change a report for a period already closed.
- **Decision:** (1) the report **is not persisted**: it is calculated with a single aggregated query by a read port. (2) It groups by `book_id, book_title, category_name` **as they were frozen in the item**. (3) `loan_item.category_name` **has no FK** on purpose. (4) If a book changed categories within the range, the report shows **two rows**: it doesn't pick a winner.
- **Discarded alternatives:** join with the live catalog (report stops being stable); take "the most recent value in the range" (a new loan would change what was already read); materialize a report table (second source of truth).
- **Consequences:** AC-10.1 says "one row per book and frozen category name", not "one row per book". The report doesn't include members or librarians (privacy).
- **Verification:** lend a book, rename it, request the report for the same range: the report title is the old one.