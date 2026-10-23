**bookclerk_library > backup > encode**

# Module: backup::encode

## Contents

**Enums**

- [`CanonicalObject`](#canonicalobject) - One immutable canonical object (hashed before compression).

**Constants**

- [`CHUNK_TARGET_UNCOMPRESSED_BYTES`](#chunk_target_uncompressed_bytes) - Target uncompressed JSON size for one table-data chunk.

---

## bookclerk_library::backup::encode::CHUNK_TARGET_UNCOMPRESSED_BYTES

*Constant*: `usize`

Target uncompressed JSON size for one table-data chunk.

256 KiB sits in a 128–512 KiB band: large enough to avoid pathological
per-object overhead on typical catalog rows, small enough for a small-VPS
working set and for `maxResultBytes` paging. A single row larger than the
target still occupies its own chunk.



## bookclerk_library::backup::encode::CanonicalObject

*Enum*

One immutable canonical object (hashed before compression).

**Variants:**
- `Schema{ sql_contract_version: u32, statements: Vec<String> }` - Admitted `CREATE TABLE` / `CREATE INDEX` in restore order.
- `TableChunk{ table: String, columns: Vec<String>, rows: Vec<Vec<bookclerk_plugin_abi::DbValue>> }` - Bounded table rows in canonical order.
- `Identity{ entries: std::collections::BTreeMap<String, super::IdentityHighWater> }` - Identity high-water for generated columns.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CanonicalObject) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CanonicalObject`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



