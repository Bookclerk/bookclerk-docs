# bookclerk_library::database_credential

The database plugin's own credential, sealed with the DEK and kept where
the master key is.

A database plugin needs its credential before the library can open, so
it can't live in `encrypted_secrets` like a store login or a storage
plugin's keys. Whatever the plugin, it is kept here instead of in an
environment variable or a plaintext file: the API token
`bookclerk config database setup d1` mints for Cloudflare D1, and the
database credential a joining shared branch receives from its library
(D1's token, or the connection URL of a PostgreSQL library). Each is
tagged with the plugin it belongs to, so it is never handed to another:

1. The credential is sealed with the data-encryption key (XChaCha20-Poly1305),
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
| [`DatabaseCredential`](#databasecredential) | struct | A database plugin's credential (an API token, a connection URL), the plugin it belongs to, and the provider's id for it. |
| [`StagedDatabaseCredential`](#stageddatabasecredential) | struct | A credential `stage_database_credential` placed, waiting for its `database.credential` to be published. |
| [`set_database_credential_vaults`](#set_database_credential_vaults) | function | Replaces the stores credential items go to, for tests (a `bookclerk_keystore::MemoryVault`) and hosts that substitute their own. |
| [`database_credential_path`](#database_credential_path) | function | Path to `{files_dir}/database.credential`. |
| [`stage_database_credential`](#stage_database_credential) | function | Seals `credential` with a DEK given as raw bytes and puts it in `target`, for a caller that publishes `database.credential` itself: a shared branch applying its library's bootstrap as one recoverable step, beside the `master.key` it stages the same way. |
| [`forget_database_credential_item`](#forget_database_credential_item) | function | Deletes the store item `database.credential` names, leaving the file: for undoing a shared branch join whose own file restore removes it. Does nothing when there is no file or the credential is in it. |
| [`store_database_credential`](#store_database_credential) | function | Seals `credential` with `dek` and keeps it in the master key's store, replacing (and deleting the item of) any credential stored before. |
| [`load_database_credential`](#load_database_credential) | function | Reads and unseals the stored credential. |
| [`load_database_credential_for`](#load_database_credential_for) | function | `load_database_credential` for `plugin`: a stored credential that belongs to another database plugin is an error, never handed over. |
| [`remove_database_credential`](#remove_database_credential) | function | Removes the stored credential and its item. |
| [`rehome_database_credential`](#rehome_database_credential) | function | Moves a stored credential to the master key's current store, after the master key itself moved. Does nothing when none is stored or it is already there. |
| [`resolve_d1_api_token`](#resolve_d1_api_token) | function | The Cloudflare D1 API token: `BOOKCLERK_D1_API_TOKEN` or `CLOUDFLARE_API_TOKEN` when set, else the credential `bookclerk config database setup d1` stored. A stored token is registered for log redaction. |
| [`current_d1_credential`](#current_d1_credential) | function | The D1 credential this install uses, for a shared branch joining it: the environment's token (with no id) when one is set, else the stored credential, else `None`. |
| [`d1_api_token_configured`](#d1_api_token_configured) | function | True when a D1 token is available: from the environment or stored. |
| [`resolve_postgres_url`](#resolve_postgres_url) | function | The PostgreSQL connection URL: `[database.postgres].url_file` or `url` (or their environment overrides) when set, else, with `stored_credential = true`, the URL kept in `database.credential` (where a joined shared branch keeps its library's). A stored URL is registered for log redaction. |
| [`DATABASE_CREDENTIAL_FILE_NAME`](#database_credential_file_name) | constant | The credential file beside `master.key`. |
| [`MAX_SECRET_LEN`](#max_secret_len) | constant | The longest secret accepted, in bytes: a Cloudflare token is about 40, a PostgreSQL URL rarely past a few hundred. |
| [`MAX_SECRET_ID_LEN`](#max_secret_id_len) | constant | The longest secret id accepted, in bytes. |

## Structs

### DatabaseCredential

```rust
pub struct DatabaseCredential {
    /* private fields */
}
```

A database plugin's credential (an API token, a connection URL), the
plugin it belongs to, and the provider's id for it.

The id lets a later setup run revoke the token it replaces.

#### DatabaseCredential::new

```rust
pub fn new(
    plugin: DatabasePluginKind,
    secret: &str,
    secret_id: Option<&str>,
) -> Result<Self>
```

A credential for `plugin` from a secret and the provider's id for it.

##### Arguments

* `plugin` - the database plugin the secret is for.
* `secret` - the API token or connection URL; surrounding whitespace
  is trimmed.
* `secret_id` - the provider's id for the secret, if known.

##### Errors

[`LibraryError::Invalid`](README.md#libraryerror) when the secret is empty, longer than
[`MAX_SECRET_LEN`](#max_secret_len) bytes or holds whitespace or control characters,
or the id is longer than [`MAX_SECRET_ID_LEN`](#max_secret_id_len) bytes or holds
anything but ASCII letters, digits, `-` and `_`.

#### DatabaseCredential::plugin

```rust
pub fn plugin(&self) -> DatabasePluginKind
```

The database plugin the secret is for.

#### DatabaseCredential::secret

```rust
pub fn secret(&self) -> &str
```

The secret (an API token, a connection URL).

#### DatabaseCredential::secret_id

```rust
pub fn secret_id(&self) -> Option<&str>
```

The provider's id for the secret, when known.

Implements: `Clone`, `Debug`

### StagedDatabaseCredential

```rust
pub struct StagedDatabaseCredential {
    /* private fields */
}
```

A credential [`stage_database_credential`](#stage_database_credential) placed, waiting for its
`database.credential` to be published.

#### StagedDatabaseCredential::bytes

```rust
pub fn bytes(&self) -> &[u8]
```

The bytes `database.credential` must hold (write them with mode
`0600`).

#### StagedDatabaseCredential::store

```rust
pub fn store(&self) -> KeyStoreKind
```

The store the credential went to.

#### StagedDatabaseCredential::discard

```rust
pub fn discard(self)
```

Deletes the store item again, for a caller whose
`database.credential` was never published. Does nothing for a
credential kept in the file.

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

### stage_database_credential

```rust
pub fn stage_database_credential(
    files_dir: &Path,
    dek: &[u8],
    credential: &DatabaseCredential,
    target: KeyStoreKind,
) -> Result<StagedDatabaseCredential>
```

Seals `credential` with a DEK given as raw bytes and puts it in
`target`, for a caller that publishes `database.credential` itself: a
shared branch applying its library's bootstrap as one recoverable step,
beside the `master.key` it stages the same way.

#### Arguments

* `files_dir` - the files directory `database.credential` will be
  written in (where a `systemd-creds` credential lives beside it).
* `dek` - the library's 32-byte data-encryption key.
* `credential` - the credential to keep.
* `target` - the store the master key went to.

#### Errors

[`LibraryError::Invalid`](README.md#libraryerror) when `dek` is not 32 bytes, and an error when
the store can't take the item or reads it back wrong.

### forget_database_credential_item

```rust
pub fn forget_database_credential_item(files_dir: &Path)
```

Deletes the store item `database.credential` names, leaving the file:
for undoing a shared branch join whose own file restore removes it.
Does nothing when there is no file or the credential is in it.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.

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
* `credential` - the credential to keep.

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

### load_database_credential_for

```rust
pub fn load_database_credential_for(
    files_dir: &Path,
    dek: &MasterKey,
    plugin: DatabasePluginKind,
) -> Result<Option<DatabaseCredential>>
```

[`load_database_credential`](#load_database_credential) for `plugin`: a stored credential that
belongs to another database plugin is an error, never handed over.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.
* `dek` - the library's data-encryption key.
* `plugin` - the database plugin that needs the credential.

#### Errors

Whatever [`load_database_credential`](#load_database_credential) returns, and an error when the
stored credential is another plugin's.

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
`bookclerk config database setup d1` stored. A stored token is registered for
log redaction.

The master key is loaded only when a stored credential exists, so a
host without one never mints a key here.

#### Arguments

* `files_dir` - the files directory holding `master.key` and
  `database.credential`.

#### Errors

Returns an error when no token is configured, or the stored one can't
be read.

### current_d1_credential

```rust
pub fn current_d1_credential(
    files_dir: &Path,
    dek: &MasterKey,
) -> Result<Option<DatabaseCredential>>
```

The D1 credential this install uses, for a shared branch joining it:
the environment's token (with no id) when one is set, else the stored
credential, else `None`.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.
* `dek` - the library's data-encryption key.

#### Errors

[`LibraryError`](README.md#libraryerror) when the environment's token is malformed or the stored
credential can't be read.

### d1_api_token_configured

```rust
pub fn d1_api_token_configured(files_dir: &Path) -> bool
```

True when a D1 token is available: from the environment or stored.

#### Arguments

* `files_dir` - the files directory holding `database.credential`.

### resolve_postgres_url

```rust
pub fn resolve_postgres_url(config: &Config) -> Result<Zeroizing<String>>
```

The PostgreSQL connection URL: `[database.postgres].url_file` or
`url` (or their environment overrides) when set, else, with
`stored_credential = true`, the URL kept in `database.credential` (where a
joined shared branch keeps its library's). A stored URL is registered for
log redaction.

#### Arguments

* `config` - the loaded configuration.

#### Errors

Returns an error when no URL is configured, or the stored one can't be
read.

## Constants

### DATABASE_CREDENTIAL_FILE_NAME

```rust
pub const DATABASE_CREDENTIAL_FILE_NAME: &str = "database.credential";
```

The credential file beside `master.key`.

### MAX_SECRET_LEN

```rust
pub const MAX_SECRET_LEN: usize = 4096;
```

The longest secret accepted, in bytes: a Cloudflare token is about 40,
a PostgreSQL URL rarely past a few hundred.

### MAX_SECRET_ID_LEN

```rust
pub const MAX_SECRET_ID_LEN: usize = 128;
```

The longest secret id accepted, in bytes.
