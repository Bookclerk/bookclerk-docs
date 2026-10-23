**bookclerk_plugin_catalog > adapters > cargo**

# Module: adapters::cargo

## Contents

**Structs**

- [`CargoAdapter`](#cargoadapter) - crates.io discovery + `.crate` metadata hydration.

---

## bookclerk_plugin_catalog::adapters::cargo::CargoAdapter

*Struct*

crates.io discovery + `.crate` metadata hydration.

**Fields:**
- `registry_url: String` - Base URL of the package registry (for example crates.io or npm).

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **RegistryAdapter**
  - `fn source_kind(self: &Self) -> &'static str`
  - `fn search(self: &Self, q: &SearchQuery) -> Result<Vec<CatalogHit>>`
  - `fn fetch_manifest(self: &Self, coord: &PackageCoordinate) -> Result<BookclerkPackageManifest>`
  - `fn list_versions(self: &Self, name: &str) -> Result<Vec<String>>`



