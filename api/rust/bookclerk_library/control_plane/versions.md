**bookclerk_library > control_plane > versions**

# Module: control_plane::versions

## Contents

**Structs**

- [`PluginArtifact`](#pluginartifact) - One downloadable artifact of a version.
- [`PluginInstall`](#plugininstall) - The active version pointer of one plugin key.
- [`PluginVersion`](#pluginversion) - One installed version of a plugin.

**Enums**

- [`AddVersion`](#addversion) - What [`add_plugin_version`](versions.md#bookclerk_librarycontrol_planeversionsadd_plugin_version) did.

**Functions**

- [`activate_version`](#activate_version) - Moves the active pointer of `plugin_key` to `version` (first install,
- [`add_plugin_version`](#add_plugin_version) - Records a version. Installing the same `(plugin_key, version)` again
- [`approve_version_grant`](#approve_version_grant) - Records the operator-approved consent grant on a version.
- [`clear_install`](#clear_install) - Clears the active pointer (`plugins uninstall`). Version rows stay so the
- [`list_installs`](#list_installs) - Every active pointer.
- [`list_versions`](#list_versions) - Every version of `plugin_key` (or of every plugin), newest install first.
- [`load_active_version`](#load_active_version) - The active version of `plugin_key`, when a pointer names one.
- [`load_artifact_blob`](#load_artifact_blob) - Reads an archive stored by [`put_artifact_blob`](versions.md#bookclerk_librarycontrol_planeversionsput_artifact_blob).
- [`load_install`](#load_install) - The active pointer of `plugin_key`.
- [`load_version`](#load_version) - One version row.
- [`put_artifact_blob`](#put_artifact_blob) - Stores a small archive and returns its `blob:sha256:<hex>` URL.

**Constants**

- [`ARTIFACT_BLOB_SCHEME`](#artifact_blob_scheme) - Artifact URL scheme of an archive stored in `plugin_artifacts`.
- [`MAX_ARTIFACT_BLOB_BYTES`](#max_artifact_blob_bytes) - Largest archive stored in `plugin_artifacts` (base64 stays under

---

## bookclerk_library::control_plane::versions::ARTIFACT_BLOB_SCHEME

*Constant*: `&str`

Artifact URL scheme of an archive stored in `plugin_artifacts`.



## bookclerk_library::control_plane::versions::AddVersion

*Enum*

What [`add_plugin_version`](versions.md#bookclerk_librarycontrol_planeversionsadd_plugin_version) did.

**Variants:**
- `Created(PluginVersion)` - A new version row was written.
- `Existing(PluginVersion)` - The same `(plugin_key, version)` with the same artifacts already

**Methods:**

- `fn version(self: &Self) -> &PluginVersion` - The version row, created or existing.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AddVersion`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &AddVersion) -> bool`



## bookclerk_library::control_plane::versions::MAX_ARTIFACT_BLOB_BYTES

*Constant*: `usize`

Largest archive stored in `plugin_artifacts` (base64 stays under
D1's row cap).



## bookclerk_library::control_plane::versions::PluginArtifact

*Struct*

One downloadable artifact of a version.

**Fields:**
- `target: String` - Bookclerk target (`linux-x64-gnu`, …; `any` for workerd packages).
- `url: String` - Where nodes fetch the archive: `https:`, `file:`, or
- `format: String` - Archive format (`tar.gz` or `zip`).
- `archive_root: String` - Path inside the archive to the plugin root.
- `archive_sha256: String` - Lowercase hex SHA-256 of the archive bytes.
- `payload_root_sha256: String` - Payload-root digest of the extracted plugin tree; the cache directory
- `executable: String` - Relative path of the executable inside the plugin root.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginArtifact) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginArtifact`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::versions::PluginInstall

*Struct*

The active version pointer of one plugin key.

**Fields:**
- `plugin_key: String` - Canonical plugin key.
- `version: String` - Active version.
- `revision: i64` - Compare-and-swap revision.
- `updated_at: String` - RFC 3339 time of the last move.
- `updated_by: String` - Audit identity of the last writer.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginInstall`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginInstall) -> bool`



## bookclerk_library::control_plane::versions::PluginVersion

*Struct*

One installed version of a plugin.

**Fields:**
- `plugin_key: String` - Canonical plugin key.
- `version: String` - Semver package version.
- `manifest_id: String` - Manifest id (display alias).
- `manifest_toml: String` - `plugin.toml` text.
- `settings: bookclerk_plugin_abi::SettingsSchema` - Settings schema the version declares.
- `artifacts: Vec<PluginArtifact>` - Per-target artifacts.
- `publisher_verified: bool` - True when the publisher was verified at install time.
- `grant_json: String` - Approved consent grant (JSON), empty until an operator approves it.
- `coordinate: String` - Package coordinate the version was resolved from; `update` resolves
- `installed_at: String` - RFC 3339 install time.
- `installed_by: String` - Audit identity of the publisher.

**Methods:**

- `fn artifact_for(self: &Self, target: &str) -> Option<&PluginArtifact>` - The artifact for `target`, falling back to a target-independent

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PluginVersion) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginVersion`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::versions::activate_version

*Function*

Moves the active pointer of `plugin_key` to `version` (first install,
update, or rollback), pins the version's settings schema, and wakes every
node. Pointing at the version that is already active is a no-op.

# Errors

Returns an error when the actor may not write, the version does not
exist, or a write fails.

```rust
fn activate_version(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, plugin_key: &str, version: &str, node_id: &str) -> crate::error::Result<(PluginInstall, super::schemas::SchemaPinOutcome)>
```



## bookclerk_library::control_plane::versions::add_plugin_version

*Function*

Records a version. Installing the same `(plugin_key, version)` again
with the same artifact digests is a no-op (`AddVersion::Existing`);
different digests under an installed version are refused, so a version
never changes content behind a node's cache.

`installed_at` / `installed_by` are stamped here.

# Errors

Returns an error when the actor may not install, the version is
malformed, the version exists with other content, or the write fails.

```rust
fn add_plugin_version(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, version: PluginVersion) -> crate::error::Result<AddVersion>
```



## bookclerk_library::control_plane::versions::approve_version_grant

*Function*

Records the operator-approved consent grant on a version.

# Errors

Returns an error when the actor may not write, the version is missing,
or the write fails.

```rust
fn approve_version_grant(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, plugin_key: &str, version: &str, grant_json: &str) -> crate::error::Result<PluginVersion>
```



## bookclerk_library::control_plane::versions::clear_install

*Function*

Clears the active pointer (`plugins uninstall`). Version rows stay so the
plugin can be re-activated; nodes retire guests whose plugin has no
active version and sweep the cache.

# Errors

Returns an error when the actor may not write or the delete fails.

```rust
fn clear_install(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, plugin_key: &str) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::versions::list_installs

*Function*

Every active pointer.

# Errors

Returns an error when the read fails.

```rust
fn list_installs(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<PluginInstall>>
```



## bookclerk_library::control_plane::versions::list_versions

*Function*

Every version of `plugin_key` (or of every plugin), newest install first.

# Errors

Returns an error when the read or decode fails.

```rust
fn list_versions(store: &crate::store::LibraryStore, plugin_key: Option<&str>) -> crate::error::Result<Vec<PluginVersion>>
```



## bookclerk_library::control_plane::versions::load_active_version

*Function*

The active version of `plugin_key`, when a pointer names one.

# Errors

Returns an error when a read fails or the pointer names a missing row.

```rust
fn load_active_version(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<Option<PluginVersion>>
```



## bookclerk_library::control_plane::versions::load_artifact_blob

*Function*

Reads an archive stored by [`put_artifact_blob`](versions.md#bookclerk_librarycontrol_planeversionsput_artifact_blob).

# Errors

Returns an error when the read fails or the stored bytes do not match
their digest.

```rust
fn load_artifact_blob(store: &crate::store::LibraryStore, sha256: &str) -> crate::error::Result<Option<Vec<u8>>>
```



## bookclerk_library::control_plane::versions::load_install

*Function*

The active pointer of `plugin_key`.

# Errors

Returns an error when the read fails.

```rust
fn load_install(store: &crate::store::LibraryStore, plugin_key: &str) -> crate::error::Result<Option<PluginInstall>>
```



## bookclerk_library::control_plane::versions::load_version

*Function*

One version row.

# Errors

Returns an error when the read or decode fails.

```rust
fn load_version(store: &crate::store::LibraryStore, plugin_key: &str, version: &str) -> crate::error::Result<Option<PluginVersion>>
```



## bookclerk_library::control_plane::versions::put_artifact_blob

*Function*

Stores a small archive and returns its `blob:sha256:<hex>` URL.

# Errors

Returns an error when the archive exceeds [`MAX_ARTIFACT_BLOB_BYTES`](versions.md#bookclerk_librarycontrol_planeversionsmax_artifact_blob_bytes) or
the write fails.

```rust
fn put_artifact_blob(store: &crate::store::LibraryStore, bytes: &[u8]) -> crate::error::Result<String>
```



