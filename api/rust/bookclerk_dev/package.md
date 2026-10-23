**bookclerk_dev > package**

# Module: package

## Contents

**Functions**

- [`bookclerk_target`](#bookclerk_target) - Canonical Bookclerk artifact target for the host (e.g. `linux-x64-gnu`).
- [`host_target_triple`](#host_target_triple) - Host rustc target triple for the machine running the packager.
- [`normalize_bookclerk_target`](#normalize_bookclerk_target) - Map a Bookclerk target or legacy rustc triple to a Bookclerk target name.
- [`package_hosts`](#package_hosts) - Packages host binaries (`bookclerk`, `bookclerkd`, helpers) into artifacts.
- [`package_platform`](#package_platform) - Packages platform-only guests (sqlite / local) into artifacts.
- [`package_plugins`](#package_plugins) - Packages discovered plugin guests into the release artifacts directory.
- [`uses_zip_archive`](#uses_zip_archive) - Whether archives for this Bookclerk target (or rustc triple) use `.zip`.

---

## bookclerk_dev::package::bookclerk_target

*Function*

Canonical Bookclerk artifact target for the host (e.g. `linux-x64-gnu`).

```rust
fn bookclerk_target() -> &'static str
```



## bookclerk_dev::package::host_target_triple

*Function*

Host rustc target triple for the machine running the packager.

Prefer [`bookclerk_target`](package.md#bookclerk_devpackagebookclerk_target) for new archive filenames. This remains for
callers that still speak rustc triples; [`normalize_bookclerk_target`](package.md#bookclerk_devpackagenormalize_bookclerk_target) maps
either form onto a Bookclerk target name.

```rust
fn host_target_triple() -> &'static str
```



## bookclerk_dev::package::normalize_bookclerk_target

*Function*

Map a Bookclerk target or legacy rustc triple to a Bookclerk target name.

# Arguments

* `triple_or_target` - String `triple_or_target` for this call.

# Returns

`Some(...)` when found / applicable; otherwise `None`.

```rust
fn normalize_bookclerk_target(triple_or_target: &str) -> Option<&'static str>
```



## bookclerk_dev::package::package_hosts

*Function*

Packages host binaries (`bookclerk`, `bookclerkd`, helpers) into artifacts.

# Arguments

* `root` - Cargo workspace root directory.
* `out_dir` - Filesystem path (`out_dir`).
* `version` - String `version` for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn package_hosts(root: &std::path::Path, out_dir: &std::path::Path, version: &str) -> anyhow::Result<()>
```



## bookclerk_dev::package::package_platform

*Function*

Packages platform-only guests (sqlite / local) into artifacts.

# Arguments

* `root` - Cargo workspace root directory.
* `out_dir` - Filesystem path (`out_dir`).
* `version` - String `version` for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn package_platform(root: &std::path::Path, out_dir: &std::path::Path, version: &str) -> anyhow::Result<()>
```



## bookclerk_dev::package::package_plugins

*Function*

Packages discovered plugin guests into the release artifacts directory.

# Arguments

* `root` - Cargo workspace root directory.
* `out_dir` - Filesystem path (`out_dir`).
* `version` - String `version` for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn package_plugins(root: &std::path::Path, out_dir: &std::path::Path, version: &str) -> anyhow::Result<()>
```



## bookclerk_dev::package::uses_zip_archive

*Function*

Whether archives for this Bookclerk target (or rustc triple) use `.zip`.

# Arguments

* `triple_or_target` - String `triple_or_target` for this call.

# Returns

`true` when the predicate holds.

```rust
fn uses_zip_archive(triple_or_target: &str) -> bool
```



