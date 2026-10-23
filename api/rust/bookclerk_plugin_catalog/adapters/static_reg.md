**bookclerk_plugin_catalog > adapters > static_reg**

# Module: adapters::static_reg

## Contents

**Structs**

- [`StaticAdapter`](#staticadapter) - Adapter over an in-memory or remote static index.
- [`StaticIndex`](#staticindex) - Top-level static index document.
- [`StaticPackage`](#staticpackage) - One package entry in a static registry index.

**Functions**

- [`load_static_index`](#load_static_index) - Load a static index from a file path or `file://` URL.

---

## bookclerk_plugin_catalog::adapters::static_reg::StaticAdapter

*Struct*

Adapter over an in-memory or remote static index.

**Fields:**
- `index_url: String` - Absolute URL of the static registry index document.

**Methods:**

- `fn open<impl Into<String>>(index_url: impl Trait) -> Result<Self>` - Downloads and parses a static registry index from `index_url`.
- `fn from_index<impl Into<String>>(index_url: impl Trait, index: StaticIndex) -> Self` - Wraps an already-parsed static index without fetching.

**Trait Implementations:**

- **RegistryAdapter**
  - `fn source_kind(self: &Self) -> &'static str`
  - `fn search(self: &Self, q: &SearchQuery) -> Result<Vec<CatalogHit>>`
  - `fn fetch_manifest(self: &Self, coord: &PackageCoordinate) -> Result<BookclerkPackageManifest>`
  - `fn list_versions(self: &Self, name: &str) -> Result<Vec<String>>`



## bookclerk_plugin_catalog::adapters::static_reg::StaticIndex

*Struct*

Top-level static index document.

**Fields:**
- `schema_version: u32` - DTO schema version for CLI/UI JSON compatibility.
- `packages: Vec<StaticPackage>` - Packages listed in this static registry index.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> StaticIndex`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_catalog::adapters::static_reg::StaticPackage

*Struct*

One package entry in a static registry index.

**Fields:**
- `name: String` - Package name within `source`.
- `versions: std::collections::BTreeMap<String, crate::manifest::BookclerkPackageManifest>` - Known versions and manifests for this static package.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> StaticPackage`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::adapters::static_reg::load_static_index

*Function*

Load a static index from a file path or `file://` URL.

# Errors

Returns an error when the operation fails.

```rust
fn load_static_index(path_or_url: &str) -> crate::error::Result<StaticIndex>
```



