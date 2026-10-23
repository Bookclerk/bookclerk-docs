# bookclerk_dev::plugins

Directory-driven discovery, build, stage, and platform install for guests.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BuildSelection`](#buildselection) | struct | Which guest trees (and installer stack) a build/stage should include. |
| [`DiscoveredGuest`](#discoveredguest) | struct | One discovered guest (platform, optional, or example). |
| [`packages_for`](#packages_for) | function | Resolve Cargo `-p` names for `BuildSelection`. |
| [`guests_by_id`](#guests_by_id) | function | Resolves plugin manifest ids to discovered guests across every tier. |
| [`packages_for_guests`](#packages_for_guests) | function | Cargo `-p` names needed to build `guests` (workerd guests ship `modules/`). |
| [`guest_plugin_key`](#guest_plugin_key) | function | Canonical PluginKey used when staging/installing `guest`. |
| [`guest_install_leaf`](#guest_install_leaf) | function | Install-directory leaf (`pk-` + 128-bit digest) for `guest`. |
| [`build_selection`](#build_selection) | function | Which plugin sets `cargo build-app` / `cargo stage-plugins` should include. |
| [`stage_plugins`](#stage_plugins) | function | Stage optional and/or example guests into `dest` (catalog layout). |
| [`install_platform`](#install_platform) | function | Install platform guests into `$FILES_DIR/plugins/{plugin-key-fs-id}/`. |
| [`stage_platform_for_pack`](#stage_platform_for_pack) | function | Stage platform guests into a temp dir for `package-platform` bundling. |
| [`stage_optional_for_pack`](#stage_optional_for_pack) | function | Stage optional guests for `package-plugins`. |
| [`discover_platform`](#discover_platform) | function | Lists platform plugin guests under `crates/bookclerk-plugins/platform`. |
| [`discover_optional`](#discover_optional) | function | Lists optional storefront / integration guests under `…/optional`. |
| [`discover_examples`](#discover_examples) | function | Lists reference Echo example guests under `examples/`. |
| [`build_packages`](#build_packages) | function | Runs `cargo build [-p …]` for the selected packages; inherits stdio and fails on non-zero exit. |
| [`HELPER_PACKAGES`](#helper_packages) | constant | Helper binaries that ship beside hosts (also listed in workspace `default-members`). |

## Structs

### BuildSelection

```rust
pub struct BuildSelection {
    pub platform: bool,
    pub optional: bool,
    pub examples: bool,
}
```

Which guest trees (and installer stack) a build/stage should include.

**Fields**

- `platform`: Workspace `default-members` + platform guests under `crates/bookclerk-plugins/platform`.
- `optional`: Guests under `crates/bookclerk-plugins/optional`.
- `examples`: Guests under `examples`.

Implements: `Clone`, `Copy`, `Debug`, `Default`

### DiscoveredGuest

```rust
pub struct DiscoveredGuest {
    pub id: String,
    pub kind: String,
    pub dir: PathBuf,
    pub rel_dir: String,
    pub package: Option<String>,
    pub bin_name: Option<String>,
    pub manifest_name: String,
    pub runtime: String,
}
```

One discovered guest (platform, optional, or example).

**Fields**

- `id`: Stable identifier for this item.
- `kind`: Discriminant or category for this value.
- `dir`: Plugin source or install directory relative to the workspace.
- `rel_dir`: Relative to workspace root (for packaging / logs).
- `package`: Cargo / npm package name for this plugin guest.
- `bin_name`: Compiled binary name when the guest is a native executable.
- `manifest_name`: Expected `plugin.toml` package name / id.
- `runtime`: Guest runtime (`native`, `workerd`, `python`, …).

Implements: `Clone`, `Debug`

## Functions

### packages_for

```rust
pub fn packages_for(root: &Path, sel: BuildSelection) -> Result<Vec<String>>
```

Resolve Cargo `-p` names for [`BuildSelection`](#buildselection).

#### Arguments

* `root` - Cargo workspace root directory.
* `sel` - `sel` input for this call.

#### Returns

On success, the inner `Vec<String>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### guests_by_id

```rust
pub fn guests_by_id(root: &Path, ids: &[String]) -> Result<Vec<DiscoveredGuest>>
```

Resolves plugin manifest ids to discovered guests across every tier.

Lets CI and developers build or stage one guest (for example `libro`)
without the rest of its tier.

#### Arguments

* `root` - Cargo workspace root directory.
* `ids` - Plugin manifest ids (`plugin.toml` `id`); duplicates are ignored.

#### Returns

Guests in the order first requested.

#### Errors

Returns an error naming every unknown id, or when discovery fails.

### packages_for_guests

```rust
pub fn packages_for_guests(guests: &[DiscoveredGuest]) -> Vec<String>
```

