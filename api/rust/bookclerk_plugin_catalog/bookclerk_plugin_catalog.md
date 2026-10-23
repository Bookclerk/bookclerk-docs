**bookclerk_plugin_catalog**

# Module: bookclerk_plugin_catalog

## Contents

**Functions**

- [`federated_search`](#federated_search) - Federated search across configured adapters (static first, then cargo/npm).
- [`fetch_manifest_for_coordinate`](#fetch_manifest_for_coordinate) - Resolve a coordinate to a manifest using the matching adapter.
- [`list_versions_for_coordinate`](#list_versions_for_coordinate) - List published versions for a coordinate's package name.
- [`resolve_newer_version`](#resolve_newer_version) - Resolve the newest version newer than `coord.version`, if any.

---

## bookclerk_plugin_catalog::federated_search

*Function*

Federated search across configured adapters (static first, then cargo/npm).

# Arguments

* `adapters` - Registry adapters to query in order.
* `query` - Free-text query and hit limit.

# Returns

Combined catalog hits truncated to `query.limit` (clamped 1..=100).

# Errors

Returns [`CatalogError`](error.md#bookclerk_plugin_catalogerrorcatalogerror) when every adapter fails and no hits were collected.

```rust
fn federated_search(adapters: &[&dyn RegistryAdapter], query: &SearchQuery) -> Result<Vec<CatalogHit>>
```



## bookclerk_plugin_catalog::fetch_manifest_for_coordinate

*Function*

Resolve a coordinate to a manifest using the matching adapter.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_manifest_for_coordinate(coord: &PackageCoordinate, static_indexes: &[(String, StaticIndex)]) -> Result<BookclerkPackageManifest>
```



## bookclerk_plugin_catalog::list_versions_for_coordinate

*Function*

List published versions for a coordinate's package name.

# Errors

Returns an error when the operation fails.

```rust
fn list_versions_for_coordinate(coord: &PackageCoordinate, static_indexes: &[(String, StaticIndex)]) -> Result<Vec<String>>
```



## bookclerk_plugin_catalog::resolve_newer_version

*Function*

Resolve the newest version newer than `coord.version`, if any.

# Errors

Returns an error when the operation fails.

```rust
fn resolve_newer_version(coord: &PackageCoordinate, static_indexes: &[(String, StaticIndex)]) -> Result<Option<String>>
```



