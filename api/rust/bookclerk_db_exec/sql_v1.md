**bookclerk_db_exec > sql_v1**

# Module: sql_v1

## Contents

**Enums**

- [`PortableExpect`](#portableexpect) - Expected cells for [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select) (adapter-neutral).

**Functions**

- [`mixed_gate_literal_insert`](#mixed_gate_literal_insert) - INSERT that embeds [`crate::GUEST_RECEIPT_WRITE_GATE`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_write_gate) in a comment and a string.
- [`portable_aggregate_expects`](#portable_aggregate_expects) - Expected [`PORTABLE_AGGREGATE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_aggregate_select) cells in column order.
- [`portable_aggregate_mismatch`](#portable_aggregate_mismatch) - Formats a mismatch for [`PORTABLE_AGGREGATE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_aggregate_select).
- [`portable_boolean_expects`](#portable_boolean_expects) - Expected [`PORTABLE_BOOLEAN_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_boolean_select) cells in column order.
- [`portable_boolean_insert_binds`](#portable_boolean_insert_binds) - Binds for [`PORTABLE_BOOLEAN_INSERT`](sql_v1.md#bookclerk_db_execsql_v1portable_boolean_insert).
- [`portable_boolean_mismatch`](#portable_boolean_mismatch) - Formats a mismatch for [`PORTABLE_BOOLEAN_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_boolean_select).
- [`portable_div_nullif_not_zero_guard_expects`](#portable_div_nullif_not_zero_guard_expects) - Expected [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](sql_v1.md#bookclerk_db_execsql_v1portable_div_nullif_not_zero_guard) cells.
- [`portable_div_nullif_not_zero_guard_mismatch`](#portable_div_nullif_not_zero_guard_mismatch) - Formats a mismatch for [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](sql_v1.md#bookclerk_db_execsql_v1portable_div_nullif_not_zero_guard).
- [`portable_div_operands_expects`](#portable_div_operands_expects) - Expected [`PORTABLE_DIV_OPERANDS`](sql_v1.md#bookclerk_db_execsql_v1portable_div_operands) cells.
- [`portable_div_operands_mismatch`](#portable_div_operands_mismatch) - Formats a mismatch for [`PORTABLE_DIV_OPERANDS`](sql_v1.md#bookclerk_db_execsql_v1portable_div_operands).
- [`portable_integer_overflow_expects`](#portable_integer_overflow_expects) - Expected [`PORTABLE_INTEGER_OVERFLOW`](sql_v1.md#bookclerk_db_execsql_v1portable_integer_overflow) cells.
- [`portable_integer_overflow_mismatch`](#portable_integer_overflow_mismatch) - Formats a mismatch for [`PORTABLE_INTEGER_OVERFLOW`](sql_v1.md#bookclerk_db_execsql_v1portable_integer_overflow).
- [`portable_json_object_semantics_expects`](#portable_json_object_semantics_expects) - Expected [`PORTABLE_JSON_OBJECT_SEMANTICS`](sql_v1.md#bookclerk_db_execsql_v1portable_json_object_semantics) cells.
- [`portable_json_object_semantics_mismatch`](#portable_json_object_semantics_mismatch) - Formats a mismatch for [`PORTABLE_JSON_OBJECT_SEMANTICS`](sql_v1.md#bookclerk_db_execsql_v1portable_json_object_semantics).
- [`portable_json_path_like_compare_expects`](#portable_json_path_like_compare_expects) - Expected [`PORTABLE_JSON_PATH_LIKE_COMPARE`](sql_v1.md#bookclerk_db_execsql_v1portable_json_path_like_compare) cells.
- [`portable_json_path_like_compare_mismatch`](#portable_json_path_like_compare_mismatch) - Formats a mismatch for [`PORTABLE_JSON_PATH_LIKE_COMPARE`](sql_v1.md#bookclerk_db_execsql_v1portable_json_path_like_compare).
- [`portable_like_expects`](#portable_like_expects) - Expected [`PORTABLE_LIKE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_like_select) cells (`c8` bind is `'A'`).
- [`portable_lowercase_insert_binds`](#portable_lowercase_insert_binds) - Binds for [`PORTABLE_INSERT_OR_IGNORE_RETURNING_LC`](sql_v1.md#bookclerk_db_execsql_v1portable_insert_or_ignore_returning_lc).
- [`portable_min_max_null_expects`](#portable_min_max_null_expects) - Expected [`PORTABLE_MIN_MAX_NULL`](sql_v1.md#bookclerk_db_execsql_v1portable_min_max_null) cells.
- [`portable_min_max_null_mismatch`](#portable_min_max_null_mismatch) - Formats a mismatch for [`PORTABLE_MIN_MAX_NULL`](sql_v1.md#bookclerk_db_execsql_v1portable_min_max_null).
- [`portable_nested_integer_arith_expects`](#portable_nested_integer_arith_expects) - Expected [`PORTABLE_NESTED_INTEGER_ARITH`](sql_v1.md#bookclerk_db_execsql_v1portable_nested_integer_arith) cells.
- [`portable_nested_integer_arith_mismatch`](#portable_nested_integer_arith_mismatch) - Formats a mismatch for [`PORTABLE_NESTED_INTEGER_ARITH`](sql_v1.md#bookclerk_db_execsql_v1portable_nested_integer_arith).
- [`portable_order_nulls_asc_expects`](#portable_order_nulls_asc_expects) - Expected [`PORTABLE_ORDER_NULLS_ASC`](sql_v1.md#bookclerk_db_execsql_v1portable_order_nulls_asc) cells.
- [`portable_order_nulls_asc_mismatch`](#portable_order_nulls_asc_mismatch) - Formats a mismatch for ASC NULL ordering.
- [`portable_order_nulls_desc_expects`](#portable_order_nulls_desc_expects) - Expected [`PORTABLE_ORDER_NULLS_DESC`](sql_v1.md#bookclerk_db_execsql_v1portable_order_nulls_desc) cells.
- [`portable_order_nulls_desc_mismatch`](#portable_order_nulls_desc_mismatch) - Formats a mismatch for DESC NULL ordering.
- [`portable_rows_mismatch`](#portable_rows_mismatch) - Compares every row in `stmt` to `expect`.
- [`portable_runtime_edges_expects`](#portable_runtime_edges_expects) - Expected [`PORTABLE_RUNTIME_EDGES`](sql_v1.md#bookclerk_db_execsql_v1portable_runtime_edges) cells.
- [`portable_runtime_edges_mismatch`](#portable_runtime_edges_mismatch) - Formats a mismatch for [`PORTABLE_RUNTIME_EDGES`](sql_v1.md#bookclerk_db_execsql_v1portable_runtime_edges).
- [`portable_select_expects`](#portable_select_expects) - Expected [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select) cells in column order.
- [`portable_select_mismatch`](#portable_select_mismatch) - Formats a mismatch for adapter execution tests.
- [`portable_statement_mismatch`](#portable_statement_mismatch) - Compares `stmt` to `expect` and describes the first mismatch.
- [`portable_text_multilingual_expects`](#portable_text_multilingual_expects) - Expected [`PORTABLE_TEXT_MULTILINGUAL`](sql_v1.md#bookclerk_db_execsql_v1portable_text_multilingual) cells (exact UTF-8, no NFC/NFD fold).
- [`portable_text_ops_expects`](#portable_text_ops_expects) - Expected [`PORTABLE_TEXT_OPS`](sql_v1.md#bookclerk_db_execsql_v1portable_text_ops) cells.
- [`portable_text_order_expects`](#portable_text_order_expects) - Expected [`PORTABLE_TEXT_ORDER_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_text_order_select) rows.
- [`portable_text_prefix_literals_expects`](#portable_text_prefix_literals_expects) - Expected [`PORTABLE_TEXT_PREFIX_LITERALS`](sql_v1.md#bookclerk_db_execsql_v1portable_text_prefix_literals) cells.
- [`portable_text_prefix_literals_mismatch`](#portable_text_prefix_literals_mismatch) - Formats a mismatch for [`PORTABLE_TEXT_PREFIX_LITERALS`](sql_v1.md#bookclerk_db_execsql_v1portable_text_prefix_literals).
- [`portable_uncast_round_expects`](#portable_uncast_round_expects) - Expected [`PORTABLE_UNCAST_ROUND`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_round) cells (r=1.5).
- [`portable_uncast_round_mismatch`](#portable_uncast_round_mismatch) - Formats a mismatch for [`PORTABLE_UNCAST_ROUND`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_round).
- [`portable_uncast_sum_avg_expects`](#portable_uncast_sum_avg_expects) - Expected [`PORTABLE_UNCAST_SUM_AVG`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_sum_avg) cells (n=2).
- [`portable_uncast_sum_avg_mismatch`](#portable_uncast_sum_avg_mismatch) - Formats a mismatch for [`PORTABLE_UNCAST_SUM_AVG`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_sum_avg).
- [`portable_value_matches`](#portable_value_matches) - True when `got` is the exact [`DbValue`](../bookclerk_plugin_abi/db_value.md#bookclerk_plugin_abidb_valuedbvalue) for `want`.

**Constants**

- [`BINDING_DDL_AUTOINCREMENT_BLOB`](#binding_ddl_autoincrement_blob) - Binding `CREATE TABLE` covering identity + `BLOB`/`INTEGER`/`REAL`.
- [`BINDING_DDL_BLOB_DEFAULT`](#binding_ddl_blob_default) - Typed BLOB default hex.
- [`BINDING_DDL_BOOLEAN`](#binding_ddl_boolean) - Binding `CREATE TABLE` covering canonical `BOOLEAN` (not INTEGER-as-bool).
- [`BINDING_DDL_CONFLICT`](#binding_ddl_conflict) - Unique + NOT NULL table for `INSERT OR IGNORE` conflict-domain vectors.
- [`BINDING_DDL_IDENTITY`](#binding_ddl_identity) - Identity table (explicit id then omit-id).
- [`BINDING_DDL_IGNORE_SELECT`](#binding_ddl_ignore_select) - `INSERT OR IGNORE … SELECT` (plain bind source).
- [`BINDING_DDL_LIKE`](#binding_ddl_like) - Case-sensitive `LIKE` table.
- [`BINDING_DDL_LOWERCASE`](#binding_ddl_lowercase) - Lowercase / mixed-case DDL admitted by SQL v1 (idents fold) and lowered
- [`BINDING_DDL_ORDER_NULLS`](#binding_ddl_order_nulls) - Nullable column for ORDER BY NULL ordering.
- [`BINDING_DDL_TEXT_ORDER`](#binding_ddl_text_order) - TEXT collation / order table.
- [`BINDING_DDL_UNQUOTED_FOLD`](#binding_ddl_unquoted_fold) - Unquoted mixed-case table that folds to lowercase.
- [`MIXED_GATE_LITERAL_COUNT`](#mixed_gate_literal_count) - COUNT helper for mixed-gate replay assertions.
- [`MIXED_GATE_LITERAL_DDL`](#mixed_gate_literal_ddl) - Mixed DDL whose companion INSERT stores the receipt-gate text as a value.
- [`MIXED_GATE_LITERAL_SELECT`](#mixed_gate_literal_select) - SELECT the stored gate literal from [`mixed_gate_literal_insert`](sql_v1.md#bookclerk_db_execsql_v1mixed_gate_literal_insert).
- [`PORTABLE_AGGREGATE_SELECT`](#portable_aggregate_select) - Aggregate-only companion to [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select).
- [`PORTABLE_BLOB_DEFAULT_INSERT`](#portable_blob_default_insert) - INSERT using the BLOB default.
- [`PORTABLE_BLOB_DEFAULT_SELECT`](#portable_blob_default_select) - Read the default blob.
- [`PORTABLE_BOOLEAN_INSERT`](#portable_boolean_insert) - INSERT `DbValue::Boolean` + typed-null into [`BINDING_DDL_BOOLEAN`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_boolean).
- [`PORTABLE_BOOLEAN_SELECT`](#portable_boolean_select) - SELECT declared boolean columns from [`BINDING_DDL_BOOLEAN`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_boolean).
- [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](#portable_div_nullif_not_zero_guard) - `/` and `%` of a `NULLIF` that is **not** `NULLIF(expr, 0)` must still
- [`PORTABLE_DIV_OPERANDS`](#portable_div_operands) - `/` RHS covering call, unary, CAST, and parenthesized zero.
- [`PORTABLE_IDENTITY_DELETE_MAX`](#portable_identity_delete_max) - Delete the max identity row (SQLite AUTOINCREMENT must not reuse it).
- [`PORTABLE_IDENTITY_DROP`](#portable_identity_drop) - DROP identity table.
- [`PORTABLE_IDENTITY_INSERT_BAD_TYPE`](#portable_identity_insert_bad_type) - Identity omit after rollback of an explicit id (same atomic request fails).
- [`PORTABLE_IDENTITY_INSERT_EXPLICIT`](#portable_identity_insert_explicit) - Explicit id 100.
- [`PORTABLE_IDENTITY_INSERT_OMIT`](#portable_identity_insert_omit) - Omit id (must yield 101 after explicit 100).
- [`PORTABLE_IDENTITY_SELECT_MAX`](#portable_identity_select_max) - Largest id.
- [`PORTABLE_IGNORE_SELECT`](#portable_ignore_select) - Plain `INSERT OR IGNORE … SELECT ? RETURNING id`.
- [`PORTABLE_IGNORE_SELECT_ORDER_LIMIT`](#portable_ignore_select_order_limit) - `ORDER BY` + `LIMIT` source.
- [`PORTABLE_IGNORE_SELECT_UNION`](#portable_ignore_select_union) - Compound `UNION ALL` source.
- [`PORTABLE_IGNORE_SELECT_WITH`](#portable_ignore_select_with) - `WITH` source for `INSERT OR IGNORE`.
- [`PORTABLE_INSERT`](#portable_insert) - Seed row for [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select). Bind [`PORTABLE_INSERT_BLOB`](sql_v1.md#bookclerk_db_execsql_v1portable_insert_blob) at `?`.
- [`PORTABLE_INSERT_BLOB`](#portable_insert_blob) - Blob payload for [`PORTABLE_INSERT`](sql_v1.md#bookclerk_db_execsql_v1portable_insert).
- [`PORTABLE_INSERT_OR_IGNORE_NOT_NULL`](#portable_insert_or_ignore_not_null) - NOT NULL ignore must still error (not swallowed).
- [`PORTABLE_INSERT_OR_IGNORE_RETURNING_LC`](#portable_insert_or_ignore_returning_lc) - Lowercase `INSERT OR IGNORE … RETURNING` against [`BINDING_DDL_LOWERCASE`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_lowercase).
- [`PORTABLE_INSERT_OR_IGNORE_UNIQUE`](#portable_insert_or_ignore_unique) - Unique-conflict ignore (second insert is a no-op).
- [`PORTABLE_INTEGER_OVERFLOW`](#portable_integer_overflow) - INTEGER overflow (`+` `-` `*` `abs(i64::MIN)`) yields NULL on every backend.
- [`PORTABLE_JSON_OBJECT_SEMANTICS`](#portable_json_object_semantics) - `json_object` nulls, duplicate keys, nested objects, and 16-pair (32-arg) arity.
- [`PORTABLE_JSON_PATH_LIKE_COMPARE`](#portable_json_path_like_compare) - `'$.…'` compared as TEXT (not a `json_extract` path) — Postgres must COLLATE.
- [`PORTABLE_LIKE_INSERT`](#portable_like_insert) - Seed `'A'` for LIKE vectors.
- [`PORTABLE_LIKE_NON_ASCII`](#portable_like_non_ascii) - Non-ASCII that must not Unicode-fold under LIKE.
- [`PORTABLE_LIKE_SELECT`](#portable_like_select) - Case-sensitive LIKE probes (A/a, prefix, GLOB metacharacters, NULL).
- [`PORTABLE_LOWERCASE_SELECT`](#portable_lowercase_select) - SELECT stored blob + boolean from [`BINDING_DDL_LOWERCASE`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_lowercase).
- [`PORTABLE_LOWER_NON_ASCII_EXPECT`](#portable_lower_non_ascii_expect) - SQLite's result for [`PORTABLE_LOWER_NON_ASCII_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_lower_non_ascii_select), and Postgres once
- [`PORTABLE_LOWER_NON_ASCII_INSERT`](#portable_lower_non_ascii_insert) - Inserts a non-ASCII `body` so `lower` can be compared with SQLite.
- [`PORTABLE_LOWER_NON_ASCII_SELECT`](#portable_lower_non_ascii_select) - `lower` of the row inserted by [`PORTABLE_LOWER_NON_ASCII_INSERT`](sql_v1.md#bookclerk_db_execsql_v1portable_lower_non_ascii_insert).
- [`PORTABLE_LOWER_PLACEHOLDER_SELECT`](#portable_lower_placeholder_select) - `lower` of a bound placeholder. Postgres must collate the parameter
- [`PORTABLE_LOWER_SUBSTR_SELECT`](#portable_lower_substr_select) - `lower(substr(body, ?, ?))` with integer binds.
- [`PORTABLE_MIN_MAX_NULL`](#portable_min_max_null) - Scalar min/max NULL-poison (any NULL argument → NULL).
- [`PORTABLE_NESTED_INTEGER_ARITH`](#portable_nested_integer_arith) - Nested INTEGER `+`/`*`/`abs` plus TEXT-inside-overflow (`length('ä') + 1`).
- [`PORTABLE_ORDER_NULLS_ASC`](#portable_order_nulls_asc) - ASC: NULL sorts as smallest.
- [`PORTABLE_ORDER_NULLS_DESC`](#portable_order_nulls_desc) - DESC: NULL sorts as smallest (last).
- [`PORTABLE_ORDER_NULLS_INSERT_1`](#portable_order_nulls_insert_1) - Seed three rows: 1, NULL, 2.
- [`PORTABLE_ORDER_NULLS_INSERT_2`](#portable_order_nulls_insert_2) - 2 seed.
- [`PORTABLE_ORDER_NULLS_INSERT_NULL`](#portable_order_nulls_insert_null) - NULL seed.
- [`PORTABLE_RUNTIME_EDGES`](#portable_runtime_edges) - Runtime-edge operators/helpers (`/` `%` zero → NULL, `round` halfway, `substr`, `length`, `replace`, `abs`, `lower`).
- [`PORTABLE_SELECT`](#portable_select) - Scalar portable helpers (no aggregates).
- [`PORTABLE_TEXT_MULTILINGUAL`](#portable_text_multilingual) - Full Unicode scalar TEXT (UTF-8), including supplementary-plane Han, RTL,
- [`PORTABLE_TEXT_OPS`](#portable_text_ops) - TEXT comparison + scalar min/max + lower/upper.
- [`PORTABLE_TEXT_ORDER_INSERT_A`](#portable_text_order_insert_a) - Lowercase a.
- [`PORTABLE_TEXT_ORDER_INSERT_B`](#portable_text_order_insert_b) - Seed ASCII + non-ASCII rows.
- [`PORTABLE_TEXT_ORDER_INSERT_E`](#portable_text_order_insert_e) - ASCII e.
- [`PORTABLE_TEXT_ORDER_INSERT_EACUTE`](#portable_text_order_insert_eacute) - Non-ASCII é (U+00E9).
- [`PORTABLE_TEXT_ORDER_SELECT`](#portable_text_order_select) - Binary TEXT order (byte/code-point, not locale).
- [`PORTABLE_TEXT_PREFIX_LITERALS`](#portable_text_prefix_literals) - TEXT literals whose contents look like bind placeholders or `COLLATE`.
- [`PORTABLE_UNCAST_ROUND`](#portable_uncast_round) - Uncast `round` (per-row; not mixed with aggregates).
- [`PORTABLE_UNCAST_SUM_AVG`](#portable_uncast_sum_avg) - Uncast integer `sum` / `avg` (aggregate-only companion).
- [`PORTABLE_UNQUOTED_FOLD_INSERT`](#portable_unquoted_fold_insert) - SELECT via folded lowercase name.
- [`PORTABLE_UNQUOTED_FOLD_SELECT`](#portable_unquoted_fold_select) - SELECT from folded name.

---

## bookclerk_db_exec::sql_v1::BINDING_DDL_AUTOINCREMENT_BLOB

*Constant*: `&str`

Binding `CREATE TABLE` covering identity + `BLOB`/`INTEGER`/`REAL`.



## bookclerk_db_exec::sql_v1::BINDING_DDL_BLOB_DEFAULT

*Constant*: `&str`

Typed BLOB default hex.



## bookclerk_db_exec::sql_v1::BINDING_DDL_BOOLEAN

*Constant*: `&str`

Binding `CREATE TABLE` covering canonical `BOOLEAN` (not INTEGER-as-bool).



## bookclerk_db_exec::sql_v1::BINDING_DDL_CONFLICT

*Constant*: `&str`

Unique + NOT NULL table for `INSERT OR IGNORE` conflict-domain vectors.



## bookclerk_db_exec::sql_v1::BINDING_DDL_IDENTITY

*Constant*: `&str`

Identity table (explicit id then omit-id).



## bookclerk_db_exec::sql_v1::BINDING_DDL_IGNORE_SELECT

*Constant*: `&str`

`INSERT OR IGNORE … SELECT` (plain bind source).



## bookclerk_db_exec::sql_v1::BINDING_DDL_LIKE

*Constant*: `&str`

Case-sensitive `LIKE` table.



## bookclerk_db_exec::sql_v1::BINDING_DDL_LOWERCASE

*Constant*: `&str`

Lowercase / mixed-case DDL admitted by SQL v1 (idents fold) and lowered
case-insensitively on Postgres.



## bookclerk_db_exec::sql_v1::BINDING_DDL_ORDER_NULLS

*Constant*: `&str`

Nullable column for ORDER BY NULL ordering.



## bookclerk_db_exec::sql_v1::BINDING_DDL_TEXT_ORDER

*Constant*: `&str`

TEXT collation / order table.



## bookclerk_db_exec::sql_v1::BINDING_DDL_UNQUOTED_FOLD

*Constant*: `&str`

Unquoted mixed-case table that folds to lowercase.



## bookclerk_db_exec::sql_v1::MIXED_GATE_LITERAL_COUNT

*Constant*: `&str`

COUNT helper for mixed-gate replay assertions.



## bookclerk_db_exec::sql_v1::MIXED_GATE_LITERAL_DDL

*Constant*: `&str`

Mixed DDL whose companion INSERT stores the receipt-gate text as a value.



## bookclerk_db_exec::sql_v1::MIXED_GATE_LITERAL_SELECT

*Constant*: `&str`

SELECT the stored gate literal from [`mixed_gate_literal_insert`](sql_v1.md#bookclerk_db_execsql_v1mixed_gate_literal_insert).



## bookclerk_db_exec::sql_v1::PORTABLE_AGGREGATE_SELECT

*Constant*: `&str`

Aggregate-only companion to [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select).



## bookclerk_db_exec::sql_v1::PORTABLE_BLOB_DEFAULT_INSERT

*Constant*: `&str`

INSERT using the BLOB default.



## bookclerk_db_exec::sql_v1::PORTABLE_BLOB_DEFAULT_SELECT

*Constant*: `&str`

Read the default blob.



## bookclerk_db_exec::sql_v1::PORTABLE_BOOLEAN_INSERT

*Constant*: `&str`

INSERT `DbValue::Boolean` + typed-null into [`BINDING_DDL_BOOLEAN`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_boolean).



## bookclerk_db_exec::sql_v1::PORTABLE_BOOLEAN_SELECT

*Constant*: `&str`

SELECT declared boolean columns from [`BINDING_DDL_BOOLEAN`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_boolean).



## bookclerk_db_exec::sql_v1::PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD

*Constant*: `&str`

`/` and `%` of a `NULLIF` that is **not** `NULLIF(expr, 0)` must still
gain an outer zero guard (`NULLIF(0, 1)` evaluates to `0`).



## bookclerk_db_exec::sql_v1::PORTABLE_DIV_OPERANDS

*Constant*: `&str`

`/` RHS covering call, unary, CAST, and parenthesized zero.



## bookclerk_db_exec::sql_v1::PORTABLE_IDENTITY_DELETE_MAX

*Constant*: `&str`

Delete the max identity row (SQLite AUTOINCREMENT must not reuse it).



## bookclerk_db_exec::sql_v1::PORTABLE_IDENTITY_DROP

*Constant*: `&str`

DROP identity table.



## bookclerk_db_exec::sql_v1::PORTABLE_IDENTITY_INSERT_BAD_TYPE

*Constant*: `&str`

Identity omit after rollback of an explicit id (same atomic request fails).



## bookclerk_db_exec::sql_v1::PORTABLE_IDENTITY_INSERT_EXPLICIT

*Constant*: `&str`

Explicit id 100.



## bookclerk_db_exec::sql_v1::PORTABLE_IDENTITY_INSERT_OMIT

*Constant*: `&str`

Omit id (must yield 101 after explicit 100).



## bookclerk_db_exec::sql_v1::PORTABLE_IDENTITY_SELECT_MAX

*Constant*: `&str`

Largest id.



## bookclerk_db_exec::sql_v1::PORTABLE_IGNORE_SELECT

*Constant*: `&str`

Plain `INSERT OR IGNORE … SELECT ? RETURNING id`.



## bookclerk_db_exec::sql_v1::PORTABLE_IGNORE_SELECT_ORDER_LIMIT

*Constant*: `&str`

`ORDER BY` + `LIMIT` source.



## bookclerk_db_exec::sql_v1::PORTABLE_IGNORE_SELECT_UNION

*Constant*: `&str`

Compound `UNION ALL` source.



## bookclerk_db_exec::sql_v1::PORTABLE_IGNORE_SELECT_WITH

*Constant*: `&str`

`WITH` source for `INSERT OR IGNORE`.



## bookclerk_db_exec::sql_v1::PORTABLE_INSERT

*Constant*: `&str`

Seed row for [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select). Bind [`PORTABLE_INSERT_BLOB`](sql_v1.md#bookclerk_db_execsql_v1portable_insert_blob) at `?`.



## bookclerk_db_exec::sql_v1::PORTABLE_INSERT_BLOB

*Constant*: `&[u8]`

Blob payload for [`PORTABLE_INSERT`](sql_v1.md#bookclerk_db_execsql_v1portable_insert).



## bookclerk_db_exec::sql_v1::PORTABLE_INSERT_OR_IGNORE_NOT_NULL

*Constant*: `&str`

NOT NULL ignore must still error (not swallowed).



## bookclerk_db_exec::sql_v1::PORTABLE_INSERT_OR_IGNORE_RETURNING_LC

*Constant*: `&str`

Lowercase `INSERT OR IGNORE … RETURNING` against [`BINDING_DDL_LOWERCASE`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_lowercase).



## bookclerk_db_exec::sql_v1::PORTABLE_INSERT_OR_IGNORE_UNIQUE

*Constant*: `&str`

Unique-conflict ignore (second insert is a no-op).



## bookclerk_db_exec::sql_v1::PORTABLE_INTEGER_OVERFLOW

*Constant*: `&str`

INTEGER overflow (`+` `-` `*` `abs(i64::MIN)`) yields NULL on every backend.



## bookclerk_db_exec::sql_v1::PORTABLE_JSON_OBJECT_SEMANTICS

*Constant*: `&str`

`json_object` nulls, duplicate keys, nested objects, and 16-pair (32-arg) arity.

Length comparisons distinguish JSON `null` retention from merge-patch
deletion without depending on exact JSON spacing. Postgres lowering casts
`json_build_object` to TEXT so `length(json_object(…))` is legal.



## bookclerk_db_exec::sql_v1::PORTABLE_JSON_PATH_LIKE_COMPARE

*Constant*: `&str`

`'$.…'` compared as TEXT (not a `json_extract` path) — Postgres must COLLATE.



## bookclerk_db_exec::sql_v1::PORTABLE_LIKE_INSERT

*Constant*: `&str`

Seed `'A'` for LIKE vectors.



## bookclerk_db_exec::sql_v1::PORTABLE_LIKE_NON_ASCII

*Constant*: `&str`

Non-ASCII that must not Unicode-fold under LIKE.



## bookclerk_db_exec::sql_v1::PORTABLE_LIKE_SELECT

*Constant*: `&str`

Case-sensitive LIKE probes (A/a, prefix, GLOB metacharacters, NULL).



## bookclerk_db_exec::sql_v1::PORTABLE_LOWERCASE_SELECT

*Constant*: `&str`

SELECT stored blob + boolean from [`BINDING_DDL_LOWERCASE`](sql_v1.md#bookclerk_db_execsql_v1binding_ddl_lowercase).



## bookclerk_db_exec::sql_v1::PORTABLE_LOWER_NON_ASCII_EXPECT

*Constant*: `&str`

SQLite's result for [`PORTABLE_LOWER_NON_ASCII_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_lower_non_ascii_select), and Postgres once
the argument is `COLLATE "C"`.



## bookclerk_db_exec::sql_v1::PORTABLE_LOWER_NON_ASCII_INSERT

*Constant*: `&str`

Inserts a non-ASCII `body` so `lower` can be compared with SQLite.

SQLite `lower` folds ASCII only, so `É` stays `É`. Postgres `lower` uses
the argument collation's `LC_CTYPE`. The typed lowering wraps the column
as `lower((body COLLATE "C"))`, which keeps `É`.



## bookclerk_db_exec::sql_v1::PORTABLE_LOWER_NON_ASCII_SELECT

*Constant*: `&str`

`lower` of the row inserted by [`PORTABLE_LOWER_NON_ASCII_INSERT`](sql_v1.md#bookclerk_db_execsql_v1portable_lower_non_ascii_insert).



## bookclerk_db_exec::sql_v1::PORTABLE_LOWER_PLACEHOLDER_SELECT

*Constant*: `&str`

`lower` of a bound placeholder. Postgres must collate the parameter
(`lower(($1 COLLATE "C"))`) so `É` stays `É`, matching SQLite.



## bookclerk_db_exec::sql_v1::PORTABLE_LOWER_SUBSTR_SELECT

*Constant*: `&str`

`lower(substr(body, ?, ?))` with integer binds.

The placeholders are the substr index and length, not text arguments of
`lower`. Postgres rejects `COLLATE` on bigint, so they stay bare. It also
rejects `substr(text, bigint, bigint)`, so lowering clamps those arguments
into `int4` and casts them to `INTEGER`. A length of `i64::MAX` still
means the rest of the string. The first character of the non-ASCII row is
still `É`.



## bookclerk_db_exec::sql_v1::PORTABLE_MIN_MAX_NULL

*Constant*: `&str`

Scalar min/max NULL-poison (any NULL argument → NULL).



## bookclerk_db_exec::sql_v1::PORTABLE_NESTED_INTEGER_ARITH

*Constant*: `&str`

Nested INTEGER `+`/`*`/`abs` plus TEXT-inside-overflow (`length('ä') + 1`).



## bookclerk_db_exec::sql_v1::PORTABLE_ORDER_NULLS_ASC

*Constant*: `&str`

ASC: NULL sorts as smallest.



## bookclerk_db_exec::sql_v1::PORTABLE_ORDER_NULLS_DESC

*Constant*: `&str`

DESC: NULL sorts as smallest (last).



## bookclerk_db_exec::sql_v1::PORTABLE_ORDER_NULLS_INSERT_1

*Constant*: `&str`

Seed three rows: 1, NULL, 2.



## bookclerk_db_exec::sql_v1::PORTABLE_ORDER_NULLS_INSERT_2

*Constant*: `&str`

2 seed.



## bookclerk_db_exec::sql_v1::PORTABLE_ORDER_NULLS_INSERT_NULL

*Constant*: `&str`

NULL seed.



## bookclerk_db_exec::sql_v1::PORTABLE_RUNTIME_EDGES

*Constant*: `&str`

Runtime-edge operators/helpers (`/` `%` zero → NULL, `round` halfway, `substr`, `length`, `replace`, `abs`, `lower`).



## bookclerk_db_exec::sql_v1::PORTABLE_SELECT

*Constant*: `&str`

Scalar portable helpers (no aggregates).

Column order matches [`portable_select_expects`](sql_v1.md#bookclerk_db_execsql_v1portable_select_expects). `json_object` is
sampled through `json_extract` so the cell is a scalar TEXT value rather
than a JSON document (spacing is engine-specific). Postgres lowering also
casts `json_build_object` to TEXT so `length(json_object(…))` is legal.
Aggregates live in [`PORTABLE_AGGREGATE_SELECT`]: a mixed aggregate +
non-aggregate SELECT list is SQLite-shaped, not Bookclerk SQL v1
(PostgreSQL requires `GROUP BY`). `round`/`sum`/`avg` are `CAST` to
`INTEGER` so cells stay in the universal `DbValue` domain (Postgres
`NUMERIC` is not a wire type).



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_MULTILINGUAL

*Constant*: `&str`

Full Unicode scalar TEXT (UTF-8), including supplementary-plane Han, RTL,
combining sequences, and NFC vs NFD that must not be silently normalized.



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_OPS

*Constant*: `&str`

TEXT comparison + scalar min/max + lower/upper.



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_ORDER_INSERT_A

*Constant*: `&str`

Lowercase a.



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_ORDER_INSERT_B

*Constant*: `&str`

Seed ASCII + non-ASCII rows.



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_ORDER_INSERT_E

*Constant*: `&str`

ASCII e.



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_ORDER_INSERT_EACUTE

*Constant*: `&str`

Non-ASCII é (U+00E9).



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_ORDER_SELECT

*Constant*: `&str`

Binary TEXT order (byte/code-point, not locale).



## bookclerk_db_exec::sql_v1::PORTABLE_TEXT_PREFIX_LITERALS

*Constant*: `&str`

TEXT literals whose contents look like bind placeholders or `COLLATE`.



## bookclerk_db_exec::sql_v1::PORTABLE_UNCAST_ROUND

*Constant*: `&str`

Uncast `round` (per-row; not mixed with aggregates).



## bookclerk_db_exec::sql_v1::PORTABLE_UNCAST_SUM_AVG

*Constant*: `&str`

Uncast integer `sum` / `avg` (aggregate-only companion).



## bookclerk_db_exec::sql_v1::PORTABLE_UNQUOTED_FOLD_INSERT

*Constant*: `&str`

SELECT via folded lowercase name.



## bookclerk_db_exec::sql_v1::PORTABLE_UNQUOTED_FOLD_SELECT

*Constant*: `&str`

SELECT from folded name.



## bookclerk_db_exec::sql_v1::PortableExpect

*Enum*

Expected cells for [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select) (adapter-neutral).

**Variants:**
- `Int(i64)` - Exact `DbValue::Int64`.
- `Text(&'static str)` - Exact `DbValue::Text`.
- `Float(f64)` - Exact `DbValue::Float64` (tolerance only vs another float).
- `Bool(bool)` - Exact `DbValue::Boolean`.
- `NullBool` - Exact `DbValue::Null` with `DbType::Bool`.
- `Null` - Exact `DbValue::Null` (any declared type).

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PortableExpect`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::sql_v1::mixed_gate_literal_insert

*Function*

INSERT that embeds [`crate::GUEST_RECEIPT_WRITE_GATE`](guest_receipt.md#bookclerk_db_execguest_receiptguest_receipt_write_gate) in a comment and a string.

```rust
fn mixed_gate_literal_insert() -> String
```



## bookclerk_db_exec::sql_v1::portable_aggregate_expects

*Function*

Expected [`PORTABLE_AGGREGATE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_aggregate_select) cells in column order.

```rust
fn portable_aggregate_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_aggregate_mismatch

*Function*

Formats a mismatch for [`PORTABLE_AGGREGATE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_aggregate_select).

```rust
fn portable_aggregate_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_boolean_expects

*Function*

Expected [`PORTABLE_BOOLEAN_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_boolean_select) cells in column order.

```rust
fn portable_boolean_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_boolean_insert_binds

*Function*

Binds for [`PORTABLE_BOOLEAN_INSERT`](sql_v1.md#bookclerk_db_execsql_v1portable_boolean_insert).

```rust
fn portable_boolean_insert_binds() -> Vec<bookclerk_plugin_abi::DbValue>
```



## bookclerk_db_exec::sql_v1::portable_boolean_mismatch

*Function*

Formats a mismatch for [`PORTABLE_BOOLEAN_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_boolean_select).

```rust
fn portable_boolean_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_div_nullif_not_zero_guard_expects

*Function*

Expected [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](sql_v1.md#bookclerk_db_execsql_v1portable_div_nullif_not_zero_guard) cells.

```rust
fn portable_div_nullif_not_zero_guard_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_div_nullif_not_zero_guard_mismatch

*Function*

Formats a mismatch for [`PORTABLE_DIV_NULLIF_NOT_ZERO_GUARD`](sql_v1.md#bookclerk_db_execsql_v1portable_div_nullif_not_zero_guard).

```rust
fn portable_div_nullif_not_zero_guard_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_div_operands_expects

*Function*

Expected [`PORTABLE_DIV_OPERANDS`](sql_v1.md#bookclerk_db_execsql_v1portable_div_operands) cells.

```rust
fn portable_div_operands_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_div_operands_mismatch

*Function*

Formats a mismatch for [`PORTABLE_DIV_OPERANDS`](sql_v1.md#bookclerk_db_execsql_v1portable_div_operands).

```rust
fn portable_div_operands_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_integer_overflow_expects

*Function*

Expected [`PORTABLE_INTEGER_OVERFLOW`](sql_v1.md#bookclerk_db_execsql_v1portable_integer_overflow) cells.

```rust
fn portable_integer_overflow_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_integer_overflow_mismatch

*Function*

Formats a mismatch for [`PORTABLE_INTEGER_OVERFLOW`](sql_v1.md#bookclerk_db_execsql_v1portable_integer_overflow).

```rust
fn portable_integer_overflow_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_json_object_semantics_expects

*Function*

Expected [`PORTABLE_JSON_OBJECT_SEMANTICS`](sql_v1.md#bookclerk_db_execsql_v1portable_json_object_semantics) cells.

```rust
fn portable_json_object_semantics_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_json_object_semantics_mismatch

*Function*

Formats a mismatch for [`PORTABLE_JSON_OBJECT_SEMANTICS`](sql_v1.md#bookclerk_db_execsql_v1portable_json_object_semantics).

```rust
fn portable_json_object_semantics_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_json_path_like_compare_expects

*Function*

Expected [`PORTABLE_JSON_PATH_LIKE_COMPARE`](sql_v1.md#bookclerk_db_execsql_v1portable_json_path_like_compare) cells.

```rust
fn portable_json_path_like_compare_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_json_path_like_compare_mismatch

*Function*

Formats a mismatch for [`PORTABLE_JSON_PATH_LIKE_COMPARE`](sql_v1.md#bookclerk_db_execsql_v1portable_json_path_like_compare).

```rust
fn portable_json_path_like_compare_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_like_expects

*Function*

Expected [`PORTABLE_LIKE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_like_select) cells (`c8` bind is `'A'`).

```rust
fn portable_like_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_lowercase_insert_binds

*Function*

Binds for [`PORTABLE_INSERT_OR_IGNORE_RETURNING_LC`](sql_v1.md#bookclerk_db_execsql_v1portable_insert_or_ignore_returning_lc).

```rust
fn portable_lowercase_insert_binds() -> Vec<bookclerk_plugin_abi::DbValue>
```



## bookclerk_db_exec::sql_v1::portable_min_max_null_expects

*Function*

Expected [`PORTABLE_MIN_MAX_NULL`](sql_v1.md#bookclerk_db_execsql_v1portable_min_max_null) cells.

```rust
fn portable_min_max_null_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_min_max_null_mismatch

*Function*

Formats a mismatch for [`PORTABLE_MIN_MAX_NULL`](sql_v1.md#bookclerk_db_execsql_v1portable_min_max_null).

```rust
fn portable_min_max_null_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_nested_integer_arith_expects

*Function*

Expected [`PORTABLE_NESTED_INTEGER_ARITH`](sql_v1.md#bookclerk_db_execsql_v1portable_nested_integer_arith) cells.

```rust
fn portable_nested_integer_arith_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_nested_integer_arith_mismatch

*Function*

Formats a mismatch for [`PORTABLE_NESTED_INTEGER_ARITH`](sql_v1.md#bookclerk_db_execsql_v1portable_nested_integer_arith).

```rust
fn portable_nested_integer_arith_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_order_nulls_asc_expects

*Function*

Expected [`PORTABLE_ORDER_NULLS_ASC`](sql_v1.md#bookclerk_db_execsql_v1portable_order_nulls_asc) cells.

```rust
fn portable_order_nulls_asc_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_order_nulls_asc_mismatch

*Function*

Formats a mismatch for ASC NULL ordering.

```rust
fn portable_order_nulls_asc_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_order_nulls_desc_expects

*Function*

Expected [`PORTABLE_ORDER_NULLS_DESC`](sql_v1.md#bookclerk_db_execsql_v1portable_order_nulls_desc) cells.

```rust
fn portable_order_nulls_desc_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_order_nulls_desc_mismatch

*Function*

Formats a mismatch for DESC NULL ordering.

```rust
fn portable_order_nulls_desc_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_rows_mismatch

*Function*

Compares every row in `stmt` to `expect`.

```rust
fn portable_rows_mismatch(stmt: &bookclerk_plugin_abi::StatementResult, expect: &[&[PortableExpect]], label: &str) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_runtime_edges_expects

*Function*

Expected [`PORTABLE_RUNTIME_EDGES`](sql_v1.md#bookclerk_db_execsql_v1portable_runtime_edges) cells.

```rust
fn portable_runtime_edges_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_runtime_edges_mismatch

*Function*

Formats a mismatch for [`PORTABLE_RUNTIME_EDGES`](sql_v1.md#bookclerk_db_execsql_v1portable_runtime_edges).

```rust
fn portable_runtime_edges_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_select_expects

*Function*

Expected [`PORTABLE_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_select) cells in column order.

```rust
fn portable_select_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_select_mismatch

*Function*

Formats a mismatch for adapter execution tests.

```rust
fn portable_select_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_statement_mismatch

*Function*

Compares `stmt` to `expect` and describes the first mismatch.

```rust
fn portable_statement_mismatch(stmt: &bookclerk_plugin_abi::StatementResult, expect: &[PortableExpect], label: &str) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_text_multilingual_expects

*Function*

Expected [`PORTABLE_TEXT_MULTILINGUAL`](sql_v1.md#bookclerk_db_execsql_v1portable_text_multilingual) cells (exact UTF-8, no NFC/NFD fold).

```rust
fn portable_text_multilingual_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_text_ops_expects

*Function*

Expected [`PORTABLE_TEXT_OPS`](sql_v1.md#bookclerk_db_execsql_v1portable_text_ops) cells.

```rust
fn portable_text_ops_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_text_order_expects

*Function*

Expected [`PORTABLE_TEXT_ORDER_SELECT`](sql_v1.md#bookclerk_db_execsql_v1portable_text_order_select) rows.

```rust
fn portable_text_order_expects() -> &'static [&'static [PortableExpect]]
```



## bookclerk_db_exec::sql_v1::portable_text_prefix_literals_expects

*Function*

Expected [`PORTABLE_TEXT_PREFIX_LITERALS`](sql_v1.md#bookclerk_db_execsql_v1portable_text_prefix_literals) cells.

```rust
fn portable_text_prefix_literals_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_text_prefix_literals_mismatch

*Function*

Formats a mismatch for [`PORTABLE_TEXT_PREFIX_LITERALS`](sql_v1.md#bookclerk_db_execsql_v1portable_text_prefix_literals).

```rust
fn portable_text_prefix_literals_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_uncast_round_expects

*Function*

Expected [`PORTABLE_UNCAST_ROUND`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_round) cells (r=1.5).

```rust
fn portable_uncast_round_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_uncast_round_mismatch

*Function*

Formats a mismatch for [`PORTABLE_UNCAST_ROUND`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_round).

```rust
fn portable_uncast_round_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_uncast_sum_avg_expects

*Function*

Expected [`PORTABLE_UNCAST_SUM_AVG`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_sum_avg) cells (n=2).

```rust
fn portable_uncast_sum_avg_expects() -> &'static [PortableExpect]
```



## bookclerk_db_exec::sql_v1::portable_uncast_sum_avg_mismatch

*Function*

Formats a mismatch for [`PORTABLE_UNCAST_SUM_AVG`](sql_v1.md#bookclerk_db_execsql_v1portable_uncast_sum_avg).

```rust
fn portable_uncast_sum_avg_mismatch(stmt: &bookclerk_plugin_abi::StatementResult) -> Option<String>
```



## bookclerk_db_exec::sql_v1::portable_value_matches

*Function*

True when `got` is the exact [`DbValue`](../bookclerk_plugin_abi/db_value.md#bookclerk_plugin_abidb_valuedbvalue) for `want`.

```rust
fn portable_value_matches(got: &bookclerk_plugin_abi::DbValue, want: PortableExpect) -> bool
```



