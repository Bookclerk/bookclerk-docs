# bookclerk_library::database_credential

A hosted database's API credential, sealed with the DEK and kept where
the master key is.

A hosted database (Cloudflare D1) needs an API token before the library
can open, so it can't live in `encrypted_secrets` like a store login.
`bookclerk database setup d1` mints one and keeps it here instead of in
an environment variable:

1. The token is sealed with the data-encryption key (XChaCha20-Poly1305),
   so its bytes are never stored in the clear anywhere.
2. The sealed blob goes in the same store as the master key: in
   `database.credential` itself on a file install, or in an OS keychain
   item or a `systemd-creds` credential on a keychain or TPM install,
   with `database.credential` then holding only a pointer to it.

| Header | `database.credential` holds |
| --- | --- |
| `BCD1` | DEK fingerprint, nonce, ciphertext |
| `BCD3` | DEK fingerprint, OS keychain item id |
| `BCD4` | DEK fingerprint, `systemd-creds` item id |

The keychain and `systemd-creds` items are of
`ItemKind::DatabaseCredential`, under their own service and credential
names, so they are never confused with the master key's. Reading fails
closed: a missing item or a credential sealed under another key is an
error, never an empty token.

The file and the items are written by this install only, under the same
protection as `master.key` itself, so they are not an untrusted-input
boundary (see `docs/adr/untrusted-input-fuzzing.md`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`DatabaseCredential`](#databasecredential) | struct | A hosted database's API token, and the provider's id for it. |
| [`set_database_credential_vaults`](#set_database_credential_vaults) | function | Replaces the stores credential items go to, for tests (a `bookclerk_keystore::MemoryVault`) and hosts that substitute their own. |
| [`database_credential_path`](#database_credential_path) | function | Path to `{files_dir}/database.credential`. |
| [`store_database_credential`](#store_database_credential) | function | Seals `credential` with `dek` and keeps it in the master key's store, replacing (and deleting the item of) any credential stored before. |
| [`load_database_credential`](#load_database_credential) | function | Reads and unseals the stored credential. |
| [`remove_database_credential`](#remove_database_credential) | function | Removes the stored credential and its item. |
| [`rehome_database_credential`](#rehome_database_credential) | function | Moves a stored credential to the master key's current store, after the master key itself moved. Does nothing when none is stored or it is already there. |
| [`resolve_d1_api_token`](#resolve_d1_api_token) | function | The Cloudflare D1 API token: `BOOKCLERK_D1_API_TOKEN` or `CLOUDFLARE_API_TOKEN` when set, else the credential `bookclerk database setup d1` stored. A stored token is registered for log redaction. |
| [`d1_api_token_configured`](#d1_api_token_configured) | function | True when a D1 token is available: from the environment or stored. |
| [`DATABASE_CREDENTIAL_FILE_NAME`](#database_credential_file_name) | constant | The credential file beside `master.key`. |

## Structs

### DatabaseCredential

```rust
pub struct DatabaseCredential {
    /* private fields */
}
```

A hosted database's API token, and the provider's id for it.

The id lets a later setup run revoke the token it replaces.

#### DatabaseCredential::new

```rust
pub fn new(token: &str, token_id: Option<&str>) -> Result<Self>
```

A credential from a token and the provider's id for it.

##### Arguments

* `token` - the bearer token; surrounding whitespace is trimmed.
* `token_id` - the provider's id for the token, if known.

##### Errors

[`LibraryError::Invalid`](README.md#libraryerror) when the token is empty, longer than
1024 bytes or holds whitespace or control characters, or the id is
longer than 128 bytes or holds anything but ASCII letters, digits,
`-` and `_`.

#### DatabaseCredential::token

```rust
pub fn token(&self) -> &str
```

The bearer token.

#### DatabaseCredential::token_id

```rust
pub fn token_id(&self) -> Option<&str>
```

The provider's id for the token, when known.

Implements: `Clone`, `Debug`

## Functions

### set_database_credential_vaults

```rust
pub fn set_database_credential_vaults(
    keychain: Arc<dyn Vault>,
    creds: Arc<SystemdCredsFactory>,
)
```

Replaces the stores credential items go to, for tests (a
[`bookclerk_keystore::MemoryVault`](../bookclerk_keystore/README.md#memoryvault)) and hosts that substitute their own.

#### Arguments

* `keychain` - the keychain credential items are kept in.
* `creds` - builds the `systemd-creds` store for a files directory.

### database_credential_path

```rust
pub fn database_credential_path(files_dir: &Path) -> PathBuf
```

Path to `{files_dir}/database.credential`.

### store_database_credential

```rust
pub fn store_database_credential(
    files_dir: &Path,
    dek: &MasterKey,
    credential: &DatabaseCredential,
) -> Result<KeyStoreKind>
```

Seals `credential` with `dek` and keeps it in the master key's store,
replacing (and deleting the item of) any credential stored before.

The new item is written and read back before `database.credential`
names it; a failure leaves the previous credential as it was.

#### Arguments

* `files_dir` - the files directory holding `master.key`.
* `dek` - the library's data-encryption key (from `master.key`).
* `credential` - the token to keep.

#### Returns

The store the sealed credential went to.

#### Errors

Returns an error when there is no master key, the store can't take the
item, or the file can't be written.

### load_database_credential

```rust
pub fn load_database_credential(
    files_dir: &Path,
    dek: &MasterKey,
) -> Result<Option<DatabaseCredential>>
```

Reads and unseals the stored credential.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.
* `dek` - the library's data-encryption key.

#### Returns

The credential, or `None` when none is stored.

#### Errors

Fails closed: a damaged file, a missing or unreadable item, or a
credential sealed under another key is an error.

### remove_database_credential

```rust
pub fn remove_database_credential(files_dir: &Path) -> Result<bool>
```

Removes the stored credential and its item.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.

#### Returns

True when a credential was removed.

#### Errors

Returns an error when the file exists but can't be read or removed.

### rehome_database_credential

```rust
pub fn rehome_database_credential(files_dir: &Path, dek: &MasterKey) -> Result<()>
```

Moves a stored credential to the master key's current store, after the
master key itself moved. Does nothing when none is stored or it is
already there.

#### Arguments

* `files_dir` - the files directory holding both files.
* `dek` - the library's data-encryption key.

#### Errors

Returns an error when the credential can't be read or stored; it then
stays where it was.

### resolve_d1_api_token

```rust
pub fn resolve_d1_api_token(files_dir: &Path) -> Result<Zeroizing<String>>
```

The Cloudflare D1 API token: `BOOKCLERK_D1_API_TOKEN` or
`CLOUDFLARE_API_TOKEN` when set, else the credential
`bookclerk database setup d1` stored. A stored token is registered for
log redaction.

The master key is loaded only when a stored credential exists, so a
host without one never mints a key here.

#### Arguments

* `files_dir` - the files directory holding `master.key` and
  `database.credential`.

#### Errors

Returns an error when no token is configured, or the stored one can't
be read.

### d1_api_token_configured

```rust
pub fn d1_api_token_configured(files_dir: &Path) -> bool
```

True when a D1 token is available: from the environment or stored.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.

## Constants

### DATABASE_CREDENTIAL_FILE_NAME

```rust
pub const DATABASE_CREDENTIAL_FILE_NAME: &str = "database.credential";
```

The credential file beside `master.key`.
