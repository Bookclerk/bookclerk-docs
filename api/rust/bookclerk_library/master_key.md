**bookclerk_library > master_key**

# Module: master_key

## Contents

**Structs**

- [`MasterKey`](#masterkey) - 32-byte data-encryption key (zeroized on drop).
- [`MasterKeyResolution`](#masterkeyresolution) - DEK loaded from `master.key`, plus whether this call created the file.

**Enums**

- [`MasterKeyFormat`](#masterkeyformat) - On-disk `master.key` envelope kind.

**Functions**

- [`configure_master_key`](#configure_master_key) - Load or mint the DEK and cache it for this process.
- [`configure_master_key_with`](#configure_master_key_with) - Load or mint the DEK using an explicit password (env / config), then cache it.
- [`discard_minted_master_key`](#discard_minted_master_key) - Delete a `master.key` this process just created and drop it from the cache.
- [`encode_master_key_file`](#encode_master_key_file) - The `master.key` file contents for `dek`: `BCK2` (Argon2id +
- [`inspect_master_key`](#inspect_master_key) - Inspect the on-disk envelope without unlocking (missing → `None`).
- [`install_master_key`](#install_master_key) - Installs `dek` as `{files_dir}/master.key`: a clerk joining a library
- [`master_key_fingerprint`](#master_key_fingerprint) - SHA-256 hex fingerprint of the unwrapped DEK.
- [`master_key_path`](#master_key_path) - Path to `{files_dir}/master.key`.
- [`prepare_master_key_dir`](#prepare_master_key_dir) - Creates the directory that holds `master.key` and sets it to mode
- [`require_master_key`](#require_master_key) - Return the process-cached DEK, or resolve from `files_dir` if not configured.
- [`resolve_master_key`](#resolve_master_key) - Resolve the DEK from env + `master.key` (minting when needed).
- [`resolve_master_key_detailed`](#resolve_master_key_detailed) - Resolve the DEK and report whether `master.key` was created by this call.
- [`resolve_master_key_with`](#resolve_master_key_with) - Resolve the DEK with an explicit password override.
- [`seal_with_dek`](#seal_with_dek) - Seal plaintext with the DEK (random nonce). Fast — no Argon2.
- [`uncache_master_key`](#uncache_master_key) - Drops `key` from the process cache when it is the cached DEK.
- [`unseal_with_dek`](#unseal_with_dek) - Unseal ciphertext with the DEK.
- [`warn_unwrapped_master_key`](#warn_unwrapped_master_key) - Warns that the `master.key` at `path` was written as `BCK1` (the DEK
- [`wrap_master_key`](#wrap_master_key) - Wrap an existing `BCK1` `master.key` with `password` (`BCK2`).

**Constants**

- [`AUTH_PASSWORD_ENV`](#auth_password_env) - Env var that wraps the DEK in `master.key` (optional).
- [`MASTER_KEY_FILE_NAME`](#master_key_file_name) - Bootstrap key file under `BOOKCLERK_FILES_DIR` (auto-minted when missing).

---

## bookclerk_library::master_key::AUTH_PASSWORD_ENV

*Constant*: `&str`

Env var that wraps the DEK in `master.key` (optional).



## bookclerk_library::master_key::MASTER_KEY_FILE_NAME

*Constant*: `&str`

Bootstrap key file under `BOOKCLERK_FILES_DIR` (auto-minted when missing).



## bookclerk_library::master_key::MasterKey

*Struct*

32-byte data-encryption key (zeroized on drop).

**Methods:**

- `fn as_bytes(self: &Self) -> &[u8; 32]` - Returns the 32-byte DEK without copying.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`
- **Zeroize**
  - `fn zeroize(self: & mut Self)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> MasterKey`



## bookclerk_library::master_key::MasterKeyFormat

*Enum*

On-disk `master.key` envelope kind.

**Variants:**
- `Raw` - Raw 32-byte DEK (`BCK1`).
- `Wrapped` - Password-wrapped DEK (`BCK2`).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MasterKeyFormat`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MasterKeyFormat) -> bool`



## bookclerk_library::master_key::MasterKeyResolution

*Struct*

DEK loaded from `master.key`, plus whether this call created the file.

**Fields:**
- `key: MasterKey` - Unwrapped data-encryption key.
- `minted: bool` - True when this call created `master.key` (lost create races are false).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MasterKeyResolution`



## bookclerk_library::master_key::configure_master_key

*Function*

Load or mint the DEK and cache it for this process.

Password comes from `BOOKCLERK_AUTH_PASSWORD` only. Prefer
[`configure_master_key_with`](master_key.md#bookclerk_librarymaster_keyconfigure_master_key_with) when `[auth].password` may apply.

Call once at CLI/daemon startup after `paths.ensure_dirs()`.

# Errors

Returns an error when the operation fails.

```rust
fn configure_master_key(files_dir: &std::path::Path) -> crate::error::Result<MasterKey>
```



## bookclerk_library::master_key::configure_master_key_with

*Function*

Load or mint the DEK using an explicit password (env / config), then cache it.

When `password` is set and `master.key` is still `BCK1`, re-wraps to `BCK2`
in place (same DEK bytes — sealed ciphertext stays valid). Empty `password`
is treated as absent.

If the resolved DEK identity differs from the previously cached one (e.g. a
replaced `master.key`), the plaintext unseal cache is flushed so callers
cannot observe stale plaintext under the new key.

# Errors

Returns an error when the operation fails.

```rust
fn configure_master_key_with(files_dir: &std::path::Path, password: Option<&str>) -> crate::error::Result<MasterKey>
```



## bookclerk_library::master_key::discard_minted_master_key

*Function*

Delete a `master.key` this process just created and drop it from the cache.

# Errors

Returns an error when the file exists and cannot be removed.

```rust
fn discard_minted_master_key(files_dir: &std::path::Path, key: &MasterKey) -> crate::error::Result<()>
```



## bookclerk_library::master_key::encode_master_key_file

*Function*

The `master.key` file contents for `dek`: `BCK2` (Argon2id +
XChaCha20-Poly1305) when `password` is set and non-empty after
trimming, `BCK1` otherwise. A caller that stages the file itself (a
clerk applying its bootstrap as one recoverable step) writes these
bytes with mode `0600`.

# Errors

Returns `LibraryError::Invalid` when `dek` is not 32 bytes, and the
wrapping error.

```rust
fn encode_master_key_file(dek: &[u8], password: Option<&str>) -> crate::error::Result<zeroize::Zeroizing<Vec<u8>>>
```



## bookclerk_library::master_key::inspect_master_key

*Function*

Inspect the on-disk envelope without unlocking (missing → `None`).

# Errors

Returns an error when the operation fails.

```rust
fn inspect_master_key(files_dir: &std::path::Path) -> crate::error::Result<Option<MasterKeyFormat>>
```



## bookclerk_library::master_key::install_master_key

*Function*

Installs `dek` as `{files_dir}/master.key`: a clerk joining a library
receives the library's DEK (sealed to its node key, never the
passphrase-wrapped file) and keeps it under its own `password`.

Writes [`encode_master_key_file`](master_key.md#bookclerk_librarymaster_keyencode_master_key_file)'s bytes, mode `0600`, replacing any
existing file atomically. The process cache is not changed: the daemon
restarts on the new bootstrap.

# Errors

Returns an error when `dek` is not 32 bytes or the file cannot be
written.

```rust
fn install_master_key(files_dir: &std::path::Path, dek: &[u8], password: Option<&str>) -> crate::error::Result<()>
```



## bookclerk_library::master_key::master_key_fingerprint

*Function*

SHA-256 hex fingerprint of the unwrapped DEK.

This identifies the cluster secret root. It is not a password hash: the
passphrase, when set, is stretched with Argon2id before it unwraps this
key. Wrapping changes the file bytes and leaves this fingerprint stable.

```rust
fn master_key_fingerprint(key: &MasterKey) -> String
```



## bookclerk_library::master_key::master_key_path

*Function*

Path to `{files_dir}/master.key`.

```rust
fn master_key_path(files_dir: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_library::master_key::prepare_master_key_dir

*Function*

Creates the directory that holds `master.key` and sets it to mode
`0700` on Unix, as a first mint does.

# Errors

Returns an error when the directory cannot be created or hardened.

```rust
fn prepare_master_key_dir(files_dir: &std::path::Path) -> crate::error::Result<()>
```



## bookclerk_library::master_key::require_master_key

*Function*

Return the process-cached DEK, or resolve from `files_dir` if not configured.

# Errors

Returns an error when the operation fails.

```rust
fn require_master_key(files_dir: Option<&std::path::Path>) -> crate::error::Result<MasterKey>
```



## bookclerk_library::master_key::resolve_master_key

*Function*

Resolve the DEK from env + `master.key` (minting when needed).

# Errors

Returns an error when the operation fails.

```rust
fn resolve_master_key(files_dir: &std::path::Path) -> crate::error::Result<MasterKey>
```



## bookclerk_library::master_key::resolve_master_key_detailed

*Function*

Resolve the DEK and report whether `master.key` was created by this call.

# Errors

Returns an error when the file cannot be read, unwrapped, or created.

```rust
fn resolve_master_key_detailed(files_dir: &std::path::Path, password: Option<&str>) -> crate::error::Result<MasterKeyResolution>
```



## bookclerk_library::master_key::resolve_master_key_with

*Function*

Resolve the DEK with an explicit password override.

# Errors

Returns an error when the operation fails.

```rust
fn resolve_master_key_with(files_dir: &std::path::Path, password: Option<&str>) -> crate::error::Result<MasterKey>
```



## bookclerk_library::master_key::seal_with_dek

*Function*

Seal plaintext with the DEK (random nonce). Fast — no Argon2.

# Errors

Returns an error when the operation fails.

```rust
fn seal_with_dek(plaintext: &[u8], dek: &MasterKey) -> crate::error::Result<(Vec<u8>, Vec<u8>)>
```



## bookclerk_library::master_key::uncache_master_key

*Function*

Drops `key` from the process cache when it is the cached DEK.

```rust
fn uncache_master_key(key: &MasterKey)
```



## bookclerk_library::master_key::unseal_with_dek

*Function*

Unseal ciphertext with the DEK.

# Errors

Returns an error when the operation fails.

```rust
fn unseal_with_dek(ciphertext: &[u8], nonce: &[u8], dek: &MasterKey) -> crate::error::Result<Vec<u8>>
```



## bookclerk_library::master_key::warn_unwrapped_master_key

*Function*

Warns that the `master.key` at `path` was written as `BCK1` (the DEK
unwrapped at rest), and how to wrap it.

```rust
fn warn_unwrapped_master_key(path: &std::path::Path)
```



## bookclerk_library::master_key::wrap_master_key

*Function*

Wrap an existing `BCK1` `master.key` with `password` (`BCK2`).

No-op (still unlocks) when already `BCK2` and `password` is correct.
Fails if the file is missing — mint via [`configure_master_key_with`](master_key.md#bookclerk_librarymaster_keyconfigure_master_key_with) first.

# Errors

Returns an error when the operation fails.

```rust
fn wrap_master_key(files_dir: &std::path::Path, password: &str) -> crate::error::Result<MasterKey>
```



