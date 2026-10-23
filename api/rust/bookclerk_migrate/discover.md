**bookclerk_migrate > discover**

# Module: discover

## Contents

**Structs**

- [`ClassicSource`](#classicsource) - Paths discovered in a classic Libation Files directory.

**Functions**

- [`discover_source`](#discover_source) - Discover importable files under `root`.

---

## bookclerk_migrate::discover::ClassicSource

*Struct*

Paths discovered in a classic Libation Files directory.

**Fields:**
- `root: std::path::PathBuf` - Filesystem root of the classic Libation Files directory.
- `settings_json: Option<std::path::PathBuf>` - Path to Libation `Settings.json` when present.
- `accounts_settings: Option<std::path::PathBuf>` - Path to Libation `AccountsSettings.json` when present.
- `library_db: Option<std::path::PathBuf>` - Path to Libation `libation.db` (or equivalent) when present.
- `file_locations: Option<std::path::PathBuf>` - Path to Libation file-locations JSON when present.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ClassicSource`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::discover::discover_source

*Function*

Discover importable files under `root`.

Accepts either a classic Libation Files directory or a directory that contains
one of the expected files (for Docker `/config` layouts).

# Arguments

* `root` - Cargo workspace root directory.

# Returns

On success, the inner `ClassicSource` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn discover_source(root: &std::path::Path) -> crate::error::Result<ClassicSource>
```



