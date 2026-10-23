**bookclerk_plugin_catalog > adapters > pypi**

# Module: adapters::pypi

## Contents

**Structs**

- [`PypiAdapter`](#pypiadapter) - PyPI JSON API adapter. Search is not supported by PyPI; use static registries

---

## bookclerk_plugin_catalog::adapters::pypi::PypiAdapter

*Struct*

PyPI JSON API adapter. Search is not supported by PyPI; use static registries
for discovery and exact `pypi:name==version` coordinates for install.

**Fields:**
- `base_url: String` - PyPI simple/API base URL for this adapter.

**Trait Implementations:**

- **RegistryAdapter**
  - `fn source_kind(self: &Self) -> &'static str`
  - `fn search(self: &Self, q: &SearchQuery) -> Result<Vec<CatalogHit>>`
  - `fn fetch_manifest(self: &Self, coord: &PackageCoordinate) -> Result<BookclerkPackageManifest>`
  - `fn list_versions(self: &Self, name: &str) -> Result<Vec<String>>`
- **Default**
  - `fn default() -> Self`



