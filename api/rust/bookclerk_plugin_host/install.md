**bookclerk_plugin_host > install**

# Module: install

## Contents

**Structs**

- [`AddedVersion`](#addedversion) - What [`add_package_version`](install.md#bookclerk_plugin_hostinstalladd_package_version) recorded.
- [`InstallRequest`](#installrequest) - An operator's `install`: what `bookclerk plugins install` and
- [`InstalledVersion`](#installedversion) - What [`install_version`](install.md#bookclerk_plugin_hostinstallinstall_version) did.
- [`VersionRequest`](#versionrequest) - One package to install.

**Enums**

- [`Activate`](#activate) - When installing moves the active pointer.

**Functions**

- [`add_package_version`](#add_package_version) - Verifies `package`, records a [`PluginVersion`](../bookclerk_library/control_plane/versions.md#bookclerk_librarycontrol_planeversionspluginversion), and moves the active
- [`enroll_version`](#enroll_version) - Creates the default instance of a installed plugin with its schema
- [`grant_covers_version`](#grant_covers_version) - True when `grant` still covers everything `version` requests, so an
- [`host_artifact`](#host_artifact) - The version artifact this node runs.
- [`install_plugin_key`](#install_plugin_key) - The plugin key an install installs under.
- [`install_version`](#install_version) - Installs an operator's package as a version, carries an earlier
- [`is_sha256_hex`](#is_sha256_hex) - True for a canonical SHA-256 digest: exactly 64 lowercase hex digits.
- [`materialize_owner_version`](#materialize_owner_version) - [`materialize_version`](install.md#bookclerk_plugin_hostinstallmaterialize_version) for a version an owner cluster deploys to this
- [`materialize_version`](#materialize_version) - Makes the active version's tree available under
- [`open_installed_plugin`](#open_installed_plugin) - Opens a materialized version tree as a spawnable plugin carrying the
- [`sweep_version_cache`](#sweep_version_cache) - Removes cached digests not in `keep` and older than `grace`, plus stale
- [`version_cache_path`](#version_cache_path) - Cached tree of one payload-root digest.
- [`version_cache_root`](#version_cache_root) - Cache root: `<files_dir>/plugins/.versions`.
- [`version_consent_request`](#version_consent_request) - Consent the version's manifest requests (what `plugins approve` shows).
- [`version_grant`](#version_grant) - The consent grant recorded on `version`.

**Constants**

- [`VERSION_CACHE_DIR`](#version_cache_dir) - Directory under `plugins/` holding materialized version trees.
- [`VERSION_CACHE_GRACE`](#version_cache_grace) - How long an unreferenced cached digest survives before the sweep.

---

## bookclerk_plugin_host::install::Activate

*Enum*

When installing moves the active pointer.

**Variants:**
- `Always` - Always point at the new version (`update`, `--activate`).
- `IfNone` - Only when the plugin has no active version yet (first install).
- `Never` - Never; install only.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Activate`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Activate) -> bool`



## bookclerk_plugin_host::install::AddedVersion

*Struct*

What [`add_package_version`](install.md#bookclerk_plugin_hostinstalladd_package_version) recorded.

**Fields:**
- `version: bookclerk_library::control_plane::PluginVersion` - The version row.
- `created: bool` - True when this call wrote the row (false for an identical re-install).
- `pointer: Option<bookclerk_library::control_plane::PluginInstall>` - Pointer after activation, when this call moved it.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> AddedVersion`



## bookclerk_plugin_host::install::InstallRequest

*Struct*

An operator's `install`: what `bookclerk plugins install` and
`POST /api/config/plugin-versions` both install.

**Fields:**
- `package: bookclerk_plugin_catalog::BookclerkPackageManifest` - Resolved package manifest.
- `coordinate: bookclerk_plugin_catalog::PackageCoordinate` - Where the package came from. A local archive's coordinate is
- `archive: Option<std::path::PathBuf>` - Local archive to install instead of fetching the coordinate.
- `target: Option<String>` - Install only this target instead of every target in the package.
- `offline: bool` - Refuse network fetches.
- `activate: bool` - Make the version active even when another version already is.
- `approve: bool` - Record the requested consent as approved when no earlier approval

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstallRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::install::InstalledVersion

*Struct*

What [`install_version`](install.md#bookclerk_plugin_hostinstallinstall_version) did.

**Fields:**
- `version: bookclerk_library::control_plane::PluginVersion` - The installed (and maybe approved) version.
- `created: bool` - False when this exact version was already installed.
- `active: bool` - True when the version is the plugin's active version afterwards.
- `instance: bookclerk_library::control_plane::PluginInstance` - The plugin's default instance.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> InstalledVersion`



## bookclerk_plugin_host::install::VERSION_CACHE_DIR

*Constant*: `&str`

Directory under `plugins/` holding materialized version trees.



## bookclerk_plugin_host::install::VERSION_CACHE_GRACE

*Constant*: `std::time::Duration`

How long an unreferenced cached digest survives before the sweep.



## bookclerk_plugin_host::install::VersionRequest

*Struct*

One package to install.

**Fields:**
- `plugin_key: String` - Canonical plugin key the version is recorded under.
- `package: bookclerk_plugin_catalog::BookclerkPackageManifest` - Install-grade package manifest.
- `archive: Option<std::path::PathBuf>` - Local archive. `None` fetches every target's artifact from its URL
- `target: Option<String>` - Target the local archive is built for (default: this host's).
- `coordinate: String` - Package coordinate recorded for `update`; empty for a local archive.
- `offline: bool` - Refuse remote downloads (a local archive needs none).
- `publisher_verified: bool` - Publisher verification result (recorded on the version).
- `grant: Option<crate::consent::PluginGrant>` - Consent grant approved at install time, if any.
- `activate: Activate` - When to move the active pointer.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> VersionRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::install::add_package_version

*Function*

Verifies `package`, records a [`PluginVersion`](../bookclerk_library/control_plane/versions.md#bookclerk_librarycontrol_planeversionspluginversion), and moves the active
pointer per `VersionRequest::activate`.

A local archive's digest is computed here (it is the operator's own
file); a remote artifact must match the digest its package manifest
declares. Archives up to the library blob cap are stored in the
database when no shared URL exists, so every host can materialize them.

# Errors

Returns an error for a platform key (platform guests ship with the
host), a malformed or yanked package, a digest mismatch, a tree that
disagrees with the package, or a library write failure.

```rust
fn add_package_version(store: &bookclerk_library::LibraryStore, actor: &bookclerk_library::control_plane::ConfigActor, files_dir: &std::path::Path, node_id: &str, request: VersionRequest) -> crate::Result<AddedVersion>
```



## bookclerk_plugin_host::install::enroll_version

*Function*

Creates the default instance of a installed plugin with its schema
defaults, as enrollment does for a host-local plugin. Idempotent.

# Errors

Returns an error when an instance or document write fails.

```rust
fn enroll_version(store: &bookclerk_library::LibraryStore, actor: &bookclerk_library::control_plane::ConfigActor, version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<bookclerk_library::control_plane::PluginInstance>
```



## bookclerk_plugin_host::install::grant_covers_version

*Function*

True when `grant` still covers everything `version` requests, so an
update may carry an earlier approval forward. A widened request needs a
fresh `plugins approve`.

```rust
fn grant_covers_version(grant: &crate::consent::PluginGrant, version: &bookclerk_library::control_plane::PluginVersion) -> bool
```



## bookclerk_plugin_host::install::host_artifact

*Function*

The version artifact this node runs.

# Errors

Returns an error naming the host target when the version has none for
it, so placement can exclude this host.

```rust
fn host_artifact(version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<&bookclerk_library::control_plane::PluginArtifact>
```



## bookclerk_plugin_host::install::install_plugin_key

*Function*

The plugin key an install installs under.

A registry coordinate names a stable key. A local archive's path does
not: a new version of an already installed plugin arrives as another
file, so its key is the key its alias is already installed under, and
only a first install takes the path-derived key.

# Errors

Returns an error when the coordinate yields no key or the alias is
installed under several keys.

```rust
fn install_plugin_key(store: &bookclerk_library::LibraryStore, files_dir: &std::path::Path, coordinate: &bookclerk_plugin_catalog::PackageCoordinate, package: &bookclerk_plugin_catalog::BookclerkPackageManifest) -> crate::Result<String>
```



## bookclerk_plugin_host::install::install_version

*Function*

Installs an operator's package as a version, carries an earlier
approval forward when it still covers the new version (or approves when
asked), and enrolls the plugin's default instance.

One alias names one plugin: a version may not take an alias another
plugin's active version already answers to.

# Errors

Returns an error when the key cannot be derived, the alias belongs to
another plugin, or installing, approval, or enrollment fails.

```rust
fn install_version(store: &bookclerk_library::LibraryStore, actor: &bookclerk_library::control_plane::ConfigActor, files_dir: &std::path::Path, node_id: &str, request: InstallRequest) -> crate::Result<InstalledVersion>
```



## bookclerk_plugin_host::install::is_sha256_hex

*Function*

True for a canonical SHA-256 digest: exactly 64 lowercase hex digits.

Every digest that names a path (the version cache) or is compared
against a computed digest is held to this form, so an owner- or
registry-supplied value such as `..`, an absolute path, or one with a
separator can never select a directory outside the cache.

```rust
fn is_sha256_hex(raw: &str) -> bool
```



## bookclerk_plugin_host::install::materialize_owner_version

*Function*

[`materialize_version`](install.md#bookclerk_plugin_hostinstallmaterialize_version) for a version an owner cluster deploys to this
host: a `blob:` archive is read from the library as usual (the caller
fetched it over the owner link and checked its digest), and any other
URL is fetched only as an owner `https:` URL
([`bookclerk_plugin_catalog::fetch_owner_artifact`]: no `file:`, no
local or private hosts, every redirect hop checked the same way).

A tree already cached under the artifact's payload digest is reused
without a fetch or an archive check, so the caller must have checked
the whole artifact (URL, archive digest and payload digest) against the
host operator's approval first
(`RemotePluginConsent::approved_artifact_matches`).

# Arguments

* `store` - This host's library.
* `files_dir` - This host's files dir.
* `version` - The owner's version, with the grant this host runs it with.

# Errors

As [`materialize_version`](install.md#bookclerk_plugin_hostinstallmaterialize_version), and when the owner's URL is refused.

```rust
fn materialize_owner_version(store: &bookclerk_library::LibraryStore, files_dir: &std::path::Path, version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<std::path::PathBuf>
```



## bookclerk_plugin_host::install::materialize_version

*Function*

Makes the active version's tree available under
`plugins/.versions/<payload digest>` and returns that path.

A cached digest is reused as is; otherwise the archive is fetched (from
the library blob store, a `file:` path, or HTTPS), its digest checked,
extracted under `.staging`, the payload-root digest checked against the
version, and renamed into place under the plugin mutation lock.

# Errors

Returns an error when the version has no artifact for this host, a
digest is not 64 lowercase hex digits ([`is_sha256_hex`](install.md#bookclerk_plugin_hostinstallis_sha256_hex); checked before
any path is formed or a cached tree reused), the artifact cannot be
fetched, or a digest does not match.

```rust
fn materialize_version(store: &bookclerk_library::LibraryStore, files_dir: &std::path::Path, version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<std::path::PathBuf>
```



## bookclerk_plugin_host::install::open_installed_plugin

*Function*

Opens a materialized version tree as a spawnable plugin carrying the
version's identity and consent grant.

# Errors

Returns an error when the tree's manifest is unreadable or names another
plugin, or the version has no approved grant yet.

```rust
fn open_installed_plugin(root: &std::path::Path, version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<crate::DiscoveredPlugin>
```



## bookclerk_plugin_host::install::sweep_version_cache

*Function*

Removes cached digests not in `keep` and older than `grace`, plus stale
version staging directories. Returns how many entries were removed.

# Errors

Returns an error when the mutation lock cannot be taken.

```rust
fn sweep_version_cache(files_dir: &std::path::Path, keep: &std::collections::BTreeSet<String>, grace: std::time::Duration) -> crate::Result<usize>
```



## bookclerk_plugin_host::install::version_cache_path

*Function*

Cached tree of one payload-root digest.

# Errors

Returns an error when `payload_root_sha256` is not a canonical SHA-256
([`is_sha256_hex`](install.md#bookclerk_plugin_hostinstallis_sha256_hex)): it is joined onto the cache root, so anything else
could name a directory outside it.

```rust
fn version_cache_path(files_dir: &std::path::Path, payload_root_sha256: &str) -> crate::Result<std::path::PathBuf>
```



## bookclerk_plugin_host::install::version_cache_root

*Function*

Cache root: `<files_dir>/plugins/.versions`.

```rust
fn version_cache_root(files_dir: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_plugin_host::install::version_consent_request

*Function*

Consent the version's manifest requests (what `plugins approve` shows).

# Errors

Returns an error when the recorded `plugin.toml` or plugin key no longer
parses.

```rust
fn version_consent_request(version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<crate::consent::PluginGrant>
```



## bookclerk_plugin_host::install::version_grant

*Function*

The consent grant recorded on `version`.

# Errors

Returns an error when no operator has approved the version yet or the
recorded grant is malformed.

```rust
fn version_grant(version: &bookclerk_library::control_plane::PluginVersion) -> crate::Result<crate::consent::PluginGrant>
```



