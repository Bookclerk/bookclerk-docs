**bookclerk_plugin_catalog > target**

# Module: target

## Contents

**Enums**

- [`ArchiveFormat`](#archiveformat) - Archive format for a target.

**Functions**

- [`host_bookclerk_target`](#host_bookclerk_target) - Host Bookclerk target for the machine compiling this code.
- [`normalize_target`](#normalize_target) - Map a rustc triple (or Bookclerk target) to a Bookclerk target name.
- [`rust_triple`](#rust_triple) - Rustc triple corresponding to a Bookclerk target (for packaging).
- [`select_target`](#select_target) - Select an artifact entry matching the host (or override) target.

**Constants**

- [`TARGETS`](#targets) - Canonical Bookclerk install targets (not raw rustc triples).

---

## bookclerk_plugin_catalog::target::ArchiveFormat

*Enum*

Archive format for a target.

**Variants:**
- `TarGz` - Gzip-compressed tar archive (default for non-Windows targets).
- `Zip` - Zip archive (default for Windows targets).

**Methods:**

- `fn for_target(bookclerk_target: &str) -> Self` - Returns the conventional archive format for a Bookclerk target id.
- `fn extension(self: Self) -> &'static str` - File extension without dot (for example `m4b` or `mp3`).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ArchiveFormat) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ArchiveFormat`



## bookclerk_plugin_catalog::target::TARGETS

*Constant*: `&[&str]`

Canonical Bookclerk install targets (not raw rustc triples).



## bookclerk_plugin_catalog::target::host_bookclerk_target

*Function*

Host Bookclerk target for the machine compiling this code.

```rust
fn host_bookclerk_target() -> &'static str
```



## bookclerk_plugin_catalog::target::normalize_target

*Function*

Map a rustc triple (or Bookclerk target) to a Bookclerk target name.

```rust
fn normalize_target(triple_or_target: &str) -> Option<&'static str>
```



## bookclerk_plugin_catalog::target::rust_triple

*Function*

Rustc triple corresponding to a Bookclerk target (for packaging).

```rust
fn rust_triple(bookclerk_target: &str) -> Option<&'static str>
```



## bookclerk_plugin_catalog::target::select_target

*Function*

Select an artifact entry matching the host (or override) target.

# Errors

Returns an error when the operation fails.

```rust
fn select_target<'a, T, impl Fn(&T) -> &str>(artifacts: &'a [T], target_of: impl Trait, prefer: Option<&str>) -> crate::error::Result<&'a T>
```



