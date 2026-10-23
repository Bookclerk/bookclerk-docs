**bookclerk_plugin_source_audible > db**

# Module: db

## Contents

**Functions**

- [`delete_audible_account_from_db`](#delete_audible_account_from_db) - Remove an Audible account secret from the DB.
- [`list_audible_accounts_from_db`](#list_audible_accounts_from_db) - List all Audible accounts stored in the DB for this scope.
- [`load_authenticator_from_db`](#load_authenticator_from_db) - Load an [`Authenticator`](https://docs.rs/audible_rs/latest/audible_rs/auth/struct.Authenticator.html) from the scoped `encrypted_secrets` table.
- [`load_widevine_cdm_from_db`](#load_widevine_cdm_from_db) - Load a Widevine `.wvd` device blob from `encrypted_secrets`.
- [`save_authenticator_to_db`](#save_authenticator_to_db) - Persist an [`Authenticator`](https://docs.rs/audible_rs/latest/audible_rs/auth/struct.Authenticator.html) via the Audible [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope).
- [`save_widevine_cdm_to_db`](#save_widevine_cdm_to_db) - Persist a raw Widevine `.wvd` device blob into `encrypted_secrets`.

---

## bookclerk_plugin_source_audible::db::delete_audible_account_from_db

*Function*

Remove an Audible account secret from the DB.

# Errors

Returns an error when the operation fails.

```rust
fn delete_audible_account_from_db(scope: &bookclerk_library::SourceScope, account_name: &str) -> crate::error::Result<()>
```



## bookclerk_plugin_source_audible::db::list_audible_accounts_from_db

*Function*

List all Audible accounts stored in the DB for this scope.

Returns `(account_id, name)` tuples extracted from `encrypted_secrets` rows.

# Errors

Returns an error when the operation fails.

```rust
fn list_audible_accounts_from_db(scope: &bookclerk_library::SourceScope) -> crate::error::Result<Vec<(String, String)>>
```



## bookclerk_plugin_source_audible::db::load_authenticator_from_db

*Function*

Load an [`Authenticator`](https://docs.rs/audible_rs/latest/audible_rs/auth/struct.Authenticator.html) from the scoped `encrypted_secrets` table.

Registers a write-back callback so that token refreshes and cookie
exchanges persist back to the DB automatically.

Returns `None` when no secret for the given `account_name` exists.

# Errors

Returns an error when the operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn load_authenticator_from_db(scope: &bookclerk_library::SourceScope, account_name: &str) -> crate::error::Result<Option<audible_rs::auth::Authenticator>>
```



## bookclerk_plugin_source_audible::db::load_widevine_cdm_from_db

*Function*

Load a Widevine `.wvd` device blob from `encrypted_secrets`.

Returns `None` when no CDM for `account_id` is stored yet.

# Errors

Returns an error when the operation fails.

```rust
fn load_widevine_cdm_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<Option<Vec<u8>>>
```



## bookclerk_plugin_source_audible::db::save_authenticator_to_db

*Function*

Persist an [`Authenticator`](https://docs.rs/audible_rs/latest/audible_rs/auth/struct.Authenticator.html) via the Audible [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope).

The audible-rs envelope is serialized with `Protection::Plain` (no inner
Argon2), then sealed with the process DEK (`sealed-v1`).

# Errors

Returns an error when the operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn save_authenticator_to_db(auth: &audible_rs::auth::Authenticator, scope: &bookclerk_library::SourceScope, account_name: &str) -> crate::error::Result<()>
```



## bookclerk_plugin_source_audible::db::save_widevine_cdm_to_db

*Function*

Persist a raw Widevine `.wvd` device blob into `encrypted_secrets`.

`account_id` identifies which account the CDM was provisioned for.
The blob is sealed with the process DEK (`sealed-v1`).

# Errors

Returns an error when the operation fails.

```rust
fn save_widevine_cdm_to_db(scope: &bookclerk_library::SourceScope, account_id: &str, wvd_bytes: &[u8]) -> crate::error::Result<()>
```



