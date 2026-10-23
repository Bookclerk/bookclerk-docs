**bookclerk_dev > plugins**

# Module: plugins

## Contents

**Structs**

- [`BuildSelection`](#buildselection) - Which guest trees (and installer stack) a build/stage should include.
- [`DiscoveredGuest`](#discoveredguest) - One discovered guest (platform, optional, or example).

**Functions**

- [`build_packages`](#build_packages) - Runs `cargo build [-p …]` for the selected packages; inherits stdio and fails on non-zero exit.
- [`build_selection`](#build_selection) - Which plugin sets `cargo build-app` / `cargo stage-plugins` should include.
- [`discover_examples`](#discover_examples) - Lists reference Echo example guests under `examples/`.
- [`discover_optional`](#discover_optional) - Lists optional storefront / integration guests under `…/optional`.
- [`discover_platform`](#discover_platform) - Lists platform plugin guests under `crates/bookclerk-plugins/platform`.
- [`guest_install_leaf`](#guest_install_leaf) - Install-directory leaf (`pk-` + 128-bit digest) for `guest`.
- [`guest_plugin_key`](#guest_plugin_key) - Canonical PluginKey used when staging/installing `guest`.
- [`guests_by_id`](#guests_by_id) - Resolves plugin manifest ids to discovered guests across every tier.
- [`install_platform`](#install_platform) - Install platform guests into `$FILES_DIR/plugins/{plugin-key-fs-id}/`.
- [`packages_for`](#packages_for) - Resolve Cargo `-p` names for [`BuildSelection`](plugins.md#bookclerk_devpluginsbuildselection).
- [`packages_for_guests`](#packages_for_guests) - Cargo `-p` names needed to build `guests` (workerd guests ship `modules/`).
- [`stage_optional_for_pack`](#stage_optional_for_pack) - Stage optional guests for `package-plugins`.
- [`stage_platform_for_pack`](#stage_platform_for_pack) - Stage platform guests into a temp dir for `package-platform` bundling.
- [`stage_plugins`](#stage_plugins) - Stage optional and/or example guests into `dest` (catalog layout).

**Constants**

- [`HELPER_PACKAGES`](#helper_packages) - Helper binaries that ship beside hosts (also listed in workspace `default-members`).

---

## bookclerk_dev::plugins::BuildSelection

*Struct*

Which guest trees (and installer stack) a build/stage should include.

**Fields:**
- `platform: bool` - Workspace `default-members` + platform guests under `crates/bookclerk-plugins/platform`.
- `optional: bool` - Guests under `crates/bookclerk-plugins/optional`.
- `examples: bool` - Guests under `examples`.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BuildSelection`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> BuildSelection`



## bookclerk_dev::plugins::DiscoveredGuest

*Struct*

One discovered guest (platform, optional, or example).

**Fields:**
- `id: String` - Stable identifier for this item.
- `kind: String` - Discriminant or category for this value.
- `dir: std::path::PathBuf` - Plugin source or install directory relative to the workspace.
- `rel_dir: String` - Relative to workspace root (for packaging / logs).
- `package: Option<String>` - Cargo / npm package name for this plugin guest.
- `bin_name: Option<String>` - Compiled binary name when the guest is a native executable.
- `manifest_name: String` - Expected `plugin.toml` package name / id.
- `runtime: String` - Guest runtime (`native`, `workerd`, `python`, …).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DiscoveredGuest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_dev::plugins::HELPER_PACKAGES

*Constant*: `&[&str]`

Helper binaries that ship beside hosts (also listed in workspace `default-members`).



## bookclerk_dev::plugins::build_packages

*Function*

Runs `cargo build [-p …]` for the selected packages; inherits stdio and fails on non-zero exit.

# Arguments

* `root` - Cargo workspace root directory.
* `release` - Build with `--release` when true.
* `packages` - Cargo package names passed as `-p`.

# Errors

Returns an error when `cargo` cannot be spawned or exits non-zero.

```rust
fn build_packages(root: &std::path::Path, release: bool, packages: &[String]) -> anyhow::Result<()>
```



## bookclerk_dev::plugins::build_selection

*Function*

Which plugin sets `cargo build-app` / `cargo stage-plugins` should include.

# Arguments

* `root` - Cargo workspace root directory.
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.
* `sel` - `sel` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn build_selection(root: &std::path::Path, release: bool, sel: BuildSelection) -> anyhow::Result<()>
```



## bookclerk_dev::plugins::discover_examples

*Function*

Lists reference Echo example guests under `examples/`.

# Arguments

* `root` - Cargo workspace root directory.

# Returns

On success, the inner `Vec<DiscoveredGuest>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn discover_examples(root: &std::path::Path) -> anyhow::Result<Vec<DiscoveredGuest>>
```



## bookclerk_dev::plugins::discover_optional

*Function*

Lists optional storefront / integration guests under `…/optional`.

# Arguments

* `root` - Cargo workspace root directory.

# Returns

On success, the inner `Vec<DiscoveredGuest>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn discover_optional(root: &std::path::Path) -> anyhow::Result<Vec<DiscoveredGuest>>
```



## bookclerk_dev::plugins::discover_platform

*Function*

Lists platform plugin guests under `crates/bookclerk-plugins/platform`.

# Arguments

* `root` - Cargo workspace root directory.

# Returns

On success, the inner `Vec<DiscoveredGuest>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn discover_platform(root: &std::path::Path) -> anyhow::Result<Vec<DiscoveredGuest>>
```



## bookclerk_dev::plugins::guest_install_leaf

*Function*

Install-directory leaf (`pk-` + 128-bit digest) for `guest`.

# Errors

Returns when [`guest_plugin_key`](plugins.md#bookclerk_devpluginsguest_plugin_key) fails.

```rust
fn guest_install_leaf(guest: &DiscoveredGuest) -> anyhow::Result<String>
```



## bookclerk_dev::plugins::guest_plugin_key

*Function*

Canonical PluginKey used when staging/installing `guest`.

Platform artifacts use `platform:bookclerk/{package}`. Workspace
Cargo packages use the crates.io `cargo:` form so dest leaves stay stable.
Other trees use a path key of the source directory.

# Errors

Returns when the id or coordinate cannot form a PluginKey.

```rust
fn guest_plugin_key(guest: &DiscoveredGuest) -> anyhow::Result<bookclerk_plugin_catalog::PluginKey>
```



## bookclerk_dev::plugins::guests_by_id

*Function*

Resolves plugin manifest ids to discovered guests across every tier.

Lets CI and developers build or stage one guest (for example `libro`)
without the rest of its tier.

# Arguments

* `root` - Cargo workspace root directory.
* `ids` - Plugin manifest ids (`plugin.toml` `id`); duplicates are ignored.

# Returns

Guests in the order first requested.

# Errors

Returns an error naming every unknown id, or when discovery fails.

```rust
fn guests_by_id(root: &std::path::Path, ids: &[String]) -> anyhow::Result<Vec<DiscoveredGuest>>
```



## bookclerk_dev::plugins::install_platform

*Function*

Install platform guests into `$FILES_DIR/plugins/{plugin-key-fs-id}/`.

# Arguments

* `root` - Cargo workspace root directory.
* `files_dir` - Bookclerk files directory to wipe and recreate.
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn install_platform(root: &std::path::Path, files_dir: &std::path::Path, release: bool) -> anyhow::Result<()>
```



## bookclerk_dev::plugins::packages_for

*Function*

Resolve Cargo `-p` names for [`BuildSelection`](plugins.md#bookclerk_devpluginsbuildselection).

# Arguments

* `root` - Cargo workspace root directory.
* `sel` - `sel` input for this call.

# Returns

On success, the inner `Vec<String>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn packages_for(root: &std::path::Path, sel: BuildSelection) -> anyhow::Result<Vec<String>>
```



## bookclerk_dev::plugins::packages_for_guests

*Function*

Cargo `-p` names needed to build `guests` (workerd guests ship `modules/`).

# Arguments

* `guests` - Guests resolved by [`guests_by_id`](plugins.md#bookclerk_devpluginsguests_by_id) or tier discovery.

# Returns

Unique native package names in input order.

```rust
fn packages_for_guests(guests: &[DiscoveredGuest]) -> Vec<String>
```



## bookclerk_dev::plugins::stage_optional_for_pack

*Function*

Stage optional guests for `package-plugins`.

# Arguments

* `root` - Cargo workspace root directory.
* `dest` - Filesystem path (`dest`).
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn stage_optional_for_pack(root: &std::path::Path, dest: &std::path::Path, release: bool) -> anyhow::Result<()>
```



## bookclerk_dev::plugins::stage_platform_for_pack

*Function*

Stage platform guests into a temp dir for `package-platform` bundling.

# Arguments

* `root` - Cargo workspace root directory.
* `dest` - Filesystem path (`dest`).
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.
* `skip_build` - Boolean flag `skip_build`.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn stage_platform_for_pack(root: &std::path::Path, dest: &std::path::Path, release: bool, skip_build: bool) -> anyhow::Result<()>
```



## bookclerk_dev::plugins::stage_plugins

*Function*

Stage optional and/or example guests into `dest` (catalog layout).

# Arguments

* `root` - Cargo workspace root directory.
* `dest` - Filesystem path (`dest`).
* `release` - When true, install under `target/release/`; otherwise `target/debug/`.
* `optional` - Stage every guest under `crates/bookclerk-plugins/optional`.
* `examples` - Stage every guest under `examples`.
* `plugin_ids` - Additional individual guests by manifest id (`--plugin`).
* `skip_build` - Boolean flag `skip_build`.

# Returns

The successful result value for this operation.

# Errors

Returns an error when nothing is selected, an id is unknown or names a
platform guest (those are installed by `install-platform`), or the
underlying I/O, parse, or build operation fails.

```rust
fn stage_plugins(root: &std::path::Path, dest: &std::path::Path, release: bool, optional: bool, examples: bool, plugin_ids: &[String], skip_build: bool) -> anyhow::Result<()>
```



