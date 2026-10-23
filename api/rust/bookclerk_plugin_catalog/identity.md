**bookclerk_plugin_catalog > identity**

# Module: identity

## Contents

**Structs**

- [`ArtifactIdentity`](#artifactidentity) - Identity of one exact installed build: [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey) + version + hashes.
- [`PlatformArtifact`](#platformartifact) - One Bookclerk-shipped platform package.
- [`PluginInstallIdentity`](#plugininstallidentity) - Evaluated identity for one discovered install directory.
- [`PluginKey`](#pluginkey) - Stable identity across upgrades (not version, not artifact hash).

**Enums**

- [`PluginProvenance`](#pluginprovenance) - How Bookclerk established (or lost) trust in an installed tree.
- [`ProvenanceScheme`](#provenancescheme) - Provenance scheme in a canonical [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey).

**Functions**

- [`is_first_party_database_adapter`](#is_first_party_database_adapter) - True when `key` names a host-controlled first-party database adapter.
- [`is_first_party_destination`](#is_first_party_destination) - True when `key` names a host-controlled first-party destination (`local` / `s3`).
- [`is_first_party_integration`](#is_first_party_integration) - True when `key` names a host-controlled first-party integration.
- [`is_platform_plugin_key`](#is_platform_plugin_key) - True when `key` is a known Bookclerk platform [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey).
- [`platform_artifact`](#platform_artifact) - Looks up a known platform artifact by package name + manifest id.

**Constants**

- [`CRATES_IO_INDEX`](#crates_io_index) - Default Cargo registry origin used in canonical `cargo:` PluginKeys.
- [`FIRST_PARTY_DATABASE_ADAPTERS`](#first_party_database_adapters) - Host-controlled first-party database adapter identities.
- [`FIRST_PARTY_DESTINATIONS`](#first_party_destinations) - Host-controlled first-party destination identities (`local`, `s3`).
- [`PLATFORM_ARTIFACTS`](#platform_artifacts) - Known installer-shipped platform packages. Host-controlled; never read from
- [`PLATFORM_PRODUCT`](#platform_product) - Product name recorded on Bookclerk-shipped platform artifacts.
- [`PLUGIN_KEY_FS_ID_HEX_CHARS`](#plugin_key_fs_id_hex_chars) - Hex characters of SHA-256 used in `PluginKey::fs_id` (128 bits).

---

## bookclerk_plugin_catalog::identity::ArtifactIdentity

*Struct*

Identity of one exact installed build: [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey) + version + hashes.

**Fields:**
- `plugin_key: PluginKey` - Logical plugin (stable across upgrades).
- `version: String` - Package / manifest version string.
- `manifest_sha256: String` - SHA-256 of the installed `plugin.toml` bytes.
- `payload_root_sha256: String` - Deterministic Merkle-style root over the immutable installed payload.
- `archive_sha256: Option<String>` - SHA-256 of the downloaded archive, when the install came from a registry.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ArtifactIdentity`
- **PartialEq**
  - `fn eq(self: &Self, other: &ArtifactIdentity) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::identity::CRATES_IO_INDEX

*Constant*: `&str`

Default Cargo registry origin used in canonical `cargo:` PluginKeys.



## bookclerk_plugin_catalog::identity::FIRST_PARTY_DATABASE_ADAPTERS

*Constant*: `&[PlatformArtifact]`

Host-controlled first-party database adapter identities.

Secret injection, library-path grants, and host-private connect params
require this exact package + manifest id **and** a ledger-backed
provenance. A third-party tree that merely reuses the `sqlite` /
`postgres` / `d1` alias never matches.



## bookclerk_plugin_catalog::identity::FIRST_PARTY_DESTINATIONS

*Constant*: `&[PlatformArtifact]`

Host-controlled first-party destination identities (`local`, `s3`).

Typed `[output.local]` / `[output.s3]` secrets and extra jail grants require
this exact package on `platform:` or the canonical crates.io `cargo:` index
**and** a verified artifact. A third-party tree that merely reuses the
`local` / `s3` alias never matches. The live manifest alias is **not**
consulted here.



## bookclerk_plugin_catalog::identity::PLATFORM_ARTIFACTS

*Constant*: `&[PlatformArtifact]`

Known installer-shipped platform packages. Host-controlled; never read from
plugin-authored manifest fields other than matching `id` after provenance
is already established.



## bookclerk_plugin_catalog::identity::PLATFORM_PRODUCT

*Constant*: `&str`

Product name recorded on Bookclerk-shipped platform artifacts.



## bookclerk_plugin_catalog::identity::PLUGIN_KEY_FS_ID_HEX_CHARS

*Constant*: `usize`

Hex characters of SHA-256 used in `PluginKey::fs_id` (128 bits).



## bookclerk_plugin_catalog::identity::PlatformArtifact

*Struct*

One Bookclerk-shipped platform package.

**Fields:**
- `package_name: &'static str` - Cargo package name staged by `install-platform`.
- `manifest_id: &'static str` - Manifest alias (`sqlite`, `local`).
- `allowed_entrypoints: &'static [&'static str]` - Entrypoints the installer envelope permits.
- `allowed_bindings: &'static [&'static str]` - Host bindings the installer envelope permits.

**Methods:**

- `fn plugin_key(self: Self) -> PluginKey` - [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey) for this platform package.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlatformArtifact) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PlatformArtifact`



## bookclerk_plugin_catalog::identity::PluginInstallIdentity

*Struct*

Evaluated identity for one discovered install directory.

**Fields:**
- `plugin_key: PluginKey` - Stable logical identity (provenance + package; not the alias).
- `alias: String` - Human-facing alias from `plugin.toml` `id`.
- `artifact: ArtifactIdentity` - Exact installed build.
- `provenance: PluginProvenance` - Host-evaluated provenance (never plugin-authored).

**Methods:**

- `fn alias(self: &Self) -> &str` - Display alias (`plugin.toml` id). Never a security principal.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginInstallIdentity`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginInstallIdentity) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_catalog::identity::PluginKey

*Struct*

Stable identity across upgrades (not version, not artifact hash).

Canonical text (deterministic, unambiguous). Registry index URL and
package name are `#`-separated so a registry path cannot be confused
with a package path. The human alias is **not** included (it lives on
the manifest / receipt `runtime.id`):

- `cargo:{index_url}#{package}`
- `npm:{index_url}#{package}`
- `pypi:{index_url}#{package}`
- `registry:{index_url}#{package}`
- `platform:bookclerk/{package}`
- `path:file://{normalized_absolute_path}`

**Methods:**

- `fn parse(raw: &str) -> Result<Self>` - Builds a key from already-canonical text. Prefer the typed constructors.
- `fn platform(package_name: &str, manifest_id: &str) -> Result<Self>` - Platform-bundled key (`platform:bookclerk/{package}`).
- `fn from_coordinate(coordinate: &PackageCoordinate, manifest_id: &str) -> Result<Self>` - Registry-sourced key (Cargo / npm / PyPI / static index).
- `fn from_install_path(root: &Path, manifest_id: &str) -> Result<Self>` - Filesystem / development-tree key for an install directory.
- `fn canonical(self: &Self) -> &str` - Canonical textual form.
- `fn digest_sha256(self: &Self) -> String` - SHA-256 (hex) of the canonical UTF-8 bytes.
- `fn fs_id(self: &Self) -> String` - Filesystem-safe directory leaf (`pk-` + 32 hex chars / 128 bits of SHA-256).
- `fn scheme(self: &Self) -> ProvenanceScheme` - Provenance scheme.
- `fn package(self: &Self) -> String` - Package coordinate / name when present (empty for `path:` keys).

**Traits:** Eq

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> std::result::Result<<S as >::Ok, <S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginKey`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginKey) -> bool`
- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> std::result::Result<Self, <D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **FromStr**
  - `fn from_str(s: &str) -> Result<Self>`



## bookclerk_plugin_catalog::identity::PluginProvenance

*Enum*

How Bookclerk established (or lost) trust in an installed tree.

**Variants:**
- `PlatformBundled` - Host-staged Bookclerk platform artifact; content hashes match the receipt.
- `VerifiedStaged` - Installed from a registry/archive; receipt hashes still match.
- `LocalDevelopment` - Local/dev tree with no verified receipt (or path-only identity).
- `Modified` - Receipt exists but the immutable payload no longer matches.
- `Installed` - Materialized from a cluster install: the payload-root digest of the

**Methods:**

- `fn grants_platform_defaults(self: Self) -> bool` - True when this provenance may receive installer platform default grants.
- `fn is_verified_artifact(self: Self) -> bool` - True when discovery verified the installed payload against the host ledger.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginProvenance) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginProvenance`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`



## bookclerk_plugin_catalog::identity::ProvenanceScheme

*Enum*

Provenance scheme in a canonical [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey).

**Variants:**
- `Cargo` - crates.io / Cargo registry.
- `Npm` - npm registry.
- `Pypi` - PyPI.
- `Registry` - Static HTTPS plugin index.
- `Platform` - Bookclerk-shipped installer artifact.
- `Path` - Local filesystem install / development tree.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Canonical scheme token used in [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey) text.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialEq**
  - `fn eq(self: &Self, other: &ProvenanceScheme) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ProvenanceScheme`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_catalog::identity::is_first_party_database_adapter

*Function*

True when `key` names a host-controlled first-party database adapter.

Matches exact package name on `platform:` or the canonical crates.io
`cargo:` index. Callers that inject host-private state must also require
verified provenance and the expected alias on the discovered plugin.

```rust
fn is_first_party_database_adapter(key: &PluginKey) -> bool
```



## bookclerk_plugin_catalog::identity::is_first_party_destination

*Function*

True when `key` names a host-controlled first-party destination (`local` / `s3`).

```rust
fn is_first_party_destination(key: &PluginKey) -> bool
```



## bookclerk_plugin_catalog::identity::is_first_party_integration

*Function*

True when `key` names a host-controlled first-party integration.

Matches exact package name on `platform:` or the canonical crates.io
`cargo:` index. Callers that grant host-implied network access must also
require verified provenance on the discovered plugin.

```rust
fn is_first_party_integration(key: &PluginKey) -> bool
```



## bookclerk_plugin_catalog::identity::is_platform_plugin_key

*Function*

True when `key` is a known Bookclerk platform [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey).

```rust
fn is_platform_plugin_key(key: &PluginKey) -> bool
```



## bookclerk_plugin_catalog::identity::platform_artifact

*Function*

Looks up a known platform artifact by package name + manifest id.

```rust
fn platform_artifact(package_name: &str, manifest_id: &str) -> Option<&'static PlatformArtifact>
```



