**bookclerk_dev**

# Module: bookclerk_dev

## Contents

**Modules**

- [`package`](#package) - Release packaging aligned with `docs/plugin-registry.md` artifact naming.
- [`plugins`](#plugins) - Directory-driven discovery, build, stage, and platform install for guests.

**Functions**

- [`default_artifacts`](#default_artifacts) - Default directory for packaged release artifacts under the workspace.
- [`default_files_dir`](#default_files_dir) - Dev files root: `$BOOKCLERK_FILES_DIR`, else `<workspace>/BookclerkFiles`.
- [`ensure_workerd_for_profile`](#ensure_workerd_for_profile) - Ensure the pinned Cloudflare `workerd` binary is present under `target/<profile>/`.
- [`reset_files_dir`](#reset_files_dir) - Wipe a Bookclerk files directory (config, DB, master.key, plugins, caches).
- [`workspace_root`](#workspace_root) - Resolves the Cargo workspace root from `CARGO_MANIFEST_DIR` or cwd.
- [`workspace_version`](#workspace_version) - Workspace version from `CARGO_PKG_VERSION` (matches `[workspace.package].version`).

---

## bookclerk_dev::default_artifacts

*Function*

Default directory for packaged release artifacts under the workspace.

# Arguments

* `root` - Cargo workspace root directory.

# Returns

Absolute path to the artifacts directory.

```rust
fn default_artifacts(root: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_dev::default_files_dir

*Function*

Dev files root: `$BOOKCLERK_FILES_DIR`, else `<workspace>/BookclerkFiles`.

```rust
fn default_files_dir() -> std::path::PathBuf
```



## bookclerk_dev::ensure_workerd_for_profile

*Function*

Ensure the pinned Cloudflare `workerd` binary is present under `target/<profile>/`.

# Arguments

* `root` - Cargo workspace root directory.
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.

# Returns

Absolute path to the ensured `workerd` binary.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn ensure_workerd_for_profile(root: &std::path::Path, release: bool) -> anyhow::Result<std::path::PathBuf>
```



## Module: package

Release packaging aligned with `docs/plugin-registry.md` artifact naming.



## Module: plugins

Directory-driven discovery, build, stage, and platform install for guests.



## bookclerk_dev::reset_files_dir

*Function*

Wipe a Bookclerk files directory (config, DB, master.key, plugins, caches).

# Arguments

* `files_dir` - Bookclerk files directory to wipe and recreate.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn reset_files_dir(files_dir: &std::path::Path) -> anyhow::Result<()>
```



## bookclerk_dev::workspace_root

*Function*

Resolves the Cargo workspace root from `CARGO_MANIFEST_DIR` or cwd.

# Returns

Absolute path to the Cargo workspace root (parent of `crates/`).

# Errors

Returns an error when the crate is not nested under `crates/` as expected.

```rust
fn workspace_root() -> anyhow::Result<std::path::PathBuf>
```



## bookclerk_dev::workspace_version

*Function*

Workspace version from `CARGO_PKG_VERSION` (matches `[workspace.package].version`).

```rust
fn workspace_version() -> &'static str
```



