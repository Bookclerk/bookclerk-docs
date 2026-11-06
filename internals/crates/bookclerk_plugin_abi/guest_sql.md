# bookclerk_plugin_abi::guest_sql

Host-side grammar and scope checks for guest-authored typed SQL.

Overwriting [`crate::DbPlanStatementKind`](db_execute.md#dbplanstatementkind) is classification, not
authorization. Guests may only run canonical DML/SELECT against allowed
tables, with bind counts and result-selection fields that match the
statement. Host-authored schema batches must not use this path.

| Item | Kind | Summary |
| --- | --- | --- |
| [`GuestSqlPolicy`](#guestsqlpolicy) | struct | Host-issued scope for one granted database binding. |
| [`GuestSqlRefs`](#guestsqlrefs) | struct | Tables, columns, and functions referenced by guest SQL. |
| [`is_reserved_binding_name`](#is_reserved_binding_name) | function | True when `name` is reserved for host bookkeeping in a binding database. |
| [`authorize_guest_sql_policy`](#authorize_guest_sql_policy) | function | Authorizes parsed table/column/function refs against a host-issued policy. |
| [`statement_is_ddl`](#statement_is_ddl) | function | True when `sql` is a DDL statement (`CREATE` / `ALTER` / `DROP`). |
| [`validate_sql_v1_grammar`](#validate_sql_v1_grammar) | function | Bookclerk SQL v1 grammar (plugin SQL), before scope/security authorization. |
| [`guest_statement_kind`](#guest_statement_kind) | function | Classifies guest SQL the same way the host stamps `DbStatement.kind`. |
| [`returning_single_row_proven`](#returning_single_row_proven) | function | Proves from SQL shape alone that a `RETURNING` statement yields at most one row. |
| [`validate_guest_execute_request`](#validate_guest_execute_request) | function | Rejects guest SQL that is outside the Bookclerk binding grammar. |
| [`validate_guest_execute_request_for_policy`](#validate_guest_execute_request_for_policy) | function | `validate_guest_execute_request` with policy-dependent grammar. |
| [`parse_guest_sql_refs`](#parse_guest_sql_refs) | function | Parsed table, column, and function names from one guest statement. |
| [`BOOKCLERK_RESERVED_PREFIX`](#bookclerk_reserved_prefix) | constant | Identifier prefix reserved for host bookkeeping inside a plugin-owned binding database. |

## Structs

### GuestSqlPolicy

```rust
pub struct GuestSqlPolicy {
    /* private fields */
}
```

Host-issued scope for one granted database binding.

Empty [`Self::deny_all`](#guestsqlpolicydeny_all) denies every table. Catalog identifiers
(`encrypted_secrets`, `sqlite_*`, `pg_*`, `information_schema`) are always
refused even if listed. [`Self::host_authoritative`](#guestsqlpolicyhost_authoritative) skips table-scope
authorization so a Cap'n `DatabaseSession` can be the single authorization
authority; typing still runs at execute.

#### GuestSqlPolicy::deny_all

```rust
pub fn deny_all() -> Self
```

No tables, columns, or functions.

#### GuestSqlPolicy::host_authoritative

```rust
pub fn host_authoritative() -> Self
```

Broker passthrough: grammar/size checks still run; table scope is the
host `DatabaseSession`'s responsibility. Typing and TEXT lowering still
run at execute against the host catalog.

#### GuestSqlPolicy::binding_owned

```rust
pub fn binding_owned() -> Self
```

Plugin-owned isolated database binding (Workers-D1-like ownership).

Ordinary binding execute is query/DML only. Durable `CREATE`/`DROP` is
admitted only via [`Self::binding_migration`](#guestsqlpolicybinding_migration) (host migration engine).
Any table may be named except identifiers starting with
[`BOOKCLERK_RESERVED_PREFIX`](#bookclerk_reserved_prefix) (host bookkeeping), engine catalog
identifiers, and schema-qualified names. Grammar and size
checks still run. Functions are the Bookclerk SQL v1 portable set.

#### GuestSqlPolicy::binding_migration

```rust
pub fn binding_migration() -> Self
```

Binding policy that admits bounded idempotent DDL for a migration apply.

Same table/function rules as [`Self::binding_owned`](#guestsqlpolicybinding_owned), plus
`CREATE`/`DROP` `TABLE`/`INDEX` with `IF [NOT] EXISTS`. `ALTER` and
`CREATE TABLE AS` stay refused. Product `BindingGuestDatabase::execute`
must not use this constructor.

#### GuestSqlPolicy::is_binding_owned

```rust
pub fn is_binding_owned(&self) -> bool
```

True when this policy is a plugin-owned binding scope.

#### GuestSqlPolicy::is_migration_context

```rust
pub fn is_migration_context(&self) -> bool
```

True when bounded binding DDL is admitted (migration apply only).

#### GuestSqlPolicy::allow_tables

```rust
pub fn allow_tables(tables: impl IntoIterator<Item = impl AsRef<str>>) -> Self
```

Allows `tables` with builtin scalar functions and any column on those tables.

#### GuestSqlPolicy::with_sql_types

```rust
pub fn with_sql_types(self, env: SqlTypeEnv) -> Self
```

Attaches a loaded [`SqlTypeEnv`](sql_types.md#sqltypeenv) (durable catalog snapshot).

#### GuestSqlPolicy::sql_types

```rust
pub fn sql_types(&self) -> &SqlTypeEnv
```

Column types used by fail-closed expression checking.

#### GuestSqlPolicy::functions

```rust
pub fn functions(&self) -> impl Iterator<Item = &str>
```

Lower-cased scalar function names this policy admits, in sorted order.

Lets a harness compare the policy against an independent view of the
functions a statement calls (the fuzzing ADR's SQLite authorizer
oracle); authorization itself happens in [`authorize_guest_sql_policy`](#authorize_guest_sql_policy).

#### GuestSqlPolicy::restrict_columns

```rust
pub fn restrict_columns(
    self,
    table: &str,
    cols: impl IntoIterator<Item = impl AsRef<str>>,
) -> Self
```

Restricts `table` to `cols` (empty set denies every column on that table).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### GuestSqlRefs

```rust
pub struct GuestSqlRefs {
    pub tables: Vec<String>,
    pub columns: Vec<(Option<String>, String)>,
    pub functions: Vec<String>,
    pub ctes: Vec<String>,
}
```

Tables, columns, and functions referenced by guest SQL.

**Fields**

- `tables`: FROM/JOIN/INTO/UPDATE/TABLE names.
- `columns`: `(table, column)` pairs; `table` is `None` when unqualified.
- `functions`: Function names (`ident(`).
- `ctes`: CTE names in scope (not physical tables).

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### is_reserved_binding_name

```rust
pub fn is_reserved_binding_name(name: &str) -> bool
```

True when `name` is reserved for host bookkeeping in a binding database.

Case-insensitive on the normalized identifier; quoted names are unwrapped
first so `"Bookclerk_x"` is reserved too.

### authorize_guest_sql_policy

```rust
pub fn authorize_guest_sql_policy(
    req: &ExecuteRequest,
    policy: &GuestSqlPolicy,
) -> Result<()>
```

Authorizes parsed table/column/function refs against a host-issued policy.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a referenced object is outside
`policy`.

### statement_is_ddl

```rust
pub fn statement_is_ddl(sql: &str) -> bool
```

True when `sql` is a DDL statement (`CREATE` / `ALTER` / `DROP`).

Used by hosts to skip receipt write-predicates (DDL has no `WHERE`
clause); classification only, never authorization.

### validate_sql_v1_grammar

```rust
pub fn validate_sql_v1_grammar(sql: &str, binding_owned: bool) -> Result<()>
```

Bookclerk SQL v1 grammar (plugin SQL), before scope/security authorization.

Bindings use the portable subset: `INSERT` / `INSERT OR IGNORE` (not
`REPLACE`), no `GLOB` / `COLLATE` / `MATCH` / `REGEXP`, and CREATE TABLE
types `INTEGER` / `REAL` / `TEXT` / `BLOB` / `BOOLEAN` only. Every guest
rejects non-v1 bind spellings (`$n`, `?NNN`). Library guests still reject
`REPLACE` as a statement form.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when `sql` is outside SQL v1.

### guest_statement_kind

```rust
pub fn guest_statement_kind(sql: &str) -> DbPlanStatementKind
```

Classifies guest SQL the same way the host stamps `DbStatement.kind`.

Leading `WITH` / `WITH RECURSIVE` CTE lists are skipped so the **main**
statement decides the kind: `WITH … SELECT` is [`DbPlanStatementKind::Select`](db_execute.md#dbplanstatementkind),
`WITH … INSERT/UPDATE/DELETE` is [`DbPlanStatementKind::Execute`](db_execute.md#dbplanstatementkind), and a
top-level `RETURNING` on that main statement is [`DbPlanStatementKind::Returning`](db_execute.md#dbplanstatementkind).

### returning_single_row_proven

```rust
pub fn returning_single_row_proven(sql: &str) -> bool
```

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

### validate_guest_execute_request

```rust
pub fn validate_guest_execute_request(req: &ExecuteRequest) -> Result<()>
```

Rejects guest SQL that is outside the Bookclerk binding grammar.

Checks statement verbs, named tables, positional bind counts,
`resultSelection` vs classified kind, and `maxRows` consistency.
Does **not** stamp the request hash or measure encoded size.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the request is not allowed.

### validate_guest_execute_request_for_policy

```rust
pub fn validate_guest_execute_request_for_policy(
    req: &ExecuteRequest,
    policy: &GuestSqlPolicy,
) -> Result<()>
```

[`validate_guest_execute_request`](#validate_guest_execute_request) with policy-dependent grammar.

Pipeline: plugin SQL → SQL-v1 grammar validation → binding
scope/security authorization → capability/limit validation → canonical
[`ExecuteRequest`](db_execute.md#executerequest). Adapter-edge lowering happens later, at execute,
against the isolated physical DB.

A [`GuestSqlPolicy::binding_migration`](#guestsqlpolicybinding_migration) policy admits bounded DDL verbs
(`CREATE` / `DROP`; `ALTER` is still classified then refused);
[`GuestSqlPolicy::binding_owned`](#guestsqlpolicybinding_owned) stays query/DML. Shapes and names are
then authorized by [`authorize_guest_sql_policy`](#authorize_guest_sql_policy). Every other policy uses
the fixed DML/SELECT grammar.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the request is not allowed.

### parse_guest_sql_refs

```rust
pub fn parse_guest_sql_refs(sql: &str) -> Result<GuestSqlRefs>
```

Parsed table, column, and function names from one guest statement.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the statement uses a form the
authorizer cannot fully resolve (fail closed).

## Constants

### BOOKCLERK_RESERVED_PREFIX

```rust
pub const BOOKCLERK_RESERVED_PREFIX: &str = "bookclerk_";
```

Identifier prefix reserved for host bookkeeping inside a plugin-owned
binding database.

Every table the host creates in a binding (`bookclerk_schema_migrations`,
`bookclerk_plugin_migrations`, `bookclerk_receipts`, `bookclerk_slots`, the
SQL catalog tables, the `INSERT ... SELECT` wrap alias) carries this prefix,
so the single rule "no plugin identifier starts with `bookclerk_`" keeps
plugin tables from colliding with present or future host bookkeeping.
Everything else (including names such as `schema_migrations` that a plugin
framework may itself want) is the plugin's to use.
