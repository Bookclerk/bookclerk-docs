**bookclerk_plugin_host > registry**

# Module: registry

## Contents

**Structs**

- [`BookclerkPackageMetadata`](#bookclerkpackagemetadata) - `[package.metadata.bookclerk]` published on crates.io.
- [`PluginCatalogEntry`](#plugincatalogentry) - One catalog hit from crates.io (or a curated index).
- [`PluginCrateName`](#plugincratename) - Parsed crates.io crate name for a Bookclerk plugin.

**Functions**

- [`family_keyword`](#family_keyword) - Family-specific crates.io keyword (`bookclerk-source`, …).
- [`host_target_triple`](#host_target_triple) - Host target triple used when selecting release assets.
- [`validate_plugin_id`](#validate_plugin_id) - Validate plugin id segment used in crate names and `plugin.toml`.

**Constants**

- [`CRATE_NAME_PREFIX`](#crate_name_prefix) - Required crates.io name prefix: `bookclerk-plugin-{family}-{id}`.
- [`PRODUCT_KEYWORD`](#product_keyword) - Shared crates.io keyword (`bookclerk`) expected on published plugin crates.
- [`REGISTRY_KEYWORD`](#registry_keyword) - Keyword every published plugin crate should include.

---

## bookclerk_plugin_host::registry::BookclerkPackageMetadata

*Struct*

`[package.metadata.bookclerk]` published on crates.io.

**Fields:**
- `api_version: u32` - Plugin ABI / package metadata API version (must be >= 1).
- `family: crate::manifest::PluginFamily` - Plugin family encoded in the crates.io name.
- `id: String` - Plugin id (`[a-z0-9_]{2,32}`), unique across kinds in one host namespace.
- `display_name: Option<String>` - Optional human-readable name for catalog UI.
- `artifact_base_url: Option<String>` - Template with `{tag}`, `{version}`, `{target}`, `{crate}` placeholders.
- `artifact_url: Option<String>` - Full download URL template (overrides base + conventional filename).
- `archive_root: Option<String>` - Optional path inside the release archive where `plugin.toml` lives.
- `min_host: Option<String>` - Optional minimum Bookclerk host version required to run this plugin.

**Methods:**

- `fn validate_against_crate_name(self: &Self, crate_name: &str) -> Result<()>` - Ensure metadata matches the crate naming taxonomy.
- `fn artifact_download_url(self: &Self, crate_name: &str, version: &str, target: &str) -> Option<String>` - Build a download URL for a release asset.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BookclerkPackageMetadata`
- **PartialEq**
  - `fn eq(self: &Self, other: &BookclerkPackageMetadata) -> bool`



## bookclerk_plugin_host::registry::CRATE_NAME_PREFIX

*Constant*: `&str`

Required crates.io name prefix: `bookclerk-plugin-{family}-{id}`.



## bookclerk_plugin_host::registry::PRODUCT_KEYWORD

*Constant*: `&str`

Shared crates.io keyword (`bookclerk`) expected on published plugin crates.



## bookclerk_plugin_host::registry::PluginCatalogEntry

*Struct*

One catalog hit from crates.io (or a curated index).

**Fields:**
- `crate_name: String` - crates.io package name (`bookclerk-plugin-{family}-{id}`).
- `version: String` - Published crate / release version string.
- `description: Option<String>` - Short crates.io description when the index provides one.
- `downloads: u64` - Download count from the registry index, when known.
- `documentation: Option<String>` - Documentation URL from crate metadata, when present.
- `repository: Option<String>` - Source repository URL from crate metadata, when present.
- `homepage: Option<String>` - Project homepage URL from crate metadata, when present.
- `parsed: Option<PluginCrateName>` - Parsed from crate name when it matches the taxonomy.
- `metadata: Option<BookclerkPackageMetadata>` - Present when crates.io returned Cargo.toml metadata (install phase).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginCatalogEntry`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginCatalogEntry) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::registry::PluginCrateName

*Struct*

Parsed crates.io crate name for a Bookclerk plugin.

**Fields:**
- `family: crate::manifest::PluginFamily` - Plugin family encoded in the crates.io name.
- `id: String` - Plugin id (`[a-z0-9_]{2,32}`), unique across kinds in one host namespace.

**Methods:**

- `fn crate_name(self: &Self) -> String` - Format as `bookclerk-plugin-{family}-{id}`.
- `fn parse(name: &str) -> Result<Self>` - Parse `bookclerk-plugin-{family}-{id}`.

**Traits:** Eq

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginCrateName`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginCrateName) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::registry::REGISTRY_KEYWORD

*Constant*: `&str`

Keyword every published plugin crate should include.



## bookclerk_plugin_host::registry::family_keyword

*Function*

Family-specific crates.io keyword (`bookclerk-source`, …).

```rust
fn family_keyword(family: crate::manifest::PluginFamily) -> &'static str
```



## bookclerk_plugin_host::registry::host_target_triple

*Function*

Host target triple used when selecting release assets.

```rust
fn host_target_triple() -> &'static str
```



## bookclerk_plugin_host::registry::validate_plugin_id

*Function*

Validate plugin id segment used in crate names and `plugin.toml`.

Delegates to [`bookclerk_plugin_manifest::validate_plugin_id`](../bookclerk_plugin_manifest/id.md#bookclerk_plugin_manifestidvalidate_plugin_id) (strict
`[a-z0-9_]{2,32}` grammar; unique across kinds in one host plugin namespace).

# Errors

Returns an error when the operation fails.

```rust
fn validate_plugin_id(id: &str) -> crate::Result<()>
```



