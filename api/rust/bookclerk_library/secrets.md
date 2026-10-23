**bookclerk_library > secrets**

# Module: secrets

## Contents

**Modules**

- [`secret_account_type`](#secret_account_type) - Ownership namespace for `encrypted_secrets.account_type`.
- [`secret_kind`](#secret_kind) - Well-known `kind` values for the `encrypted_secrets` table.

**Structs**

- [`EncryptedBlob`](#encryptedblob) - Raw output from [`encrypt_secret`](secrets.md#bookclerk_librarysecretsencrypt_secret) (legacy Argon2id path).
- [`EncryptedSecretRecord`](#encryptedsecretrecord) - A row from the `encrypted_secrets` table.
- [`SecretStore`](#secretstore) - Thin wrapper around a [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html) for `encrypted_secrets` CRUD.

**Functions**

- [`b64_string_to_bytes`](#b64_string_to_bytes) - Decode a `b64:`-prefixed string to bytes. Returns `None` if not prefixed.
- [`build_sealed_record`](#build_sealed_record) - Build a [`EncryptedSecretRecord`](secrets.md#bookclerk_librarysecretsencryptedsecretrecord) skeleton sealed with the process DEK.
- [`bytes_to_b64_string`](#bytes_to_b64_string) - Seal arbitrary bytes as base64 string — used by D1 transport layer.
- [`clear_unseal_cache`](#clear_unseal_cache) - Drop every cached plaintext unseal.
- [`decrypt_secret`](#decrypt_secret) - Decrypt ciphertext using Argon2id + XChaCha20-Poly1305 (legacy `json-encrypted`).
- [`delete_secret`](#delete_secret) - Delete a secret by its composite key. No-op if it does not exist.
- [`delete_secret_on`](#delete_secret_on) - Delete a secret by composite key on an open connection or transaction.
- [`delete_secrets_for_account`](#delete_secrets_for_account) - Delete integration-account secrets for `account_id` under `provider`
- [`encrypt_secret`](#encrypt_secret) - Encrypt `plaintext` with Argon2id key derivation + XChaCha20-Poly1305 (legacy).
- [`get_secret`](#get_secret) - Fetch a single secret by its composite key. Returns `None` if not found.
- [`list_secrets`](#list_secrets) - List all secrets of a given `kind`.
- [`unseal_secret`](#unseal_secret) - Unseal a `sealed-v1` record using the process DEK.
- [`upsert_secret`](#upsert_secret) - Insert or replace a secret in the DB using a single ON CONFLICT statement.
- [`upsert_secret_on`](#upsert_secret_on) - Insert or replace a secret using an open connection or transaction.

**Constants**

- [`CIPHER_ALGORITHM`](#cipher_algorithm) - Cipher algorithm identifier stored alongside ciphertext rows.
- [`FORMAT_SEALED_V1`](#format_sealed_v1) - Format tag for new writes: DEK-sealed XChaCha20-Poly1305 (no per-row Argon2).
- [`KDF_ALGORITHM`](#kdf_algorithm) - KDF algorithm identifier. Legacy only.
- [`KDF_M_COST`](#kdf_m_cost) - Argon2id memory cost in KiB (64 MiB — OWASP minimum). Legacy only.
- [`KDF_P_COST`](#kdf_p_cost) - Argon2id parallelism factor. Legacy only.
- [`KDF_T_COST`](#kdf_t_cost) - Argon2id time cost (iterations). Legacy only.

---

## bookclerk_library::secrets::CIPHER_ALGORITHM

*Constant*: `&str`

Cipher algorithm identifier stored alongside ciphertext rows.



## bookclerk_library::secrets::EncryptedBlob

*Struct*

Raw output from [`encrypt_secret`](secrets.md#bookclerk_librarysecretsencrypt_secret) (legacy Argon2id path).

**Fields:**
- `kdf_salt: Vec<u8>` - Random salt for legacy Argon2 key derivation.
- `cipher_nonce: Vec<u8>` - AEAD nonce bytes used with `ciphertext`.
- `ciphertext: Vec<u8>` - Encrypted payload bytes for this secret.



## bookclerk_library::secrets::EncryptedSecretRecord

*Struct*

A row from the `encrypted_secrets` table.

**Fields:**
- `id: Option<i64>` - Surrogate primary key assigned by the database.
- `kind: String` - Secret kind discriminator (`source_auth`, `s3`, …).
- `provider: Option<String>` - Source / service name (`"audible"`, `"libro"`, `"s3"`, …) or `None`.
- `account_type: String` - Ownership namespace — see [`secret_account_type`](secrets/secret_account_type.md) constants.
- `account_id: Option<String>` - Per-provider account identifier (file stem, email, …) or `None`.
- `name: String` - Human-readable label / file-stem equivalent (e.g. `"alice.audible"`).
- `format: String` - Payload format:
- `ciphertext: Vec<u8>` - Encrypted payload bytes for this secret.
- `kdf_algorithm: Option<String>` - KDF algorithm id for legacy rows (for example `argon2id`).
- `kdf_salt: Option<Vec<u8>>` - Random salt for legacy Argon2 key derivation.
- `kdf_m_cost: Option<u32>` - Argon2 memory cost in KiB for legacy rows.
- `kdf_t_cost: Option<u32>` - Argon2 time cost (iterations) for legacy rows.
- `kdf_p_cost: Option<u32>` - Argon2 parallel lane count stored for legacy `json-encrypted` rows.
- `cipher_algorithm: Option<String>` - Cipher algorithm identifier (e.g. `"xchacha20poly1305"`) or `None`.
- `cipher_nonce: Option<Vec<u8>>` - AEAD nonce bytes used with `ciphertext`.
- `created_at: String` - RFC 3339 timestamp when the row was inserted.
- `updated_at: String` - RFC 3339 timestamp when the row was last modified.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EncryptedSecretRecord`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::secrets::FORMAT_SEALED_V1

*Constant*: `&str`

Format tag for new writes: DEK-sealed XChaCha20-Poly1305 (no per-row Argon2).



## bookclerk_library::secrets::KDF_ALGORITHM

*Constant*: `&str`

KDF algorithm identifier. Legacy only.



## bookclerk_library::secrets::KDF_M_COST

*Constant*: `u32`

Argon2id memory cost in KiB (64 MiB — OWASP minimum). Legacy only.



## bookclerk_library::secrets::KDF_P_COST

*Constant*: `u32`

Argon2id parallelism factor. Legacy only.



## bookclerk_library::secrets::KDF_T_COST

*Constant*: `u32`

Argon2id time cost (iterations). Legacy only.



## bookclerk_library::secrets::SecretStore

*Struct*

Thin wrapper around a [`DatabaseConnection`](https://docs.rs/sea_orm/latest/sea_orm/database/db_connection/struct.DatabaseConnection.html) for `encrypted_secrets` CRUD.

Construct with `SecretStore::new` or call the standalone `async fn`
helpers directly. All methods are `async`.

**Generic Parameters:**
- 'a

**Methods:**

- `fn new(db: &'a DatabaseConnection) -> Self` - Constructs a store handle over an existing database connection.
- `fn upsert(self: &Self, record: &EncryptedSecretRecord) -> Result<()>` - Inserts or replaces a sealed secret row.
- `fn get(self: &Self, kind: &str, provider: Option<&str>, account_type: &str, account_id: Option<&str>, name: &str) -> Result<Option<EncryptedSecretRecord>>` - Loads one sealed secret by identity columns, if present.
- `fn list(self: &Self, kind: &str) -> Result<Vec<EncryptedSecretRecord>>` - Lists sealed secrets with the given `kind`.
- `fn delete(self: &Self, kind: &str, provider: Option<&str>, account_type: &str, account_id: Option<&str>, name: &str) -> Result<()>` - Deletes one sealed secret by identity columns.



## bookclerk_library::secrets::b64_string_to_bytes

*Function*

Decode a `b64:`-prefixed string to bytes. Returns `None` if not prefixed.

```rust
fn b64_string_to_bytes(s: &str) -> Option<Vec<u8>>
```



## bookclerk_library::secrets::build_sealed_record

*Function*

Build a [`EncryptedSecretRecord`](secrets.md#bookclerk_librarysecretsencryptedsecretrecord) skeleton sealed with the process DEK.

Callers must fill in `kind`, `provider`, `account_type`, `account_id`,
`name`, and timestamps before upserting.

# Errors

Returns an error when the operation fails.

```rust
fn build_sealed_record(plaintext: &[u8], kind: &str, provider: &str, account_type: &str, account_id: &str, name: &str) -> crate::error::Result<EncryptedSecretRecord>
```



## bookclerk_library::secrets::bytes_to_b64_string

*Function*

Seal arbitrary bytes as base64 string — used by D1 transport layer.

```rust
fn bytes_to_b64_string(bytes: &[u8]) -> String
```



## bookclerk_library::secrets::clear_unseal_cache

*Function*

Drop every cached plaintext unseal.

Call when the process DEK identity changes. Password wrap of the same DEK
(`BCK1` → `BCK2`) does not require this — ciphertext and plaintext stay valid.

```rust
fn clear_unseal_cache()
```



## bookclerk_library::secrets::decrypt_secret

*Function*

Decrypt ciphertext using Argon2id + XChaCha20-Poly1305 (legacy `json-encrypted`).

Use [`unseal_secret`](secrets.md#bookclerk_librarysecretsunseal_secret) for new reads. This function handles legacy DB rows.

# Errors

Returns an error when the operation fails.

```rust
fn decrypt_secret(ciphertext: &[u8], password: &str, kdf_salt: &[u8], cipher_nonce: &[u8]) -> crate::error::Result<Vec<u8>>
```



## bookclerk_library::secrets::delete_secret

*Function*

Delete a secret by its composite key. No-op if it does not exist.

# Errors

Returns an error when the operation fails.

```rust
fn delete_secret(db: &sea_orm::DatabaseConnection, kind: &str, provider: Option<&str>, account_type: &str, account_id: Option<&str>, name: &str) -> crate::error::Result<()>
```



## bookclerk_library::secrets::delete_secret_on

*Function*

Delete a secret by composite key on an open connection or transaction.

# Errors

Returns an error when the operation fails.

```rust
fn delete_secret_on<C>(db: &C, kind: &str, provider: Option<&str>, account_type: &str, account_id: Option<&str>, name: &str) -> crate::error::Result<()>
```



## bookclerk_library::secrets::delete_secrets_for_account

*Function*

Delete integration-account secrets for `account_id` under `provider`
(the owning plugin key).

Operator-typed secrets (e.g. S3 destination credentials) are never
matched — they have `account_type = "operator"` and survive account
revocation.

# Errors

Returns an error when the operation fails.

```rust
fn delete_secrets_for_account(db: &sea_orm::DatabaseConnection, provider: &str, account_id: &str) -> crate::error::Result<()>
```



## bookclerk_library::secrets::encrypt_secret

*Function*

Encrypt `plaintext` with Argon2id key derivation + XChaCha20-Poly1305 (legacy).

Use [`build_sealed_record`](secrets.md#bookclerk_librarysecretsbuild_sealed_record) for new writes. This function is retained for
legacy test compat and the json-encrypted migration path.

# Errors

Returns an error when the operation fails.

```rust
fn encrypt_secret(plaintext: &[u8], password: &str) -> crate::error::Result<EncryptedBlob>
```



## bookclerk_library::secrets::get_secret

*Function*

Fetch a single secret by its composite key. Returns `None` if not found.

# Errors

Returns an error when the operation fails.

```rust
fn get_secret(db: &sea_orm::DatabaseConnection, kind: &str, provider: Option<&str>, account_type: &str, account_id: Option<&str>, name: &str) -> crate::error::Result<Option<EncryptedSecretRecord>>
```



## bookclerk_library::secrets::list_secrets

*Function*

List all secrets of a given `kind`.

# Errors

Returns an error when the operation fails.

```rust
fn list_secrets(db: &sea_orm::DatabaseConnection, kind: &str) -> crate::error::Result<Vec<EncryptedSecretRecord>>
```



## Module: secret_account_type

Ownership namespace for `encrypted_secrets.account_type`.

Rows tagged `integration` are store or portal credentials tied to a user
account; they are purged when that account is revoked. Rows tagged
`operator` are destination / control-plane secrets (e.g. S3 keys) that
outlive any individual store account and must never be touched by
[`delete_secrets_for_account`](secrets.md#bookclerk_librarysecretsdelete_secrets_for_account).



## Module: secret_kind

Well-known `kind` values for the `encrypted_secrets` table.

Runtime credentials live here. Bootstrap credentials
(`BOOKCLERK_AUTH_PASSWORD`, `BOOKCLERK_D1_API_TOKEN`,
`BOOKCLERK_DATABASE_POSTGRES_URL`) are env-only and never stored in this
table. `BOOKCLERK_OPERATOR_TOKEN` may override the durable
[`OPERATOR_TOKEN`](secrets/secret_kind.md#bookclerk_librarysecretssecret_kindoperator_token) row without writing it.



## bookclerk_library::secrets::unseal_secret

*Function*

Unseal a `sealed-v1` record using the process DEK.

Also handles legacy `json-encrypted` (using `BOOKCLERK_AUTH_PASSWORD`) and
rejects `json` plaintext.

Successful unseals are cached process-wide (keyed by secret identity +
ciphertext fingerprint) so repeated loads during acquire/scan do not
re-decrypt. Callers — including content-source plugins — need not cache.

# Errors

Returns an error when the operation fails.

```rust
fn unseal_secret(record: &EncryptedSecretRecord) -> crate::error::Result<Vec<u8>>
```



## bookclerk_library::secrets::upsert_secret

*Function*

Insert or replace a secret in the DB using a single ON CONFLICT statement.

provider and account_id should always be `Some` for new writes.

# Errors

Returns an error when the operation fails.

```rust
fn upsert_secret(db: &sea_orm::DatabaseConnection, record: &EncryptedSecretRecord) -> crate::error::Result<()>
```



## bookclerk_library::secrets::upsert_secret_on

*Function*

Insert or replace a secret using an open connection or transaction.

# Errors

Returns an error when the operation fails.

```rust
fn upsert_secret_on<C>(db: &C, record: &EncryptedSecretRecord) -> crate::error::Result<()>
```



