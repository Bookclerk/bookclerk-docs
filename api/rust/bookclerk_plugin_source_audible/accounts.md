**bookclerk_plugin_source_audible > accounts**

# Module: accounts

## Contents

**Structs**

- [`AccountInfo`](#accountinfo) - Summary of a configured account.

**Enums**

- [`AccountStatus`](#accountstatus) - Token health.

**Functions**

- [`import_auth_file`](#import_auth_file) - Import an audible-rs auth file into the `encrypted_secrets` table.
- [`import_libation_accounts_json`](#import_libation_accounts_json) - Import classic Libation `AccountsSettings.json` account metadata only.
- [`import_mkb79_auth_json`](#import_mkb79_auth_json) - Import mkb79/audible-cli legacy auth JSON (LibationCli: `import-account`).

---

## bookclerk_plugin_source_audible::accounts::AccountInfo

*Struct*

Summary of a configured account.

**Fields:**
- `account_id: String` - Account Identifier.
- `marketplace: String` - Marketplace.
- `label: Option<String>` - Operator-visible label for this account or item.
- `status: AccountStatus` - Status.
- `auth_file: Option<String>` - Reference to where credentials live (`encrypted_secrets:<id>`), for display only.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> AccountInfo`



## bookclerk_plugin_source_audible::accounts::AccountStatus

*Enum*

Token health.

**Variants:**
- `Valid` - Valid variant.
- `ExpiringSoon` - Expiring soon variant.
- `Expired` - Expired variant.
- `MissingRefresh` - Missing refresh variant.
- `Unknown` - Unknown variant.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - As str.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AccountStatus) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AccountStatus`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::accounts::import_auth_file

*Function*

Import an audible-rs auth file into the `encrypted_secrets` table.

Reads (decrypting via `BOOKCLERK_AUTH_PASSWORD` when needed) a user-supplied
auth file, then persists the authenticator into the DB. No `Accounts/` file
is written.

# Errors

Returns an error when the operation fails.

```rust
fn import_auth_file(scope: &bookclerk_library::SourceScope, source: &std::path::Path, label: Option<&str>, force: bool) -> crate::error::Result<AccountInfo>
```



## bookclerk_plugin_source_audible::accounts::import_libation_accounts_json

*Function*

Import classic Libation `AccountsSettings.json` account metadata only.

IdentityTokens are not converted (discarded file-based auth path). Use
[`import_auth_file`](accounts.md#bookclerk_plugin_source_audibleaccountsimport_auth_file) / [`import_mkb79_auth_json`](accounts.md#bookclerk_plugin_source_audibleaccountsimport_mkb79_auth_json) or interactive login for
credentials.

# Errors

Returns an error when the operation fails.

```rust
fn import_libation_accounts_json(path: &std::path::Path) -> crate::error::Result<Vec<AccountInfo>>
```



## bookclerk_plugin_source_audible::accounts::import_mkb79_auth_json

*Function*

Import mkb79/audible-cli legacy auth JSON (LibationCli: `import-account`).

# Errors

Returns an error when the operation fails.

```rust
fn import_mkb79_auth_json(scope: &bookclerk_library::SourceScope, source: &std::path::Path, label: Option<&str>, force: bool) -> crate::error::Result<AccountInfo>
```



