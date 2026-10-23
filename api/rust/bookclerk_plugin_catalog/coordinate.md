**bookclerk_plugin_catalog > coordinate**

# Module: coordinate

## Contents

**Structs**

- [`PackageCoordinate`](#packagecoordinate) - Exact package coordinate: source + name + version.

**Enums**

- [`RegistrySource`](#registrysource) - Where a package was discovered / will be fetched from.

---

## bookclerk_plugin_catalog::coordinate::PackageCoordinate

*Struct*

Exact package coordinate: source + name + version.

Display forms:
- `cargo:bookclerk-plugin-source-example@1.2.3`
- `npm:@publisher/bookclerk-plugin-source-example@1.2.3`
- `pypi:bookclerk-plugin-source-example==1.2.3`
- `registry:https://example.com/index.json#community/example@1.2.3`
- `local:/path/to/archive.tar.gz` (version may be `0.0.0` placeholder)

**Fields:**
- `source: RegistrySource` - Registry source (crates.io, npm, static index URL, …).
- `name: String` - Package name within `source`.
- `version: String` - Resolved or candidate package version string.

**Methods:**

- `fn parse(raw: &str) -> Result<Self>` - Parse a source-qualified coordinate string.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PackageCoordinate) -> bool`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PackageCoordinate`



## bookclerk_plugin_catalog::coordinate::RegistrySource

*Enum*

Where a package was discovered / will be fetched from.

**Variants:**
- `Cargo{ registry_url: String }` - crates.io (or alternate Cargo registry).
- `Npm{ registry_url: String }` - npm registry.
- `Pypi{ simple_url: String }` - PyPI (exact lookup only for search-less discovery).
- `Static{ index_url: String }` - Static HTTPS JSON/TOML index.
- `LocalArchive` - Local archive path (offline / fixture install).

**Methods:**

- `fn kind_name(self: &Self) -> &'static str` - Returns the short source kind name (`cargo`, `npm`, `pypi`, `registry`, `local`).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RegistrySource`
- **PartialEq**
  - `fn eq(self: &Self, other: &RegistrySource) -> bool`



