# bookclerk_dev::package

Release packaging aligned with `docs/plugin-registry.md` artifact naming.

| Item | Kind | Summary |
| --- | --- | --- |
| [`host_target_triple`](#host_target_triple) | function | Host rustc target triple for the machine running the packager. |
| [`bookclerk_target`](#bookclerk_target) | function | Canonical Bookclerk artifact target for the host (e.g. `linux-x64-gnu`). |
| [`normalize_bookclerk_target`](#normalize_bookclerk_target) | function | Map a Bookclerk target or legacy rustc triple to a Bookclerk target name. |
| [`uses_zip_archive`](#uses_zip_archive) | function | Whether archives for this Bookclerk target (or rustc triple) use `.zip`. |
| [`package_plugins`](#package_plugins) | function | Packages discovered plugin guests into the release artifacts directory. |
| [`package_hosts`](#package_hosts) | function | Packages host binaries (`bookclerk`, `bookclerkd`, helpers) into artifacts. |
| [`package_platform`](#package_platform) | function | Packages platform-only guests (sqlite / local) into artifacts. |

## Functions

### host_target_triple

```rust
pub fn host_target_triple() -> &'static str
```

Host rustc target triple for the machine running the packager.

Prefer [`bookclerk_target`](#bookclerk_target) for new archive filenames. This remains for
callers that still speak rustc triples; [`normalize_bookclerk_target`](#normalize_bookclerk_target) maps
either form onto a Bookclerk target name.

### bookclerk_target

```rust
pub fn bookclerk_target() -> &'static str
```

Canonical Bookclerk artifact target for the host (e.g. `linux-x64-gnu`).

### normalize_bookclerk_target

```rust
pub fn normalize_bookclerk_target(triple_or_target: &str) -> Option<&'static str>
```

Map a Bookclerk target or legacy rustc triple to a Bookclerk target name.

#### Arguments

* `triple_or_target` - String `triple_or_target` for this call.

#### Returns

`Some(...)` when found / applicable; otherwise `None`.

### uses_zip_archive

```rust
pub fn uses_zip_archive(triple_or_target: &str) -> bool
```

Whether archives for this Bookclerk target (or rustc triple) use `.zip`.

#### Arguments

* `triple_or_target` - String `triple_or_target` for this call.

#### Returns

`true` when the predicate holds.

### package_plugins

```rust
pub fn package_plugins(root: &Path, out_dir: &Path, version: &str) -> Result<()>
```

Packages discovered plugin guests into the release artifacts directory.

#### Arguments

* `root` - Cargo workspace root directory.
* `out_dir` - Filesystem path (`out_dir`).
* `version` - String `version` for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### package_hosts

```rust
pub fn package_hosts(root: &Path, out_dir: &Path, version: &str) -> Result<()>
```

Packages host binaries (`bookclerk`, `bookclerkd`, helpers) into artifacts.

#### Arguments

* `root` - Cargo workspace root directory.
* `out_dir` - Filesystem path (`out_dir`).
* `version` - String `version` for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### package_platform

```rust
pub fn package_platform(root: &Path, out_dir: &Path, version: &str) -> Result<()>
```

Packages platform-only guests (sqlite / local) into artifacts.

#### Arguments

* `root` - Cargo workspace root directory.
* `out_dir` - Filesystem path (`out_dir`).
* `version` - String `version` for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.
