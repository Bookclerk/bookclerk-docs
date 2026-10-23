# bookclerk_library::secrets

Encrypted secrets store — DB-backed credential storage.

## Overview

Secrets are kept in the `encrypted_secrets` table (M10 migration). Each row
stores opaque `ciphertext` with column-level metadata describing which KDF
and cipher were used, so every secret carries its own self-contained
decryption recipe.

### Algorithms

- **New writes**: `sealed-v1` — XChaCha20-Poly1305 with the process DEK
  (from `master.key`). No per-row Argon2. No plaintext writes ever.
- **Legacy reads**: `json-encrypted` (Argon2id + BOOKCLERK_AUTH_PASSWORD)
  still decrypted. `json` (plaintext) is migrated on read if master key
  is available, otherwise rejected.

### Unseal cache

[`unseal_secret`](#unseal_secret) keeps a process-wide plaintext cache (identity +
ciphertext fingerprint). Content-source plugins should just call
load/unseal normally — repeated acquire/scan loads stay cheap without
per-plugin caches. Upsert/delete invalidate the matching entries.

### Bootstrap secrets (NOT stored here)

`BOOKCLERK_AUTH_PASSWORD`, `BOOKCLERK_DATABASE_POSTGRES_URL`, and
`BOOKCLERK_D1_API_TOKEN` / `CLOUDFLARE_API_TOKEN` are **env-only bootstrap**
credentials. They are needed to open the DB or bootstrap the master key and
cannot be stored here. `config.toml` also stays on disk.

