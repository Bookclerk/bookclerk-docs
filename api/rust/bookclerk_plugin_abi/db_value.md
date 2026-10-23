# bookclerk_plugin_abi::db_value

Universal Cap'n database value domain (`DbValue`).

Baseline cells are typed null, bool, int64, finite float64, UTF-8 text, and
bytes. SeaORM and JSON bridges live in `bookclerk-db-exec` at the adapter edge.

| Item | Kind | Summary |
| --- | --- | --- |
| [`DbType`](#dbtype) | enum | Expected SQL type for a typed null (Cap'n `DbType`). |
| [`DbValue`](#dbvalue) | enum | One parameter or cell in the universal database domain. |
| [`db_type_from_declared`](#db_type_from_declared) | function | Maps a declared SQL column type onto the universal `DbType`. |
| [`normalize_db_value_for_column`](#normalize_db_value_for_column) | function | Normalizes one result cell against its declared column `DbType`. |

## Enums

### DbType

```rust
pub enum DbType {
    Unspecified,
    Bool,
    Int64,
    Float64,
    Text,
    Bytes,
}
```

Expected SQL type for a typed null (Cap'n `DbType`).

**Variants**

- `Unspecified`: Adapter may infer a type (unspecified null).
- `Bool`: Boolean null.
- `Int64`: Signed 64-bit integer null.
- `Float64`: Finite 64-bit float null.
- `Text`: UTF-8 text null.
- `Bytes`: Bytea / blob null.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<DbType>`, `PartialEq`, `Serialize`

### DbValue

```rust
pub enum DbValue {
    Null(DbType),
    Boolean(bool),
    Int64(i64),
    Float64(f64),
    Text(String),
    Bytes(Vec<u8>),
}
```

One parameter or cell in the universal database domain.

**Variants**

- `Null`: Typed SQL null.
- `Boolean`: Boolean.
- `Int64`: Signed 64-bit integer.
- `Float64`: Finite 64-bit float. Non-finite values are rejected at the bridge.
- `Text`: UTF-8 text.
- `Bytes`: Opaque bytes (not a `b64:` string).

#### DbValue::null

```rust
pub const fn null(ty: DbType) -> Self
```

Typed null of `ty`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `PartialEq`, `Serialize`

## Functions

### db_type_from_declared

```rust
pub fn db_type_from_declared(decl: &str) -> DbType
```

Maps a declared SQL column type onto the universal [`DbType`](#dbtype).

Shared by every adapter so declared-type result normalization is
identical across engines (SQLite decltype, D1 `pragma_table_info` type,
PostgreSQL type names). Unknown declarations map to
[`DbType::Unspecified`](#dbtype).

### normalize_db_value_for_column

```rust
pub fn normalize_db_value_for_column(value: DbValue, declared: DbType) -> DbValue
```

Normalizes one result cell against its declared column [`DbType`](#dbtype).

The universal-value contract requires identical observable `DbValue`
variants across adapters, so declared-type metadata (never engine storage
affinity) decides the variant:

- every SQL NULL in a declared column becomes `Null(<declared>)`;
- `0` / `1` integers in a declared `BOOL` column become [`DbValue::Boolean`](#dbvalue)
  (SQLite-family engines store booleans as INTEGER);
- integers in a declared `FLOAT64` column become [`DbValue::Float64`](#dbvalue)
  (JSON channels drop the `.0` on whole floats).

Cells in undeclared / computed columns are returned unchanged.
