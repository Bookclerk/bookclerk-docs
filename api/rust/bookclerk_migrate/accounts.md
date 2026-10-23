**bookclerk_migrate > accounts**

# Module: accounts

## Contents

**Structs**

- [`AccountsImportSummary`](#accountsimportsummary) - Result of importing classic `AccountsSettings.json` metadata (credentials are never sealed here).

**Functions**

- [`import_accounts`](#import_accounts) - Import account metadata from `AccountsSettings.json`.

---

## bookclerk_migrate::accounts::AccountsImportSummary

*Struct*

Result of importing classic `AccountsSettings.json` metadata (credentials are never sealed here).

**Fields:**
- `accounts: usize` - Number of account metadata rows upserted (or that would be, on dry-run).
- `credentials: usize` - Always `0` — credentials are not written by migrate.
- `warnings: Vec<String>` - Non-fatal skip/parse warnings collected during the import.
- `account_id_map: std::collections::HashMap<(String, String), String>` - Classic AccountId (email) + locale → canonical account_id used in DB.



## bookclerk_migrate::accounts::import_accounts

*Function*

Import account metadata from `AccountsSettings.json`.

`skip_auth` is retained for CLI compatibility; credentials are never sealed
here (Audible IdentityTokens are not converted in migrate).

# Arguments

* `path` - Filesystem path involved in this operation.
* `store` - Destination library (opened by the caller through the adapter).
* `force` - When true, overwrite or force a full remote rescan.
* `skip_auth` - Boolean flag `skip_auth`.
* `dry_run` - Boolean flag `dry_run`.

# Returns

On success, the inner `AccountsImportSummary` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn import_accounts(path: &std::path::Path, store: &bookclerk_library::LibraryStore, plugin_key: &str, _force: bool, _skip_auth: bool, dry_run: bool) -> crate::error::Result<AccountsImportSummary>
```



