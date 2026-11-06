# bookclerk_db_exec::sql_v1

Shared Bookclerk SQL v1 execution vectors (binding DDL + portable helpers).

Hosts emit these canonical statements. Adapters lower at execute: every
backend rewrites `INSERT OR IGNORE` to unique/PK `ON CONFLICT DO NOTHING`;
Postgres also applies binding DDL type rewrites and helper lowering.
Every name in `GuestSqlPolicy::binding_owned` portable functions
(except denied `hex`) appears here.

| Item | Kind | Summary |
| --- | --- | --- |
| [`PortableExpect`](#portableexpect) | enum | Expected cells for `PORTABLE_SELECT` (adapter-neutral). |
| [`portable_select_expects`](#portable_select_expects) | function | Expected `PORTABLE_SELECT` cells in column order. |
| [`portable_boolean_expects`](#portable_boolean_expects) | function | Expected `PORTABLE_BOOLEAN_SELECT` cells in column order. |
| [`portable_boolean_insert_binds`](#portable_boolean_insert_binds) | function | Binds for `PORTABLE_BOOLEAN_INSERT`. |
| [`portable_lowercase_insert_binds`](#portable_lowercase_insert_binds) | function | Binds for `PORTABLE_INSERT_OR_IGNORE_RETURNING_LC`. |
| [`portable_aggregate_expects`](#portable_aggregate_expects) | function | Expected `PORTABLE_AGGREGATE_SELECT` cells in column order. |
| [`portable_value_matches`](#portable_value_matches) | function | True when `got` is the exact `DbValue` for `want`. |
| [`portable_select_mismatch`](#portable_select_mismatch) | function | Formats a mismatch for adapter execution tests. |
| [`portable_boolean_mismatch`](#portable_boolean_mismatch) | function | Formats a mismatch for `PORTABLE_BOOLEAN_SELECT`. |
| [`portable_aggregate_mismatch`](#portable_aggregate_mismatch) | function | Formats a mismatch for `PORTABLE_AGGREGATE_SELECT`. |
| [`portable_statement_mismatch`](#portable_statement_mismatch) | function | Compares `stmt` to `expect` and describes the first mismatch. |
| [`mixed_gate_literal_insert`](#mixed_gate_literal_insert) | function | INSERT that embeds `crate::GUEST_RECEIPT_WRITE_GATE` in a comment and a string. |
| [`portable_min_max_null_expects`](#portable_min_max_null_expects) | function | Expected `PORTABLE_MIN_MAX_NULL` cells. |
| [`portable_order_nulls_asc_expects`](#portable_order_nulls_asc_expects) | function | Expected `PORTABLE_ORDER_NULLS_ASC` cells. |
| [`portable_order_nulls_desc_expects`](#portable_order_nulls_desc_expects) | function | Expected `PORTABLE_ORDER_NULLS_DESC` cells. |
| [`portable_uncast_round_expects`](#portable_uncast_round_expects) | function | Expected `PORTABLE_UNCAST_ROUND` cells (r=1.5). |
| [`portable_uncast_sum_avg_expects`](#portable_uncast_sum_avg_expects) | function | Expected `PORTABLE_UNCAST_SUM_AVG` cells (n=2). |
| [`portable_runtime_edges_expects`](#portable_runtime_edges_expects) | function | Expected `PORTABLE_RUNTIME_EDGES` cells. |
| [`portable_integer_overflow_expects`](#portable_integer_overflow_expects) | function | Expected `PORTABLE_INTEGER_OVERFLOW` cells. |
| [`portable_integer_overflow_mismatch`](#portable_integer_overflow_mismatch) | function | Formats a mismatch for `PORTABLE_INTEGER_OVERFLOW`. |
| [`portable_nested_integer_arith_expects`](#portable_nested_integer_arith_expects) | function | Expected `PORTABLE_NESTED_INTEGER_ARITH` cells. |
| [`portable_nested_integer_arith_mismatch`](#portable_nested_integer_arith_mismatch) | function | Formats a mismatch for `PORTABLE_NESTED_INTEGER_ARITH`. |
| [`portable_json_path_like_compare_expects`](#portable_json_path_like_compare_expects) | function | Expected `PORTABLE_JSON_PATH_LIKE_COMPARE` cells. |
| [`portable_json_path_like_compare_mismatch`](#portable_json_path_like_compare_mismatch) | function | Formats a mismatch for `PORTABLE_JSON_PATH_LIKE_COMPARE`. |
| [`portable_div_operands_expects`](#portable_div_operands_expects) | function | Expected `PORTABLE_DIV_OPERANDS` cells. |
| [`portable_div_operands_mismatch`](#portable_div_operands_mismatch) | function | Formats a mismatch for `PORTABLE_DIV_OPERANDS`. |
| [`portable_div_nullif_not_zero_guard_expects`](#portable_div_nullif_not_zero_guard_expects) | function | Expected `PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD` cells. |
| [`portable_div_nullif_not_zero_guard_mismatch`](#portable_div_nullif_not_zero_guard_mismatch) | function | Formats a mismatch for `PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`. |
| [`portable_json_object_semantics_expects`](#portable_json_object_semantics_expects) | function | Expected `PORTABLE_JSON_OBJECT_SEMANTICS` cells. |
| [`portable_json_object_semantics_mismatch`](#portable_json_object_semantics_mismatch) | function | Formats a mismatch for `PORTABLE_JSON_OBJECT_SEMANTICS`. |
| [`portable_text_prefix_literals_expects`](#portable_text_prefix_literals_expects) | function | Expected `PORTABLE_TEXT_PREFIX_LITERALS` cells. |
| [`portable_text_prefix_literals_mismatch`](#portable_text_prefix_literals_mismatch) | function | Formats a mismatch for `PORTABLE_TEXT_PREFIX_LITERALS`. |
| [`portable_runtime_edges_mismatch`](#portable_runtime_edges_mismatch) | function | Formats a mismatch for `PORTABLE_RUNTIME_EDGES`. |
| [`portable_min_max_null_mismatch`](#portable_min_max_null_mismatch) | function | Formats a mismatch for `PORTABLE_MIN_MAX_NULL`. |
| [`portable_order_nulls_asc_mismatch`](#portable_order_nulls_asc_mismatch) | function | Formats a mismatch for ASC NULL ordering. |
| [`portable_order_nulls_desc_mismatch`](#portable_order_nulls_desc_mismatch) | function | Formats a mismatch for DESC NULL ordering. |
| [`portable_uncast_round_mismatch`](#portable_uncast_round_mismatch) | function | Formats a mismatch for `PORTABLE_UNCAST_ROUND`. |
| [`portable_uncast_sum_avg_mismatch`](#portable_uncast_sum_avg_mismatch) | function | Formats a mismatch for `PORTABLE_UNCAST_SUM_AVG`. |
| [`portable_rows_mismatch`](#portable_rows_mismatch) | function | Compares every row in `stmt` to `expect`. |
| [`portable_like_expects`](#portable_like_expects) | function | Expected `PORTABLE_LIKE_SELECT` cells (`c8` bind is `'A'`). |
| [`portable_text_order_expects`](#portable_text_order_expects) | function | Expected `PORTABLE_TEXT_ORDER_SELECT` rows. |
| [`portable_text_ops_expects`](#portable_text_ops_expects) | function | Expected `PORTABLE_TEXT_OPS` cells. |
| [`portable_text_multilingual_expects`](#portable_text_multilingual_expects) | function | Expected `PORTABLE_TEXT_MULTILINGUAL` cells (exact UTF-8, no NFC/NFD fold). |
| [`BINDING_DDL_AUTOINCREMENT_BLOB`](#binding_ddl_autoincrement_blob) | constant | Binding `CREATE TABLE` covering identity + `BLOB`/`INTEGER`/`REAL`. |
| [`BINDING_DDL_BOOLEAN`](#binding_ddl_boolean) | constant | Binding `CREATE TABLE` covering canonical `BOOLEAN` (not INTEGER-as-bool). |
| [`PORTABLE_BOOLEAN_INSERT`](#portable_boolean_insert) | constant | INSERT `DbValue::Boolean` + typed-null into `BINDING_DDL_BOOLEAN`. |
| [`PORTABLE_BOOLEAN_SELECT`](#portable_boolean_select) | constant | SELECT declared boolean columns from `BINDING_DDL_BOOLEAN`. |
| [`BINDING_DDL_LOWERCASE`](#binding_ddl_lowercase) | constant | Lowercase / mixed-case DDL admitted by SQL v1 (idents fold) and lowered case-insensitively on Postgres. |
| [`PORTABLE_INSERT_OR_IGNORE_RETURNING_LC`](#portable_insert_or_ignore_returning_lc) | constant | Lowercase `INSERT OR IGNORE … RETURNING` against `BINDING_DDL_LOWERCASE`. |
| [`PORTABLE_LOWERCASE_SELECT`](#portable_lowercase_select) | constant | SELECT stored blob + boolean from `BINDING_DDL_LOWERCASE`. |
| [`PORTABLE_INSERT`](#portable_insert) | constant | Seed row for `PORTABLE_SELECT`. Bind `PORTABLE_INSERT_BLOB` at `?`. |
| [`PORTABLE_INSERT_BLOB`](#portable_insert_blob) | constant | Blob payload for `PORTABLE_INSERT`. |
| [`PORTABLE_LOWER_NON_ASCII_INSERT`](#portable_lower_non_ascii_insert) | constant | Inserts a non-ASCII `body` so `lower` can be compared with SQLite. |
| [`PORTABLE_LOWER_NON_ASCII_SELECT`](#portable_lower_non_ascii_select) | constant | `lower` of the row inserted by `PORTABLE_LOWER_NON_ASCII_INSERT`. |
| [`PORTABLE_LOWER_NON_ASCII_EXPECT`](#portable_lower_non_ascii_expect) | constant | SQLite's result for `PORTABLE_LOWER_NON_ASCII_SELECT`, and Postgres once the argument is `COLLATE "C"`. |
| [`PORTABLE_LOWER_PLACEHOLDER_SELECT`](#portable_lower_placeholder_select) | constant | `lower` of a bound placeholder. Postgres must collate the parameter (`lower(($1 COLLATE "C"))`) so `É` stays `É`, matching SQLite. |
| [`PORTABLE_LOWER_SUBSTR_SELECT`](#portable_lower_substr_select) | constant | `lower(substr(body, ?, ?))` with integer binds. |
| [`PORTABLE_SELECT`](#portable_select) | constant | Scalar portable helpers (no aggregates). |
| [`PORTABLE_AGGREGATE_SELECT`](#portable_aggregate_select) | constant | Aggregate-only companion to `PORTABLE_SELECT`. |
| [`MIXED_GATE_LITERAL_DDL`](#mixed_gate_literal_ddl) | constant | Mixed DDL whose companion INSERT stores the receipt-gate text as a value. |
| [`MIXED_GATE_LITERAL_SELECT`](#mixed_gate_literal_select) | constant | SELECT the stored gate literal from `mixed_gate_literal_insert`. |
| [`MIXED_GATE_LITERAL_COUNT`](#mixed_gate_literal_count) | constant | COUNT helper for mixed-gate replay assertions. |
| [`BINDING_DDL_CONFLICT`](#binding_ddl_conflict) | constant | Unique + NOT NULL table for `INSERT OR IGNORE` conflict-domain vectors. |
| [`PORTABLE_INSERT_OR_IGNORE_UNIQUE`](#portable_insert_or_ignore_unique) | constant | Unique-conflict ignore (second insert is a no-op). |
| [`PORTABLE_INSERT_OR_IGNORE_NOT_NULL`](#portable_insert_or_ignore_not_null) | constant | NOT NULL ignore must still error (not swallowed). |
| [`PORTABLE_MIN_MAX_NULL`](#portable_min_max_null) | constant | Scalar min/max NULL-poison (any NULL argument → NULL). |
| [`BINDING_DDL_ORDER_NULLS`](#binding_ddl_order_nulls) | constant | Nullable column for ORDER BY NULL ordering. |
| [`PORTABLE_ORDER_NULLS_INSERT_1`](#portable_order_nulls_insert_1) | constant | Seed three rows: 1, NULL, 2. |
| [`PORTABLE_ORDER_NULLS_INSERT_NULL`](#portable_order_nulls_insert_null) | constant | NULL seed. |
| [`PORTABLE_ORDER_NULLS_INSERT_2`](#portable_order_nulls_insert_2) | constant | 2 seed. |
| [`PORTABLE_ORDER_NULLS_ASC`](#portable_order_nulls_asc) | constant | ASC: NULL sorts as smallest. |
| [`PORTABLE_ORDER_NULLS_DESC`](#portable_order_nulls_desc) | constant | DESC: NULL sorts as smallest (last). |
| [`BINDING_DDL_IDENTITY`](#binding_ddl_identity) | constant | Identity table (explicit id then omit-id). |
| [`PORTABLE_IDENTITY_INSERT_EXPLICIT`](#portable_identity_insert_explicit) | constant | Explicit id 100. |
| [`PORTABLE_IDENTITY_INSERT_OMIT`](#portable_identity_insert_omit) | constant | Omit id (must yield 101 after explicit 100). |
| [`PORTABLE_IDENTITY_SELECT_MAX`](#portable_identity_select_max) | constant | Largest id. |
| [`PORTABLE_IDENTITY_DELETE_MAX`](#portable_identity_delete_max) | constant | Delete the max identity row (SQLite AUTOINCREMENT must not reuse it). |
| [`BINDING_DDL_UNQUOTED_FOLD`](#binding_ddl_unquoted_fold) | constant | Unquoted mixed-case table that folds to lowercase. |
| [`PORTABLE_UNQUOTED_FOLD_INSERT`](#portable_unquoted_fold_insert) | constant | SELECT via folded lowercase name. |
| [`PORTABLE_UNQUOTED_FOLD_SELECT`](#portable_unquoted_fold_select) | constant | SELECT from folded name. |
| [`PORTABLE_UNCAST_ROUND`](#portable_uncast_round) | constant | Uncast `round` (per-row; not mixed with aggregates). |
| [`PORTABLE_UNCAST_SUM_AVG`](#portable_uncast_sum_avg) | constant | Uncast integer `sum` / `avg` (aggregate-only companion). |
| [`PORTABLE_RUNTIME_EDGES`](#portable_runtime_edges) | constant | Runtime-edge operators/helpers (`/` `%` zero → NULL, `round` halfway, `substr`, `length`, `replace`, `abs`, `lower`). |
| [`PORTABLE_INTEGER_OVERFLOW`](#portable_integer_overflow) | constant | INTEGER overflow (`+` `-` `*` `abs(i64::MIN)`) yields NULL on every backend. |
| [`PORTABLE_NESTED_INTEGER_ARITH`](#portable_nested_integer_arith) | constant | Nested INTEGER `+`/`*`/`abs` plus TEXT-inside-overflow (`length('ä') + 1`). |
| [`PORTABLE_JSON_PATH_LIKE_COMPARE`](#portable_json_path_like_compare) | constant | `'$.…'` compared as TEXT (not a `json_extract` path) — Postgres must COLLATE. |
| [`PORTABLE_DIV_OPERANDS`](#portable_div_operands) | constant | `/` RHS covering call, unary, CAST, and parenthesized zero. |
| [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](#portable_div_nullif_not_zero_guard) | constant | `/` and `%` of a `NULLIF` that is **not** `NULLIF(expr, 0)` must still gain an outer zero guard (`NULLIF(0, 1)` evaluates to `0`). |
| [`PORTABLE_JSON_OBJECT_SEMANTICS`](#portable_json_object_semantics) | constant | `json_object` nulls, duplicate keys, nested objects, and 16-pair (32-arg) arity. |
| [`PORTABLE_TEXT_PREFIX_LITERALS`](#portable_text_prefix_literals) | constant | TEXT literals whose contents look like bind placeholders or `COLLATE`. |
| [`BINDING_DDL_IGNORE_SELECT`](#binding_ddl_ignore_select) | constant | `INSERT OR IGNORE … SELECT` (plain bind source). |
| [`PORTABLE_IGNORE_SELECT`](#portable_ignore_select) | constant | Plain `INSERT OR IGNORE … SELECT ? RETURNING id`. |
| [`PORTABLE_IGNORE_SELECT_WITH`](#portable_ignore_select_with) | constant | `WITH` source for `INSERT OR IGNORE`. |
| [`PORTABLE_IGNORE_SELECT_UNION`](#portable_ignore_select_union) | constant | Compound `UNION ALL` source. |
| [`PORTABLE_IGNORE_SELECT_ORDER_LIMIT`](#portable_ignore_select_order_limit) | constant | `ORDER BY` + `LIMIT` source. |
| [`BINDING_DDL_LIKE`](#binding_ddl_like) | constant | Case-sensitive `LIKE` table. |
| [`PORTABLE_LIKE_INSERT`](#portable_like_insert) | constant | Seed `'A'` for LIKE vectors. |
| [`PORTABLE_LIKE_SELECT`](#portable_like_select) | constant | Case-sensitive LIKE probes (A/a, prefix, GLOB metacharacters, NULL). |
| [`PORTABLE_LIKE_NON_ASCII`](#portable_like_non_ascii) | constant | Non-ASCII that must not Unicode-fold under LIKE. |
| [`BINDING_DDL_BLOB_DEFAULT`](#binding_ddl_blob_default) | constant | Typed BLOB default hex. |
| [`PORTABLE_BLOB_DEFAULT_INSERT`](#portable_blob_default_insert) | constant | INSERT using the BLOB default. |
| [`PORTABLE_BLOB_DEFAULT_SELECT`](#portable_blob_default_select) | constant | Read the default blob. |
| [`BINDING_DDL_TEXT_ORDER`](#binding_ddl_text_order) | constant | TEXT collation / order table. |
| [`PORTABLE_TEXT_ORDER_INSERT_B`](#portable_text_order_insert_b) | constant | Seed ASCII + non-ASCII rows. |
| [`PORTABLE_TEXT_ORDER_INSERT_A`](#portable_text_order_insert_a) | constant | Lowercase a. |
| [`PORTABLE_TEXT_ORDER_INSERT_EACUTE`](#portable_text_order_insert_eacute) | constant | Non-ASCII é (U+00E9). |
| [`PORTABLE_TEXT_ORDER_INSERT_E`](#portable_text_order_insert_e) | constant | ASCII e. |
| [`PORTABLE_TEXT_ORDER_SELECT`](#portable_text_order_select) | constant | Binary TEXT order (byte/code-point, not locale). |
| [`PORTABLE_TEXT_OPS`](#portable_text_ops) | constant | TEXT comparison + scalar min/max + lower/upper. |
| [`PORTABLE_TEXT_MULTILINGUAL`](#portable_text_multilingual) | constant | Full Unicode scalar TEXT (UTF-8), including supplementary-plane Han, RTL, combining sequences, and NFC vs NFD that must not be silently normalized. |
| [`PORTABLE_IDENTITY_INSERT_BAD_TYPE`](#portable_identity_insert_bad_type) | constant | Identity omit after rollback of an explicit id (same atomic request fails). |
| [`PORTABLE_IDENTITY_DROP`](#portable_identity_drop) | constant | DROP identity table. |

## Enums

### PortableExpect

```rust
pub enum PortableExpect {
    Int(i64),
    Text(&'static str),
    Float(f64),
    Bool(bool),
    NullBool,
    Null,
}
```

Expected cells for [`PORTABLE_SELECT`](#portable_select) (adapter-neutral).

**Variants**

- `Int`: Exact `DbValue::Int64`.
- `Text`: Exact `DbValue::Text`.
- `Float`: Exact `DbValue::Float64` (tolerance only vs another float).
- `Bool`: Exact `DbValue::Boolean`.
- `NullBool`: Exact `DbValue::Null` with `DbType::Bool`.
- `Null`: Exact `DbValue::Null` (any declared type).

Implements: `Clone`, `Copy`, `Debug`

## Functions

### portable_select_expects

```rust
pub fn portable_select_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_SELECT`](#portable_select) cells in column order.

### portable_boolean_expects

```rust
pub fn portable_boolean_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_BOOLEAN_SELECT`](#portable_boolean_select) cells in column order.

### portable_boolean_insert_binds

```rust
pub fn portable_boolean_insert_binds() -> Vec<DbValue>
```

Binds for [`PORTABLE_BOOLEAN_INSERT`](#portable_boolean_insert).

### portable_lowercase_insert_binds

```rust
pub fn portable_lowercase_insert_binds() -> Vec<DbValue>
```

Binds for [`PORTABLE_INSERT_OR_IGNORE_RETURNING_LC`](#portable_insert_or_ignore_returning_lc).

### portable_aggregate_expects

```rust
pub fn portable_aggregate_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_AGGREGATE_SELECT`](#portable_aggregate_select) cells in column order.

### portable_value_matches

```rust
pub fn portable_value_matches(got: &DbValue, want: PortableExpect) -> bool
```

True when `got` is the exact [`DbValue`](../bookclerk_plugin_abi/db_value.md#dbvalue) for `want`.

### portable_select_mismatch

```rust
pub fn portable_select_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for adapter execution tests.

### portable_boolean_mismatch

```rust
pub fn portable_boolean_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_BOOLEAN_SELECT`](#portable_boolean_select).

### portable_aggregate_mismatch

```rust
pub fn portable_aggregate_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_AGGREGATE_SELECT`](#portable_aggregate_select).

### portable_statement_mismatch

```rust
pub fn portable_statement_mismatch(
    stmt: &StatementResult,
    expect: &[PortableExpect],
    label: &str,
) -> Option<String>
```

Compares `stmt` to `expect` and describes the first mismatch.

### mixed_gate_literal_insert

```rust
pub fn mixed_gate_literal_insert() -> String
```

INSERT that embeds [`crate::GUEST_RECEIPT_WRITE_GATE`](guest_receipt.md#guest_receipt_write_gate) in a comment and a string.

### portable_min_max_null_expects

```rust
pub fn portable_min_max_null_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_MIN_MAX_NULL`](#portable_min_max_null) cells.

### portable_order_nulls_asc_expects

```rust
pub fn portable_order_nulls_asc_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_ORDER_NULLS_ASC`](#portable_order_nulls_asc) cells.

### portable_order_nulls_desc_expects

```rust
pub fn portable_order_nulls_desc_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_ORDER_NULLS_DESC`](#portable_order_nulls_desc) cells.

### portable_uncast_round_expects

```rust
pub fn portable_uncast_round_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_UNCAST_ROUND`](#portable_uncast_round) cells (r=1.5).

### portable_uncast_sum_avg_expects

```rust
pub fn portable_uncast_sum_avg_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_UNCAST_SUM_AVG`](#portable_uncast_sum_avg) cells (n=2).

### portable_runtime_edges_expects

```rust
pub fn portable_runtime_edges_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_RUNTIME_EDGES`](#portable_runtime_edges) cells.

### portable_integer_overflow_expects

```rust
pub fn portable_integer_overflow_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_INTEGER_OVERFLOW`](#portable_integer_overflow) cells.

### portable_integer_overflow_mismatch

```rust
pub fn portable_integer_overflow_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_INTEGER_OVERFLOW`](#portable_integer_overflow).

### portable_nested_integer_arith_expects

```rust
pub fn portable_nested_integer_arith_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_NESTED_INTEGER_ARITH`](#portable_nested_integer_arith) cells.

### portable_nested_integer_arith_mismatch

```rust
pub fn portable_nested_integer_arith_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_NESTED_INTEGER_ARITH`](#portable_nested_integer_arith).

### portable_json_path_like_compare_expects

```rust
pub fn portable_json_path_like_compare_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_JSON_PATH_LIKE_COMPARE`](#portable_json_path_like_compare) cells.

### portable_json_path_like_compare_mismatch

```rust
pub fn portable_json_path_like_compare_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_JSON_PATH_LIKE_COMPARE`](#portable_json_path_like_compare).

### portable_div_operands_expects

```rust
pub fn portable_div_operands_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_DIV_OPERANDS`](#portable_div_operands) cells.

### portable_div_operands_mismatch

```rust
pub fn portable_div_operands_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_DIV_OPERANDS`](#portable_div_operands).

### portable_div_nullif_not_zero_guard_expects

```rust
pub fn portable_div_nullif_not_zero_guard_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](#portable_div_nullif_not_zero_guard) cells.

### portable_div_nullif_not_zero_guard_mismatch

```rust
pub fn portable_div_nullif_not_zero_guard_mismatch(
    stmt: &StatementResult,
) -> Option<String>
```

Formats a mismatch for [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](#portable_div_nullif_not_zero_guard).

### portable_json_object_semantics_expects

```rust
pub fn portable_json_object_semantics_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_JSON_OBJECT_SEMANTICS`](#portable_json_object_semantics) cells.

### portable_json_object_semantics_mismatch

```rust
pub fn portable_json_object_semantics_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_JSON_OBJECT_SEMANTICS`](#portable_json_object_semantics).

### portable_text_prefix_literals_expects

```rust
pub fn portable_text_prefix_literals_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_TEXT_PREFIX_LITERALS`](#portable_text_prefix_literals) cells.

### portable_text_prefix_literals_mismatch

```rust
pub fn portable_text_prefix_literals_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_TEXT_PREFIX_LITERALS`](#portable_text_prefix_literals).

### portable_runtime_edges_mismatch

```rust
pub fn portable_runtime_edges_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_RUNTIME_EDGES`](#portable_runtime_edges).

### portable_min_max_null_mismatch

```rust
pub fn portable_min_max_null_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_MIN_MAX_NULL`](#portable_min_max_null).

### portable_order_nulls_asc_mismatch

```rust
pub fn portable_order_nulls_asc_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for ASC NULL ordering.

### portable_order_nulls_desc_mismatch

```rust
pub fn portable_order_nulls_desc_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for DESC NULL ordering.

### portable_uncast_round_mismatch

```rust
pub fn portable_uncast_round_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_UNCAST_ROUND`](#portable_uncast_round).

### portable_uncast_sum_avg_mismatch

```rust
pub fn portable_uncast_sum_avg_mismatch(stmt: &StatementResult) -> Option<String>
```

Formats a mismatch for [`PORTABLE_UNCAST_SUM_AVG`](#portable_uncast_sum_avg).

### portable_rows_mismatch

```rust
pub fn portable_rows_mismatch(
    stmt: &StatementResult,
    expect: &[&[PortableExpect]],
    label: &str,
) -> Option<String>
```

Compares every row in `stmt` to `expect`.

### portable_like_expects

```rust
pub fn portable_like_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_LIKE_SELECT`](#portable_like_select) cells (`c8` bind is `'A'`).

### portable_text_order_expects

```rust
pub fn portable_text_order_expects() -> &'static [&'static [PortableExpect]]
```

Expected [`PORTABLE_TEXT_ORDER_SELECT`](#portable_text_order_select) rows.

### portable_text_ops_expects

```rust
pub fn portable_text_ops_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_TEXT_OPS`](#portable_text_ops) cells.

### portable_text_multilingual_expects

```rust
pub fn portable_text_multilingual_expects() -> &'static [PortableExpect]
```

Expected [`PORTABLE_TEXT_MULTILINGUAL`](#portable_text_multilingual) cells (exact UTF-8, no NFC/NFD fold).

## Constants

### BINDING_DDL_AUTOINCREMENT_BLOB

```rust
pub const BINDING_DDL_AUTOINCREMENT_BLOB: &str = "CREATE TABLE IF NOT EXISTS typed (\
     id INTEGER PRIMARY KEY AUTOINCREMENT, \
     n INTEGER, body TEXT, payload TEXT, blob BLOB, r REAL)";
```

Binding `CREATE TABLE` covering identity + `BLOB`/`INTEGER`/`REAL`.

### BINDING_DDL_BOOLEAN

```rust
pub const BINDING_DDL_BOOLEAN: &str = "CREATE TABLE IF NOT EXISTS flags (\
     id INTEGER PRIMARY KEY AUTOINCREMENT, \
     flag BOOLEAN, \
     missing BOOLEAN)";
```

Binding `CREATE TABLE` covering canonical `BOOLEAN` (not INTEGER-as-bool).

### PORTABLE_BOOLEAN_INSERT

```rust
pub const PORTABLE_BOOLEAN_INSERT: &str = "INSERT INTO flags (flag, missing) VALUES (?, ?)";
```

INSERT `DbValue::Boolean` + typed-null into [`BINDING_DDL_BOOLEAN`](#binding_ddl_boolean).

### PORTABLE_BOOLEAN_SELECT

```rust
pub const PORTABLE_BOOLEAN_SELECT: &str = "SELECT flag, missing FROM flags";
```

SELECT declared boolean columns from [`BINDING_DDL_BOOLEAN`](#binding_ddl_boolean).

### BINDING_DDL_LOWERCASE

```rust
pub const BINDING_DDL_LOWERCASE: &str = "create table if not exists typed_lc (\
     id integer primary key autoincrement, \
     blob blob, flag boolean)";
```

Lowercase / mixed-case DDL admitted by SQL v1 (idents fold) and lowered
case-insensitively on Postgres.

### PORTABLE_INSERT_OR_IGNORE_RETURNING_LC

```rust
pub const PORTABLE_INSERT_OR_IGNORE_RETURNING_LC: &str = "insert or ignore into typed_lc (blob, flag) values (?, ?) returning id";
```

Lowercase `INSERT OR IGNORE … RETURNING` against [`BINDING_DDL_LOWERCASE`](#binding_ddl_lowercase).

### PORTABLE_LOWERCASE_SELECT

```rust
pub const PORTABLE_LOWERCASE_SELECT: &str = "SELECT blob, flag FROM typed_lc";
```

SELECT stored blob + boolean from [`BINDING_DDL_LOWERCASE`](#binding_ddl_lowercase).

### PORTABLE_INSERT

```rust
pub const PORTABLE_INSERT: &str = "INSERT INTO typed (n, body, payload, blob, r) VALUES (2, 'Ab', '{\"k\":\"v\"}', ?, 1.5)";
```

Seed row for [`PORTABLE_SELECT`](#portable_select). Bind [`PORTABLE_INSERT_BLOB`](#portable_insert_blob) at `?`.

### PORTABLE_INSERT_BLOB

```rust
pub const PORTABLE_INSERT_BLOB: &[u8] = _;
```

Blob payload for [`PORTABLE_INSERT`](#portable_insert).

### PORTABLE_LOWER_NON_ASCII_INSERT

```rust
pub const PORTABLE_LOWER_NON_ASCII_INSERT: &str = "INSERT INTO typed (n, body, payload, blob, r) VALUES (7, 'É', '{}', ?, 0)";
```

Inserts a non-ASCII `body` so `lower` can be compared with SQLite.

SQLite `lower` folds ASCII only, so `É` stays `É`. Postgres `lower` uses
the argument collation's `LC_CTYPE`. The typed lowering wraps the column
as `lower((body COLLATE "C"))`, which keeps `É`.

### PORTABLE_LOWER_NON_ASCII_SELECT

```rust
pub const PORTABLE_LOWER_NON_ASCII_SELECT: &str = "SELECT lower(body) FROM typed WHERE n = 7";
```

`lower` of the row inserted by [`PORTABLE_LOWER_NON_ASCII_INSERT`](#portable_lower_non_ascii_insert).

### PORTABLE_LOWER_NON_ASCII_EXPECT

```rust
pub const PORTABLE_LOWER_NON_ASCII_EXPECT: &str = "É";
```

SQLite's result for [`PORTABLE_LOWER_NON_ASCII_SELECT`](#portable_lower_non_ascii_select), and Postgres once
the argument is `COLLATE "C"`.

### PORTABLE_LOWER_PLACEHOLDER_SELECT

```rust
pub const PORTABLE_LOWER_PLACEHOLDER_SELECT: &str = "SELECT lower(?) FROM typed WHERE n = 7";
```

`lower` of a bound placeholder. Postgres must collate the parameter
(`lower(($1 COLLATE "C"))`) so `É` stays `É`, matching SQLite.

### PORTABLE_LOWER_SUBSTR_SELECT

```rust
pub const PORTABLE_LOWER_SUBSTR_SELECT: &str = "SELECT lower(substr(body, ?, ?)) FROM typed WHERE n = 7";
```

`lower(substr(body, ?, ?))` with integer binds.

The placeholders are the substr index and length, not text arguments of
`lower`. Postgres rejects `COLLATE` on bigint, so they stay bare. It also
rejects `substr(text, bigint, bigint)`, so lowering clamps those arguments
into `int4` and casts them to `INTEGER`. A length of `i64::MAX` still
means the rest of the string. The first character of the non-ASCII row is
still `É`.

### PORTABLE_SELECT

```rust
pub const PORTABLE_SELECT: &str = "SELECT \
     abs(-3) AS c0, \
     coalesce(NULL, 4) AS c1, \
     ifnull(NULL, 5) AS c2, \
     length(body) AS c3, \
     lower(body) AS c4, \
     upper(body) AS c5, \
     trim(' x ') AS c6, \
     substr(body, 1, 1) AS c7, \
     CAST(round(1.4) AS INTEGER) AS c8, \
     nullif(n, 9) AS c9, \
     min(1, 2) AS c10, \
     max(3, 1) AS c11, \
     json_extract(payload, '$.k') AS c12, \
     json_extract(json_object('k', 'v'), '$.k') AS c13, \
     json_valid(payload) AS c14, \
     json_valid(body) AS c15, \
     replace(body, 'A', 'Z') AS c16, \
     CAST(n AS INTEGER) AS c17 \
     FROM typed";
```

Scalar portable helpers (no aggregates).

Column order matches [`portable_select_expects`](#portable_select_expects). `json_object` is
sampled through `json_extract` so the cell is a scalar TEXT value rather
than a JSON document (spacing is engine-specific). Postgres lowering also
casts `json_build_object` to TEXT so `length(json_object(…))` is legal.
Aggregates live in [`PORTABLE_AGGREGATE_SELECT`]: a mixed aggregate +
non-aggregate SELECT list is SQLite-shaped, not Bookclerk SQL v1
(PostgreSQL requires `GROUP BY`). `round`/`sum`/`avg` are `CAST` to
`INTEGER` so cells stay in the universal `DbValue` domain (Postgres
`NUMERIC` is not a wire type).

### PORTABLE_AGGREGATE_SELECT

```rust
pub const PORTABLE_AGGREGATE_SELECT: &str = "SELECT \
     count(*) AS a0, \
     CAST(sum(n) AS INTEGER) AS a1, \
     min(n) AS a2, \
     max(n) AS a3, \
     CAST(avg(n) AS INTEGER) AS a4 \
     FROM typed";
```

Aggregate-only companion to [`PORTABLE_SELECT`](#portable_select).

### MIXED_GATE_LITERAL_DDL

```rust
pub const MIXED_GATE_LITERAL_DDL: &str = "CREATE TABLE IF NOT EXISTS gated_notes (id INTEGER PRIMARY KEY, body TEXT)";
```

Mixed DDL whose companion INSERT stores the receipt-gate text as a value.

### MIXED_GATE_LITERAL_SELECT

```rust
pub const MIXED_GATE_LITERAL_SELECT: &str = "SELECT body FROM gated_notes WHERE id = 1";
```

SELECT the stored gate literal from [`mixed_gate_literal_insert`](#mixed_gate_literal_insert).

### MIXED_GATE_LITERAL_COUNT

```rust
pub const MIXED_GATE_LITERAL_COUNT: &str = "SELECT count(*) FROM gated_notes";
```

COUNT helper for mixed-gate replay assertions.

### BINDING_DDL_CONFLICT

```rust
pub const BINDING_DDL_CONFLICT: &str = "CREATE TABLE IF NOT EXISTS conflicted (\
     id INTEGER PRIMARY KEY, \
     k TEXT NOT NULL UNIQUE, \
     v TEXT NOT NULL)";
```

Unique + NOT NULL table for `INSERT OR IGNORE` conflict-domain vectors.

### PORTABLE_INSERT_OR_IGNORE_UNIQUE

```rust
pub const PORTABLE_INSERT_OR_IGNORE_UNIQUE: &str = "INSERT OR IGNORE INTO conflicted (id, k, v) VALUES (1, 'a', 'x')";
```

Unique-conflict ignore (second insert is a no-op).

### PORTABLE_INSERT_OR_IGNORE_NOT_NULL

```rust
pub const PORTABLE_INSERT_OR_IGNORE_NOT_NULL: &str = "INSERT OR IGNORE INTO conflicted (id, k, v) VALUES (2, 'b', NULL)";
```

NOT NULL ignore must still error (not swallowed).

### PORTABLE_MIN_MAX_NULL

```rust
pub const PORTABLE_MIN_MAX_NULL: &str = "SELECT max(NULL, 3) AS c0, min(NULL, 3) AS c1, max(1, 3) AS c2";
```

Scalar min/max NULL-poison (any NULL argument → NULL).

### BINDING_DDL_ORDER_NULLS

```rust
pub const BINDING_DDL_ORDER_NULLS: &str = "CREATE TABLE IF NOT EXISTS ordered_n (n INTEGER)";
```

Nullable column for ORDER BY NULL ordering.

### PORTABLE_ORDER_NULLS_INSERT_1

```rust
pub const PORTABLE_ORDER_NULLS_INSERT_1: &str = "INSERT INTO ordered_n (n) VALUES (1)";
```

Seed three rows: 1, NULL, 2.

### PORTABLE_ORDER_NULLS_INSERT_NULL

```rust
pub const PORTABLE_ORDER_NULLS_INSERT_NULL: &str = "INSERT INTO ordered_n (n) VALUES (NULL)";
```

NULL seed.

### PORTABLE_ORDER_NULLS_INSERT_2

```rust
pub const PORTABLE_ORDER_NULLS_INSERT_2: &str = "INSERT INTO ordered_n (n) VALUES (2)";
```

2 seed.

### PORTABLE_ORDER_NULLS_ASC

```rust
pub const PORTABLE_ORDER_NULLS_ASC: &str = "SELECT n FROM ordered_n ORDER BY n ASC";
```

ASC: NULL sorts as smallest.

### PORTABLE_ORDER_NULLS_DESC

```rust
pub const PORTABLE_ORDER_NULLS_DESC: &str = "SELECT n FROM ordered_n ORDER BY n DESC";
```

DESC: NULL sorts as smallest (last).

### BINDING_DDL_IDENTITY

```rust
pub const BINDING_DDL_IDENTITY: &str = "CREATE TABLE IF NOT EXISTS ident (id INTEGER PRIMARY KEY AUTOINCREMENT, n INTEGER)";
```

Identity table (explicit id then omit-id).

### PORTABLE_IDENTITY_INSERT_EXPLICIT

```rust
pub const PORTABLE_IDENTITY_INSERT_EXPLICIT: &str = "INSERT INTO ident (id, n) VALUES (100, 1)";
```

Explicit id 100.

### PORTABLE_IDENTITY_INSERT_OMIT

```rust
pub const PORTABLE_IDENTITY_INSERT_OMIT: &str = "INSERT INTO ident (n) VALUES (2)";
```

Omit id (must yield 101 after explicit 100).

### PORTABLE_IDENTITY_SELECT_MAX

```rust
pub const PORTABLE_IDENTITY_SELECT_MAX: &str = "SELECT max(id) FROM ident";
```

Largest id.

### PORTABLE_IDENTITY_DELETE_MAX

```rust
pub const PORTABLE_IDENTITY_DELETE_MAX: &str = "DELETE FROM ident WHERE id = (SELECT max(id) FROM ident)";
```

Delete the max identity row (SQLite AUTOINCREMENT must not reuse it).

### BINDING_DDL_UNQUOTED_FOLD

```rust
pub const BINDING_DDL_UNQUOTED_FOLD: &str = "CREATE TABLE IF NOT EXISTS FoldMe (id INTEGER PRIMARY KEY, n INTEGER)";
```

Unquoted mixed-case table that folds to lowercase.

### PORTABLE_UNQUOTED_FOLD_INSERT

```rust
pub const PORTABLE_UNQUOTED_FOLD_INSERT: &str = "INSERT INTO FoldMe (id, n) VALUES (1, 7)";
```

SELECT via folded lowercase name.

### PORTABLE_UNQUOTED_FOLD_SELECT

```rust
pub const PORTABLE_UNQUOTED_FOLD_SELECT: &str = "SELECT n FROM foldme";
```

SELECT from folded name.

### PORTABLE_UNCAST_ROUND

```rust
pub const PORTABLE_UNCAST_ROUND: &str = "SELECT round(r, 2) AS r0 FROM typed";
```

Uncast `round` (per-row; not mixed with aggregates).

### PORTABLE_UNCAST_SUM_AVG

```rust
pub const PORTABLE_UNCAST_SUM_AVG: &str = "SELECT sum(n) AS s, avg(n) AS a FROM typed";
```

Uncast integer `sum` / `avg` (aggregate-only companion).

### PORTABLE_RUNTIME_EDGES

```rust
pub const PORTABLE_RUNTIME_EDGES: &str = "SELECT \
     1 / 0 AS d0, \
     4 % 0 AS m0, \
     CAST(round(1.5) AS INTEGER) AS r0, \
     CAST(round(2.5) AS INTEGER) AS r1, \
     substr('hello', 2, 2) AS s0, \
     length('hi') AS ln, \
     replace('aa', 'a', 'b') AS rp, \
     abs(-3) AS ab, \
     lower('Hi') AS lo";
```

Runtime-edge operators/helpers (`/` `%` zero → NULL, `round` halfway, `substr`, `length`, `replace`, `abs`, `lower`).

### PORTABLE_INTEGER_OVERFLOW

```rust
pub const PORTABLE_INTEGER_OVERFLOW: &str = "SELECT \
     9223372036854775807 + 1 AS o0, \
     (-9223372036854775807 - 1) - 1 AS o1, \
     9223372036854775807 * 2 AS o2, \
     abs(-9223372036854775807 - 1) AS o3";
```

INTEGER overflow (`+` `-` `*` `abs(i64::MIN)`) yields NULL on every backend.

### PORTABLE_NESTED_INTEGER_ARITH

```rust
pub const PORTABLE_NESTED_INTEGER_ARITH: &str = "SELECT \
     1 + abs(-2) AS n0, \
     1 + 2 * 3 AS n1, \
     1 + abs(-9223372036854775807 - 1) AS n2, \
     length('ä') + 1 AS n3";
```

Nested INTEGER `+`/`*`/`abs` plus TEXT-inside-overflow (`length('ä') + 1`).

### PORTABLE_JSON_PATH_LIKE_COMPARE

```rust
pub const PORTABLE_JSON_PATH_LIKE_COMPARE: &str = "SELECT CASE WHEN '$.ä' < 'a' THEN 1 ELSE 0 END AS c0";
```

`'$.…'` compared as TEXT (not a `json_extract` path) — Postgres must COLLATE.

### PORTABLE_DIV_OPERANDS

```rust
pub const PORTABLE_DIV_OPERANDS: &str = "SELECT \
     10 / abs(0) AS d0, \
     10 / -0 AS d1, \
     10 / CAST(0 AS INTEGER) AS d2, \
     10 / (0) AS d3";
```

`/` RHS covering call, unary, CAST, and parenthesized zero.

### PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD

```rust
pub const PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD: &str = "SELECT \
     1 / NULLIF(0, 1) AS d0, \
     1 % NULLIF(0, 1) AS m0, \
     10 / NULLIF(2, 0) AS already";
```

`/` and `%` of a `NULLIF` that is **not** `NULLIF(expr, 0)` must still
gain an outer zero guard (`NULLIF(0, 1)` evaluates to `0`).

### PORTABLE_JSON_OBJECT_SEMANTICS

```rust
pub const PORTABLE_JSON_OBJECT_SEMANTICS: &str = "SELECT \
     CASE WHEN length(json_object('a', 1, 'b', NULL)) > length(json_object('a', 1)) \
          THEN 1 ELSE 0 END AS null_retained, \
     CASE WHEN length(json_object('a', 1, 'a', NULL)) > 2 THEN 1 ELSE 0 END AS dup_null_retained, \
     json_extract(json_extract(json_object('n', json_object('k', 'v')), '$.n'), '$.k') AS nested, \
     json_extract(json_object(\
        'k00', 'v00', 'k01', 'v01', 'k02', 'v02', 'k03', 'v03', \
        'k04', 'v04', 'k05', 'v05', 'k06', 'v06', 'k07', 'v07', \
        'k08', 'v08', 'k09', 'v09', 'k10', 'v10', 'k11', 'v11', \
        'k12', 'v12', 'k13', 'v13', 'k14', 'v14', 'k15', 'v15'\
     ), '$.k15') AS max_arity";
```

`json_object` nulls, duplicate keys, nested objects, and 16-pair (32-arg) arity.

Length comparisons distinguish JSON `null` retention from merge-patch
deletion without depending on exact JSON spacing. Postgres lowering casts
`json_build_object` to TEXT so `length(json_object(…))` is legal.

### PORTABLE_TEXT_PREFIX_LITERALS

```rust
pub const PORTABLE_TEXT_PREFIX_LITERALS: &str = "SELECT '$ä' AS a, '{ä' AS b, 'COLLATEä' AS c";
```

TEXT literals whose contents look like bind placeholders or `COLLATE`.

### BINDING_DDL_IGNORE_SELECT

```rust
pub const BINDING_DDL_IGNORE_SELECT: &str = "CREATE TABLE IF NOT EXISTS ign_sel (id INTEGER PRIMARY KEY)";
```

`INSERT OR IGNORE … SELECT` (plain bind source).

### PORTABLE_IGNORE_SELECT

```rust
pub const PORTABLE_IGNORE_SELECT: &str = "INSERT OR IGNORE INTO ign_sel (id) SELECT ? RETURNING id";
```

Plain `INSERT OR IGNORE … SELECT ? RETURNING id`.

### PORTABLE_IGNORE_SELECT_WITH

```rust
pub const PORTABLE_IGNORE_SELECT_WITH: &str = "INSERT OR IGNORE INTO ign_sel (id) WITH s(id) AS (SELECT ?) SELECT * FROM s RETURNING id";
```

`WITH` source for `INSERT OR IGNORE`.

### PORTABLE_IGNORE_SELECT_UNION

```rust
pub const PORTABLE_IGNORE_SELECT_UNION: &str = "INSERT OR IGNORE INTO ign_sel (id) SELECT ? UNION ALL SELECT ? RETURNING id";
```

Compound `UNION ALL` source.

### PORTABLE_IGNORE_SELECT_ORDER_LIMIT

```rust
pub const PORTABLE_IGNORE_SELECT_ORDER_LIMIT: &str = "INSERT OR IGNORE INTO ign_sel (id) SELECT ? AS id ORDER BY id LIMIT 1 RETURNING id";
```

`ORDER BY` + `LIMIT` source.

### BINDING_DDL_LIKE

```rust
pub const BINDING_DDL_LIKE: &str = "CREATE TABLE IF NOT EXISTS liked (body TEXT)";
```

Case-sensitive `LIKE` table.

### PORTABLE_LIKE_INSERT

```rust
pub const PORTABLE_LIKE_INSERT: &str = "INSERT INTO liked (body) VALUES ('A')";
```

Seed `'A'` for LIKE vectors.

### PORTABLE_LIKE_SELECT

```rust
pub const PORTABLE_LIKE_SELECT: &str = "SELECT \
     CASE WHEN 'A' LIKE 'a' THEN 1 ELSE 0 END AS c0, \
     CASE WHEN 'A' LIKE 'A' THEN 1 ELSE 0 END AS c1, \
     CASE WHEN 'A' LIKE 'A%' THEN 1 ELSE 0 END AS c2, \
     CASE WHEN 'A' LIKE 'A*' THEN 1 ELSE 0 END AS c3, \
     CASE WHEN 'A' LIKE 'A?' THEN 1 ELSE 0 END AS c4, \
     CASE WHEN 'A' LIKE 'A[' THEN 1 ELSE 0 END AS c5, \
     CASE WHEN 'ab' LIKE 'a_b' THEN 1 ELSE 0 END AS c6, \
     CASE WHEN 'A' LIKE NULL THEN 1 ELSE 0 END AS c7, \
     CASE WHEN body LIKE ? THEN 1 ELSE 0 END AS c8 \
     FROM liked";
```

Case-sensitive LIKE probes (A/a, prefix, GLOB metacharacters, NULL).

### PORTABLE_LIKE_NON_ASCII

```rust
pub const PORTABLE_LIKE_NON_ASCII: &str = "SELECT CASE WHEN 'İ' LIKE 'i' THEN 1 ELSE 0 END AS c0";
```

Non-ASCII that must not Unicode-fold under LIKE.

### BINDING_DDL_BLOB_DEFAULT

```rust
pub const BINDING_DDL_BLOB_DEFAULT: &str = "CREATE TABLE IF NOT EXISTS blobdef (id INTEGER PRIMARY KEY, payload BLOB DEFAULT X'deadbeef')";
```

Typed BLOB default hex.

### PORTABLE_BLOB_DEFAULT_INSERT

```rust
pub const PORTABLE_BLOB_DEFAULT_INSERT: &str = "INSERT INTO blobdef (id) VALUES (1)";
```

INSERT using the BLOB default.

### PORTABLE_BLOB_DEFAULT_SELECT

```rust
pub const PORTABLE_BLOB_DEFAULT_SELECT: &str = "SELECT payload FROM blobdef WHERE id = 1";
```

Read the default blob.

### BINDING_DDL_TEXT_ORDER

```rust
pub const BINDING_DDL_TEXT_ORDER: &str = "CREATE TABLE IF NOT EXISTS textord (body TEXT)";
```

TEXT collation / order table.

### PORTABLE_TEXT_ORDER_INSERT_B

```rust
pub const PORTABLE_TEXT_ORDER_INSERT_B: &str = "INSERT INTO textord (body) VALUES ('B')";
```

Seed ASCII + non-ASCII rows.

### PORTABLE_TEXT_ORDER_INSERT_A

```rust
pub const PORTABLE_TEXT_ORDER_INSERT_A: &str = "INSERT INTO textord (body) VALUES ('a')";
```

Lowercase a.

### PORTABLE_TEXT_ORDER_INSERT_EACUTE

```rust
pub const PORTABLE_TEXT_ORDER_INSERT_EACUTE: &str = "INSERT INTO textord (body) VALUES ('é')";
```

Non-ASCII é (U+00E9).

### PORTABLE_TEXT_ORDER_INSERT_E

```rust
pub const PORTABLE_TEXT_ORDER_INSERT_E: &str = "INSERT INTO textord (body) VALUES ('e')";
```

ASCII e.

### PORTABLE_TEXT_ORDER_SELECT

```rust
pub const PORTABLE_TEXT_ORDER_SELECT: &str = "SELECT body FROM textord ORDER BY body";
```

Binary TEXT order (byte/code-point, not locale).

### PORTABLE_TEXT_OPS

```rust
pub const PORTABLE_TEXT_OPS: &str = "SELECT \
     CASE WHEN 'A' < 'a' THEN 1 ELSE 0 END AS c0, \
     CASE WHEN 'A' > 'a' THEN 1 ELSE 0 END AS c1, \
     min('A', 'a') AS c2, \
     max('A', 'a') AS c3, \
     lower('Ab') AS c4, \
     upper('Ab') AS c5";
```

TEXT comparison + scalar min/max + lower/upper.

### PORTABLE_TEXT_MULTILINGUAL

```rust
pub const PORTABLE_TEXT_MULTILINGUAL: &str = "SELECT \
     '汉语' AS cjk, \
     '日本語' AS ja, \
     '한국어' AS ko, \
     '𠀀' AS han_ext, \
     'مرحبا' AS ar, \
     'שלום' AS he, \
     'नमस्ते' AS hi, \
     'สวัสดี' AS th, \
     'γειά' AS el, \
     'привет' AS ru, \
     '🎵🦀' AS emoji, \
     'café' AS nfc, \
     'cafe\u{0301}' AS nfd";
```

Full Unicode scalar TEXT (UTF-8), including supplementary-plane Han, RTL,
combining sequences, and NFC vs NFD that must not be silently normalized.

### PORTABLE_IDENTITY_INSERT_BAD_TYPE

```rust
pub const PORTABLE_IDENTITY_INSERT_BAD_TYPE: &str = "INSERT INTO ident (id, n) VALUES (100, 'x')";
```

Identity omit after rollback of an explicit id (same atomic request fails).

### PORTABLE_IDENTITY_DROP

```rust
pub const PORTABLE_IDENTITY_DROP: &str = "DROP TABLE IF EXISTS ident";
```

DROP identity table.
