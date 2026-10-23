**bookclerk_plugin_abi > guest_sql**

# Module: guest_sql

## Contents

**Structs**

- [`GuestSqlPolicy`](#guestsqlpolicy) - Host-issued scope for one granted database binding.
- [`GuestSqlRefs`](#guestsqlrefs) - Tables, columns, and functions referenced by guest SQL.

**Functions**

- [`authorize_guest_sql_policy`](#authorize_guest_sql_policy) - Authorizes parsed table/column/function refs against a host-issued policy.
- [`guest_statement_kind`](#guest_statement_kind) - Classifies guest SQL the same way the host stamps `DbStatement.kind`.
- [`is_reserved_binding_name`](#is_reserved_binding_name) - True when `name` is reserved for host bookkeeping in a binding database.
- [`parse_guest_sql_refs`](#parse_guest_sql_refs) - Parsed table, column, and function names from one guest statement.
- [`returning_single_row_proven`](#returning_single_row_proven) - Proves from SQL shape alone that a `RETURNING` statement yields at most
- [`statement_is_ddl`](#statement_is_ddl) - True when `sql` is a DDL statement (`CREATE` / `ALTER` / `DROP`).
- [`validate_guest_execute_request`](#validate_guest_execute_request) - Rejects guest SQL that is outside the Bookclerk binding grammar.
- [`validate_guest_execute_request_for_policy`](#validate_guest_execute_request_for_policy) - [`validate_guest_execute_request`](guest_sql.md#bookclerk_plugin_abiguest_sqlvalidate_guest_execute_request) with policy-dependent grammar.
- [`validate_sql_v1_grammar`](#validate_sql_v1_grammar) - Bookclerk SQL v1 grammar (plugin SQL), before scope/security authorization.

**Constants**

- [`BOOKCLERK_RESERVED_PREFIX`](#bookclerk_reserved_prefix) - Identifier prefix reserved for host bookkeeping inside a plugin-owned

---

## bookclerk_plugin_abi::guest_sql::BOOKCLERK_RESERVED_PREFIX

*Constant*: `&str`

Identifier prefix reserved for host bookkeeping inside a plugin-owned
binding database.

Every table the host creates in a binding (`bookclerk_schema_migrations`,
`bookclerk_plugin_migrations`, `bookclerk_receipts`, `bookclerk_slots`, the
SQL catalog tables, the `INSERT ... SELECT` wrap alias) carries this prefix,
so the single rule "no plugin identifier starts with `bookclerk_`" keeps
plugin tables from colliding with present or future host bookkeeping.
Everything else (including names such as `schema_migrations` that a plugin
framework may itself want) is the plugin's to use.



## bookclerk_plugin_abi::guest_sql::GuestSqlPolicy

*Struct*

Host-issued scope for one granted database binding.

Empty `Self::deny_all` denies every table. Catalog identifiers
(`encrypted_secrets`, `sqlite_*`, `pg_*`, `information_schema`) are always
refused even if listed. `Self::host_authoritative` skips table-scope
authorization so a Cap'n `DatabaseSession` can be the single authorization
authority; typing still runs at execute.

**Methods:**

- `fn deny_all() -> Self` - No tables, columns, or functions.
- `fn host_authoritative() -> Self` - Broker passthrough: grammar/size checks still run; table scope is the
- `fn binding_owned() -> Self` - Plugin-owned isolated database binding (Workers-D1-like ownership).
- `fn binding_migration() -> Self` - Binding policy that admits bounded idempotent DDL for a migration apply.
- `fn is_binding_owned(self: &Self) -> bool` - True when this policy is a plugin-owned binding scope.
- `fn is_migration_context(self: &Self) -> bool` - True when bounded binding DDL is admitted (migration apply only).
- `fn allow_tables<impl AsRef<str>, impl IntoIterator<Item = impl AsRef<str>>>(tables: impl Trait) -> Self` - Allows `tables` with builtin scalar functions and any column on those tables.
- `fn with_sql_types(self: Self, env: SqlTypeEnv) -> Self` - Attaches a loaded [`SqlTypeEnv`](sql_types.md#bookclerk_plugin_abisql_typessqltypeenv) (durable catalog snapshot).
- `fn sql_types(self: &Self) -> &SqlTypeEnv` - Column types used by fail-closed expression checking.
- `fn functions(self: &Self) -> impl Trait` - Lower-cased scalar function names this policy admits, in sorted order.
- `fn restrict_columns<impl AsRef<str>, impl IntoIterator<Item = impl AsRef<str>>>(self: Self, table: &str, cols: impl Trait) -> Self` - Restricts `table` to `cols` (empty set denies every column on that table).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> GuestSqlPolicy`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &GuestSqlPolicy) -> bool`



## bookclerk_plugin_abi::guest_sql::GuestSqlRefs

*Struct*

Tables, columns, and functions referenced by guest SQL.

**Fields:**
- `tables: Vec<String>` - FROM/JOIN/INTO/UPDATE/TABLE names.
- `columns: Vec<(Option<String>, String)>` - `(table, column)` pairs; `table` is `None` when unqualified.
- `functions: Vec<String>` - Function names (`ident(`).
- `ctes: Vec<String>` - CTE names in scope (not physical tables).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> GuestSqlRefs`
- **Clone**
  - `fn clone(self: &Self) -> GuestSqlRefs`
- **PartialEq**
  - `fn eq(self: &Self, other: &GuestSqlRefs) -> bool`



## bookclerk_plugin_abi::guest_sql::authorize_guest_sql_policy

*Function*

Authorizes parsed table/column/function refs against a host-issued policy.

# Errors

Returns `PluginError::invalid_params` when a referenced object is outside
`policy`.

```rust
fn authorize_guest_sql_policy(req: &crate::ExecuteRequest, policy: &GuestSqlPolicy) -> crate::Result<()>
```



## bookclerk_plugin_abi::guest_sql::guest_statement_kind

*Function*

Classifies guest SQL the same way the host stamps `DbStatement.kind`.

Leading `WITH` / `WITH RECURSIVE` CTE lists are skipped so the **main**
statement decides the kind: `WITH … SELECT` is `DbPlanStatementKind::Select`,
`WITH … INSERT/UPDATE/DELETE` is `DbPlanStatementKind::Execute`, and a
top-level `RETURNING` on that main statement is `DbPlanStatementKind::Returning`.

```rust
fn guest_statement_kind(sql: &str) -> crate::DbPlanStatementKind
```



## bookclerk_plugin_abi::guest_sql::is_reserved_binding_name

*Function*

True when `name` is reserved for host bookkeeping in a binding database.

Case-insensitive on the normalized identifier; quoted names are unwrapped
first so `"Bookclerk_x"` is reserved too.

```rust
fn is_reserved_binding_name(name: &str) -> bool
```



## bookclerk_plugin_abi::guest_sql::parse_guest_sql_refs

*Function*

Parsed table, column, and function names from one guest statement.

# Errors

Returns `PluginError::invalid_params` when the statement uses a form the
authorizer cannot fully resolve (fail closed).

```rust
fn parse_guest_sql_refs(sql: &str) -> crate::Result<GuestSqlRefs>
```



## bookclerk_plugin_abi::guest_sql::returning_single_row_proven

*Function*

Proves from SQL shape alone that a `RETURNING` statement yields at most
one row.

Conservative and fail-closed — a caller-asserted `maxRows` is never
trusted. A single-statement `INSERT` (after any leading CTE list) is
proven when its row source is exactly one top-level `VALUES` tuple, or a
table-free `SELECT` with no compound or grouping whose select list holds
only placeholders, literals, operators, `COALESCE`, `MIN` / `MAX` over
such items, and scalar subqueries, so it cannot return a set. `INSERT … SELECT … FROM`,
`UPDATE … RETURNING`, `DELETE … RETURNING`, and multi-tuple `VALUES` are
never proven, because the SQL text does not bound how many rows they
touch.

```rust
fn returning_single_row_proven(sql: &str) -> bool
```



## bookclerk_plugin_abi::guest_sql::statement_is_ddl

*Function*

True when `sql` is a DDL statement (`CREATE` / `ALTER` / `DROP`).

Used by hosts to skip receipt write-predicates (DDL has no `WHERE`
clause); classification only, never authorization.

```rust
fn statement_is_ddl(sql: &str) -> bool
```



## bookclerk_plugin_abi::guest_sql::validate_guest_execute_request

*Function*

Rejects guest SQL that is outside the Bookclerk binding grammar.

Checks statement verbs, named tables, positional bind counts,
`resultSelection` vs classified kind, and `maxRows` consistency.
Does **not** stamp the request hash or measure encoded size.

# Errors

Returns `PluginError::invalid_params` when the request is not allowed.

```rust
fn validate_guest_execute_request(req: &crate::ExecuteRequest) -> crate::Result<()>
```



## bookclerk_plugin_abi::guest_sql::validate_guest_execute_request_for_policy

*Function*

[`validate_guest_execute_request`](guest_sql.md#bookclerk_plugin_abiguest_sqlvalidate_guest_execute_request) with policy-dependent grammar.

Pipeline: plugin SQL → SQL-v1 grammar validation → binding
scope/security authorization → capability/limit validation → canonical
[`ExecuteRequest`](db_execute.md#bookclerk_plugin_abidb_executeexecuterequest). Adapter-edge lowering happens later, at execute,
against the isolated physical DB.

A `GuestSqlPolicy::binding_migration` policy admits bounded DDL verbs
(`CREATE` / `DROP`; `ALTER` is still classified then refused);
`GuestSqlPolicy::binding_owned` stays query/DML. Shapes and names are
then authorized by [`authorize_guest_sql_policy`](guest_sql.md#bookclerk_plugin_abiguest_sqlauthorize_guest_sql_policy). Every other policy uses
the fixed DML/SELECT grammar.

# Errors

Returns `PluginError::invalid_params` when the request is not allowed.

```rust
fn validate_guest_execute_request_for_policy(req: &crate::ExecuteRequest, policy: &GuestSqlPolicy) -> crate::Result<()>
```



## bookclerk_plugin_abi::guest_sql::validate_sql_v1_grammar

*Function*

Bookclerk SQL v1 grammar (plugin SQL), before scope/security authorization.

Bindings use the portable subset: `INSERT` / `INSERT OR IGNORE` (not
`REPLACE`), no `GLOB` / `COLLATE` / `MATCH` / `REGEXP`, and CREATE TABLE
types `INTEGER` / `REAL` / `TEXT` / `BLOB` / `BOOLEAN` only. Every guest
rejects non-v1 bind spellings (`$n`, `?NNN`). Library guests still reject
`REPLACE` as a statement form.

# Errors

Returns `PluginError::invalid_params` when `sql` is outside SQL v1.

```rust
fn validate_sql_v1_grammar(sql: &str, binding_owned: bool) -> crate::Result<()>
```



