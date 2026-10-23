**bookclerk_library > store > plugin_kv**

# Module: store::plugin_kv

## Contents

**Structs**

- [`KvEntry`](#kventry) - One live key/value row.
- [`KvListPage`](#kvlistpage) - One page of `LibraryStore::kv_list`.
- [`KvWriteSpec`](#kvwritespec) - A write plus the namespace limits it must satisfy.

**Enums**

- [`KvPutOutcome`](#kvputoutcome) - Why a write was refused, so the binding can map it onto a wire code.

**Constants**

- [`KV_LIST_DEFAULT_LIMIT`](#kv_list_default_limit) - Default page size for `LibraryStore::kv_list` when the caller passes 0.

---

## bookclerk_library::store::plugin_kv::KV_LIST_DEFAULT_LIMIT

*Constant*: `u32`

Default page size for `LibraryStore::kv_list` when the caller passes 0.



## bookclerk_library::store::plugin_kv::KvEntry

*Struct*

One live key/value row.

**Fields:**
- `key: String` - Key within the namespace.
- `value: Vec<u8>` - Value bytes.
- `metadata: String` - Opaque JSON metadata text; empty when none.
- `expires_at: Option<chrono::DateTime<chrono::Utc>>` - Absolute expiry, when set.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvEntry`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvEntry) -> bool`



## bookclerk_library::store::plugin_kv::KvListPage

*Struct*

One page of `LibraryStore::kv_list`.

**Fields:**
- `keys: Vec<KvEntry>` - Keys in lexical order (values are not loaded).
- `list_complete: bool` - True when no further page exists.
- `cursor: String` - Opaque cursor for the next page; empty when complete.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvListPage`
- **Default**
  - `fn default() -> KvListPage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvListPage) -> bool`



## bookclerk_library::store::plugin_kv::KvPutOutcome

*Enum*

Why a write was refused, so the binding can map it onto a wire code.

**Variants:**
- `Written` - The row was written.
- `ValueTooLarge{ len: usize, max: u32 }` - `value` exceeds `max_value_bytes`.
- `KeyLimitReached{ max: u32 }` - The namespace already holds `max_keys` other live keys.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvPutOutcome) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> KvPutOutcome`



## bookclerk_library::store::plugin_kv::KvWriteSpec

*Struct*

A write plus the namespace limits it must satisfy.

**Fields:**
- `value: Vec<u8>` - Value bytes.
- `metadata: String` - Opaque JSON metadata text; empty when none.
- `expires_at: Option<chrono::DateTime<chrono::Utc>>` - Absolute expiry; `None` never expires.
- `max_value_bytes: u32` - Largest value this namespace accepts.
- `max_keys: u32` - Most live keys this namespace holds.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvWriteSpec`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvWriteSpec) -> bool`



