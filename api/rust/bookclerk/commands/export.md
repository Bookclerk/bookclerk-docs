**bookclerk > commands > export**

# Module: commands::export

## Contents

**Functions**

- [`export_csv`](#export_csv) - Writes library rows as CSV with the classic export column set.
- [`export_json`](#export_json) - Writes library rows as pretty-printed JSON (`BookRecord` array).
- [`export_xlsx`](#export_xlsx) - Writes library rows to an `.xlsx` workbook with the same columns as CSV.
- [`filter_books`](#filter_books) - Keeps books whose uuid, product id, ISBN, or ASIN matches `asins` (case-insensitive).
- [`load_books`](#load_books) - Lists library books, optionally scoped to one account id.

---

## bookclerk::commands::export::export_csv

*Function*

Writes library rows as CSV with the classic export column set.

```rust
fn export_csv(path: &std::path::Path, books: &[bookclerk_library::BookRecord]) -> anyhow::Result<()>
```



## bookclerk::commands::export::export_json

*Function*

Writes library rows as pretty-printed JSON (`BookRecord` array).

```rust
fn export_json(path: &std::path::Path, books: &[bookclerk_library::BookRecord]) -> anyhow::Result<()>
```



## bookclerk::commands::export::export_xlsx

*Function*

Writes library rows to an `.xlsx` workbook with the same columns as CSV.

```rust
fn export_xlsx(path: &std::path::Path, books: &[bookclerk_library::BookRecord]) -> anyhow::Result<()>
```



## bookclerk::commands::export::filter_books

*Function*

Keeps books whose uuid, product id, ISBN, or ASIN matches `asins` (case-insensitive).

```rust
fn filter_books(books: Vec<bookclerk_library::BookRecord>, asins: Option<&[String]>) -> Vec<bookclerk_library::BookRecord>
```



## bookclerk::commands::export::load_books

*Function*

Lists library books, optionally scoped to one account id.

```rust
fn load_books(store: &bookclerk_library::LibraryStore, account: Option<&str>) -> anyhow::Result<Vec<bookclerk_library::BookRecord>>
```



