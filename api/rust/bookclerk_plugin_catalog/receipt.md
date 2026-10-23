**bookclerk_plugin_catalog > receipt**

# Module: receipt

## Contents

**Structs**

- [`InstallReceipt`](#installreceipt) - Record written after a successful install / update.

**Constants**

- [`RECEIPT_FILE`](#receipt_file) - Filename for the receipt beside `plugin.toml`.

---

## bookclerk_plugin_catalog::receipt::InstallReceipt

*Struct*

Record written after a successful install / update.

**Fields:**
- `schema_version: u32` - DTO schema version for CLI/UI JSON compatibility.
- `plugin_key: String` - Provenance-qualified logical plugin identity (no version, no hash).
- `provenance: crate::identity::PluginProvenance` - Host-evaluated provenance at install time (re-checked at discovery).
- `coordinate: crate::coordinate::PackageCoordinate` - Fully qualified package coordinate when version is known.
- `version: String` - Resolved or candidate package version string.
- `registry_url: Option<String>` - Base URL of the package registry (for example crates.io or npm).
- `artifact_url: String` - Resolved download URL used for this install.
- `target: String` - Host target triple used to select release artifacts.
- `archive_sha256: String` - Lowercase hex SHA-256 of the downloadable archive bytes (install-time).
- `manifest_sha256: String` - SHA-256 of the installed `plugin.toml` bytes.
- `payload_root_sha256: String` - Deterministic Merkle-style root over immutable packaged files.
- `executable_sha256: Option<String>` - Optional SHA-256 of the extracted executable bytes.
- `protocol: String` - Host↔guest wire protocol id (for example `workers-rpc`).
- `api_version: u32` - Plugin ABI version negotiated with the host.
- `runtime: crate::kind::RuntimeIdentity` - Bookclerk runtime identity (kind + plugin id) when known.
- `requested_sandbox: crate::manifest::SandboxRequest` - Publisher-requested sandbox snapshot recorded at install time.
- `approved_network: String` - Host-approved network mode (may be stricter than requested).
- `installed_at: chrono::DateTime<chrono::Utc>` - RFC 3339 time when this install was activated.
- `update_constraint: Option<String>` - Optional update constraint (for example pinned version range).

**Methods:**

- `fn path_in(plugin_root: &Path) -> PathBuf` - Path to receipt inside an installed plugin directory.
- `fn plugin_key(self: &Self) -> Result<PluginKey>` - Parsed [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey) from `Self::plugin_key`.
- `fn platform_bundled(plugin_key: PluginKey, version: &str, manifest: &PluginManifest, manifest_sha256: String, payload_root_sha256: String, product: &str, package_name: &str) -> Self` - Host-stamped receipt for a Bookclerk platform artifact.
- `fn load(plugin_root: &Path) -> Result<Self>` - Load receipt from a plugin install directory.
- `fn store(self: &Self, plugin_root: &Path) -> Result<()>` - Atomically write receipt (unique same-dir staging + replace).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> InstallReceipt`
- **PartialEq**
  - `fn eq(self: &Self, other: &InstallReceipt) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::receipt::RECEIPT_FILE

*Constant*: `&str`

Filename for the receipt beside `plugin.toml`.



