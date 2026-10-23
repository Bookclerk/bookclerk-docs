**bookclerk_plugin_abi > sql_overflow**

# Module: sql_overflow

## Contents

**Enums**

- [`OverflowDialect`](#overflowdialect) - Engine family for overflow derived-table syntax and i64::MIN CAST.

**Functions**

- [`apply_integer_overflow`](#apply_integer_overflow) - Wrap each INTEGER `+`/`-`/`*`/`abs` site in a portable overflow CASE.

---

## bookclerk_plugin_abi::sql_overflow::OverflowDialect

*Enum*

Engine family for overflow derived-table syntax and i64::MIN CAST.

**Variants:**
- `SqliteFamily` - SQLite / D1: correlated `FROM (SELECT …) _bc_ov`, `INTEGER` min CAST.
- `Postgres` - PostgreSQL: `LATERAL` + `BIGINT` min CAST.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OverflowDialect) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> OverflowDialect`



## bookclerk_plugin_abi::sql_overflow::apply_integer_overflow

*Function*

Wrap each INTEGER `+`/`-`/`*`/`abs` site in a portable overflow CASE.

Sites are applied **innermost-first** (`full.end`, then reverse `full.start`)
so right-nested `1 + abs(n)` rewrites the `abs` call before the outer add.
After each wrap, remaining (outer) sites shift by the inserted byte count.

# Errors

Returns when a site is empty, past `sql`, not on a UTF-8 boundary, or cannot
be wrapped as the recorded operator.

```rust
fn apply_integer_overflow(dialect: OverflowDialect, sql: &str, sites: &[crate::sql_proof::IntegerArithSite]) -> crate::Result<String>
```



