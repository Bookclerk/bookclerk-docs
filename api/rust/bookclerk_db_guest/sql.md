**bookclerk_db_guest > sql**

# Module: sql

## Contents

**Structs**

- [`GuestExecResult`](#guestexecresult) - Mutating statement outcome inside the session worker.
- [`GuestQueryResult`](#guestqueryresult) - Read-only statement result inside the session worker.
- [`GuestRow`](#guestrow) - One query row projected for session paging and tests.
- [`GuestStatement`](#gueststatement) - SQL text, JSON-encoded binds, and optional txn id for the session worker.

**Functions**

- [`guest_sql`](#guest_sql) - Builds a [`GuestStatement`](sql.md#bookclerk_db_guestsqlgueststatement) for plain SQL without binds.

---

## bookclerk_db_guest::sql::GuestExecResult

*Struct*

Mutating statement outcome inside the session worker.

**Fields:**
- `last_insert_id: u64` - Last auto-increment value when applicable.
- `rows_affected: u64` - Rows affected by the statement.



## bookclerk_db_guest::sql::GuestQueryResult

*Struct*

Read-only statement result inside the session worker.

**Fields:**
- `rows: Vec<GuestRow>` - Result rows in engine order.



## bookclerk_db_guest::sql::GuestRow

*Struct*

One query row projected for session paging and tests.

**Fields:**
- `values: std::collections::BTreeMap<String, serde_json::Value>` - Column name → JSON cell value.



## bookclerk_db_guest::sql::GuestStatement

*Struct*

SQL text, JSON-encoded binds, and optional txn id for the session worker.

**Fields:**
- `sql: String` - SQL text with positional placeholders.
- `values: Vec<serde_json::Value>` - Ordered bind values (JSON encoding for typed nulls and bytes).
- `txn_id: Option<String>` - Active guest transaction id, if any.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> GuestStatement`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_guest::sql::guest_sql

*Function*

Builds a [`GuestStatement`](sql.md#bookclerk_db_guestsqlgueststatement) for plain SQL without binds.

```rust
fn guest_sql<impl Into<String>>(sql: impl Trait) -> GuestStatement
```



