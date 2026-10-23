**bookclerk_plugin_source_chirp > db**

# Module: db

## Contents

**Functions**

- [`delete_auth_from_db`](#delete_auth_from_db) - Remove a Chirp account secret.
- [`list_auth_from_db`](#list_auth_from_db) - List Chirp accounts for this plugin only.
- [`load_auth_from_db`](#load_auth_from_db) - Load a [`ChirpAuthFile`](auth.md#bookclerk_plugin_source_chirpauthchirpauthfile) for this plugin only.
- [`save_auth_to_db`](#save_auth_to_db) - Persist a [`ChirpAuthFile`](auth.md#bookclerk_plugin_source_chirpauthchirpauthfile) via the plugin scope.

---

## bookclerk_plugin_source_chirp::db::delete_auth_from_db

*Function*

Remove a Chirp account secret.

# Errors

Returns an error when the operation fails.

```rust
fn delete_auth_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



## bookclerk_plugin_source_chirp::db::list_auth_from_db

*Function*

List Chirp accounts for this plugin only.

# Errors

Returns an error when the operation fails.

```rust
fn list_auth_from_db(scope: &bookclerk_library::SourceScope) -> crate::error::Result<Vec<(String, crate::auth::ChirpAuthFile)>>
```



## bookclerk_plugin_source_chirp::db::load_auth_from_db

*Function*

Load a [`ChirpAuthFile`](auth.md#bookclerk_plugin_source_chirpauthchirpauthfile) for this plugin only.

# Errors

Returns an error when the operation fails.

```rust
fn load_auth_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<Option<crate::auth::ChirpAuthFile>>
```



## bookclerk_plugin_source_chirp::db::save_auth_to_db

*Function*

Persist a [`ChirpAuthFile`](auth.md#bookclerk_plugin_source_chirpauthchirpauthfile) via the plugin scope.

# Errors

Returns an error when the operation fails.

```rust
fn save_auth_to_db(auth: &crate::auth::ChirpAuthFile, scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



