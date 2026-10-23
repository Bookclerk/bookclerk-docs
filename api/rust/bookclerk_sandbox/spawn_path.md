**bookclerk_sandbox > spawn_path**

# Module: spawn_path

## Contents

**Enums**

- [`SpawnPathError`](#spawnpatherror) - Why a spawn path was rejected.

**Functions**

- [`canonicalize`](#canonicalize) - [`std::fs::canonicalize`](https://doc.rust-lang.org/stable/std/fs/fn.canonicalize.html) that also works inside a Windows AppContainer.
- [`create_process_path`](#create_process_path) - Path passed to `CreateProcess`, `Command`, or a child that parses Win32 paths.
- [`require_absolute_or_name`](#require_absolute_or_name) - Absolute path with no NUL, or a single PATH lookup name (`cargo`, `python3`).
- [`require_absolute_spawn_path`](#require_absolute_spawn_path) - Like [`require_absolute_or_name`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_absolute_or_name), but the path must be absolute (no PATH names).
- [`require_existing_regular_file`](#require_existing_regular_file) - Existing regular file after resolving relative paths against the process cwd.
- [`require_helper_beside_or_absolute`](#require_helper_beside_or_absolute) - Accepts `path` when it equals `beside`'s sibling named `helper_name`, or is an
- [`require_spawn_executable`](#require_spawn_executable) - Absolute existing regular file (or a single PATH name), with no interior NUL.
- [`require_under_root`](#require_under_root) - Canonicalizes `path` and requires it to stay under canonical `root`.

---

## bookclerk_sandbox::spawn_path::SpawnPathError

*Enum*

Why a spawn path was rejected.

**Variants:**
- `Empty` - Empty `OsStr` / path.
- `Nul` - Interior NUL byte (or wide NUL on Windows).
- `NotAbsolute(std::path::PathBuf)` - Relative path with separators (not a single PATH lookup name).
- `NotFile(std::path::PathBuf)` - Absolute path that is not a regular file when one is required.
- `OutsideRoot{ path: std::path::PathBuf, root: std::path::PathBuf }` - Canonical path escapes the trusted root.
- `Canonicalize{ path: std::path::PathBuf, source: std::io::Error }` - `canonicalize` failed for an existing path that must resolve.

**Trait Implementations:**

- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::spawn_path::canonicalize

*Function*

[`std::fs::canonicalize`](https://doc.rust-lang.org/stable/std/fs/fn.canonicalize.html) that also works inside a Windows AppContainer.

On Windows, `canonicalize` asks the mount manager for the DOS drive letter
(`VOLUME_NAME_DOS`), which an AppContainer token may not query, so every
path fails with "Access is denied" even when the file is readable. The
fallback resolves the volume-relative final path from the open handle,
re-attaches the caller's drive, and accepts it only when it opens as the
very same file (volume serial + file index); otherwise the original error
stands. `GetFinalPathNameByHandleW` follows reparse points; a junction to
another volume yields a different identity and falls back to the original
error. Callers that later reopen the returned path by name (rather than by
the handle used here) still see the usual TOCTOU gap — this helper proves
identity at the moment of canonicalize, not at a later open.

# Errors

Returns the [`std::fs::canonicalize`](https://doc.rust-lang.org/stable/std/fs/fn.canonicalize.html) error when the path cannot be
resolved (or, on Windows, cannot be proven identical by handle).

```rust
fn canonicalize(path: &std::path::Path) -> std::io::Result<std::path::PathBuf>
```



## bookclerk_sandbox::spawn_path::create_process_path

*Function*

Path passed to `CreateProcess`, `Command`, or a child that parses Win32 paths.

Windows canonical paths use the `\\?\` prefix. An AppContainer can open
that form for read and still be denied process creation
(`ERROR_ACCESS_DENIED`). Pinned `workerd` rejects the same prefix: its
filesystem parser treats `?` as the first component and aborts. The prefix
is removed only after callers have already canonicalized, so the Win32 path
names the same resolved file. Other platforms return `path` unchanged.

```rust
fn create_process_path(path: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_sandbox::spawn_path::require_absolute_or_name

*Function*

Absolute path with no NUL, or a single PATH lookup name (`cargo`, `python3`).

Does not require the path to exist (useful for generated argv file operands).
Lexical `..` in an absolute spelling is allowed; callers that need a real
file should use [`require_spawn_executable`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_spawn_executable) (canonicalize).

Prefer not using this for operator-selected tools like `$CARGO` — those may
legitimately be relative (`./custom-cargo`). Use it when the product
contract is “absolute or PATH name” for a helper beside the host.

# Errors

Returns [`SpawnPathError`](spawn_path.md#bookclerk_sandboxspawn_pathspawnpatherror) when the path is empty, contains NUL, or is a
multi-component relative path.

```rust
fn require_absolute_or_name(path: &std::path::Path) -> Result<std::path::PathBuf, SpawnPathError>
```



## bookclerk_sandbox::spawn_path::require_absolute_spawn_path

*Function*

Like [`require_absolute_or_name`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_absolute_or_name), but the path must be absolute (no PATH names).

# Errors

Returns [`SpawnPathError`](spawn_path.md#bookclerk_sandboxspawn_pathspawnpatherror) when the path is empty, contains NUL, or is not absolute.

```rust
fn require_absolute_spawn_path(path: &std::path::Path) -> Result<std::path::PathBuf, SpawnPathError>
```



## bookclerk_sandbox::spawn_path::require_existing_regular_file

*Function*

Existing regular file after resolving relative paths against the process cwd.

Absolute and multi-component relative paths (`./worker`, `bin/tool`) are
accepted when they canonicalize to a regular file. A bare name is accepted
only when that name exists as a file under the current directory — never as
an ambient `PATH` lookup. Use this for media worker / jail path knobs that
historically required `path.is_file()`.

# Errors

Returns [`SpawnPathError`](spawn_path.md#bookclerk_sandboxspawn_pathspawnpatherror) when the path is empty/NUL, cannot be resolved, or
is not a regular file.

```rust
fn require_existing_regular_file(path: &std::path::Path) -> Result<std::path::PathBuf, SpawnPathError>
```



## bookclerk_sandbox::spawn_path::require_helper_beside_or_absolute

*Function*

Accepts `path` when it equals `beside`'s sibling named `helper_name`, or is an
absolute existing file (env override).

Equality with the beside-host helper is the preferred product path. Absolute
env overrides still require [`require_spawn_executable`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_spawn_executable); that is an
explicit operator trust decision, not proof the binary is safe.

Bare PATH names (single component, non-absolute) are rejected for this API
unless they match the beside-host helper. Callers that intentionally want
PATH lookup must use [`require_absolute_or_name`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_absolute_or_name) / [`require_spawn_executable`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_spawn_executable)
directly.

# Errors

Returns [`SpawnPathError`](spawn_path.md#bookclerk_sandboxspawn_pathspawnpatherror) when validation fails.

```rust
fn require_helper_beside_or_absolute(path: &std::path::Path, helper_name: &str, beside: Option<&std::path::Path>) -> Result<std::path::PathBuf, SpawnPathError>
```



## bookclerk_sandbox::spawn_path::require_spawn_executable

*Function*

Absolute existing regular file (or a single PATH name), with no interior NUL.

Absolute paths are canonicalized so symlink targets and lexical `..` resolve
to the file that will actually be executed. PATH lookup names are returned
as-is for the OS to resolve at spawn time.

Prefer [`require_existing_regular_file`](spawn_path.md#bookclerk_sandboxspawn_pathrequire_existing_regular_file) for configured/env worker paths that
must identify a real file (including cwd-relative `./bin`) and must not
authorize ambient PATH lookup for a bare name.

# Errors

Returns [`SpawnPathError`](spawn_path.md#bookclerk_sandboxspawn_pathspawnpatherror) when validation fails, canonicalize fails, or an
absolute path is not a regular file.

```rust
fn require_spawn_executable(path: &std::path::Path) -> Result<std::path::PathBuf, SpawnPathError>
```



## bookclerk_sandbox::spawn_path::require_under_root

*Function*

Canonicalizes `path` and requires it to stay under canonical `root`.

When `path` does not exist yet, canonicalizes its parent and rejoins the
final component so generated files under a trusted session directory work.

# Errors

Returns [`SpawnPathError`](spawn_path.md#bookclerk_sandboxspawn_pathspawnpatherror) when the path is unsafe or escapes `root`.

```rust
fn require_under_root(path: &std::path::Path, root: &std::path::Path) -> Result<std::path::PathBuf, SpawnPathError>
```



