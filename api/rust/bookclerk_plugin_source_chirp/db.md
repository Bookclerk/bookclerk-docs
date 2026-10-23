# bookclerk_plugin_source_chirp::db

DB-backed credential storage for Chirp accounts.

Credentials live in `encrypted_secrets` under `provider =` the plugin id
(`chirp`), accessed only through [`SourceScope`](../bookclerk_library/scope.md#sourcescope).

| Item | Kind | Summary |
| --- | --- | --- |
| [`save_auth_to_db`](#save_auth_to_db) | function | Persist a `ChirpAuthFile` via the plugin scope. |
| [`load_auth_from_db`](#load_auth_from_db) | function | Load a `ChirpAuthFile` for this plugin only. |
| [`list_auth_from_db`](#list_auth_from_db) | function | List Chirp accounts for this plugin only. |
| [`delete_auth_from_db`](#delete_auth_from_db) | function | Remove a Chirp account secret. |

## Functions

### save_auth_to_db

```rust
pub async fn save_auth_to_db(
    auth: &ChirpAuthFile,
    scope: &SourceScope,
    account_id: &str,
) -> Result<()>
```

Persist a [`ChirpAuthFile`](index.md#chirpauthfile) via the plugin scope.

#### Errors

Returns an error when the operation fails.

### load_auth_from_db

```rust
pub async fn load_auth_from_db(
    scope: &SourceScope,
    account_id: &str,
) -> Result<Option<ChirpAuthFile>>
```

Load a [`ChirpAuthFile`](index.md#chirpauthfile) for this plugin only.

#### Errors

Returns an error when the operation fails.

### list_auth_from_db

```rust
pub async fn list_auth_from_db(
    scope: &SourceScope,
) -> Result<Vec<(String, ChirpAuthFile)>>
```

List Chirp accounts for this plugin only.

#### Errors

Returns an error when the operation fails.

### delete_auth_from_db

```rust
pub async fn delete_auth_from_db(scope: &SourceScope, account_id: &str) -> Result<()>
```

Remove a Chirp account secret.

#### Errors

Returns an error when the operation fails.
