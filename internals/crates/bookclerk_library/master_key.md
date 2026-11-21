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
| OS keychain | `master.key` is a `BCK3` pointer naming a keychain item that holds the `BCK1`/`BCK2` envelope |

### Key stores

The envelope (`BCK1` or `BCK2`) lives either in `master.key` itself or in
one OS keychain item ([`bookclerk_keystore`](../bookclerk_keystore/README.md)). On a keychain install
`master.key` holds a `BCK3` pointer: the item's id and the DEK's SHA-256
fingerprint, never key bytes. Loading follows whatever `master.key` says
and fails closed: a missing item, a locked keychain or a key that doesn't
match the pointer's fingerprint is an error, never a fresh mint.
[`set_key_store_preference`](#set_key_store_preference) picks where a *new* key goes (a first mint,
a shared branch's installed key, an import); [`move_master_key`](#move_master_key) moves an
existing one.

A later password can wrap an existing `BCK1` file in place (CLI
`config master-key wrap`, `config set auth.password`, or daemon config
reload). The operator API token is **independent** and short-lived for
HTTP auth only. Rotating it does **not** re-encrypt library secrets.

Prefer a single opaque ciphertext blob per secret (not per-field columns):
Audible envelopes are already opaque, updates stay atomic, and one unseal
recovers the whole credential.

| Item | Kind | Summary |
| --- | --- | --- |
| [`KeyStorePreference`](#keystorepreference) | struct | Where a new key goes, and whether that place is required. |
| [`MasterKey`](#masterkey) | struct | 32-byte data-encryption key (zeroized on drop). |
| [`MasterKeyResolution`](#masterkeyresolution) | struct | DEK loaded from `master.key`, plus whether this call created the file. |
| [`StoredMasterKey`](#storedmasterkey) | struct | A key `store_master_key` placed, waiting for its `master.key` to be published. |
| [`MasterKeyFormat`](#masterkeyformat) | enum | On-disk `master.key` envelope kind. |
| [`set_key_store_preference`](#set_key_store_preference) | function | Sets where this process writes new keys (first mint, a shared branch's installed key, an import). Existing keys stay where `master.key` says. |
| [`key_store_preference`](#key_store_preference) | function | The current store preference. |
| [`set_keychain_vault`](#set_keychain_vault) | function | Replaces the keychain this process uses, for tests (a `bookclerk_keystore::MemoryVault`) and for hosts that substitute their own store. |
| [`set_systemd_creds_vault`](#set_systemd_creds_vault) | function | Replaces how this process builds its `systemd-creds` store, for tests (a fake tool, or a `bookclerk_keystore::MemoryVault`). |
| [`master_key_path`](#master_key_path) | function | Path to `{files_dir}/master.key`. |
| [`inspect_master_key`](#inspect_master_key) | function | Inspect the on-disk envelope without unlocking (missing → `None`). |
| [`master_key_is_wrapped`](#master_key_is_wrapped) | function | Whether the key is password-wrapped (`BCK2`), reading the keychain item when `master.key` is a pointer. `None` when there is no `master.key`. |
| [`configure_master_key`](#configure_master_key) | function | Load or mint the DEK and cache it for this process. |
| [`configure_master_key_with`](#configure_master_key_with) | function | Load or mint the DEK using an explicit password (env / config), then cache it. |
| [`require_master_key`](#require_master_key) | function | Return the process-cached DEK, or resolve from `files_dir` if not configured. |
| [`resolve_master_key`](#resolve_master_key) | function | Resolve the DEK from env + `master.key` (minting when needed). |
| [`resolve_master_key_with`](#resolve_master_key_with) | function | Resolve the DEK with an explicit password override. |
| [`resolve_master_key_detailed`](#resolve_master_key_detailed) | function | Resolve the DEK and report whether `master.key` was created by this call. |
| [`master_key_fingerprint`](#master_key_fingerprint) | function | SHA-256 hex fingerprint of the unwrapped DEK. |
| [`uncache_master_key`](#uncache_master_key) | function | Drops `key` from the process cache when it is the cached DEK. |
| [`discard_minted_master_key`](#discard_minted_master_key) | function | Delete a `master.key` this process just created and drop it from the cache. |
| [`wrap_master_key`](#wrap_master_key) | function | Wrap an existing `BCK1` key with `password` (`BCK2`), in `master.key` or in the OS keychain item it points at. |
| [`install_master_key`](#install_master_key) | function | Installs `dek` as `{files_dir}/master.key`: a shared branch joining a library receives the library's DEK (sealed to its node key, never the passphrase-wrapped file) and keeps it under its own `password`. |
| [`prepare_master_key_dir`](#prepare_master_key_dir) | function | Creates the directory that holds `master.key` and sets it to mode `0700` on Unix, as a first mint does. |
| [`warn_unwrapped_master_key`](#warn_unwrapped_master_key) | function | Warns that the `master.key` at `path` was written as `BCK1` (the DEK unwrapped at rest), and how to wrap it. |
| [`encode_master_key_file`](#encode_master_key_file) | function | The key envelope for `dek`: `BCK2` (Argon2id + XChaCha20-Poly1305) when `password` is set and non-empty after trimming, `BCK1` otherwise. It touches no store; `store_master_key` places an envelope where the store preference says. |
| [`store_master_key`](#store_master_key) | function | The bytes `master.key` must hold for `dek`, after putting the envelope where the `key_store_preference` says: the envelope itself for the file store, or a `BCK3` pointer to the keychain item this call just wrote. A caller that stages the file itself (a shared branch applying its bootstrap as one recoverable step) writes these bytes with mode `0600`. |
| [`move_master_key`](#move_master_key) | function | Moves the existing key to `to`, keeping its bytes (and its password wrap) unchanged, so every sealed secret stays readable. |
| [`master_key_envelope`](#master_key_envelope) | function | The key's envelope exactly as it is stored (`BCK1`, or `BCK2` under the auth password), for an administrator who seals it into a unit credential themselves (`systemd-creds encrypt --name=bookclerk-master-key` on systemd older than 256, or a system-wide credential store). A `systemd-creds` move then adopts that credential instead of sealing its own copy, because the bytes match. |
| [`export_master_key`](#export_master_key) | function | A recovery copy of the key: a `BCK2` envelope wrapped with `recovery_password`, readable by `import_master_key` on any machine. |
| [`import_master_key`](#import_master_key) | function | Restores a key from a recovery copy (`export_master_key`, or a copied `master.key` in `BCK1`/`BCK2`) on an install that has no `master.key`, placing it where the `key_store_preference` says. |
| [`seal_with_dek`](#seal_with_dek) | function | Seal plaintext with the DEK (random nonce). Fast — no Argon2. |
| [`unseal_with_dek`](#unseal_with_dek) | function | Unseal ciphertext with the DEK. |
| [`SystemdCredsFactory`](#systemdcredsfactory) | type alias | Builds the `systemd-creds` store for a files directory. |
| [`AUTH_PASSWORD_ENV`](#auth_password_env) | constant | Env var that wraps the DEK in `master.key` (optional). |
| [`MASTER_KEY_FILE_NAME`](#master_key_file_name) | constant | Bootstrap key file under `BOOKCLERK_FILES_DIR` (auto-minted when missing). |

## Structs

### KeyStorePreference

```rust
pub struct KeyStorePreference {
    pub kind: KeyStoreKind,
    pub required: bool,
}
```

Where a new key goes, and whether that place is required.

**Fields**

- `kind`: The store new keys are written to.
- `required`: True when the operator set `[auth].key_store`: a keychain that can't
  be reached is then an error. False for a host's default, which falls
  back to the file with a warning.

#### KeyStorePreference::from_config

```rust
pub fn from_config(configured: Option<KeyStoreKind>, default: KeyStoreKind) -> Self
```

The preference for `configured` (`[auth].key_store`), or the host's
`default` when unset.

##### Arguments

* `configured` - the operator's setting, if any.
* `default` - the host's default store.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

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

### StoredMasterKey

```rust
pub struct StoredMasterKey {
    /* private fields */
}
```

A key [`store_master_key`](#store_master_key) placed, waiting for its `master.key` to be
published.

#### StoredMasterKey::bytes

```rust
pub fn bytes(&self) -> &[u8]
```

The bytes `master.key` must hold (write them with mode `0600`).

#### StoredMasterKey::store

```rust
pub fn store(&self) -> KeyStoreKind
```

The store the key went to: the file, or the store `master.key`
points at.

#### StoredMasterKey::discard

```rust
pub fn discard(self)
```

Deletes the store item again, for a caller whose `master.key` was
never published. Does nothing for a key kept in the file.

## Enums

### MasterKeyFormat

```rust
pub enum MasterKeyFormat {
    Raw,
    Wrapped,
    Keychain,
    SystemdCreds,
}
```

On-disk `master.key` envelope kind.

**Variants**

- `Raw`: Raw 32-byte DEK (`BCK1`).
- `Wrapped`: Password-wrapped DEK (`BCK2`).
- `Keychain`: A pointer to an OS keychain item (`BCK3`); the item holds `BCK1` or
  `BCK2`.
- `SystemdCreds`: A pointer to a `systemd-creds` credential (`BCK4`); the credential
  holds `BCK1` or `BCK2`.

#### MasterKeyFormat::header

```rust
pub fn header(self) -> &'static str
```

The four-character header: `BCK1`, `BCK2`, `BCK3` or `BCK4`.

#### MasterKeyFormat::store

```rust
pub fn store(self) -> KeyStoreKind
```

The store this format means.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### set_key_store_preference

```rust
pub fn set_key_store_preference(preference: KeyStorePreference)
```

Sets where this process writes new keys (first mint, a shared branch's
installed key, an import). Existing keys stay where `master.key` says.

#### Arguments

* `preference` - the store and whether it is required.

### key_store_preference

```rust
pub fn key_store_preference() -> KeyStorePreference
```

The current store preference.

### set_keychain_vault

```rust
pub fn set_keychain_vault(vault: Arc<dyn Vault>)
```

Replaces the keychain this process uses, for tests (a
[`bookclerk_keystore::MemoryVault`](../bookclerk_keystore/README.md#memoryvault)) and for hosts that substitute
their own store.

#### Arguments

* `vault` - the keychain to read and write items in.

### set_systemd_creds_vault

```rust
pub fn set_systemd_creds_vault(factory: Arc<SystemdCredsFactory>)
```

Replaces how this process builds its `systemd-creds` store, for tests
(a fake tool, or a [`bookclerk_keystore::MemoryVault`](../bookclerk_keystore/README.md#memoryvault)).

#### Arguments

* `factory` - builds the store for a files directory.

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

### master_key_is_wrapped

```rust
pub fn master_key_is_wrapped(files_dir: &Path) -> Result<Option<bool>>
```

Whether the key is password-wrapped (`BCK2`), reading the keychain item
when `master.key` is a pointer. `None` when there is no `master.key`.

#### Errors

Returns an error when `master.key` or the item it names can't be read.

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

This identifies the library secret root. It is not a password hash: the
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

Wrap an existing `BCK1` key with `password` (`BCK2`), in `master.key`
or in the OS keychain item it points at.

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

Installs `dek` as `{files_dir}/master.key`: a shared branch joining a library
receives the library's DEK (sealed to its node key, never the
passphrase-wrapped file) and keeps it under its own `password`.

Writes [`store_master_key`](#store_master_key)'s bytes (the envelope, or a pointer to the
keychain item it just wrote), mode `0600`, replacing any
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

The key envelope for `dek`: `BCK2` (Argon2id + XChaCha20-Poly1305) when
`password` is set and non-empty after trimming, `BCK1` otherwise. It
touches no store; [`store_master_key`](#store_master_key) places an envelope where the store
preference says.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when `dek` is not 32 bytes, and the
wrapping error.

### store_master_key

```rust
pub fn store_master_key(
    files_dir: &Path,
    dek: &[u8],
    password: Option<&str>,
) -> Result<StoredMasterKey>
```

The bytes `master.key` must hold for `dek`, after putting the envelope
where the [`key_store_preference`](#key_store_preference) says: the envelope itself for the
file store, or a `BCK3` pointer to the keychain item this call just
wrote. A caller that stages the file itself (a shared branch applying
its bootstrap as one recoverable step) writes these bytes with mode
`0600`.

#### Arguments

* `files_dir` - the files directory `master.key` will be written in
  (where a `systemd-creds` credential lives beside it).
* `dek` - the 32-byte data-encryption key.
* `password` - wraps the envelope as `BCK2` when set and non-empty.

#### Errors

Returns [`LibraryError::Invalid`](README.md#libraryerror) when `dek` is not 32 bytes, the
wrapping error, and a keychain failure when the keychain is required.

### move_master_key

```rust
pub fn move_master_key(
    files_dir: &Path,
    to: KeyStoreKind,
    password: Option<&str>,
) -> Result<MasterKeyFormat>
```

Moves the existing key to `to`, keeping its bytes (and its password
wrap) unchanged, so every sealed secret stays readable.

To the keychain or `systemd-creds`: writes a new item, reads it back,
replaces `master.key` with a pointer atomically, then deletes any item
the key was in before. To the file: writes the envelope over the pointer
atomically, reloads it, then deletes the old item. Moving to the store
the key is already in does nothing. Unlike starting a library, an
explicit move never falls back: a store that can't take the key is an
error.

#### Arguments

* `files_dir` - the files directory holding `master.key`.
* `to` - the store to move to.
* `password` - the auth password, needed when the key is wrapped.

#### Returns

The format `master.key` has afterwards.

#### Errors

Returns an error when there is no key, it can't be unwrapped, or the
target store can't take it. On error the key stays where it was.

### master_key_envelope

```rust
pub fn master_key_envelope(
    files_dir: &Path,
    password: Option<&str>,
) -> Result<Zeroizing<Vec<u8>>>
```

The key's envelope exactly as it is stored (`BCK1`, or `BCK2` under the
auth password), for an administrator who seals it into a unit
credential themselves (`systemd-creds encrypt --name=bookclerk-master-key`
on systemd older than 256, or a system-wide credential store). A
`systemd-creds` move then adopts that credential instead of sealing its
own copy, because the bytes match.

#### Arguments

* `files_dir` - the files directory holding `master.key`.
* `password` - the auth password, needed when the key is wrapped (the
  envelope stays wrapped; the password only proves it opens).

#### Errors

Returns an error when there is no key or it can't be unwrapped.

### export_master_key

```rust
pub fn export_master_key(
    files_dir: &Path,
    password: Option<&str>,
    recovery_password: &str,
) -> Result<Zeroizing<Vec<u8>>>
```

A recovery copy of the key: a `BCK2` envelope wrapped with
`recovery_password`, readable by [`import_master_key`](#import_master_key) on any machine.

#### Arguments

* `files_dir` - the files directory holding `master.key`.
* `password` - the auth password, needed when the key is wrapped.
* `recovery_password` - the password the recovery copy is wrapped with.

#### Errors

Returns an error when `recovery_password` is empty, or there is no key or
it can't be unwrapped.

### import_master_key

```rust
pub fn import_master_key(
    files_dir: &Path,
    recovery: &[u8],
    recovery_password: Option<&str>,
    password: Option<&str>,
) -> Result<String>
```

Restores a key from a recovery copy ([`export_master_key`](#export_master_key), or a copied
`master.key` in `BCK1`/`BCK2`) on an install that has no `master.key`,
placing it where the [`key_store_preference`](#key_store_preference) says.

#### Arguments

* `files_dir` - the files directory to write `master.key` in.
* `recovery` - the recovery copy's bytes.
* `recovery_password` - the password the copy is wrapped with, if it is.
* `password` - this install's auth password, which wraps the key at rest
  when set.

#### Returns

The SHA-256 hex fingerprint of the restored key.

#### Errors

Returns an error when `master.key` already exists, the copy can't be
unwrapped, or the key can't be stored.

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

## Type aliases

### SystemdCredsFactory

```rust
pub type SystemdCredsFactory = dyn Fn(&Path) -> Arc<dyn Vault> + Send + Sync;
```

Builds the `systemd-creds` store for a files directory.

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
