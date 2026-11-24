# bookclerk_keystore

OS keychain access for the library's data-encryption key.

Bookclerk can keep the library's data-encryption key (DEK) in the
operating system's credential store instead of in `master.key`:

| Platform | Store |
| --- | --- |
| macOS | Keychain Services (the login keychain) |
| Windows | Credential Manager (DPAPI, per user) |
| Linux and other Unix | Secret Service over the session D-Bus (GNOME Keyring, KWallet), in a `plain` session so the hosts link no block cipher |

This crate is deliberately small. It names one keychain item per install
([`KeychainItemId`](#keychainitemid), a random UUID minted with the key), reads, writes and
deletes that item's bytes through a [`Vault`](#vault), and encodes the pointer
record ([`KeychainPointer`](#keychainpointer)) that `master.key` holds on a keychain
install. What the item's bytes mean (`BCK1` or `BCK2`, the fingerprint
check) stays in `bookclerk-library`'s `master_key` module.

The same stores also keep a hosted database's API token, already sealed
with the DEK, as a second [`ItemKind`](#itemkind) with its own service and
credential names.

[`OsKeychain`](#oskeychain) is the real store. [`MemoryVault`](#memoryvault) keeps items in memory,
for tests and for hosts that must never touch the keychain.

Errors never carry item bytes: the keychain libraries' own error values can
hold the secret they failed to decode, so [`KeystoreError`](#keystoreerror) keeps only a
category and a safe message.

See `docs/adr/master-key-store.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Vault`](#vault) | trait | A store of master-key items: the OS keychain, or memory in tests. |
| [`SystemdCredsVault`](#systemdcredsvault) | struct | A key store that seals items with `systemd-creds`. |
| [`KeystoreError`](#keystoreerror) | struct | A key store failure, without any item bytes. |
| [`KeychainItemId`](#keychainitemid) | struct | Identifier of one install's master-key item: the item's account name under `KEYCHAIN_SERVICE`. |
| [`KeychainPointer`](#keychainpointer) | struct | What `master.key` holds on a keychain or `systemd-creds` install: which store and item carry the key, and the fingerprint the key inside must match. No key material. |
| [`OsKeychain`](#oskeychain) | struct | The operating system's credential store. |
| [`MemoryVault`](#memoryvault) | struct | Items kept in process memory: for tests, and for hosts that must not touch the OS keychain. |
| [`ItemKind`](#itemkind) | enum | What an item holds, which decides the names it is stored under. |
| [`KeystoreErrorKind`](#keystoreerrorkind) | enum | Why a keychain operation failed. |
| [`ErrorStore`](#errorstore) | enum | Which store a `KeystoreError` came from, so messages name the store the operator configured. |
| [`PointerStore`](#pointerstore) | enum | Which store a pointer record names. |
| [`PointerError`](#pointererror) | enum | Why a byte string is not a pointer record. |
| [`Result`](#result) | type alias | Result alias for keychain operations. |
| [`CREDENTIAL_NAME`](#credential_name) | constant | The credential name the key is sealed under, and the name a service unit hands it in as (`LoadCredentialEncrypted=bookclerk-master-key`). |
| [`DATABASE_CREDENTIAL_NAME`](#database_credential_name) | constant | The credential name a database credential is sealed under, and the name a service unit hands one in as (`LoadCredentialEncrypted=bookclerk-database-credential`). |
| [`KEYCHAIN_SERVICE`](#keychain_service) | constant | Keychain service name every Bookclerk master-key item is stored under. |
| [`DATABASE_CREDENTIAL_SERVICE`](#database_credential_service) | constant | Keychain service name a database credential item is stored under (the DEK-sealed API token of a hosted database such as Cloudflare D1). |
| [`POINTER_MAGIC`](#pointer_magic) | constant | Four-byte header of a `master.key` pointer record for an OS keychain item (`BCK3`). |
| [`CREDS_POINTER_MAGIC`](#creds_pointer_magic) | constant | Four-byte header of a `master.key` pointer record for a key sealed with `systemd-creds` (`BCK4`): the same layout as `POINTER_MAGIC`'s. |
| [`FINGERPRINT_LEN`](#fingerprint_len) | constant | Length in bytes of a key fingerprint (SHA-256 of the DEK). |
| [`POINTER_LEN`](#pointer_len) | constant | Length in bytes of an encoded `KeychainPointer`: header, 16-byte item id, 32-byte fingerprint. |
| [`MAX_ITEM_LEN`](#max_item_len) | constant | Largest item, in bytes, a `Vault` accepts. |

## Traits

### Vault

```rust
pub trait Vault: Send + Sync {
    fn get(&self, item: KeychainItemId) -> Result<Zeroizing<Vec<u8>>>;
    fn set(&self, item: KeychainItemId, bytes: &[u8]) -> Result<()>;
    fn delete(&self, item: KeychainItemId) -> Result<()>;
}
```

A store of master-key items: the OS keychain, or memory in tests.

#### Vault::get

```rust
fn get(&self, item: KeychainItemId) -> Result<Zeroizing<Vec<u8>>>
```

Reads an item's bytes.

##### Errors

[`KeystoreErrorKind::NotFound`](#keystoreerrorkind) when the item is missing, and the
store's other failures.

#### Vault::set

```rust
fn set(&self, item: KeychainItemId, bytes: &[u8]) -> Result<()>
```

Creates or replaces an item.

##### Errors

[`KeystoreErrorKind::Rejected`](#keystoreerrorkind) when `bytes` is longer than
[`MAX_ITEM_LEN`](#max_item_len), and the store's failures.

#### Vault::delete

```rust
fn delete(&self, item: KeychainItemId) -> Result<()>
```

Deletes an item. Deleting a missing item succeeds.

##### Errors

The store's failures other than a missing item.

Implemented by: `MemoryVault`, `OsKeychain`, `SystemdCredsVault`

## Structs

### SystemdCredsVault

```rust
pub struct SystemdCredsVault {
    /* private fields */
}
```

A key store that seals items with `systemd-creds`.

Items live in one directory (the files directory) as
`master-key-<item>.cred` (or `database-credential-<item>.cred` for
[`ItemKind::DatabaseCredential`](#itemkind)). See the module docs.

#### SystemdCredsVault::new

```rust
pub fn new(dir: impl Into<PathBuf>) -> Self
```

A store sealing items into `dir` with the system's `systemd-creds`,
reading a unit-provided copy from `$CREDENTIALS_DIRECTORY` when set.

##### Arguments

* `dir` - the directory sealed items go in (the files directory).

#### SystemdCredsVault::with_program

```rust
pub fn with_program(
    dir: impl Into<PathBuf>,
    program: Option<PathBuf>,
    credentials_dir: Option<PathBuf>,
) -> Self
```

A store with an explicit tool and credentials directory, for tests
and for systems that install `systemd-creds` elsewhere.

##### Arguments

* `dir` - the directory sealed items go in.
* `program` - the `systemd-creds` binary, or `None` for none.
* `credentials_dir` - the unit's credentials directory, if any.

#### SystemdCredsVault::for_kind

```rust
pub fn for_kind(self, kind: ItemKind) -> Self
```

The same store, sealing items of `kind` (its own file prefix and
credential name) instead of master-key items.

##### Arguments

* `kind` - the kind of item to seal.

#### SystemdCredsVault::sealed_path

```rust
pub fn sealed_path(&self, item: KeychainItemId) -> PathBuf
```

Where `item` is sealed: `<dir>/<prefix>-<item>.cred`, the prefix
being `master-key` or `database-credential` ([`ItemKind::file_prefix`](#itemkindfile_prefix)).

##### Arguments

* `item` - the item id the pointer file names.

#### SystemdCredsVault::unit_credential

```rust
pub fn unit_credential(&self) -> Option<PathBuf>
```

The copy the service unit handed in, when there is one.

Implements: `Clone`, `Debug`, `Vault`

### KeystoreError

```rust
pub struct KeystoreError {
    pub kind: KeystoreErrorKind,
    pub store: ErrorStore,
    pub message: String,
}
```

A key store failure, without any item bytes.

**Fields**

- `kind`: What went wrong, for callers that branch on it.
- `store`: The store that failed.
- `message`: A human-readable explanation that never contains item bytes.

#### KeystoreError::new

```rust
pub fn new(kind: KeystoreErrorKind, message: impl Into<String>) -> Self
```

Builds an OS keychain error of `kind` with `message`; see
[`KeystoreError::in_store`](#keystoreerrorin_store) for another store.

##### Arguments

* `kind` - the failure category.
* `message` - a safe, secret-free explanation.

#### KeystoreError::in_store

```rust
pub fn in_store(self, store: ErrorStore) -> Self
```

The same error, naming `store` as the one that failed.

##### Arguments

* `store` - the store the error came from.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### KeychainItemId

```rust
pub struct KeychainItemId(/* private */);
```

Identifier of one install's master-key item: the item's account name
under [`KEYCHAIN_SERVICE`](#keychain_service).

#### KeychainItemId::random

```rust
pub fn random() -> Self
```

Mints a fresh random item id (UUID v4).

#### KeychainItemId::as_bytes

```rust
pub fn as_bytes(&self) -> &[u8; 16]
```

The id's 16 raw bytes, as the pointer record stores them.

#### KeychainItemId::from_bytes

```rust
pub fn from_bytes(bytes: [u8; 16]) -> Self
```

Rebuilds an id from the 16 bytes a pointer record holds.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `PartialEq`

### KeychainPointer

```rust
pub struct KeychainPointer {
    pub store: PointerStore,
    pub item: KeychainItemId,
    pub fingerprint: [u8; 32],
}
```

What `master.key` holds on a keychain or `systemd-creds` install: which
store and item carry the key, and the fingerprint the key inside must
match. No key material.

**Fields**

- `store`: The store that holds the item.
- `item`: The item that holds the key envelope.
- `fingerprint`: SHA-256 of the unwrapped DEK, the same value `library_identity`
  stores. A key read from the item must hash to this.

#### KeychainPointer::encode

```rust
pub fn encode(&self) -> [u8; 52]
```

Encodes the record: the store's header (`BCK3` or `BCK4`), the item
id, the fingerprint.

#### KeychainPointer::parse

```rust
pub fn parse(raw: &[u8]) -> Result<Self, PointerError>
```

Parses a record [`Self::encode`](#keychainpointerencode) wrote.

##### Arguments

* `raw` - the whole contents of `master.key`.

##### Errors

[`PointerError::NotAPointer`](#pointererror) when the header is neither `BCK3` nor
`BCK4`, and [`PointerError::BadLength`](#pointererror) when the length is not
exactly [`POINTER_LEN`](#pointer_len).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### OsKeychain

```rust
pub struct OsKeychain {
    /* private fields */
}
```

The operating system's credential store.

On macOS and Windows the native store (through `keyring-core`) is opened
on first use and reused. On Linux and other Unix systems each call talks
to the Secret Service on the session D-Bus; a headless server usually has
none, and every call then fails with [`KeystoreErrorKind::Unavailable`](#keystoreerrorkind).

#### OsKeychain::new

```rust
pub fn new() -> Self
```

A handle to this machine's credential store for master-key items.

#### OsKeychain::for_kind

```rust
pub fn for_kind(kind: ItemKind) -> Self
```

A handle to this machine's credential store for items of `kind`.

##### Arguments

* `kind` - decides the keychain service items are stored under.

Implements: `Default`, `Vault`

### MemoryVault

```rust
pub struct MemoryVault {
    /* private fields */
}
```

Items kept in process memory: for tests, and for hosts that must not touch
the OS keychain.

#### MemoryVault::new

```rust
pub fn new() -> Self
```

An empty vault.

#### MemoryVault::fail_with

```rust
pub fn fail_with(&self, kind: Option<KeystoreErrorKind>)
```

Makes every later call fail with `kind` (`None` restores normal use).

##### Arguments

* `kind` - the failure to report, or `None`.

#### MemoryVault::len

```rust
pub fn len(&self) -> usize
```

How many items the vault holds.

#### MemoryVault::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when the vault holds no items.

Implements: `Default`, `Vault`

## Enums

### ItemKind

```rust
pub enum ItemKind {
    MasterKey,
    DatabaseCredential,
}
```

What an item holds, which decides the names it is stored under.

Each kind has its own keychain service, `systemd-creds` credential name
and sealed-file prefix, so an item of one kind can never be read as the
other (a unit-provided master key is never returned for a database
credential, and a sealed file can't be renamed into the other's place).

**Variants**

- `MasterKey`: The library's data-encryption key envelope (`BCK1`/`BCK2`).
- `DatabaseCredential`: A hosted database's API credential, already sealed with the DEK.

#### ItemKind::service

```rust
pub fn service(self) -> &'static str
```

The keychain service items of this kind are stored under.

#### ItemKind::credential_name

```rust
pub fn credential_name(self) -> &'static str
```

The `systemd-creds` credential name items of this kind are sealed
under, and the name a service unit hands one in as.

#### ItemKind::file_prefix

```rust
pub fn file_prefix(self) -> &'static str
```

The file-name prefix of a sealed `systemd-creds` item:
`<prefix>-<item>.cred`.

#### ItemKind::label

```rust
pub fn label(self) -> &'static str
```

The human-readable label a keychain shows for an item of this kind.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `Hash`, `PartialEq`

### KeystoreErrorKind

```rust
pub enum KeystoreErrorKind {
    Unavailable,
    AccessDenied,
    NotFound,
    Corrupt,
    Rejected,
}
```

Why a keychain operation failed.

**Variants**

- `Unavailable`: No credential store is reachable on this machine or session: no
  Secret Service on the session bus, no login keychain, no D-Bus.
- `AccessDenied`: The store is there but refused access (locked collection, the user
  denied the prompt, or the app's signature changed).
- `NotFound`: The named item does not exist.
- `Corrupt`: The item exists but its contents are not what Bookclerk wrote.
- `Rejected`: The store rejected the request (too long, bad attributes, ambiguous).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ErrorStore

```rust
pub enum ErrorStore {
    OsKeychain,
    SystemdCreds,
}
```

Which store a [`KeystoreError`](#keystoreerror) came from, so messages name the store
the operator configured.

**Variants**

- `OsKeychain`: The OS keychain (also the label for the in-memory test store).
- `SystemdCreds`: A `systemd-creds` credential ([`SystemdCredsVault`](#systemdcredsvault)).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### PointerStore

```rust
pub enum PointerStore {
    Keychain,
    SystemdCreds,
}
```

Which store a pointer record names.

**Variants**

- `Keychain`: The OS keychain ([`OsKeychain`](#oskeychain)); the record starts `BCK3`.
- `SystemdCreds`: A credential sealed with `systemd-creds` ([`SystemdCredsVault`](#systemdcredsvault)); the
  record starts `BCK4`.

#### PointerStore::magic

```rust
pub fn magic(self) -> &'static [u8; 4]
```

The record header for this store.

#### PointerStore::for_header

```rust
pub fn for_header(raw: &[u8]) -> Option<Self>
```

The store whose records start with `raw`'s first four bytes, if any.

##### Arguments

* `raw` - the start of `master.key`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PointerError

```rust
pub enum PointerError {
    NotAPointer,
    BadLength(usize),
}
```

Why a byte string is not a pointer record.

**Variants**

- `NotAPointer`: The bytes start with neither [`POINTER_MAGIC`](#pointer_magic) nor
  [`CREDS_POINTER_MAGIC`](#creds_pointer_magic).
- `BadLength`: The header is right but the record is not exactly [`POINTER_LEN`](#pointer_len) bytes.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, KeystoreError>;
```

Result alias for keychain operations.

## Constants

### CREDENTIAL_NAME

```rust
pub const CREDENTIAL_NAME: &str = "bookclerk-master-key";
```

The credential name the key is sealed under, and the name a service
unit hands it in as (`LoadCredentialEncrypted=bookclerk-master-key`).

### DATABASE_CREDENTIAL_NAME

```rust
pub const DATABASE_CREDENTIAL_NAME: &str = "bookclerk-database-credential";
```

The credential name a database credential is sealed under, and the name
a service unit hands one in as
(`LoadCredentialEncrypted=bookclerk-database-credential`).

### KEYCHAIN_SERVICE

```rust
pub const KEYCHAIN_SERVICE: &str = "app.bookclerk.master-key";
```

Keychain service name every Bookclerk master-key item is stored under.

The item's account (user) name is its [`KeychainItemId`](#keychainitemid), so two installs
for one OS user (a development checkout beside an installed app) keep
separate items.

### DATABASE_CREDENTIAL_SERVICE

```rust
pub const DATABASE_CREDENTIAL_SERVICE: &str = "app.bookclerk.database-credential";
```

Keychain service name a database credential item is stored under (the
DEK-sealed API token of a hosted database such as Cloudflare D1).

### POINTER_MAGIC

```rust
pub const POINTER_MAGIC: &[u8; 4] = b"BCK3";
```

Four-byte header of a `master.key` pointer record for an OS keychain
item (`BCK3`).

### CREDS_POINTER_MAGIC

```rust
pub const CREDS_POINTER_MAGIC: &[u8; 4] = b"BCK4";
```

Four-byte header of a `master.key` pointer record for a key sealed with
`systemd-creds` (`BCK4`): the same layout as [`POINTER_MAGIC`](#pointer_magic)'s.

### FINGERPRINT_LEN

```rust
pub const FINGERPRINT_LEN: usize = 32;
```

Length in bytes of a key fingerprint (SHA-256 of the DEK).

### POINTER_LEN

```rust
pub const POINTER_LEN: usize = _;
```

Length in bytes of an encoded [`KeychainPointer`]: header, 16-byte item
id, 32-byte fingerprint.

### MAX_ITEM_LEN

```rust
pub const MAX_ITEM_LEN: usize = 2048;
```

Largest item, in bytes, a [`Vault`](#vault) accepts.

A `BCK2` envelope is 108 bytes; Windows Credential Manager caps a blob at
2,560 bytes. Anything larger is refused before it reaches the store.
