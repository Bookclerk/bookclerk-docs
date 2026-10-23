**bookclerk_sandbox > guest_ipc**

# Module: guest_ipc

## Contents

**Functions**

- [`create_guest_ipc_dir`](#create_guest_ipc_dir) - Create a mode `0700` directory under `/tmp` whose longest guest socket fits
- [`ensure_guest_ipc_fits`](#ensure_guest_ipc_fits) - Reject `dir` when `{dir}/.s.PGSQL.65535` does not fit in a macOS `sockaddr_un`.

**Constants**

- [`MACOS_SUN_PATH_CAPACITY`](#macos_sun_path_capacity) - macOS `sun_path` capacity. Rust rejects a path whose length is greater than

---

## bookclerk_sandbox::guest_ipc::MACOS_SUN_PATH_CAPACITY

*Constant*: `usize`

macOS `sun_path` capacity. Rust rejects a path whose length is greater than
or equal to this value.



## bookclerk_sandbox::guest_ipc::create_guest_ipc_dir

*Function*

Create a mode `0700` directory under `/tmp` whose longest guest socket fits
in a macOS `sockaddr_un`.

The returned path is canonical, so a `/tmp` symlink (`/private/tmp` on
macOS) is what the jail profile and the guest both see.

# Errors

Returns when `/tmp` cannot hold the directory, or when the canonical path
plus `.s.PGSQL.65535` does not fit. The error includes the path length and
the directory that was too long.

```rust
fn create_guest_ipc_dir() -> Result<std::path::PathBuf, String>
```



## bookclerk_sandbox::guest_ipc::ensure_guest_ipc_fits

*Function*

Reject `dir` when `{dir}/.s.PGSQL.65535` does not fit in a macOS `sockaddr_un`.

# Errors

The message includes the socket path length and `dir`.

```rust
fn ensure_guest_ipc_fits(dir: &std::path::Path) -> Result<(), String>
```