`BOOKCLERK_OPERATOR_TOKEN` is an optional **env override** for the durable
operator token row (`kind = operator_token`); see [`crate::operator_token`](operator_token.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`EncryptedSecretRecord`](#encryptedsecretrecord) | struct | A row from the `encrypted_secrets` table. |
| [`EncryptedBlob`](#encryptedblob) | struct | Raw output from `encrypt_secret` (legacy Argon2id path). |
| [`SecretStore`](#secretstore) | struct | Thin wrapper around a `DatabaseConnection` for `encrypted_secrets` CRUD. |
| [`clear_unseal_cache`](#clear_unseal_cache) | function | Drop every cached plaintext unseal. |
| [`encrypt_secret`](#encrypt_secret) | function | Encrypt `plaintext` with Argon2id key derivation + XChaCha20-Poly1305 (legacy). |
| [`decrypt_secret`](#decrypt_secret) | function | Decrypt ciphertext using Argon2id + XChaCha20-Poly1305 (legacy `json-encrypted`). |
| [`build_sealed_record`](#build_sealed_record) | function | Build a `EncryptedSecretRecord` skeleton sealed with the process DEK. |
| [`unseal_secret`](#unseal_secret) | function | Unseal a `sealed-v1` record using the process DEK. |
| [`bytes_to_b64_string`](#bytes_to_b64_string) | function | Seal arbitrary bytes as base64 string — used by D1 transport layer. |
| [`b64_string_to_bytes`](#b64_string_to_bytes) | function | Decode a `b64:`-prefixed string to bytes. Returns `None` if not prefixed. |
| [`upsert_secret`](#upsert_secret) | function | Insert or replace a secret in the DB using a single ON CONFLICT statement. |
| [`upsert_secret_on`](#upsert_secret_on) | function | Insert or replace a secret using an open connection or transaction. |
| [`get_secret`](#get_secret) | function | Fetch a single secret by its composite key. Returns `None` if not found. |
| [`list_secrets`](#list_secrets) | function | List all secrets of a given `kind`. |
| [`delete_secrets_for_account`](#delete_secrets_for_account) | function | Delete integration-account secrets for `account_id` under `provider` (the owning plugin key). |
| [`delete_secret`](#delete_secret) | function | Delete a secret by its composite key. No-op if it does not exist. |
| [`delete_secret_on`](#delete_secret_on) | function | Delete a secret by composite key on an open connection or transaction. |
| [`FORMAT_SEALED_V1`](#format_sealed_v1) | constant | Format tag for new writes: DEK-sealed XChaCha20-Poly1305 (no per-row Argon2). |
| [`KDF_M_COST`](#kdf_m_cost) | constant | Argon2id memory cost in KiB (64 MiB — OWASP minimum). Legacy only. |
| [`KDF_T_COST`](#kdf_t_cost) | constant | Argon2id time cost (iterations). Legacy only. |
| [`KDF_P_COST`](#kdf_p_cost) | constant | Argon2id parallelism factor. Legacy only. |
| [`KDF_ALGORITHM`](#kdf_algorithm) | constant | KDF algorithm identifier. Legacy only. |
| [`CIPHER_ALGORITHM`](#cipher_algorithm) | constant | Cipher algorithm identifier stored alongside ciphertext rows. |

## Structs

### EncryptedSecretRecord

```rust
pub struct EncryptedSecretRecord {
    pub id: Option<i64>,
    pub kind: String,
    pub provider: Option<String>,
    pub account_type: String,
    pub account_id: Option<String>,
    pub name: String,
    pub format: String,
    pub ciphertext: Vec<u8>,
    pub kdf_algorithm: Option<String>,
    pub kdf_salt: Option<Vec<u8>>,
    pub kdf_m_cost: Option<u32>,
    pub kdf_t_cost: Option<u32>,
    pub kdf_p_cost: Option<u32>,
    pub cipher_algorithm: Option<String>,
    pub cipher_nonce: Option<Vec<u8>>,
    pub created_at: String,
    pub updated_at: String,
}
```

A row from the `encrypted_secrets` table.

**Fields**

- `id`: Surrogate primary key assigned by the database.
- `kind`: Secret kind discriminator (`source_auth`, `s3`, …).
- `provider`: Source / service name (`"audible"`, `"libro"`, `"s3"`, …) or `None`.
- `account_type`: Ownership namespace — see [`secret_account_type`](secret_account_type.md) constants.
- `account_id`: Per-provider account identifier (file stem, email, …) or `None`.
- `name`: Human-readable label / file-stem equivalent (e.g. `"alice.audible"`).
- `format`: Payload format:
  - `"sealed-v1"` — XChaCha20-Poly1305 with process DEK (no per-row Argon2)
  - `"audible-rs-auth"` — sealed-v1 wrapping a plain audible-rs envelope
  - `"json-encrypted"` — legacy: JSON encrypted with Argon2id + XChaCha20-Poly1305
- `ciphertext`: Encrypted payload bytes for this secret.
- `kdf_algorithm`: KDF algorithm id for legacy rows (for example `argon2id`).
- `kdf_salt`: Random salt for legacy Argon2 key derivation.
- `kdf_m_cost`: Argon2 memory cost in KiB for legacy rows.
- `kdf_t_cost`: Argon2 time cost (iterations) for legacy rows.
- `kdf_p_cost`: Argon2 parallel lane count stored for legacy `json-encrypted` rows.
- `cipher_algorithm`: Cipher algorithm identifier (e.g. `"xchacha20poly1305"`) or `None`.
- `cipher_nonce`: AEAD nonce bytes used with `ciphertext`.
- `created_at`: RFC 3339 timestamp when the row was inserted.
- `updated_at`: RFC 3339 timestamp when the row was last modified.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EncryptedBlob

```rust
pub struct EncryptedBlob {
    pub kdf_salt: Vec<u8>,
    pub cipher_nonce: Vec<u8>,
    pub ciphertext: Vec<u8>,
}
```

Raw output from [`encrypt_secret`](#encrypt_secret) (legacy Argon2id path).

**Fields**

- `kdf_salt`: Random salt for legacy Argon2 key derivation.
- `cipher_nonce`: AEAD nonce bytes used with `ciphertext`.
- `ciphertext`: Encrypted payload bytes for this secret.

### SecretStore

```rust
pub struct SecretStore<'a> {
    /* private fields */
}
```

Thin wrapper around a [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html) for `encrypted_secrets` CRUD.

Construct with [`SecretStore::new`](#secretstorenew) or call the standalone `async fn`
helpers directly. All methods are `async`.

#### SecretStore::new

```rust
pub fn new(db: &'a DatabaseConnection) -> Self
```

Constructs a store handle over an existing database connection.

##### Arguments

* `db` - SeaORM connection owned by the caller for the lifetime of this handle.

##### Returns

A [`SecretStore`](#secretstore) that delegates to the standalone CRUD helpers.

#### SecretStore::upsert

```rust
pub async fn upsert(&self, record: &EncryptedSecretRecord) -> Result<()>
```

Inserts or replaces a sealed secret row.

##### Arguments

* `record` - Fully populated sealed secret identity and ciphertext.

##### Errors

Returns [`LibraryError`](index.md#libraryerror) when the database write fails.

#### SecretStore::get

```rust
pub async fn get(
    &self,
    kind: &str,
    provider: Option<&str>,
    account_type: &str,
    account_id: Option<&str>,
    name: &str,
) -> Result<Option<EncryptedSecretRecord>>
```

Loads one sealed secret by identity columns, if present.

##### Arguments

* `kind` - Secret kind (`source_auth`, `s3`, …).
* `provider` - Optional provider / plugin id filter.
* `account_type` - `integration` or `operator`.
* `account_id` - Optional account id filter.
* `name` - Logical secret name within that identity.

##### Returns

`Ok(Some(record))` when found, `Ok(None)` otherwise.

##### Errors

Returns [`LibraryError`](index.md#libraryerror) when the database read fails.

#### SecretStore::list

```rust
pub async fn list(&self, kind: &str) -> Result<Vec<EncryptedSecretRecord>>
```

Lists sealed secrets with the given `kind`.

##### Arguments

* `kind` - Secret kind to filter on.

##### Returns

All matching sealed rows (ciphertext included).

##### Errors

Returns [`LibraryError`](index.md#libraryerror) when the database read fails.

#### SecretStore::delete

```rust
pub async fn delete(
    &self,
    kind: &str,
    provider: Option<&str>,
    account_type: &str,
    account_id: Option<&str>,
    name: &str,
) -> Result<()>
```

Deletes one sealed secret by identity columns.

##### Arguments

* `kind` - Secret kind (`source_auth`, `s3`, …).
* `provider` - Optional provider / plugin id filter.
* `account_type` - `integration` or `operator`.
* `account_id` - Optional account id filter.
* `name` - Logical secret name within that identity.

##### Errors

Returns [`LibraryError`](index.md#libraryerror) when the database delete fails.

## Functions

### clear_unseal_cache

```rust
pub fn clear_unseal_cache()
```

Drop every cached plaintext unseal.

Call when the process DEK identity changes. Password wrap of the same DEK
(`BCK1` → `BCK2`) does not require this — ciphertext and plaintext stay valid.

### encrypt_secret

```rust
pub fn encrypt_secret(plaintext: &[u8], password: &str) -> Result<EncryptedBlob>
```

Encrypt `plaintext` with Argon2id key derivation + XChaCha20-Poly1305 (legacy).

Use [`build_sealed_record`](#build_sealed_record) for new writes. This function is retained for
legacy test compat and the json-encrypted migration path.

#### Errors

Returns an error when the operation fails.

### decrypt_secret

```rust
pub fn decrypt_secret(
    ciphertext: &[u8],
    password: &str,
    kdf_salt: &[u8],
    cipher_nonce: &[u8],
) -> Result<Vec<u8>>
```

Decrypt ciphertext using Argon2id + XChaCha20-Poly1305 (legacy `json-encrypted`).

Use [`unseal_secret`](#unseal_secret) for new reads. This function handles legacy DB rows.

#### Errors

Returns an error when the operation fails.

### build_sealed_record

```rust
pub fn build_sealed_record(
    plaintext: &[u8],
    kind: &str,
    provider: &str,
    account_type: &str,
    account_id: &str,
    name: &str,
) -> Result<EncryptedSecretRecord>
```

Build a [`EncryptedSecretRecord`](#encryptedsecretrecord) skeleton sealed with the process DEK.

Callers must fill in `kind`, `provider`, `account_type`, `account_id`,
`name`, and timestamps before upserting.

#### Errors

Returns an error when the operation fails.

### unseal_secret

```rust
pub fn unseal_secret(record: &EncryptedSecretRecord) -> Result<Vec<u8>>
```

Unseal a `sealed-v1` record using the process DEK.

Also handles legacy `json-encrypted` (using `BOOKCLERK_AUTH_PASSWORD`) and
rejects `json` plaintext.

Successful unseals are cached process-wide (keyed by secret identity +
ciphertext fingerprint) so repeated loads during acquire/scan do not
re-decrypt. Callers — including content-source plugins — need not cache.

#### Errors

Returns an error when the operation fails.

### bytes_to_b64_string

```rust
pub fn bytes_to_b64_string(bytes: &[u8]) -> String
```

Seal arbitrary bytes as base64 string — used by D1 transport layer.

### b64_string_to_bytes

```rust
pub fn b64_string_to_bytes(s: &str) -> Option<Vec<u8>>
```

Decode a `b64:`-prefixed string to bytes. Returns `None` if not prefixed.

### upsert_secret

```rust
pub async fn upsert_secret(
    db: &DatabaseConnection,
    record: &EncryptedSecretRecord,
) -> Result<()>
```

Insert or replace a secret in the DB using a single ON CONFLICT statement.

provider and account_id should always be `Some` for new writes.

#### Errors

Returns an error when the operation fails.

### upsert_secret_on

```rust
pub async fn upsert_secret_on<C: ConnectionTrait>(
    db: &C,
    record: &EncryptedSecretRecord,
) -> Result<()>
```

Insert or replace a secret using an open connection or transaction.

#### Errors

Returns an error when the operation fails.

### get_secret

```rust
pub async fn get_secret(
    db: &DatabaseConnection,
    kind: &str,
    provider: Option<&str>,
    account_type: &str,
    account_id: Option<&str>,
    name: &str,
) -> Result<Option<EncryptedSecretRecord>>
```

Fetch a single secret by its composite key. Returns `None` if not found.

#### Errors

Returns an error when the operation fails.

### list_secrets

```rust
pub async fn list_secrets(
    db: &DatabaseConnection,
    kind: &str,
) -> Result<Vec<EncryptedSecretRecord>>
```

List all secrets of a given `kind`.

#### Errors

Returns an error when the operation fails.

### delete_secrets_for_account

```rust
pub async fn delete_secrets_for_account(
    db: &DatabaseConnection,
    provider: &str,
    account_id: &str,
) -> Result<()>
```

Delete integration-account secrets for `account_id` under `provider`
(the owning plugin key).

Operator-typed secrets (e.g. S3 destination credentials) are never
matched — they have `account_type = "operator"` and survive account
revocation.

#### Errors

Returns an error when the operation fails.

### delete_secret

```rust
pub async fn delete_secret(
    db: &DatabaseConnection,
    kind: &str,
    provider: Option<&str>,
    account_type: &str,
    account_id: Option<&str>,
    name: &str,
) -> Result<()>
```

Delete a secret by its composite key. No-op if it does not exist.

#### Errors

Returns an error when the operation fails.

### delete_secret_on

```rust
pub async fn delete_secret_on<C: ConnectionTrait>(
    db: &C,
    kind: &str,
    provider: Option<&str>,
    account_type: &str,
    account_id: Option<&str>,
    name: &str,
) -> Result<()>
```

Delete a secret by composite key on an open connection or transaction.

#### Errors

Returns an error when the operation fails.

## Constants

### FORMAT_SEALED_V1

```rust
pub const FORMAT_SEALED_V1: &str = "sealed-v1";
```

Format tag for new writes: DEK-sealed XChaCha20-Poly1305 (no per-row Argon2).

### KDF_M_COST

```rust
pub const KDF_M_COST: u32 = 65_536;
```

Argon2id memory cost in KiB (64 MiB — OWASP minimum). Legacy only.

### KDF_T_COST

```rust
pub const KDF_T_COST: u32 = 3;
```

Argon2id time cost (iterations). Legacy only.

### KDF_P_COST

```rust
pub const KDF_P_COST: u32 = 1;
```

Argon2id parallelism factor. Legacy only.

### KDF_ALGORITHM

```rust
pub const KDF_ALGORITHM: &str = "argon2id";
```

KDF algorithm identifier. Legacy only.

### CIPHER_ALGORITHM

```rust
pub const CIPHER_ALGORITHM: &str = "xchacha20poly1305";
```

Cipher algorithm identifier stored alongside ciphertext rows.
