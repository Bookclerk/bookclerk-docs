**bookclerk_plugin_source_libro > db**

# Module: db

## Contents

**Functions**

- [`delete_auth_from_db`](#delete_auth_from_db) - Remove a Libro.fm account secret.
- [`list_auth_from_db`](#list_auth_from_db) - List Libro.fm accounts for this plugin only.
- [`load_auth_from_db`](#load_auth_from_db) - Load a [`LibroAuthFile`](auth.md#bookclerk_plugin_source_libroauthlibroauthfile) for this plugin only.
- [`save_auth_to_db`](#save_auth_to_db) - Persist a [`LibroAuthFile`](auth.md#bookclerk_plugin_source_libroauthlibroauthfile) via the plugin scope.

---

## bookclerk_plugin_source_libro::db::delete_auth_from_db

*Function*

Remove a Libro.fm account secret.

# Errors

Returns an error when the operation fails.

```rust
fn delete_auth_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



## bookclerk_plugin_source_libro::db::list_auth_from_db

*Function*

List Libro.fm accounts for this plugin only.

# Errors

Returns an error when the operation fails.

```rust
fn list_auth_from_db(scope: &bookclerk_library::SourceScope) -> crate::error::Result<Vec<(String, crate::auth::LibroAuthFile)>>
```



## bookclerk_plugin_source_libro::db::load_auth_from_db

*Function*

Load a [`LibroAuthFile`](auth.md#bookclerk_plugin_source_libroauthlibroauthfile) for this plugin only.

# Errors

Returns an error when the operation fails.

```rust
fn load_auth_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<Option<crate::auth::LibroAuthFile>>
```



## bookclerk_plugin_source_libro::db::save_auth_to_db

*Function*

Persist a [`LibroAuthFile`](auth.md#bookclerk_plugin_source_libroauthlibroauthfile) via the plugin scope.

# Errors

Returns an error when the operation fails.

```rust
fn save_auth_to_db(auth: &crate::auth::LibroAuthFile, scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



