**bookclerk_plugin_catalog > adapters > npm**

# Module: adapters::npm

## Contents

**Structs**

- [`NpmAdapter`](#npmadapter) - npm registry adapter.

---

## bookclerk_plugin_catalog::adapters::npm::NpmAdapter

*Struct*

npm registry adapter.

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



