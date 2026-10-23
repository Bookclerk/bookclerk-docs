**bookclerk_plugin_catalog > manifest**

# Module: manifest

## Contents

**Structs**

- [`ArtifactTarget`](#artifacttarget) - One precompiled artifact for a Bookclerk target.
- [`BookclerkPackageManifest`](#bookclerkpackagemanifest) - Canonical install-grade package metadata.
- [`PackageLinks`](#packagelinks) - Optional documentation / support links.
- [`PublisherIdentity`](#publisheridentity) - Optional publisher signing identity (when available).
- [`SandboxRequest`](#sandboxrequest) - Publisher-requested sandbox (informational until host approves).

**Functions**

- [`normalize_protocol`](#normalize_protocol) - Normalize optional `protocol` for Workers RPC packages.
- [`parse_sha256_hex`](#parse_sha256_hex) - Decode hex SHA-256 into 32 bytes.
- [`validate_sha256_hex`](#validate_sha256_hex) - Validate lowercase hex SHA-256 (64 chars).

**Constants**

- [`MANIFEST_SCHEMA_VERSION`](#manifest_schema_version) - Current package manifest schema.
- [`PROTOCOL_WORKERS_RPC`](#protocol_workers_rpc) - Product wire protocol for Workers RPC (`api_version = 3`).

---

## bookclerk_plugin_catalog::manifest::ArtifactTarget

*Struct*

One precompiled artifact for a Bookclerk target.

**Fields:**
- `target: String` - Bookclerk target (`linux-x64-gnu`, …) or legacy rustc triple.
- `url: String` - HTTPS download URL (or `file:` for fixtures).
- `archive_sha256: String` - Lowercase hex SHA-256 of the archive bytes (required for unattended install).
- `archive_root: String` - Path inside the archive to the plugin root (default `.`).
- `executable: String` - Relative path to the executable inside the archive root.
- `executable_sha256: Option<String>` - Optional SHA-256 of the extracted executable.

**Methods:**

- `fn bookclerk_target(self: &Self) -> String` - Bookclerk target id selected for this host.
- `fn archive_format(self: &Self) -> ArchiveFormat` - Archive format for this artifact (`tar.gz` or `zip`).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ArtifactTarget`
- **PartialEq**
  - `fn eq(self: &Self, other: &ArtifactTarget) -> bool`



## bookclerk_plugin_catalog::manifest::BookclerkPackageManifest

*Struct*

Canonical install-grade package metadata.

**Fields:**
- `schema_version: u32` - DTO schema version for CLI/UI JSON compatibility.
- `protocol: Option<String>` - Optional wire label. Prefer absent; when present use `workers-rpc`.
- `api_version: u32` - Plugin ABI version negotiated with the host.
- `api_version_max: Option<u32>` - Maximum plugin ABI version this package claims to support.
- `min_bookclerk: Option<String>` - Optional minimum Bookclerk host version required to install/run.
- `kind: crate::kind::PluginKind` - Destination backend kind (`local`, `s3`, …).
- `id: String` - Plugin alias (`[a-z0-9_]{2,32}`), unique within one host plugin namespace.
- `display_name: Option<String>` - Optional human-readable name for catalog and UI.
- `description: Option<String>` - Short package description from the registry, when present.
- `coordinate: Option<crate::coordinate::PackageCoordinate>` - Filled by adapters; may be omitted in static index entries that nest under name/version.
- `artifacts: Vec<ArtifactTarget>` - Per-target downloadable artifacts for this package version.
- `sandbox: SandboxRequest` - Publisher-requested sandbox surface (informational until approved).
- `links: PackageLinks` - Optional documentation and support links.
- `yanked: bool` - When true, this version was yanked and must not be installed unattended.
- `released_at: Option<String>` - RFC 3339 release timestamp from the registry, when known.
- `publisher: Option<PublisherIdentity>` - Optional publisher identity for trust decisions.

**Methods:**

- `fn runtime(self: &Self) -> RuntimeIdentity` - Returns the [`RuntimeIdentity`](kind.md#bookclerk_plugin_catalogkindruntimeidentity) (kind + id) for this package.
- `fn effective_protocol(self: &Self) -> String` - Effective protocol string for receipts / wire defaults.
- `fn validate_for_install(self: &Self) -> Result<()>` - Validate schema + require digests for install-grade manifests.
- `fn from_json(text: &str) -> Result<Self>` - Parses a package manifest from a JSON document.
- `fn from_toml(text: &str) -> Result<Self>` - Parse from TOML (`[package.metadata.bookclerk]` style flat table or root).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BookclerkPackageManifest`
- **PartialEq**
  - `fn eq(self: &Self, other: &BookclerkPackageManifest) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::manifest::MANIFEST_SCHEMA_VERSION

*Constant*: `u32`

Current package manifest schema.



## bookclerk_plugin_catalog::manifest::PROTOCOL_WORKERS_RPC

*Constant*: `&str`

Product wire protocol for Workers RPC (`api_version = 3`).



## bookclerk_plugin_catalog::manifest::PackageLinks

*Struct*

Optional documentation / support links.

**Fields:**
- `repository: Option<String>` - Source repository URL from package metadata, when present.
- `documentation: Option<String>` - Documentation URL from package metadata, when present.
- `homepage: Option<String>` - Project homepage URL from package metadata, when present.
- `license: Option<String>` - SPDX or human-readable license string from package metadata.
- `support: Option<String>` - Support / issue-tracker URL from package metadata, when present.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PackageLinks) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PackageLinks`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> PackageLinks`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::manifest::PublisherIdentity

*Struct*

Optional publisher signing identity (when available).

**Fields:**
- `name: Option<String>` - Package name within `source`.
- `url: Option<String>` - HTTPS URL for this resource (artifact, publisher, …).
- `key_id: Option<String>` - Publisher signing key identifier when signatures are present.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PublisherIdentity) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PublisherIdentity`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> PublisherIdentity`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::manifest::SandboxRequest

*Struct*

Publisher-requested sandbox (informational until host approves).

**Fields:**
- `network: String` - Requested guest network mode (`deny` or `outbound`).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SandboxRequest`
- **PartialEq**
  - `fn eq(self: &Self, other: &SandboxRequest) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::manifest::normalize_protocol

*Function*

Normalize optional `protocol` for Workers RPC packages.

Accepts absent / empty / `workers-rpc` only.

# Errors

Returns an error when the operation fails.

```rust
fn normalize_protocol(protocol: Option<&str>) -> crate::error::Result<String>
```



## bookclerk_plugin_catalog::manifest::parse_sha256_hex

*Function*

Decode hex SHA-256 into 32 bytes.

# Errors

Returns an error when the operation fails.

```rust
fn parse_sha256_hex(s: &str) -> crate::error::Result<[u8; 32]>
```



## bookclerk_plugin_catalog::manifest::validate_sha256_hex

*Function*

Validate lowercase hex SHA-256 (64 chars).

# Errors

Returns an error when the operation fails.

```rust
fn validate_sha256_hex(s: &str) -> crate::error::Result<()>
```



