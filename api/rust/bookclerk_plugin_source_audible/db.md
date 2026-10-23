# bookclerk_plugin_source_audible::db

DB-backed credential storage for Audible accounts.

Credentials live in `encrypted_secrets` under `provider =` the plugin id
(`audible`), accessed only through [`SourceScope`](../bookclerk_library/scope.md#sourcescope) — the same boundary as
third-party source plugins.

Each credential is stored as `format = "sealed-v1"` wrapping a
`Protection::Plain` audible-rs envelope. The outer XChaCha20-Poly1305 seal
(process DEK from `master.key`) provides at-rest protection; audible-rs
inner encryption is intentionally bypassed so key derivation happens once
at startup rather than on every credential access.

| Item | Kind | Summary |
| --- | --- | --- |
| [`save_authenticator_to_db`](#save_authenticator_to_db) | function | Persist an `Authenticator` via the Audible `SourceScope`. |
| [`load_authenticator_from_db`](#load_authenticator_from_db) | function | Load an `Authenticator` from the scoped `encrypted_secrets` table. |
| [`list_audible_accounts_from_db`](#list_audible_accounts_from_db) | function | List all Audible accounts stored in the DB for this scope. |
| [`delete_audible_account_from_db`](#delete_audible_account_from_db) | function | Remove an Audible account secret from the DB. |
| [`save_widevine_cdm_to_db`](#save_widevine_cdm_to_db) | function | Persist a raw Widevine `.wvd` device blob into `encrypted_secrets`. |
| [`load_widevine_cdm_from_db`](#load_widevine_cdm_from_db) | function | Load a Widevine `.wvd` device blob from `encrypted_secrets`. |

## Functions

### save_authenticator_to_db

```rust
pub async fn save_authenticator_to_db(
    auth: &Authenticator,
    scope: &SourceScope,
    account_name: &str,
) -> Result<()>
```

Persist an [`Authenticator`](https://docs.rs/audible_rs/latest/audible_rs/auth/struct.Authenticator.html) via the Audible [`SourceScope`](../bookclerk_library/scope.md#sourcescope).

The audible-rs envelope is serialized with `Protection::Plain` (no inner
Argon2), then sealed with the process DEK (`sealed-v1`).

#### Errors

Returns an error when the operation fails.

#### Panics

Panics when an internal invariant does not hold.

### load_authenticator_from_db

```rust
pub async fn load_authenticator_from_db(
    scope: &SourceScope,
    account_name: &str,
) -> Result<Option<Authenticator>>
```

Load an [`Authenticator`](https://docs.rs/audible_rs/latest/audible_rs/auth/struct.Authenticator.html) from the scoped `encrypted_secrets` table.

Registers a write-back callback so that token refreshes and cookie
exchanges persist back to the DB automatically.

Returns `None` when no secret for the given `account_name` exists.

#### Errors

Returns an error when the operation fails.

#### Panics

Panics when an internal invariant does not hold.

### list_audible_accounts_from_db

```rust
pub async fn list_audible_accounts_from_db(
    scope: &SourceScope,
) -> Result<Vec<(String, String)>>
```

List all Audible accounts stored in the DB for this scope.

Returns `(account_id, name)` tuples extracted from `encrypted_secrets` rows.

#### Errors

Returns an error when the operation fails.

### delete_audible_account_from_db

```rust
pub async fn delete_audible_account_from_db(
    scope: &SourceScope,
    account_name: &str,
) -> Result<()>
```

Remove an Audible account secret from the DB.

#### Errors

Returns an error when the operation fails.

### save_widevine_cdm_to_db

```rust
pub async fn save_widevine_cdm_to_db(
    scope: &SourceScope,
    account_id: &str,
    wvd_bytes: &[u8],
) -> Result<()>
```

Persist a raw Widevine `.wvd` device blob into `encrypted_secrets`.

`account_id` identifies which account the CDM was provisioned for.
The blob is sealed with the process DEK (`sealed-v1`).

#### Errors

Returns an error when the operation fails.

### load_widevine_cdm_from_db

```rust
pub async fn load_widevine_cdm_from_db(
    scope: &SourceScope,
    account_id: &str,
) -> Result<Option<Vec<u8>>>
```

Load a Widevine `.wvd` device blob from `encrypted_secrets`.

Returns `None` when no CDM for `account_id` is stored yet.

#### Errors

Returns an error when the operation fails.
