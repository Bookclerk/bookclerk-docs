**bookclerk_plugin_catalog > catalog**

# Module: catalog

## Contents

**Structs**

- [`CatalogHit`](#cataloghit) - One discovery hit from any registry adapter.
- [`SearchQuery`](#searchquery) - Search query passed to adapters.

**Constants**

- [`CATALOG_DTO_SCHEMA_VERSION`](#catalog_dto_schema_version) - Schema version for CLI JSON catalog output.

---

## bookclerk_plugin_catalog::catalog::CATALOG_DTO_SCHEMA_VERSION

*Constant*: `u32`

Schema version for CLI JSON catalog output.



## bookclerk_plugin_catalog::catalog::CatalogHit

*Struct*

One discovery hit from any registry adapter.

**Fields:**
- `schema_version: u32` - DTO schema version for CLI/UI JSON compatibility.
- `coordinate: Option<crate::coordinate::PackageCoordinate>` - Source-qualified coordinate when version is known; otherwise name-only hint.
- `source_kind: String` - Registry adapter kind (`cargo`, `npm`, `pypi`, `static`, …).
- `package_name: String` - Package name in the source registry.
- `version: String` - Resolved or candidate package version string.
- `description: Option<String>` - Short package description from the registry, when present.
- `downloads: Option<u64>` - Download count from the registry index, when known.
- `repository: Option<String>` - Source repository URL from package metadata, when present.
- `homepage: Option<String>` - Project homepage URL from package metadata, when present.
- `documentation: Option<String>` - Documentation URL from package metadata, when present.
- `runtime: Option<crate::kind::RuntimeIdentity>` - Bookclerk runtime identity (kind + plugin id) when known.
- `manifest: Option<crate::manifest::BookclerkPackageManifest>` - Present after metadata hydration / static index lookup.

**Methods:**

- `fn from_cargo_search<impl Into<String>, impl Into<String>>(crate_name: impl Trait, version: impl Trait, kind: Option<PluginKind>, id: Option<String>, description: Option<String>, downloads: u64, repository: Option<String>, homepage: Option<String>, documentation: Option<String>) -> Self` - Build a hit from a crates.io-shaped search result.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogHit) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogHit`



## bookclerk_plugin_catalog::catalog::SearchQuery

*Struct*

Search query passed to adapters.

**Fields:**
- `text: Option<String>` - Free-text search string passed to registry adapters.
- `limit: u32` - Maximum hits to return (clamped by the caller).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SearchQuery`
- **Default**
  - `fn default() -> SearchQuery`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



