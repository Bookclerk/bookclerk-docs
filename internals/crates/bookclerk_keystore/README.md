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

[`OsKeychain`](#oskeychain) is the real store. [`MemoryVault`](#memoryvault) keeps items in memory,
for tests and for hosts that must never touch the keychain.

Errors never carry item bytes: the keychain libraries' own error values can
hold the secret they failed to decode, so [`KeystoreError`](#keystoreerror) keeps only a
category and a safe message.

See `docs/adr/master-key-store.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Vault`](#vault) | trait | A store of master-key items: the OS keychain, or memory in tests. |
| [`KeystoreError`](#keystoreerror) | struct | A keychain failure, without any item bytes. |
| [`KeychainItemId`](#keychainitemid) | struct | Identifier of one install's master-key item: the item's account name under `KEYCHAIN_SERVICE`. |
| [`KeychainPointer`](#keychainpointer) | struct | What `master.key` holds on a keychain install: which item carries the key, and the fingerprint the key inside must match. No key material. |
| [`OsKeychain`](#oskeychain) | struct | The operating system's credential store. |
| [`MemoryVault`](#memoryvault) | struct | Items kept in process memory: for tests, and for hosts that must not touch the OS keychain. |
| [`KeystoreErrorKind`](#keystoreerrorkind) | enum | Why a keychain operation failed. |
| [`PointerError`](#pointererror) | enum | Why a byte string is not a pointer record. |
| [`Result`](#result) | type alias | Result alias for keychain operations. |
| [`KEYCHAIN_SERVICE`](#keychain_service) | constant | Keychain service name every Bookclerk master-key item is stored under. |
| [`POINTER_MAGIC`](#pointer_magic) | constant | Four-byte header of a `master.key` pointer record (`BCK3`). |
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

Implemented by: `MemoryVault`, `OsKeychain`

## Structs

### KeystoreError

```rust
pub struct KeystoreError {
    pub kind: KeystoreErrorKind,
    pub message: String,
}
```

A keychain failure, without any item bytes.

**Fields**

- `kind`: What went wrong, for callers that branch on it.
- `message`: A human-readable explanation that never contains item bytes.

#### KeystoreError::new

```rust
pub fn new(kind: KeystoreErrorKind, message: impl Into<String>) -> Self
```

Builds an error of `kind` with `message`.

##### Arguments

* `kind` - the failure category.
* `message` - a safe, secret-free explanation.

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
    pub item: KeychainItemId,
    pub fingerprint: [u8; 32],
}
```

What `master.key` holds on a keychain install: which item carries the
key, and the fingerprint the key inside must match. No key material.

**Fields**

- `item`: The keychain item that holds the key envelope.
- `fingerprint`: SHA-256 of the unwrapped DEK, the same value `library_identity`
  stores. A key read from the item must hash to this.

#### KeychainPointer::encode

```rust
pub fn encode(&self) -> [u8; 52]
```

Encodes the record: `BCK3`, the item id, the fingerprint.

#### KeychainPointer::parse

```rust
pub fn parse(raw: &[u8]) -> Result<Self, PointerError>
```

Parses a record [`Self::encode`](#keychainpointerencode) wrote.

##### Arguments

* `raw` - the whole contents of `master.key`.

##### Errors

[`PointerError::NotAPointer`](#pointererror) when the header is not `BCK3`, and
[`PointerError::BadLength`](#pointererror) when the length is not exactly
[`POINTER_LEN`](#pointer_len).

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

A handle to this machine's credential store.

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

### PointerError

```rust
pub enum PointerError {
    NotAPointer,
    BadLength(usize),
}
```

Why a byte string is not a pointer record.

**Variants**

- `NotAPointer`: The bytes don't start with [`POINTER_MAGIC`](#pointer_magic).
- `BadLength`: The header is right but the record is not exactly [`POINTER_LEN`](#pointer_len) bytes.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, KeystoreError>;
```

Result alias for keychain operations.

## Constants

### KEYCHAIN_SERVICE

```rust
pub const KEYCHAIN_SERVICE: &str = "app.bookclerk.master-key";
```

Keychain service name every Bookclerk master-key item is stored under.

The item's account (user) name is its [`KeychainItemId`](#keychainitemid), so two installs
for one OS user (a development checkout beside an installed app) keep
separate items.

### POINTER_MAGIC

```rust
pub const POINTER_MAGIC: &[u8; 4] = b"BCK3";
```

Four-byte header of a `master.key` pointer record (`BCK3`).

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
