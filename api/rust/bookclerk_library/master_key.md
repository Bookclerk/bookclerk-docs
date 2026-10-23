# bookclerk_library::master_key

Data-encryption key (DEK) for `encrypted_secrets`.

## Design

Secrets in the DB are sealed with a **random 32-byte DEK** + per-row
XChaCha20-Poly1305 nonce (format `sealed-v1`). The DEK is **not** derived
per secret (no Argon2 on every load/save).

### Bootstrap (outside the DB)

| Source | Behavior |
| --- | --- |
| `BOOKCLERK_AUTH_PASSWORD` or `[auth].password` | Wraps/unwraps the DEK in `{files_dir}/master.key` |
| Auto-mint | If neither password nor file exists, generates a DEK and writes `master.key` (mode `0600`) |

A later password can wrap an existing `BCK1` file in place (CLI
`config master-key wrap`, `config set auth.password`, or daemon config
reload). The operator API token is **independent** and short-lived for
HTTP auth only. Rotating it does **not** re-encrypt library secrets.

Prefer a single opaque ciphertext blob per secret (not per-field columns):
Audible envelopes are already opaque, updates stay atomic, and one unseal
recovers the whole credential.

| Item | Kind | Summary |
| --- | --- | --- |
| [`MasterKey`](#masterkey) | struct | 32-byte data-encryption key (zeroized on drop). |
| [`MasterKeyResolution`](#masterkeyresolution) | struct | DEK loaded from `master.key`, plus whether this call created the file. |
| [`MasterKeyFormat`](#masterkeyformat) | enum | On-disk `master.key` envelope kind. |
| [`master_key_path`](#master_key_path) | function | Path to `{files_dir}/master.key`. |
| [`inspect_master_key`](#inspect_master_key) | function | Inspect the on-disk envelope without unlocking (missing → `None`). |
| [`configure_master_key`](#configure_master_key) | function | Load or mint the DEK and cache it for this process. |
| [`configure_master_key_with`](#configure_master_key_with) | function | Load or mint the DEK using an explicit password (env / config), then cache it. |
| [`require_master_key`](#require_master_key) | function | Return the process-cached DEK, or resolve from `files_dir` if not configured. |
| [`resolve_master_key`](#resolve_master_key) | function | Resolve the DEK from env + `master.key` (minting when needed). |
| [`resolve_master_key_with`](#resolve_master_key_with) | function | Resolve the DEK with an explicit password override. |
| [`resolve_master_key_detailed`](#resolve_master_key_detailed) | function | Resolve the DEK and report whether `master.key` was created by this call. |
| [`master_key_fingerprint`](#master_key_fingerprint) | function | SHA-256 hex fingerprint of the unwrapped DEK. |
| [`uncache_master_key`](#uncache_master_key) | function | Drops `key` from the process cache when it is the cached DEK. |
| [`discard_minted_master_key`](#discard_minted_master_key) | function | Delete a `master.key` this process just created and drop it from the cache. |
| [`wrap_master_key`](#wrap_master_key) | function | Wrap an existing `BCK1` `master.key` with `password` (`BCK2`). |
| [`install_master_key`](#install_master_key) | function | Installs `dek` as `{files_dir}/master.key`: a clerk joining a library receives the library's DEK (sealed to its node key, never the passphrase-wrapped file) and keeps it under its own `password`. |
| [`prepare_master_key_dir`](#prepare_master_key_dir) | function | Creates the directory that holds `master.key` and sets it to mode `0700` on Unix, as a first mint does. |
| [`warn_unwrapped_master_key`](#warn_unwrapped_master_key) | function | Warns that the `master.key` at `path` was written as `BCK1` (the DEK unwrapped at rest), and how to wrap it. |
| [`encode_master_key_file`](#encode_master_key_file) | function | The `master.key` file contents for `dek`: `BCK2` (Argon2id + XChaCha20-Poly1305) when `password` is set and non-empty after trimming, `BCK1` otherwise. A caller that stages the file itself (a clerk applying its bootstrap as one recoverable step) writes these bytes with mode `0600`. |
| [`seal_with_dek`](#seal_with_dek) | function | Seal plaintext with the DEK (random nonce). Fast — no Argon2. |
| [`unseal_with_dek`](#unseal_with_dek) | function | Unseal ciphertext with the DEK. |
| [`AUTH_PASSWORD_ENV`](#auth_password_env) | constant | Env var that wraps the DEK in `master.key` (optional). |
| [`MASTER_KEY_FILE_NAME`](#master_key_file_name) | constant | Bootstrap key file under `BOOKCLERK_FILES_DIR` (auto-minted when missing). |

## Structs

### MasterKey

```rust
pub struct MasterKey {
    /* private fields */
}
```

32-byte data-encryption key (zeroized on drop).

#### MasterKey::as_bytes

```rust
pub fn as_bytes(&self) -> &[u8; 32]
```

Returns the 32-byte DEK without copying.

Implements: `Clone`, `Debug`, `Drop`, `Zeroize`

### MasterKeyResolution

```rust
pub struct MasterKeyResolution {
    pub key: MasterKey,
    pub minted: bool,
}
```

DEK loaded from `master.key`, plus whether this call created the file.

**Fields**

- `key`: Unwrapped data-encryption key.
- `minted`: True when this call created `master.key` (lost create races are false).

Implements: `Clone`

## Enums

### MasterKeyFormat

```rust
pub enum MasterKeyFormat {
    Raw,
    Wrapped,
}
```

On-disk `master.key` envelope kind.

**Variants**

- `Raw`: Raw 32-byte DEK (`BCK1`).
- `Wrapped`: Password-wrapped DEK (`BCK2`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### master_key_path

```rust
pub fn master_key_path(files_dir: &Path) -> PathBuf
```

Path to `{files_dir}/master.key`.

### inspect_master_key

```rust
pub fn inspect_master_key(files_dir: &Path) -> Result<Option<MasterKeyFormat>>
```

Inspect the on-disk envelope without unlocking (missing → `None`).

#### Errors

Returns an error when the operation fails.

### configure_master_key

```rust
pub fn configure_master_key(files_dir: &Path) -> Result<MasterKey>
```

Load or mint the DEK and cache it for this process.

Password comes from `BOOKCLERK_AUTH_PASSWORD` only. Prefer
[`configure_master_key_with`](#configure_master_key_with) when `[auth].password` may apply.

Call once at CLI/daemon startup after `paths.ensure_dirs()`.

#### Errors

Returns an error when the operation fails.

### configure_master_key_with

```rust
pub fn configure_master_key_with(
    files_dir: &Path,
    password: Option<&str>,
) -> Result<MasterKey>
```

Load or mint the DEK using an explicit password (env / config), then cache it.

When `password` is set and `master.key` is still `BCK1`, re-wraps to `BCK2`
in place (same DEK bytes — sealed ciphertext stays valid). Empty `password`
is treated as absent.

If the resolved DEK identity differs from the previously cached one (e.g. a
replaced `master.key`), the plaintext unseal cache is flushed so callers
cannot observe stale plaintext under the new key.

#### Errors

Returns an error when the operation fails.

### require_master_key

```rust
pub fn require_master_key(files_dir: Option<&Path>) -> Result<MasterKey>
```

Return the process-cached DEK, or resolve from `files_dir` if not configured.

#### Errors

Returns an error when the operation fails.

### resolve_master_key

```rust
pub fn resolve_master_key(files_dir: &Path) -> Result<MasterKey>
```

Resolve the DEK from env + `master.key` (minting when needed).

#### Errors

Returns an error when the operation fails.

### resolve_master_key_with

```rust
pub fn resolve_master_key_with(
    files_dir: &Path,
    password: Option<&str>,
) -> Result<MasterKey>
```

Resolve the DEK with an explicit password override.

#### Errors

Returns an error when the operation fails.

### resolve_master_key_detailed

```rust
pub fn resolve_master_key_detailed(
    files_dir: &Path,
    password: Option<&str>,
) -> Result<MasterKeyResolution>
```

Resolve the DEK and report whether `master.key` was created by this call.

#### Errors

Returns an error when the file cannot be read, unwrapped, or created.

### master_key_fingerprint

```rust
pub fn master_key_fingerprint(key: &MasterKey) -> String
```

SHA-256 hex fingerprint of the unwrapped DEK.

This identifies the cluster secret root. It is not a password hash: the
passphrase, when set, is stretched with Argon2id before it unwraps this
key. Wrapping changes the file bytes and leaves this fingerprint stable.

### uncache_master_key

```rust
pub fn uncache_master_key(key: &MasterKey)
```

Drops `key` from the process cache when it is the cached DEK.

### discard_minted_master_key

```rust
pub fn discard_minted_master_key(files_dir: &Path, key: &MasterKey) -> Result<()>
```

Delete a `master.key` this process just created and drop it from the cache.

#### Errors

Returns an error when the file exists and cannot be removed.

### wrap_master_key

```rust
pub fn wrap_master_key(files_dir: &Path, password: &str) -> Result<MasterKey>
```

Wrap an existing `BCK1` `master.key` with `password` (`BCK2`).

No-op (still unlocks) when already `BCK2` and `password` is correct.
Fails if the file is missing — mint via [`configure_master_key_with`](#configure_master_key_with) first.

#### Errors

Returns an error when the operation fails.

### install_master_key

```rust
pub fn install_master_key(
    files_dir: &Path,
    dek: &[u8],
    password: Option<&str>,
) -> Result<()>
```

Installs `dek` as `{files_dir}/master.key`: a clerk joining a library
receives the library's DEK (sealed to its node key, never the
passphrase-wrapped file) and keeps it under its own `password`.

Writes [`encode_master_key_file`](#encode_master_key_file)'s bytes, mode `0600`, replacing any
existing file atomically. The process cache is not changed: the daemon
restarts on the new bootstrap.

#### Errors

Returns an error when `dek` is not 32 bytes or the file cannot be
written.

### prepare_master_key_dir

```rust
pub fn prepare_master_key_dir(files_dir: &Path) -> Result<()>
```

Creates the directory that holds `master.key` and sets it to mode
`0700` on Unix, as a first mint does.

#### Errors

Returns an error when the directory cannot be created or hardened.

### warn_unwrapped_master_key

```rust
pub fn warn_unwrapped_master_key(path: &Path)
```

Warns that the `master.key` at `path` was written as `BCK1` (the DEK
unwrapped at rest), and how to wrap it.

### encode_master_key_file

```rust
pub fn encode_master_key_file(
    dek: &[u8],
    password: Option<&str>,
) -> Result<Zeroizing<Vec<u8>>>
```

The `master.key` file contents for `dek`: `BCK2` (Argon2id +
XChaCha20-Poly1305) when `password` is set and non-empty after
trimming, `BCK1` otherwise. A caller that stages the file itself (a
clerk applying its bootstrap as one recoverable step) writes these
bytes with mode `0600`.

#### Errors

Returns [`LibraryError::Invalid`](index.md#libraryerror) when `dek` is not 32 bytes, and the
wrapping error.

### seal_with_dek

```rust
pub fn seal_with_dek(plaintext: &[u8], dek: &MasterKey) -> Result<(Vec<u8>, Vec<u8>)>
```

Seal plaintext with the DEK (random nonce). Fast — no Argon2.

#### Errors

Returns an error when the operation fails.

### unseal_with_dek

```rust
pub fn unseal_with_dek(
    ciphertext: &[u8],
    nonce: &[u8],
    dek: &MasterKey,
) -> Result<Vec<u8>>
```

Unseal ciphertext with the DEK.

#### Errors

Returns an error when the operation fails.

## Constants

### AUTH_PASSWORD_ENV

```rust
pub const AUTH_PASSWORD_ENV: &str = "BOOKCLERK_AUTH_PASSWORD";
```

Env var that wraps the DEK in `master.key` (optional).

### MASTER_KEY_FILE_NAME

```rust
pub const MASTER_KEY_FILE_NAME: &str = "master.key";
```

Bootstrap key file under `BOOKCLERK_FILES_DIR` (auto-minted when missing).
