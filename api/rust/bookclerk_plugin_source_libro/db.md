# bookclerk_plugin_source_libro::db

DB-backed credential storage for Libro.fm accounts.

Credentials live in `encrypted_secrets` under `provider =` the plugin id
(`libro`), accessed only through [`SourceScope`](../bookclerk_library/scope.md#sourcescope).

| Item | Kind | Summary |
| --- | --- | --- |
| [`save_auth_to_db`](#save_auth_to_db) | function | Persist a `LibroAuthFile` via the plugin scope. |
| [`load_auth_from_db`](#load_auth_from_db) | function | Load a `LibroAuthFile` for this plugin only. |
| [`list_auth_from_db`](#list_auth_from_db) | function | List Libro.fm accounts for this plugin only. |
| [`delete_auth_from_db`](#delete_auth_from_db) | function | Remove a Libro.fm account secret. |

## Functions

### save_auth_to_db

```rust
pub async fn save_auth_to_db(
    auth: &LibroAuthFile,
    scope: &SourceScope,
    account_id: &str,
) -> Result<()>
```

Persist a [`LibroAuthFile`](index.md#libroauthfile) via the plugin scope.

#### Errors

Returns an error when the operation fails.

### load_auth_from_db

```rust
pub async fn load_auth_from_db(
    scope: &SourceScope,
    account_id: &str,
) -> Result<Option<LibroAuthFile>>
```

Load a [`LibroAuthFile`](index.md#libroauthfile) for this plugin only.

#### Errors

Returns an error when the operation fails.

### list_auth_from_db

```rust
pub async fn list_auth_from_db(
    scope: &SourceScope,
) -> Result<Vec<(String, LibroAuthFile)>>
```

List Libro.fm accounts for this plugin only.

#### Errors

Returns an error when the operation fails.

### delete_auth_from_db

```rust
pub async fn delete_auth_from_db(scope: &SourceScope, account_id: &str) -> Result<()>
```

Remove a Libro.fm account secret.

#### Errors

Returns an error when the operation fails.
