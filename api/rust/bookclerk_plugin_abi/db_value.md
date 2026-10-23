**bookclerk_plugin_abi > db_value**

# Module: db_value

## Contents

**Enums**

- [`DbType`](#dbtype) - Expected SQL type for a typed null (Cap'n `DbType`).
- [`DbValue`](#dbvalue) - One parameter or cell in the universal database domain.

**Functions**

- [`db_type_from_declared`](#db_type_from_declared) - Maps a declared SQL column type onto the universal [`DbType`](db_value.md#bookclerk_plugin_abidb_valuedbtype).
- [`normalize_db_value_for_column`](#normalize_db_value_for_column) - Normalizes one result cell against its declared column [`DbType`](db_value.md#bookclerk_plugin_abidb_valuedbtype).

---

## bookclerk_plugin_abi::db_value::DbType

*Enum*

Expected SQL type for a typed null (Cap'n `DbType`).

**Variants:**
- `Unspecified` - Adapter may infer a type (unspecified null).
- `Bool` - Boolean null.
- `Int64` - Signed 64-bit integer null.
- `Float64` - Finite 64-bit float null.
- `Text` - UTF-8 text null.
- `Bytes` - Bytea / blob null.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbType) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbType`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> DbType`



## bookclerk_plugin_abi::db_value::DbValue

*Enum*

One parameter or cell in the universal database domain.

**Variants:**
- `Null(DbType)` - Typed SQL null.
- `Boolean(bool)` - Boolean.
- `Int64(i64)` - Signed 64-bit integer.
- `Float64(f64)` - Finite 64-bit float. Non-finite values are rejected at the bridge.
- `Text(String)` - UTF-8 text.
- `Bytes(Vec<u8>)` - Opaque bytes (not a `b64:` string).

**Methods:**

- `fn null(ty: DbType) -> Self` - Typed null of `ty`.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbValue) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbValue`



## bookclerk_plugin_abi::db_value::db_type_from_declared

*Function*

Maps a declared SQL column type onto the universal [`DbType`](db_value.md#bookclerk_plugin_abidb_valuedbtype).

Shared by every adapter so declared-type result normalization is
identical across engines (SQLite decltype, D1 `pragma_table_info` type,
PostgreSQL type names). Unknown declarations map to
`DbType::Unspecified`.

```rust
fn db_type_from_declared(decl: &str) -> DbType
```



## bookclerk_plugin_abi::db_value::normalize_db_value_for_column

*Function*

Normalizes one result cell against its declared column [`DbType`](db_value.md#bookclerk_plugin_abidb_valuedbtype).

The universal-value contract requires identical observable `DbValue`
variants across adapters, so declared-type metadata (never engine storage
affinity) decides the variant:

- every SQL NULL in a declared column becomes `Null(<declared>)`;
- `0` / `1` integers in a declared `BOOL` column become `DbValue::Boolean`
  (SQLite-family engines store booleans as INTEGER);
- integers in a declared `FLOAT64` column become `DbValue::Float64`
  (JSON channels drop the `.0` on whole floats).

Cells in undeclared / computed columns are returned unchanged.

```rust
fn normalize_db_value_for_column(value: DbValue, declared: DbType) -> DbValue
```



