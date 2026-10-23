# bookclerk_dev

Dev workflow helpers for building and staging first-party plugin guests.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ensure_ui_dist`](#ensure_ui_dist) | function | Rebuilds `ui/dist` when `index.html` is missing or older than SPA sources. |
| [`ensure_workerd_for_profile`](#ensure_workerd_for_profile) | function | Ensure the pinned Cloudflare `workerd` binary is present under `target/<profile>/`. |
| [`workspace_root`](#workspace_root) | function | Resolves the Cargo workspace root from `CARGO_MANIFEST_DIR` or cwd. |
| [`default_artifacts`](#default_artifacts) | function | Default directory for packaged release artifacts under the workspace. |
| [`default_files_dir`](#default_files_dir) | function | Dev files root: `$BOOKCLERK_FILES_DIR`, else `<workspace>/BookclerkFiles`. |
| [`reset_files_dir`](#reset_files_dir) | function | Wipe a Bookclerk files directory (config, DB, master.key, plugins, caches). |
| [`workspace_version`](#workspace_version) | function | Workspace version from `CARGO_PKG_VERSION` (matches `[workspace.package].version`). |

## Modules

- [`package`](package.md): Release packaging aligned with `docs/plugin-registry.md` artifact naming.
- [`plugins`](plugins.md): Directory-driven discovery, build, stage, and platform install for guests.

## Functions

### ensure_ui_dist

```rust
pub fn ensure_ui_dist(root: &Path) -> Result<()>
```

Rebuilds `ui/dist` when `index.html` is missing or older than SPA sources.

`cargo dev` previously compiled only Rust crates, so a `git pull` that
changed `ui/src` still served the previous Vite output from `ui/dist`.

#### Arguments

* `root` - Cargo workspace root (the directory that contains `ui/`).

#### Errors

Returns an error when `npm ci` / `npm run build` fail, `npm` is missing, or
`ui/dist/index.html` is still absent after the build.

### ensure_workerd_for_profile

```rust
pub fn ensure_workerd_for_profile(root: &Path, release: bool) -> Result<PathBuf>
```

Ensure the pinned Cloudflare `workerd` binary is present under `target/<profile>/`.

#### Arguments

* `root` - Cargo workspace root directory.
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.

#### Returns

Absolute path to the ensured `workerd` binary.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### workspace_root

```rust
pub fn workspace_root() -> Result<PathBuf>
```

Resolves the Cargo workspace root from `CARGO_MANIFEST_DIR` or cwd.

#### Returns

Absolute path to the Cargo workspace root (parent of `crates/`).

#### Errors

Returns an error when the crate is not nested under `crates/` as expected.

### default_artifacts

```rust
pub fn default_artifacts(root: &Path) -> PathBuf
```

Default directory for packaged release artifacts under the workspace.

#### Arguments

* `root` - Cargo workspace root directory.

#### Returns

Absolute path to the artifacts directory.

### default_files_dir

```rust
pub fn default_files_dir() -> PathBuf
```

Dev files root: `$BOOKCLERK_FILES_DIR`, else `<workspace>/BookclerkFiles`.

### reset_files_dir

```rust
pub fn reset_files_dir(files_dir: &Path) -> Result<()>
```

Wipe a Bookclerk files directory (config, DB, master.key, plugins, caches).

#### Arguments

* `files_dir` - Bookclerk files directory to wipe and recreate.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### workspace_version

```rust
pub fn workspace_version() -> &'static str
```

Workspace version from `CARGO_PKG_VERSION` (matches `[workspace.package].version`).
