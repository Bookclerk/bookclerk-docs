**bookclerk_plugin_catalog > install**

# Module: install

## Contents

**Structs**

- [`InstallOptions`](#installoptions) - Options for `Installer::install_from_manifest`.
- [`InstallOutcome`](#installoutcome) - Result of a successful (or dry-run) install.
- [`Installer`](#installer) - Installer: resolve → download → verify → extract → activate.

**Functions**

- [`fetch_artifact`](#fetch_artifact) - Fetches an artifact URL (`file://` or local path, or HTTPS/localhost

**Constants**

- [`DOWNLOAD_TIMEOUT`](#download_timeout) - Download / install limits.
- [`MAX_DOWNLOAD_BYTES`](#max_download_bytes) - Hard cap on downloaded artifact size in bytes.
- [`PLUGIN_HOLD_DIR`](#plugin_hold_dir) - Host-owned directory under `$FILES_DIR` for held plugin-state during remove.

---

## bookclerk_plugin_catalog::install::DOWNLOAD_TIMEOUT

*Constant*: `std::time::Duration`

Download / install limits.



## bookclerk_plugin_catalog::install::InstallOptions

*Struct*

Options for `Installer::install_from_manifest`.

**Fields:**
- `plugins_root: std::path::PathBuf` - Directory under which plugin install folders are created.
- `target: Option<String>` - Host target triple used to select release artifacts.
- `dry_run: bool` - When true, resolve and verify without writing an install.
- `replace: bool` - When true, overwrite an existing install of the **same** [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey)
- `offline: bool` - When true, refuse network fetches (local/cache only).
- `trust: crate::trust::TrustPolicy` - Trust policy applied to community packages (digest still required).
- `skip_health: bool` - Skip health spawn (caller runs health separately).
- `approve_capabilities: bool` - When true, persist a covering consent grant after install.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstallOptions`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::install::InstallOutcome

*Struct*

Result of a successful (or dry-run) install.

**Fields:**
- `plugin_root: std::path::PathBuf` - Filesystem path of the installed plugin directory.
- `receipt: crate::receipt::InstallReceipt` - Install receipt written under the plugin root after activation.
- `dry_run: bool` - When true, resolve and verify without writing an install.
- `previous: Option<std::path::PathBuf>` - Previous install kept aside until `Installer::commit` (or restored by
- `previous_ledger: Option<crate::ledger::InstallLedgerEntry>` - Ledger row that existed for this PluginKey before activation, if any.
- `files_dir: Option<std::path::PathBuf>` - Files directory that owns `install-ledger.json` (`plugins_root` parent).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstallOutcome`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::install::Installer

*Struct*

Installer: resolve → download → verify → extract → activate.

**Unit Struct**

**Methods:**

- `fn plugin_key_for(coordinate: &PackageCoordinate, runtime_id: &str, plugins_root: &Path) -> Result<PluginKey>` - PluginKey this installer would assign for `coordinate` + runtime alias.
- `fn local_archive_coordinate(archive: &Path, manifest: &BookclerkPackageManifest) -> PackageCoordinate` - Coordinate used when installing from a local archive path.
- `fn install_from_manifest(manifest: &BookclerkPackageManifest, coordinate: &PackageCoordinate, opts: &InstallOptions) -> Result<InstallOutcome>` - Install from an already-validated package manifest (fixture / adapter output).
- `fn install_from_manifest_with_lock(lock: &PluginMutationLock, manifest: &BookclerkPackageManifest, coordinate: &PackageCoordinate, opts: &InstallOptions) -> Result<InstallOutcome>` - Install while already holding the host-local plugin mutation lock.
- `fn commit(outcome: &InstallOutcome) -> Result<()>` - Discard a replace backup after a successful health check (or when none).
- `fn rollback(outcome: &InstallOutcome) -> Result<()>` - Restore the previous install (or delete a failed first install) after a
- `fn install_local_archive(archive: &Path, manifest: &BookclerkPackageManifest, opts: &InstallOptions) -> Result<InstallOutcome>` - Install from a local archive path using an explicit manifest.
- `fn install_local_archive_with_lock(lock: &PluginMutationLock, archive: &Path, manifest: &BookclerkPackageManifest, opts: &InstallOptions) -> Result<InstallOutcome>` - Local-archive install while already holding the host mutation lock.
- `fn remove(plugins_root: &Path, spec: &str, purge_state: bool) -> Result<()>` - Remove an installed plugin directory; optionally purge `plugin-state/`.
- `fn remove_with_lock(lock: &PluginMutationLock, plugins_root: &Path, spec: &str, purge_state: bool) -> Result<()>` - Remove while already holding the host-local plugin mutation lock.
- `fn remove_with_lock_preparing_purge<F>(lock: &PluginMutationLock, plugins_root: &Path, spec: &str, purge_state: bool, prep: F) -> Result<()>` - Remove, and run `prep` on an existing plugin-state directory before purge.



## bookclerk_plugin_catalog::install::MAX_DOWNLOAD_BYTES

*Constant*: `u64`

Hard cap on downloaded artifact size in bytes.



## bookclerk_plugin_catalog::install::PLUGIN_HOLD_DIR

*Constant*: `&str`

Host-owned directory under `$FILES_DIR` for held plugin-state during remove.

Not inside a plugin-controlled install tree. Used so `--purge-state` can
rename `plugin-state/<fs-id>` aside and restore it on rollback.



## bookclerk_plugin_catalog::install::fetch_artifact

*Function*

Fetches an artifact URL (`file://` or local path, or HTTPS/localhost
HTTP bounded by [`MAX_DOWNLOAD_BYTES`](install.md#bookclerk_plugin_cataloginstallmax_download_bytes) and [`DOWNLOAD_TIMEOUT`](install.md#bookclerk_plugin_cataloginstalldownload_timeout)) into
`dest`, refusing oversized bodies.

# Errors

Returns an error when the URL is unsupported, the host is offline for a
remote URL, or the transfer fails or exceeds the cap.

```rust
fn fetch_artifact(url: &str, dest: &std::path::Path, offline: bool) -> crate::error::Result<()>
```