Cargo `-p` names needed to build `guests` (workerd guests ship `modules/`).

#### Arguments

* `guests` - Guests resolved by [`guests_by_id`](#guests_by_id) or tier discovery.

#### Returns

Unique native package names in input order.

### guest_plugin_key

```rust
pub fn guest_plugin_key(guest: &DiscoveredGuest) -> Result<PluginKey>
```

Canonical PluginKey used when staging/installing `guest`.

Platform artifacts use `platform:bookclerk/{package}`. Workspace
Cargo packages use the crates.io `cargo:` form so dest leaves stay stable.
Other trees use a path key of the source directory.

#### Errors

Returns when the id or coordinate cannot form a PluginKey.

### guest_install_leaf

```rust
pub fn guest_install_leaf(guest: &DiscoveredGuest) -> Result<String>
```

Install-directory leaf (`pk-` + 128-bit digest) for `guest`.

#### Errors

Returns when [`guest_plugin_key`](#guest_plugin_key) fails.

### build_selection

```rust
pub fn build_selection(root: &Path, release: bool, sel: BuildSelection) -> Result<()>
```

Which plugin sets `cargo build-app` / `cargo stage-plugins` should include.

#### Arguments

* `root` - Cargo workspace root directory.
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.
* `sel` - `sel` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### stage_plugins

```rust
pub fn stage_plugins(
    root: &Path,
    dest: &Path,
    release: bool,
    optional: bool,
    examples: bool,
    plugin_ids: &[String],
    skip_build: bool,
) -> Result<()>
```

Stage optional and/or example guests into `dest` (catalog layout).

#### Arguments

* `root` - Cargo workspace root directory.
* `dest` - Filesystem path (`dest`).
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.
* `optional` - Stage every guest under `crates/bookclerk-plugins/optional`.
* `examples` - Stage every guest under `examples`.
* `plugin_ids` - Additional individual guests by manifest id (`--plugin`).
* `skip_build` - Boolean flag `skip_build`.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when nothing is selected, an id is unknown or names a
platform guest (those are installed by `install-platform`), or the
underlying I/O, parse, or build operation fails.

### install_platform

```rust
pub fn install_platform(root: &Path, files_dir: &Path, release: bool) -> Result<()>
```

Install platform guests into `$FILES_DIR/plugins/{plugin-key-fs-id}/`.

#### Arguments

* `root` - Cargo workspace root directory.
* `files_dir` - Bookclerk files directory to wipe and recreate.
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### stage_platform_for_pack

```rust
pub fn stage_platform_for_pack(
    root: &Path,
    dest: &Path,
    release: bool,
    skip_build: bool,
) -> Result<()>
```

Stage platform guests into a temp dir for `package-platform` bundling.

#### Arguments

* `root` - Cargo workspace root directory.
* `dest` - Filesystem path (`dest`).
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.
* `skip_build` - Boolean flag `skip_build`.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### stage_optional_for_pack

```rust
pub fn stage_optional_for_pack(root: &Path, dest: &Path, release: bool) -> Result<()>
```

Stage optional guests for `package-plugins`.

#### Arguments

* `root` - Cargo workspace root directory.
* `dest` - Filesystem path (`dest`).
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### discover_platform

```rust
pub fn discover_platform(root: &Path) -> Result<Vec<DiscoveredGuest>>
```

Lists platform plugin guests under `crates/bookclerk-plugins/platform`.

#### Arguments

* `root` - Cargo workspace root directory.

#### Returns

On success, the inner `Vec<DiscoveredGuest>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### discover_optional

```rust
pub fn discover_optional(root: &Path) -> Result<Vec<DiscoveredGuest>>
```

Lists optional storefront / integration guests under `…/optional`.

#### Arguments

* `root` - Cargo workspace root directory.

#### Returns

On success, the inner `Vec<DiscoveredGuest>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### discover_examples

```rust
pub fn discover_examples(root: &Path) -> Result<Vec<DiscoveredGuest>>
```

Lists reference Echo example guests under `examples/`.

#### Arguments

* `root` - Cargo workspace root directory.

#### Returns

On success, the inner `Vec<DiscoveredGuest>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### build_packages

```rust
pub fn build_packages(root: &Path, release: bool, packages: &[String]) -> Result<()>
```

Runs `cargo build [-p …]` for the selected packages; inherits stdio and fails on non-zero exit.

#### Arguments

* `root` - Cargo workspace root directory.
* `release` - Build with `--release` when true.
* `packages` - Cargo package names passed as `-p`.

#### Errors

Returns an error when `cargo` cannot be spawned or exits non-zero.

## Constants

### HELPER_PACKAGES

```rust
pub const HELPER_PACKAGES: &[&str] = _;
```

Helper binaries that ship beside hosts (also listed in workspace `default-members`).
